# How to automate code quality checks in your workflow
> For the most impatient of you, there is a [template project](https://gitlab.com/mamyn0va/php-template) to allow you quickly bootstrap a PHP project with ready-to-use quality tools.

## With git hooks
>_Check the code in the editor is good, but do it before pushing to git is better!_

Git comes with a set of hooks that you can use at many steps of the workflow (pre-commit, pre-push, post-update...). A git hook is just a script that will be executed before or after a particular event. It can be a shell script, a PHP script, a perl script, a python script or whatever scripting language. It can even a binary. It just need to be an executable.

Here is the list of all available hooks:
![[githookslist.png]]

We will focus on the `pre-commit` hook.

The idea is to run the quality tools in the `pre-commit` hook to check that your modifications are compliant with the standards that you defined for your project. Thus, you make sure that you don't commit erroneous code.

What is important to know before editing the `pre-commit` script is that if some of the tools returns an error code (different from 0), then it will stop the commit process . And that's exactly what we want.

Here are two rules that I personally use:
- The `pre-commit` hook **should** only check (and fix) the files commited by the user. If you need to check/fix the whole codebase, then do it maually and notify your team to avoid a merge hell.
- Any file modified by a hook **should** not be automatically re-added to the git index. This **should** be done by the user to let him the control on what is really commited.

To begin , the file `.git/hooks/pre-commit.sample` must be renamed in `.git/hooks/pre-commit`

Then, you need to retrieve the list of files added to the git index, and to filter on the file extension you need(.php, .go, .py...)

```shell
#!/bin/bash

STAGED_FILES_CMD=`git diff --cached --name-only --diff-filter=ACMR HEAD | grep \\\\.php`
````

Here I use `bash` in order to have a portable script that could run on (almost) any computer. But any scripting language is accepted.

Now, add below a call to each of your quality tools by providing them the list of staged files, and exit with 1 in case of error to break the commit

```shell

#!/bin/bash

echo "Running pre-commit hook"

PROJECT=`php -r "echo dirname(dirname(dirname(realpath('$0'))));"``
STAGED_FILES_CMD=`git diff --cached --name-only --diff-filter=ACMR HEAD | grep \\\\.php`

echo "Checking PHP Lint..."
for FILE in $STAGED_FILES_CMD
do
	php -l -d display_errors=0 $PROJECT/$FILE
	if [$? != 0]
	then
		echo "Fix the error(s) before commit."
		exit 1
	fi
	FILES="$FILES $PROJECT/$FILE"
done

echo "Running PHP Code Beautifier and Fixer (phpcbf)..."
phpcbf --standard=phpcs.xml.dist $STAGED_FILES_CMD --colors --report=summary
if [ $? != 0 ]
then
	echo "Fix the error(s) before commit."
	exit 1
fi

echo "Running PHP-CS-FIXER..."

PHP_CS_STATUS=0

for FILE in $STAGED_FILES_CMD
do
	php-cs-fixer fix $FILE
	PHP_CS_STATUS=$(( $PHP_CS_STATUS + $? ))
done

if [ $PHP_CS_STATUS != 0]
then
	echo "Fix the error(s) before commit."
fi

echo "End of pre-commit hook"

exit $PHP_CS_STATUS
````

Here I don't run `phpcs` and  `phpmd`, because they need to be highly customized to be used in a git hook. Otherwise, a lot of errors/warnings will be raised and block the commit. Do it only when you and your colleagues are ready. If you do it too early, it might discourage you and your team

If you feel ready for that, this [phpmd file](https://gitlab.com/mamyn0va/php-template/blob/master/phpmd-pre-commit.xml.dist) would be a good start.

I also use the PHP linter (`php -l`) to ensure that all my PHP files are syntactically correct.

We could also add a call to `phpunit`, but be aware that it must not last more than a few seconds, because committing must be quick.

The limit of the git hooks is just the onje of your imagination. I use it to lint many of my project's files: JSON, XML, swagger spec, markdown, shell scripts...

-	[swagger-cli](https://github.com/BigstickCarpet/swagger-cli): lint your swagger file!
-	composer validate: lint your compser.json
-	[dockerlint](https://github.com/redcoolbeans/dockerlint): lint your Dockerfile!
-	[shellcheck](https://www.shellcheck.net/): lint your shell scripts!

Finally, you're ready to test your hook by adding a file to the git index and commiting. The `pre-commit` script will be triggered automatically. If not, check that it has the execution bit. Otherwise:
```shell
chmod +x .git/hooks/pre-commit
````

## Automatisation through Composer
Composer comes also with a set of hoos allowing you to aumatize some of your tasks

Thus, instead of manually creating a `pre-commit` script for every developers, the idea is to put the script somewhere in your codebase (e.g.in `/contrib`), and to ask Composer to install it automatically when installing or updating your project.

Just add a new section in your composer.json:

```json
{
	"scripts": {
		"post-update-cmd": [
			"mv .git/hooks/pre-commit .git/hooks/pre-commit.bak",
			"cp contrib/pre-commit .git/hooks/pre-commit",
			"chmod a+x .git/hooks/pre-commit"
		},
		"post-install-cmd": [
			"mv .git/hooks/pre-commit .git/hooks/pre-commit.bak",
			"cp contrib/pre-commit .git/hooks/pre-commit",
			"chmod a+x .git/hooks/pre-commit"
		]
	}
}
````

That way, anytime a dev will be running `composer install` or `composer update` , the script will be installed.

## In your CI/CD pipeline
The goal of the quality control in the CI/CD pipeline is two fold:
-	first check the whole codebase against your project's quality rules, and break the build in case of any error (like a unit test would do)
-	(optionally) perform a deeper analysis and publish a detailed build report that can be consulted by the devs at any time

Of couse, it doesn't make sense to use a fixer (`phpcbf`, `php-cs-fixer`) in the pipeline as it would modify the code **after** it has been pulled from the repo, and we don't want the code to be modified without any control of the devs.

Checking your whole codebase will have a significant impact on the build time. it can vary from few seconds to few minutes, depending on your code size. For your information, on a 40000 LoC project, it takes approximately 5 seconds for `phpcs` and 40 seconds for `phpmd`, with a relatively "light" analysis.

Regarding the deeper analysis, we'll use `phpcs` and `phpmd` to raise errors/warnings and to publish the results, either in "human-readable" reports for devs (e.g. on a static website) with a notification on Slack, or in a standard format (`junit`, `checkstyle`, other...) to be integrated in a statistics tool like `Sonar` or `PhpMetrics`

Here is a simple example of a gitlab-ci's pipeline allowing to build, test & check a PHP app and to publish the reports to gitlab pages

```yaml
image: composer

stages:
  - test
  - deploy
  
test:app:
  stage: test
  script:
    - apk add --no-cache $PHPIZE_DEPS
	- pecl install xdebug-2.6.0
	- docker-php-ext-enable xdebug
	- composer install
	- ./vendor/bin/phpunit --testsuite=unit --coverage-text --colors=never --log-junit reports/phpunit-junit.log --testdox-html reports/phpunit-report.html
	- ./vendor/bin/phpcs -sw --standard=phpcs_squiz.xml.dist src --basepath=. --report=full --report-file=reports/phpcs-report.log || echo "ok"
	- ./vendor/bin/phpmd src html phpmd.xml.dist --reportfile reports/phpmd-report.html --ignore-violations-on-exit
  artifacts:
    paths:
	  - reports
	expire_in: 30m
	
pages:
  stage: deploy
  dependencies:
    - test:app
  script:
    - mkdir public
	- cp ci/pages/index.html public/
	- cp reports/* public/
  artifacts:
    paths:
	  - public
  only:
    - master

````

You can find all you need to bootstrap a PHP project with quality tools and git+gitlab integration on my [template-project](https://gitlab.com/mamyn0va/php-template)

For your information, if you want to enable the `Gitlab Pages` for your project, you just have to create a job named `pages` in your pipeline. All that you put in `/public` will be accessible on the static Website of your project at the following URL: `https://<group-or-username>.pages.forge.orange-labs.fr/<project-name>`

And since we're talking about `Git Pages`, we can aldo use it to host the API documentation (PHPDoc, JavaDoc, or other), the tests reports with code coverage, and the `Swagger` documentation.

---
Have a look at the [DigitalPulp plugins](https://github.com/digitalpulp/pre-commit-php.git)