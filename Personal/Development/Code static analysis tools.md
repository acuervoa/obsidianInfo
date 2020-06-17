# Code static analysis tools PHP

The [PHP-FIG](https://www.php-fig.org/) consortium publishes a [set of recommendations](https://www.php-fig.org/psr/) that aims to standardize the use of PHP and facilitate its inteorperability. Three of then focus on the coding style and on stuff like the auto-loading, the encoding. and so on:
- **PSR1**: [Basic Coding Standard](https://www.php-fig.org/psr/psr-1/)
- **PSR2**: [Coding Style Guide](https://www.php-fig.org/psr/psr-2/)
- **PSR12**: [Extended Coding Style Guide](https://github.com/php-fig/fig-standards/blob/master/proposed/extended-coding-style-guide.md)

## Tools

### [PHP_CodeSniffer](https://github.com/squizlabs/PHP_CodeSniffer) (aka phpcs)
Detects PSR1/PSR2 violations, and a lot more with the Squiz standard that check 130 rules (7 rules for PSR1 and 42 for PSR2) and that also works with CSS and JS

#### **phpcs installation**
```shell
composer require "squizlabs/php_codesniffer"
````

To use outside of a project, you should install it globally and add `vendor/bin` to your `PATH`:
```shell
composer global require "squizlabs/php_codesniffer"
export PATH=$PATH:~/path/to/composer/vendor/bin
````

#### **phpcs usage**
Now, consider the following PHP example that contains some violations...
```php
<?php
class MaClasse{
	var $monAttribut;
	
	public function MaClasse ($monAttribut){
		$this->setMonAttribut( $monAttribut );
	}
	
	function setMonAttribut($monAttribut){
		$test;
		$this->monAttribut=$monAttribut;
	}
}
````

...and see what happens if we use phpcs with the PSR2 standard:

![[phpcs_usage.jpeg]]

12 errors are detected by the tool and 6 of them can be fixed automatically.

Let's see what happen if we re-run the tool with the Squiz standard:

![[phpcs_usage_squiz.jpeg]]

28 errors are detected by the tool and 18 of them can be fixed automatically. My opinion aboyt Squiz standard is that it is a little too verbose. Thus, what I did is that I created my own ruleset with all the PSR2 rules and added some of the Squiz rules that ake sense to me.

To go further, you can have a look at the [phpcs_psr2](https://gitlab.com/mamyn0va/php-template/blob/master/phpcs_psr2.xml.dist) and [phpcs_squiz](https://gitlab.com/mamyn0va/php-template/blob/master/phpcs_squiz.xml.dist) ruleset

### PHP Code Beautifier and Fixer (aka phpcbf)
PHP_CodeSniffer comes with another utility that fixes the main PSR1 and PSR2 violations. Just run it on a PHP file or on a code base, and it will scan all the files and try to automatically fix the violations:

![[phpcbf.jpeg]]

### [PHP Mess Detector](https://phpmd.org/) (aka php)

Through a ruleset mechanism, this tool will detect many violations towards the clean code philosophy:
- possible bugs
- suboptimal code
- overcomplicated expressions
- unused parameters, methods, properties
- ...

#### **phpmd installation**
```shell
composer require phpmd/phpmd
````

#### **phpmd usage**
```shell
phpmd MaClasse.php text cleancode,codesize,controversial,design,naming,unusedcode
````

Output
```shell
MaClasse.php:6		The method MaClasse is not named in camelCase
MaClasse.php:6		Classes should not have a constructor method with the sema name as the class
MaClasse.php:12		Avoid unused local variables such as '$test'
````

As you can see, the tool is complementary to PHP_CodeSniffer as it found three new interesting violations that weren't detected previously. I suggest you to create your own ruleset based on [existing ones](https://github.com/phpmd/phpmd/tree/2.6.0/src/main/resources/rulesets). You need to create a file named `phpmd.xml.dist` at the root of your project adn you'll be able to use it in the terminal, in your editor/IDE and in your pipeline.

Find [here](https://gitlab.com/mamyn0va/php-template/blob/master/phpmd.xml.dist) the ruleset that uses all the existing rules.

But like for phpcs, I'd advise you to preceed iteratively by starting by the mininal ruleset `unusedcode`, and to gradually include new rules that make sense for you and your project.

This is a custom ruleset that you can use to start with phpmd:
```xml
<?xml version="1.0"?>
<ruleset name="Custom rule set used in pre-commit git hook"
		 xmlns="http://pmd.sf.net/ruleset/1.0.0"
		 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		 xsi:schemaLocation="http://pmd.sf.net/ruleset/1.0.0 http://pmd.sf.net/ruleset_xml_schema.xsd"
		 xsi:noNamespaceSchemaLocation="http://pmd.sf.net/ruleset_xml_schema.xsd">
	<description>
		Custom rule set used in pre-commit git hooks
	</description>
	
	<!-- Import the entire unused code rule set -->
	<rule ref="ruleset/unusedcode.xml" />
	<!-- Import a part of design code rule set -->
	<rule ref="ruleset/design.xml">
		<exclude name="NumberOfChildren" />
		<exclude name="DepthOfInheritance" />
		<exclude name="CouplingBetweenObjects"/>
	</rule>
</ruleset>
```

### [PHP Coding Standards Fixer](https://github.com/FriendsOfPhp/PHP-CS-Fixer) (aka php-cs-fixer)

> If you had to choose only one tool, it would be this one

It does nearly the same things as **phpcbs**, namely fix PSR rules, but it does it better and deeper. Like phpcbf, it also allow you to add some other rules from a list of [172 rules](https://github.com/FriendsOfPhp/PHP-CS-Fixer#usage). The other cool thing about it is the creation of the ruleset file which is a PHP file and not a verbose XML.

#### **php-cs-fixer installation**
```shell
composer require friendsofphp/php-cs-fixer
````

#### **php-cs-fixer usage**

```shell
php-cs-fixer --verbose fix MaClasse.php
Loaded config default from "~/.php_cs.dist".
Using cache file ".php_cs.cache".
Paths from configuration file have been overriden by paths provided as command arguments.
F
Legend: ?-unknown, I-invalid file syntax, file ignored, S-Skipped, .-no changes. F-fixed, E-error
	1) MaClasse.php (class_attributes_separation, no_spaces_inside_parenthesis, declare_strict_types, blank_line_after_opening_tag, array_syntax, class_definition, 
function_declaration, visibility_required, declare_equal_normalize, binary_operator_spaces, braces)

Checked all files in 0.008 seconds, 8.000 MB memory used
````

Here is the ouput on MaClasse.php with a custom ruleset:

```php
declare(strict_types = 1);
class MaClasse
{
	public $monAttribut = [];
	
	public function __construct($monAttribut)
	{
		$this->setMonAttribut($monAttribut);
	}
	
	public function setMonAttribut($monAttribut)
	{
		$this->monAttribut = $monAttribut;
		$test = 1;
	}
}
````

And there is my custom ruleset (.php_cs.dist);

```php
$finder = PhpCsFixer\Finder::create()
    ->exclude('vendor')
    ->exclude('output')
    ->exclude('releases')
    ->in(__DIR__)
;

return PhpCsFixer\Config::create()
    ->setRiskyAllowed(true)
    ->setRules([
        '@PSR2' => true,
        'declare_strict_types' => true,
        'align_multiline_comment' => ['comment_type' => 'phpdocs_only'],
        'binary_operator_spaces' => true,
        'blank_line_after_opening_tag' => true,
        'cast_spaces' => true,
        'class_attributes_separation' => true,
        'compact_nullable_typehint' => true,
        'concat_space' => ['spacing' => 'one'],
        'declare_equal_normalize' => ['space' => 'single'],
        'function_typehint_space' => true,
        'no_blank_lines_after_class_opening' => true,
        'no_blank_lines_after_phpdoc' => true,
        'no_extra_consecutive_blank_lines' => true,
        'no_mixed_echo_print' => true,
        'no_php4_constructor' => true,
        'no_unused_imports' => true,
        'no_useless_else' => true,
        'no_useless_return' => true,
        'no_whitespace_before_comma_in_array' => true,
        'no_whitespace_in_blank_line' => true,
        'object_operator_without_whitespace' => true,
        'ordered_class_elements' => true,
        'ordered_imports' => true,
        'phpdoc_align' => true,
        'phpdoc_indent' => true,
        'yoda_style' => true,
        'protected_to_private' => true,
        'return_type_declaration' => true,
        'self_accessor' => true,
        'single_blank_line_before_namespace' => true,
        'single_quote' => true,
        'ternary_operator_spaces' => true,
        'ternary_to_null_coalescing' => true,
        'whitespace_after_comma_in_array' => true,
        'array_syntax' => ['syntax' => 'short'],
    ])
    ->setFinder($finder)
;
```

Some of these rules are so called _risky_ because they can impact the PHP's runtime, so use it carefully! On my project, I use two of them:
- `declare_strict_types` (type mode is set to `strict` instead of `weak`)
- `no_php4_construnctor` (constructors **MUST** be named `__construct`)

You can disable this with the setter `setRiskyAllowed(false)`

---
Closing on the tooling, you must consider it as an integral part of your app, that is to say that one hand, they're listed in your project's dependencies, and on the other hand, their configuration files and rulesets are also part of your codebase.

## To go further
- [phan](https://github.com/phan/phan): a powerful code static analyzer
- [phpstan](https://github.com/phpstan/phpstan): PHP Static Analyzis Tool
- [PHP metrics](https://www.phpmetrics.org/): (yet) another PHP static analyzer that outputs some user-friendly reports
- [PHP copy/paste detector](https://github.com/sebastianbergmann/phpcpd) (aka phpcpd): a duplicate code detector
- [SonarPHP](https://github.com/SonarSource/sonar-php): a static PHP code analyzer for SonarQube and SonarLint
- [phploc](https://github.com/sebastianbergmann/phploc): a tool that gives you statistics on your PHP project

---
Useful links:
- [PHP Standards Recommendations](https://www.php-fig.org/psr)
- [PHP: The Right Way](https://www.phptherightway.com/)