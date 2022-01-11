# Vim usage notes

NOTE: Almost everything here was extracted from the official Vim Help documentation.

## Documentation

In Vim the documentation is read using the Vim Help system.

### Going to the main Vim Help documentation file

Just type `:help` and press ENTER.

### Documentation online

There is an HTML version of the Vim Help pages in [https://vimhelp.org/](https://vimhelp.org/).

### Concepts, notations and some definitions

Many introduction concepts are exaplined in intro.txt. You can read this document by typing `:help intro.txt` or `:help reference`.

#### Notations

When reading the Vim Help docs, you will see that there are items in [], {} and <>, and CTRL-X. Even
when sometimes the [], {} and <> are part of what you type, the context should make this clear.

Type `:help notation` or go to [https://vimhelp.org/intro.txt.html#notation](https://vimhelp.org/intro.txt.html#notation) to read about the meaning of all the notations.

**Key notations**

The follogin are some common key notations used in Vim Help. Type `:help key-notation` or `:help keycodes` or `:help key-codes` or go to [https://vimhelp.org/intro.txt.html#key-notation](https://vimhelp.org/intro.txt.html#key-notation) to read about the meaning of all the notations.

Examples in Vim Help are often given in the <> notation. Sometimes this is just to make clear what you need to type, but often it can be typed literally. e.g.: <C-G> is CTRL-G, <Tab> is the tab key or CTRL-I.

| notation  | meaning              | equivalent | tags                      |
| --------  | -------------------- | ---------- | ------------------        |
| <Nul>     | zero                 | CTRL-@     | <Nul>                     |
| <NL>      | linefeed             | CTRL-J     |                           |
| <CR>      | carriage return      | CTRL-M     | carriage-return           |
| <Return>  | same as <CR>         |            | <Return>                  |
| <Enter>   | same as <CR>         |            | <Enter>                   |
| <EOL>     | *1                   |            | <EOL>                     |
| <Esc>     | escape               | CTRL-[     | escape <Esc>              |
| <Space>   | space                |            | space                     |
| <lt>      | less-than            | <          | *<lt>*                    |
| <Bslash>  | backslash            | \          | backslash <Bslash>        |
| <Bar>     | vertical bar         | |          | <Bar>                     |
| <Del>     | delete               |            |                           |
| <BS>      | backspace            | CTRL-H     | backspace                 |
| <Tab>     | tab                  | CTRL-I     | tab                       |
| <Up>      | cursor-up            |            | cursor-up cursor_up       |
| <Down>    | cursor-down          |            | cursor-down cursor_down   |
| <Left>    | cursor-left          |            | cursor-left cursor_left   |
| <Right>   | cursor-right         |            | cursor-right cursor_right |
| <S-...>   | shift-key            |            | shift <S-                 |
| <S-Up>    | shift-cursor-up      |            |                           |
| <S-Down>  | shift-cursor-down    |            |                           |
| <S-Left>  | shift-cursor-left    |            |                           |
| <S-Right> | shift-cursor-right   |            |                           |
| <C-...>   | control-key          |            | control ctrl <C-          |
| <C-Left>  | control-cursor-left  |            |                           |
| <C-Right> | control-cursor-right |            |                           |
| <M-...>   | alt-key or meta-key  |            | meta alt <M-              |
| <A-...>   | same as <M-...>      |            | <A-                       |

*1 - end-of-line (can be <CR>, <NL> or <CR><NL>, depends on system and 'fileformat')

#### Vim modes

Type `:help vim-modes` or go to [https://vimhelp.org/intro.txt.html#vim-modes-intro](https://vimhelp.org/intro.txt.html#vim-modes-intro) to see all modes that Vim supports.

The following are the most common modes you will be using:

- Normal mode (aka command mode): In this mode you can enter all the normal editor commands. If you start the editor you are in this mode.
- Insert mode: In this mode the text you type is inserted into the buffer, you will see "-- INSERT --" at the bottom of the window when the 'showmode' option is enabled. Type `:help insert-mode` or `:help insert.txt` or go to [https://vimhelp.org/insert.txt.html](https://vimhelp.org/insert.txt.html) to read more about how to operate in this mode.
- Replace mode: Is a special case of Insert mode. You can do the same things as in Insert mode, but for each character you enter, one character of the existing text is deleted. You will see "-- REPLACE --" at the bottom of the window when the 'showmode' option is enabled. Type `:help replace-mode` or go to [https://vimhelp.org/insert.txt.html#Replace-mode](https://vimhelp.org/insert.txt.html#Replace-mode) to read more about how to operate in this mode.
- Visual mode: This is like Normal mode, but the movement commands extend a highlighted area. When a non-movement command is used, it is executed for the highlighted area. You will see "-- VISUAL --" at the bottom of the window when the 'showmode' option is enabled. Type `:help visual-mode` or `:help visual.txt` or go to [https://vimhelp.org/visual.txt.html](https://vimhelp.org/visual.txt.html) to read more about how to operate in this mode.
- Select mode: Looks like Visual mode, but the commands accepted are quite different. This resembles the selection mode in Microsoft Windows. Typing a printable character deletes the selection, the character is inserted, and starts Insert mode. You will see "-- SELECT --" at the bottom of the window when the 'showmode' option is enabled. Type `:help select-mode` or go to [https://vimhelp.org/visual.txt.html#Select-mode](https://vimhelp.org/visual.txt.html#Select-mode) to read more about how to operate in this mode.
- Command-line mode (aka Cmdline mode): This mode is used to enter Ex commands (":"), search patterns
("/" and "?"), and filter commands ("!"). Type `:help command-line-mode` or `:help cmdline-mode` or `:help cmdline.txt` or go to [https://vimhelp.org/cmdline.txt.html](https://vimhelp.org/cmdline.txt.html) to read more about how to operate in this mode.

#### Switching between modes

The following are the most common used ways to swich between different modes. Type `:help mode-switching` or go to [https://vimhelp.org/intro.txt.html#mode-switching](https://vimhelp.org/intro.txt.html#mode-switching) for more information.

| v From To >    | Normal         | Visual | Select | Insert   | Replace  | Cmd-line |
| -------------- | -------------- | ------ | ------ | -------- | -------- | -------- |
| Normal         |                | v V ^V |   *1   |  *2      |  R gR    | : / ? !  |
| Visual         |  v V ^V <Esc>  |        |   ^G   |  c C     |   --     |   :      |
| Select         |  *3            | ^O ^G  |        |  *4      |   --     |   --     |
| Insert         |  <Esc>         |   --   |   --   |          | <Insert> |   --     |
| Replace        |  <Esc>         |   --   |   --   | <Insert> |          |   --     |
| Command-line   |  ^C <Esc>      |   --   |   --   |  :start  |   --     |          |

-- not possible

*1 - Go from Normal to Select mode with one of "gh", "gH" or "g CTRL-H" (`:help g_CTRL-H` for more info).
*2 Go from Normal mode to Insert mode by giving the command "i", "I", "a", "A", "o", "O", "c", "C", "s" or S".
*3 - Go from Select mode to Normal mode by using a non-printable command to move the cursor, without keeping the Shift key pressed.
*4 Go from Select mode to Insert mode by typing a printable character. The selection is deleted and the character is inserted.

#### Definitions

The following are some concepts you need to understand to read Vim documentation. Type `:help definitions` or go to [https://vimhelp.org/intro.txt.html#definitions](https://vimhelp.org/intro.txt.html#definitions) for more information.

- buffer: Contains lines of text, usually read from a file.
- screen (aka the Vim window): The whole area that Vim uses to work in. A screen contains one or more windows, separated by status lines and with the command line at the bottom.
- window: A view on a buffer. There can be multiple windows for one buffer.
- command line: used for messages. It scrolls up the screen when there is not enough room in the command line.
- buffer lines: The lines in the buffer. This is the same as the lines as they are read from/written to a file. They can be thousands of characters long.
- logical lines: The buffer lines with folding applied. Buffer lines in a closed fold are changed to a single logical line: "+-- 99 lines folded". They can be thousands of characters long.
- window lines: The lines displayed in a window: A range of logical lines with wrapping, line breaks, etc. applied. They can only be as long as the width of the window allows, longer lines are wrapped or truncated.
- screen lines: The lines of the screen that Vim uses. Consists of the window lines of all windows, with status lines and the command line added. They can only be as long as the width of the screen allows. When the command line gets longer it wraps and lines are scrolled to make room.

#### Content in a window in Vim

The following notes documents some things about how content is displayed in a window in Vim. Type `:help window-contents` or go to [https://vimhelp.org/intro.txt.html#window-contents](https://vimhelp.org/intro.txt.html#window-contents) for more information.

**Line wraps**

- Lines longer than the window width will wrap, unless the 'wrap' option is off.
- The 'linebreak' option can be set to wrap at a blank character.

**Line numbers**

If you set the 'number' option, all lines will be preceded with their
number. Tip: If you don't like wrapping lines to mix with the line numbers, set the 'showbreak' option to eight spaces: ":set showbreak=\ \ \ \ \ \ \ \ ".

**The ~ character**

The '~' lines indicate that the end of the buffer was reached. If the window has room after the last line of the buffer, Vim will show '~' in the last lines in the window.

**The @ character**

The '@' lines indicate that there is a line that doesn't fit in the window. If the last line in a window doesn't fit, Vim will indicate this with a '@' in the last lines in the window.

When the "lastline" flag is present in the 'display' option, you will not see '@' characters at the left side of window. If the last line doesn't fit completely, only the part that fits is shown, and the last three characters of the last line are replaced with "@@@".

If there is a single line that is too long to fit in the window, this is a special situation. Vim will show only part of the line, around where the cursor is. There are no special characters shown, so that you can edit all parts of this line.

### Searching anything

It is possible to go directly to the first match of whatever you want help on by invoking `:help {whatever}`.

Type `:help {whatever}` and press CTRL-D If you want to see all the matching entries for {whatever}.

### Searching documentation about an specific item

If you want to search for a more specific thing, you can prepend something to specify the context:

| WHAT                                   | PREPEND | EXAMPLE            |
| -------------------------------------- | ------- | ------------------ |
| Normal mode command                    |         | :help x            |
| Visual mode command                    | v_      | :help v_u          |
| Insert mode command                    | i_      | :help i_<Esc>      |
| Command-line command (aka Ex-commands) | :       | :help :quit        |
| Command-line editing                   | c_      | :help c_<Del>      |
| Vim command argument                   | -       | :help -r           |
| Option                                 | '       | :help 'textwidth'  |
| Regular expression                     | /       | :help /[           |

Type `:help help-summary` or go to [https://vimhelp.org/usr_02.txt.html#help-summary](https://vimhelp.org/usr_02.txt.html#help-summary) for more detailed information about how to use the help command.

**Documentation abount commands in different modes**

The Vim editor has many different modes. By default, the help system displays the normal-mode commands.

Normal mode commands do not have a prefix. For example, the following command displays help for the normal-mode CTRL-H command: `:help CTRL-H`.

To identify other modes, use a mode prefix. If you want the help for the insert-mode version of a command, use "i_". For CTRL-H this gives you the following command: `:help i_CTRL-H`.

Special keys are enclosed in angle brackets. To find help on the up-arrow key in Insert mode, for instance, use this command: `:help i_<Up>`.

**Documentation about errors**

If you see an error message that you don't understand, for example:

```
E37: No write since last change (use ! to override)
```

You can use the error ID at the start to find help about it: `:help E37`.

### Navigating inside the documentation

The help window is a normal editing window. You can use to following combinations for navigating inside the Vim Help documentation but also for nevigating in other files using tag files.

- Jump to a tag or any word: Position the cursor over a tag or over any word and hit `CTRL-]`. `CTRL-]` jumps to the definition of the keyword under the cursor (tags are prioritized first). Same as ":tag {name}", but here {name} is the keyword under or after cursor. When there are several matching tags for {name}, the first one is jumped to.
- Jump to a tag: Type `:tag {name}` to jump to the definition of {name}, using the information in the tags file(s). e.g. `:tag help-summary` redirects you to the same place as `:help help-summary` but the fist jumps only to those tags that are defined commonly in the top right (or top left) corner in the beginning of each section in the documentation.
- Jump back: Type `CTRL-O`. Repeat this to go further back.

### List of all help items (tags)

All tags are listed by typing `:help help-tags` or by going to [https://vimhelp.org/tags.html](https://vimhelp.org/tags.html).

### List of all commands

index.txt contains a list of all commands for each mode with a short description.

You can go to the top of this document by typing `:help index` or by going to [https://vimhelp.org/index.txt.html](https://vimhelp.org/index.txt.html).

You could also jump to specific sections in the documentation by typing for example:
- `:help insert-index` to see the list of commands for Insert mode.
- `:help normal-index` to see the list of commands for Normal mode.
- `:help visual-index` to see the list of commands for Visual mode.
- `:help ex-cmd-index` to see the list of EX commands.
- `:help ex-edit-index` to see the list of commands for Command-line editing.

### List of all options

All options are documented in detail inside options.txt (`:help options` or go to [https://vimhelp.org/options.txt.html](https://vimhelp.org/options.txt.html)) but this is a long document.

Type `:help option-list` or go to [https://vimhelp.org/quickref.txt.html#Q_op](https://vimhelp.org/quickref.txt.html#Q_op) to see the overview with the list and short explanation of each option.

### List of all builtin functions

All builtin functions are documented in detail inside builtin.txt (`:help builtin` or go to [https://vimhelp.org/builtin.txt.html](https://vimhelp.org/builtin.txt.html)) but this is a long document.

You have to overview lists of the builtin functions:

- Grouped by category: to see the list with a short description of the builtin functions grouped by what they are used for, type `:help function-list` or go to [https://vimhelp.org/usr_41.txt.html#function-list](https://vimhelp.org/usr_41.txt.html#function-list)
- Alphabetic list: for an alphabetic list with a little more of information you can type `:help builtin-function-list` or go to [https://vimhelp.org/builtin.txt.html#builtin-function-list](https://vimhelp.org/builtin.txt.html#builtin-function-list).

### List of all predefined variables

The list of all the predefined variables are documentaed in eval.txt. Type `:help vim-variables` or `:help v:` or go to [https://vimhelp.org/eval.txt.html#vim-variable](https://vimhelp.org/eval.txt.html#vim-variable) to see the list.

### List of all features

The list with a short description of all features that your vim installation could support are documented in various.txt. Type `:help +feature-list` or go to [https://vimhelp.org/various.txt.html#%2Bfeature-list](https://vimhelp.org/various.txt.html#%2Bfeature-list) to see the list.

#### Variable types

There are different types of variables like for example Number, String, Float, List, and bla bla bla. To see all the type of variables type `:help variables` or go to [https://vimhelp.org/eval.txt.html#variables](https://vimhelp.org/eval.txt.html#variables).

## The configuration file (vimrc)

vimrc is a file that contains initialization commands.

Each line in a vimrc file is executed as an Ex command line. Remember that in this mode you don't have to keep pressing ":", that way putting `:set number` is the same as putting `set number`.

As already mentioned before, you can search the documantation abou Ex commands specifically by using the : prefix, for example `:help :filetype` for information about the "filetype" command.

For specific documentation about options configured with "set", you have to use single quotes. For example `:help 'number'` for documentation about the "number" option that you enable with `set number`.

## Common tasks

### Copy a selection to the clipboard

For copying a selection to the clipboard register you type `"*y`.

### Paste from the clipboard

To put somthing from the clipboard you type `"*p`.

## Analysis and troubleshooting

### Inspect the current value of a variable

Simply run `:echo foo`.

### Inspect the current value of an option

Just add a question mark in the ond of the 'set' command, for example `set number?` shows if line numbers are enabled.

### List features supported by my current installation

All the supported features are listed when invoking `vim --version`. A feature prepended with `+` means that it is included, otherwise the `-` prefix means that it is not included.

Type for example `:help +python3` (always using the plus symbol) to see the purpose of the "+python3" or the "-python3" feature. Type `:help +feature-list` for a list of all the featured with their description.

### Call functions from command

Type `:call foo()`.

### Evaluate strings from command

Type `:exec "call foo()"`.
