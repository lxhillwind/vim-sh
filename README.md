# vim-sh

## Example

- `-v`: selected text as stdin, instead of selected line (builtin `<Visual>:!`)

```vim
<Visual>:Sh -v cat
```

- `-w`: execute shell command in new application window.

```vim
" use default detected application
:Sh -w ls -l

" force using cmd.exe
:Sh -w=cmd ls -l
```

- `-t`: execute shell command in embedded terminal. Support `-t=xxx` to
  specify cmd to prepare terminal buffer; since space is not allowd in opt,
cmd like `:bel 7sp` should be wrapped with UserCommand.

```vim
" run after :vsplit
:Sh -t=vs pwd
" use current buffer
:Sh -t= pwd
" use default layout (window height is like cmdwin)
:Sh -t pwd
```

- `<bang>`: try to reuse existing builtin tty window (implies -t option);

- `-b`: run command in background (move focus back to edit buffer).

```vim
:Sh! -b gcc % -o %:r && %:r
" also try this (if in tmux session):
:Sh -b,w=tmuxs gcc % -o %:r && %:r
```

## Usage

```console
Usage: [range]Sh[!] [-flags] [cmd...]

Example:
  Sh uname -o

Flags parsing rule:
  "," delimited; if item contains "=", it is used as sub opt; else it is combination of flags

Supported flags:
  h: display this help
  !: (:Sh! ...); try to reuse terminal window (implies -t)
  v: visual mode (char level)
  t: use builtin terminal (support sub opt, like this: -t=7split)
     sub opt is used as action to prepare terminal buffer
  w: use external terminal (support sub opt, like this: -w=urxvt,w=cmd)
     currently supported: kitty, alacritty|alac, konsole|kde, xfce4Terminal|xfce, urxvt, WindowsTerminal|wt, ConEmu|conemu, mintty, cmd, tmux, tmuxc, tmuxs, tmuxv
     order can be controlled by variable `g:sh_programs`
  c: close terminal after execution
  b: focus on current buffer / window
  f: filter, like ":{range}!cmd"
  r: like ":[range]read !cmd"
  n: dry run (echo options passed to job_start)
  S: run command directly, skipping shell
  g: open file or run command in background / gui context
     implies -S / -w option; use ":!start" in win32, job_start in other systems
     when using job_start, open / xdg-open is used when only one arg given.
```

## optional Config

### `g:sh_path`

set variable `g:sh_path` to override default shell:

- win32 default shell: msys2 shell (msys64 zsh, msys64 bash, msys32 zsh,
  msys32 bash), git shell (git bash, git x86 bash), busybox; the first
available shell is set.
- unix-like default shell: `&shell`

### `g:sh_programs`

set variable `g:sh_programs` to override default `-w` program detection order:

default is like: `['alacritty', 'urxvt', ...]`

(all available values can be viewed by executing `:Sh -h`)

#### experimental

element of `g:sh_programs` can be function, like this:

```vim
" use !empty() to turn job object into number
let g:sh_programs = [{x -> !empty(job_start(['alacritty', '-e'] + x.cmd))}]
```

If the function returns 0, then try the next element.

## Feature

### support both Vim and Neovim

(embedded terminal behavior may be different, because of neovim / vim's
different terminal implementation)

### not affected by &shell / &shellcmdflag related setting

Especially if using Windows;

like Vim's `:terminal ++shell` with unix shell syntax.

### always use shell

So things like `&&`, `||`, `if`, `*` work by default.

(though shell can be skipped intentionally by `-S` flag)

### proper % expand

- `%` with optional modifiers (`:p` / `:h` / `:t` / `:r` / `:e`) is expanded
  only when passed as a standalone argument; and it is shell-escaped (like
  `:S` modifier is always used; use unix shellescape rule even on win32).

- to make it compatible with `:!` command, `:S` modifier can be passed, and it
  will be trimmed (use custom `shellescape` instead).

This means that command like `Sh printf %s %:t:e` will print file basename
(`%s` is not expanded; `%:t:e` should NOT be quoted, as it is escaped
automatically).

`Sh printf %s %:t:e:S` also works.
