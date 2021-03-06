# vim-sh

Since commit [5b829c9799c458ef4956009a747925b141b3a373](https://github.com/lxhillwind/vim-sh/tree/5b829c9799c458ef4956009a747925b141b3a373),
only vim with `has('vim9script')` will be supported.

## Usage
Two commands are provided:

- `:Sh [cmd]...`;

- `:Terminal [cmd]...`; (like `:Sh`, but always open tty in current window)

```console
Usage: [range]Sh [-flags] [cmd...]

Example:
  Sh uname -o

Supported flags:
  h: display this help
  v: visual mode (char level)
  t: use builtin terminal (support sub opt, like this: -t=7split)
     sub opt is used as action to prepare terminal buffer
  w: use external terminal (support sub opt, like this: -w=urxvt,w=cmd)
     currently supported: alacritty, urxvt, mintty, cmd, tmux, tmuxc, tmuxs, tmuxv, konsole
  c: close terminal after execution
  b: focus on current buffer / window
  f: filter, like ":{range}!cmd"
  r: like ":[range]read !cmd"
  n: dry run (echo options passed to job_start / jobstart)
```

details:

- `-v`: selected text as stdin (this is different from filter, which is line
  level); visual mode

- `-w`: execute shell command in new application window. To see which
  application is supported, execute `:Sh -h` or see `w` flag above (the latter
  may not be updated).

- `<bang>`: try to reuse existing builtin tty window (implies -t option)

- `-t`: support `-t=xxx` to specify cmd to prepare terminal buffer; since
  space is not allowd in opt, cmd like `:bel 7sp` should be wrapped with
  UserCommand.

```vim
:Sh! [cmd]...
```

- mix these options: (order does not matter)

```vim
Sh -wv [cmd]...
```

NOTE: `:Sh -w -v [cmd]...` will not work!

## Config

### `g:sh_path`

set variable `g:sh_path` to override default shell:

- win32 default shell: msys2 shell (msys64 zsh, msys64 bash, msys32 zsh,
  msys32 bash), git shell (git bash, git x86 bash), busybox; the first
available shell is set.
- unix-like default shell: `&shell`

*If g:sh_path contains `busybox`, then sh is appended, like `busybox sh`.*

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

### always use shell

- Content after command will be passed to shell. (this is like builtin
  `:terminal` with `++shell` option)

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

### unix shell support in native Windows vim (`has('win32') == 1`)

By default, this requires [busybox-w32](https://frippery.org/busybox/) binary
in `$PATH`.

You can also use other shell by setting variable `g:sh_path`. (take
effect at runtime)

```vim
" msys2 shell default path (x64)
let g:sh_path = 'C:/msys64/usr/bin/bash.exe'

" 32-bit git for windows (x64 system)
let g:sh_path = 'C:/Program Files (x86)/Git/usr/bin/bash'
```

- `:terminal ++shell` with unix shell syntax.
