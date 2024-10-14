# Useful neovim commands and tricks

- `^` to move to first non-whitespace character, `$` for first character
- `+` and `-` move to line below and above but non-whitespace characters 15
- `j`, `k`, `l`, `h` to move down, up, left, right 1
- `G` end of file, `gg` beginning of file 14

- `%` jump to matching parenthesis
- `t` jump before character, `f` jump to character, `T` and `F` are the reverse
- `,` and `;` repeat `t`, `T`, `f`, `F` in same and opposite directions
- `/` search forward, `?` search backward

- `I`, `i`, `a`, `A` append at beginning of line / character and end of line / character
- `guu`, `gUU` and `g~~` will make the current line lowercase, uppercase or alternate case
- `40gg` to jump to the 40th line
- `D`, `Y` and `C` instead of `d$`, `y$` and `c$`

- `gx` opens link under cursor
- `gf` opens relative or absolute file path under cursor
- `CTRL v` visual block mode
- `J` join line

- `dk` and `dj` to delete 2 lines easily
- `yib` is the same as `yi(` but easier to type
- `gu`, `gU` and `g~` are operators that make the text lowercase, uppercase, or alternate case
- in VISUAL `g CTRL v` increment each line in selection based on line number, e.g. `0 0 0 0` (separated by newlines) will become `1 2 3 4`

- `d/hello` deletes until hello is found, `d/hello/e` deletes including the hello
- `#` and `*` find the identifier under cursor forwards and backwards
- in VISUAL `<` and `>` dedent and indent selection
- in VISUAL BLOCK `I` to prepend to all lines at the start of selection, `$A` to append at end of lines

- and `xp` to swap 2 characters
- CTRL a increases numbr by 1, CTRL x decreases by 1. also takes in a count
- in VISUAL `o` will go to the opposite end of the selection
- `gv` to reselect the last selection

- `CTRL wx` swap window
- `CTRL wv` split vertically and `CTRL ws` horizontally
- `CTRL wq` quit window
- `CTRL w=` equalize window sizes

- `vap` select around paragraph
- `r` replace a character
- `gp` and `gP` are like `p` and `P` but leave cursor after pasted text
- `CTRL g` show filename and line count

- `:center` centers text
- `zz`, `zt`, `zb` position cursor in middle, top and bottom of screen
- `ga` prints info about character under the cursor
- `g?` is an operator that will rot13 encode the input

- Using a count with an insert operator e.g. `o`, `a`, `I` etc will write it in insert mode that many times
- `gi` insert text in same position as where insert mode was stopped last time
- `/pattern/e` search for pattern and put cursor on the last character of pattern
- `/pattern/e+1` ^ but +1 char to the left

- `:%s/pattern//n` count number matches for pattern
- `:sort u` sort whole file or range in VISUAL mode unique
- `:sort n` sorts by number that it will find within the line
- `/apple\C` for case-sensitive search

- `g;` go to previous insert location
- `g,` to to next previous insert location
- `gi` insert mode in last edit location
- in INSERT `CTRL u` delete all characters in current line

- `''` jump to before last jump
- `[{`, `]{`, `[(` and `](` go to last/next unmatched `{` and `(`
- `g&` repeat last `:s`
- `g$`, `g0`, `g^` are like `$`, `0` and `^` but will respect wrapped lines

- `gJ` joins lines without inserting space
- `gq` wrap lines
- In O-PENDING mode we can use `v`, `V` and `CTRL v` to force operator to work charwise, linewise or blockwise
- in command line `CTRL g` and `CTRL t` next and previous match (useful for when using `/` or `?` with an operator)

- in command line `CTRL r+` paste contents from register `+` (so system clipboard, but we can use with other registers)
- in command line `CTRL u` remove all characters
- `:%norm` execute command on every line, or `:norm` for the lines selected. e.g. `:%norm $ciwhello`
- `:g` execute command on each line that match pattern, e.g. `:g/^#/d` delete all comments from bash file
- `:%s` substitute, e.g. `:%s/foo/bar/g`
- `:g/foo/s/bar/baz/g` substitute bar with baz on all lines that contain foo

- pipe output of commands into `| nvim`, super useful
- `/\%V` will search inside visual selection
- If clipboard register is `unnamedplus`, stuff we copy using system clipboard (not in vim) will also always be saved in the `"*` register, which is handy. e.g. we `dd` a line, it won't override what we copied!

- use `dj` and `dk` to delete the current line and the line above/below
- `<C-f>` edit ex mode in insert and normal modes
- use `_` instead of `^` because it takes to first character on line and accepts a motion for N lines back, and easier to reach