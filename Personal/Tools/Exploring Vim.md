# A taste of Vim

Unlike any other traditional editor, when using vim you'll spend most of your time in _normal_ mode. Within this mode you don't explicity write code, instead it is optimized for navigation and precise text changes. Your righ hand rests firmly on the core **motion** (as in movement) keys `hjkl`. These keys move the cursor around to left, down, up and right respectively. You can also move left to right word by using `w`(go to the begin of next **w**ord) or `e` (go to the **e**nd of the next word), or use `b / ge`to do the same but right to left.

Takin advantage of these keys you can navigate a file with fine granularity and strike with great vengeance and spite: `daw` and bang! You **d**elete **a** **w**ord! `das` and you remove a **s**entence! `dap` and you obliterate a **p**aragraph! Fierce!

Or you can be less menacing and more nurturing and fix stuff instead. Usin `caw`, `cas`, `cap` to **c**hange **a** **w**ord, **s**entence or **p**aragraph. But it doesn't end there. You can `ctx` to **c**hange un**t**il the first `x` in the current line, or `c$` to **c**hange everything until the end of line, or event better `ci(` to change the content between parentheses or `ci"` to do the same to the content inside quotes.

Imagine a simple string:

```javascript
const msg = 'Hello vim'
```

You can change the string by typing: `f'ci'WAT<ESC>`

```javascript
const msg = 'WAT'
````

Which means **f**ind the next `'` then **c**hange everything **i**nside `'`for `WAT`, then `<ESC>` to leave _insert_ mode back to _normal_ mode


You can then **y**ank thwe line with `yy` (which is vim jargon for copy) and **p**ut it below with `p` (again vim jargon for paste)

Yes, you got it! _yank_ is another operator like delete and change. You can use `y` just like `d` or `c` to yank a word `yaw` or `yas` yank a sentence. Moreover doubling a command like so `yy` makes the command operate on the entire line. Nifty `cc` changes a line and `dd` deletes one.

```javascript
const msg = 'WAT'
const msg = 'WAT'
````

Again `f'ci'MAN<ESC>`:

```javascript
const msg = 'WAT'
const msg = 'MAN'
````

Then go crazy join the lines with `kJ` (k to go up and J to join lines):

```javascript
const msg = 'WAT' const msg = 'MAN'
````

Rinse with `c3w+<ESC>` and we've got ourselves:

```javascript
const msg = 'WAT' + 'MAN'
```

For longer motions you can use **counts** in combination with motions and, for instance, go down 5 lines by typing `5j` (as in `{count}{motion}`). Likewise you can use these together with the operators you saw above (`d`, `c`, etc) and `d2w` delete two words or `c2s` change 2 senteces. There's longer motions too, you can `H` to move to the top of the visible area in the editor or `gg` to go to the top of the file, `L` and `G` to achieve the same but downwards. Use `{` to move up a whole paragraph or `}` to do the same but down. While `%` helps you find matching parentheses.

You can start a new line below with `o` (drops into insert mode so you can start typing) or above with `O`. You can find patterns (of text) forward within a file usin `/{pattern}` and navigate between patterns using `n` (next) and `N` (previous) or backwards using `?{pattern}`.

You can repeat previous changes using the `.` command. Just type `.` and vim repeat your last change. Likewise you can repeat motions. Type `;` and you'll repeat a motion started with `t`,`f`,`T`,`F` or type `n` to repeat a search. You can record a collection of commands using macros and replay them at your will. And there's so much more...

## The Vim Way
Let's back down a little bit, reflect and try to draw some wisdom from what we've seen thus far

**Keyboard centeredness is  at the core of vim**. Important and useful commands lie underneath your fingertips and their location is very thoughtful and coductive to efficient use and learning. For instance:
-	basic motions in the home row under you right hand `hjkl`
-	common repeaters `;` and `.`one beneath the other
-	another basic and powerful motion `f` is in your left index finger
-	basic operators are easily reachable with your left hand: `s`,`x`,`d`,`c`,`r`
-	`s`synonym for `cl` and `x` synonym for `dl` are one finger away from `d` and `c`
-	`*` and `#` motions used to find current word forward and backward. They are triggered by the middle finger of each hand

