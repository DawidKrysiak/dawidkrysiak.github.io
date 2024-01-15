---
{"dg-publish":true,"permalink":"/sys-admin-hand-book/terminal/vim-magic/","title":"Vim magic","tags":["vim","Linux","GNU","windows","macos"]}
---

## Why it's here, there are so many tutorials online?
I decided to give it a go and ditch visual IDEs completely. Trying to make neovim my primary text editor / IDE. Keeping the notes here, just in case I need a cheat sheet.
Basics of vi [openvim](https://www.openvim.com/tutorial.html)
## copy / paste within the same file
* yy to yank one line, p to paste
* ctrl+v to engage visual select, y to yank (or d to delete/cut), p to paste

## vimdiff copy between left/right panes

* Shift+V to select a line
* k or j; { or }; up or down keys to select more lines
* y (yank/copy selected lines)
* ctrl+w, left/right to move to the other pane
* p (paste)

## format JSON

* :%!python -m json.tool


(c) Dawid Krysiak https://itisoktoask.me/
