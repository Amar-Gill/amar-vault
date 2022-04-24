### :ls
List open buffers.

e.g.
```
:ls
  1 #    ".zshrc"                       line 1
  2 %a   ".vimrc"                       line 1
```

### :b [number | string]
Switch to open buffer at that number (from :ls).
e.g `:b 2` - go to `.vimrc`
switch to open buffer that closest matches that string.
e.g. `:b zsh` - go to `.zshrc`

### :bd
Delete currently open buffer.

### :find [string]
Find files (within pwd or also nested folders with fuzzy finding enabled). Hit enter to create new buffer for it.