The **operators and motions make incredible sense**, and are easy to remember through the use of mnemonics `c` means **c**hange, `cl`means **c**hange **l**etter, `caw` means **c**hange **a** **w**ord, `ciw` means **c**hange **i**nner **w**ord (as without the surrounding whitespace), `ct.` mean s change till you find `.`, and so on. This makes vim extremely conductive to learning as you'll have less need to memorize arcane commands: The commands just make sense.

[The true magic of vim is composition](https://medium.com/@mkozlows/why-atom-cant-replace-vim-433852f4b4d1) As you go building up this vocabulary of operators and motions you'll find that you can combine them your heart's content. So that, once you know all the `c`, `cl`, `caw`, `ciw`, `ct.` from the previous paragraph and you learn how `dl` works, you'll not only be able to use `dl`, you'll know that you can also combine it with all the motions you already have at your disposal and `daw`,`diw`,`dt`, etc.

## The keyboard and Normal Mode Wall
Vim is entirely keyboard driven and oftentimes is used from within a terminal (so there may be no mouse support enabled nor any options to explore with a mouse). And even though you  will be received with a prompt to use the help available via the `:h` command you will rarely heed that advice,

The best advice here is to learn the very basics first and slowly go expanding your vocabulary. The very basics are:
- Get help with `:help {whatever}` (or the shorter version `:h {whatever}`). Vim's help is insanely good.
- Move around with `hjkl`(`:h motion` or `:h movement`)
- Go into insert mode with `i`(`:h insert`)
- Go back to normal mode with `<ESC>` (`:h mode-switching`)
- Save a file with `:w`(`:h write`)
- Leave the editor with `:q` (`:h quit`)
- Open a file with `:e {filepath}` (as in `:e src/main.js`)(`:h edit`)

Notice the `:` before all these commands? The colon triggers command mode also known as ex mode. When you type a coon and a command, the command will be displayed in the lower-left part of the screen.

## Neovim is Much Nicer To Beginners
Consider following the steps below using [neovim](https://www.neovim.io/) instead of vim. Neovim has two things that make it more approachable to newcomers. One, Neovim comes with far better defaults than vim (like syntax highlighting, auto indent, etc.), very basic stuff that is not turned on in vim. Two, it has a nicer vimtutor which can be started from within neovim by running the `:Tutor` and which gives you feedback when you complete the exercises,
1. Learn the very basics inside vim starting with `vimtutor`(`:Tutor`on neovim)
2. Apply what you learn using a plugin in your editor of choice. Using your current editor will minimize the amount of things you need to learn and allow you to enrich your current editing skills with lots of microediting vim techniques. Depending on how good the vim mode of your editor is, you'll have a more or less pleasant experience.
3. When you're comfortable with basic vim commands then setup your vim configuration in `.vimrc`(or `init.vim`for neovim). Take a look at [vim-sensible](https://github.com/tpope/vim-sensible/blob/master/plugin/sensible.vim) or at [vim-wiki](https://dev.to/wiki/vim) for a minimal and pluginless configuration explained step by step. If you're using Neovim you can just jump over this.
4. Use a vim plugin manager to enrich the funcionality of vim tailored to your workflow (the most established ones are [vundle](https://github.com/VundleVim/Vundle.vim), [vim-plug](https://github.com/junegunn/vim-plug) and [pathogen](https://github.com/tpope/vim-pathogen), the Modern vim book recommends [minpac](https://github.com/k-takata/minpac). The way these plugin managers work most of the time is that they'll clone a git repo containing a serios of vim scripts that represent the plugin you need. Vim will pick those scripts on startup and run them in the context of vim
5. Check [vim awesome](https://vimawesome.com/) for plugins that interest you and the type of applications that you develop.
6. Practiced and improve your vim setup and vim configuration over time

## More resources
- [Vim](https://www.barbarianmeetscoding.com/wiki/vim) and [Neovim](https://www.barbarianmeetscoding.com/wiki/neovim) wikis
- [7 habits of effective text editing](https://www.moolenaar.net/habits.html) by Bram Moolenar creator of vim
- The Practical Vim and Modern Vim books
- [Vimcasts.org](http://vimcasts.org/episodes/archive/): A collection of short vim videos by Drew Neil the author of Practical Vim
- A [video from the Barbarian Meets Coding Vlog](https://www.youtube.com/watch?v=_uJiaVcD_s4) 