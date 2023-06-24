# Cursor Movement

- h: move left
- j: move down
- k: move up
- l: move right
- ctrl-b: page up
- ctrl-f: page down
- %: jump to matching brace
- w: jump by start of words (punctuation considered words)
- W: jump by words (spaces separate words)
- e: jump to end of words (punctuation considered words)
- E: jump to end of words (no punctuation)
- b: jump backward by words (punctuation considered words)
- B: jump backward by words (no punctuation)
- ge: jump backward to end of words (punctuation considered words)
- gE: jump backward to end of words (no punctuation)
- 0: (zero) start of line
- ^: first non-blank character of line
- $: end of line
- gg: Go to first line
- [N]G: Go To line N. No N: last line
  Note: Prefix a cursor movement command with a number to repeat it. For example, 4j moves down 4 lines.

# Inserting/Appending Text

- i: start insert mode at cursor
- I: insert at the beginning of the line
- a: append after the cursor
- A: append at the end of the line
- o: open (append) blank line below the current line (no need to press return)
- O: open blank line above the current line
- ea: append at end of word
- Esc: exit insert mode

# Editing

- r: replace a single character (does not use insert mode)
- J: join line below to the current one
- cc: change (replace) an entire line
- cw: change (replace) to the end of word
- c$: change (replace) to the end of line
- s: delete character at cursor and substitute text
- S: delete line at cursor and substitute text (same as cc)
- xp: transpose two letters (delete and paste, technically)
- u: undo
- ctrl-r: redo
- .: repeat last command
- ~: switch case
- g~iw: switch case of the current word
- gUiw: make the current word uppercase
- guiw: make the current word lowercase
- >>: indent line one column to the right
- <<: indent line one column to the left
- ==: auto-indent current line
- ddp: swap current line with next
- ddkP: swap current line with previous

# Cut and Paste

- dd: delete (cut) a line
- dw: delete the current word
- x: delete the current character
- X: delete the previous character
- D: delete from the cursor to the end of the line
- yy: yank (copy) a line
- 2yy: yank 2 lines
- yw: yank word
- y$: yank to the end of the line
- p: put (paste) the clipboard after the cursor/current line
- P: put (paste) before the cursor/current line
- :set paste: avoid unexpected effects in pasting

# Visual Mode - Marking Text

- v: start visual mode, mark lines, then do command (such as y-yank)
- V: start Linewise visual mode
- o: move to the other end of the marked area
- U: uppercase of the marked area
- ctrl+v: start visual block mode
- O: move to the other corner of the block
- aw: mark a word
- ab: a () block (with braces)
- aB: a {} block (with brackets)
- ib: inner () block
- iB: inner {} block
- Esc: exit visual mode

# Visual Mode Commands

- >: shift right
- <: shift left
- c: change (replace) marked text
- y: yank (copy) marked text
- d: delete (cut) marked text
- ~: switch case

# Cut and Paste in Visual Mode

1. Place the cursor at the start of your text.
2. ma (marks the location as point 'a')
3. Place the cursor at the end of your text.
4. d'a (cuts back to location 'a')

# Exiting

- :w: write (save) the file, but don't exit
- :wq: write (save) and quit
- :x: same as :wq
- :q: quit (fails if anything has changed)
- :q!: quit and throw away changes

# Search/Replace

- /pattern: search for pattern
- ?pattern: search backward for pattern
- n: repeat search in the same direction
- N: repeat search in the opposite direction
- :%s/old/new/g: replace all 'old' with 'new' throughout the file
- :%s/old/new/gc: replace all 'old' with 'new' throughout the file with confirmations

# Working with Multiple Files

- :e filename: Edit a file in a new buffer
- :n **/*.pl: Open all Perl files under the current directory, recursively
- :tabe filename: Edit a file in a new tab (Vim7, gVim)
- :bnext (or :bn): go to the next buffer
- :bprev (or :bp): go to the previous buffer
- :bd: delete a buffer (close a file)
- :sp filename: Open a file in a new buffer and split window
- ctrl-w s: Split windows
- ctrl-w w: switch between windows
- ctrl-w q: Quit a window
- ctrl-w v: Split windows vertically

# Tabs (Vim7)

- {i}gt: go to tab in position i
- gt: Next tab
- gT: Previous tab
- :tabr: First tab
- :tabl: Last tab
- :tabm [N]: Move current tab after tab N. No N: last. N=0: first.
- $vim -p file1 file2 fileN: Open multiple files in different tabs (Vim7)

# Command Line Options

- $vim -c "vim command" file: execute an editor command on startup

# Commands

- :setlocal fileformat=dos|unix
- :setlocal fileencoding=utf-8

# More Help

- :help: main help portal (with links to more help pages)
- :viusage: show a huge cheat sheet listing every command
