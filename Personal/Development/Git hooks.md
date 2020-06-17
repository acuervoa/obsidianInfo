## Git hooks

We can add githook that will prevent us from successfully commiting code if it detects the phrase "FIXME" in any of the modified files.

First we need  to write the hook. The simplest version might look something like this.

```shell
#!/bin/bash

# Redirect output to stderr
exec 1>&2

# Define colors
RED='\033[0;31m'
NC='\033[0m'

# Define term to search for
SEARCH_TERM="FIXME"

# Check for the presence of the SEARCH_TERM in updated files
if [[ $(git diff --cached | grep -E "^\+" | grep -v '+++ b/' | cut -c 2-) == *$SEARCH_TERM*]]
  then
    printf "${RED}Error:${NC} Found ${SEARCH_TERM} in attemped commit.\n"
	printf "Please remove all occurances of ${SEARCH_TERM} before committing.\n"
	exit 1
fi
````

Put this code into a file named "pre-comit" without extension, into the folder .git/hooks


[More info about git hooks](https://git-scm.com/docs/githooks)

## Adding Git hooks to a team project

It's actually pretty straightforward if you use [Husky](https://github.com/typicode/husky), a tool specifically developed to make sharing Git hooks easy

**Example with Prettier**
1. Install prettier with `npm i prettier --save-dev`
2. Install husky and lint-staged with `npx mrm lint-staged`
3. Add this is another script under your package.json scripts:`"prettier:": "prettier --write '**/*.js*'"`

You can find more information on automating Prettier with Git hooks on the Prettier [pre-commit hook](https://prettier.io/docs/en/precommit.html) documentation


**Awesome git-hooks repository**
- [A curated list of awesome git hooks](https://github.com/CompSciLauren/awesome-git-hooks)