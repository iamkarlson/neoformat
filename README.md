# Neoformat [![Build Status](https://travis-ci.org/sbdchd/neoformat.svg?branch=master)](https://travis-ci.org/sbdchd/neoformat)

A [Neovim](https://neovim.io) and Vim8 plugin for formatting code.

Neoformat uses a variety of formatters for many filetypes. Currently, Neoformat
will run a formatter using the current buffer data, and on success it will
update the current buffer with the formatted text. On a formatter failure,
Neoformat will try the next formatter defined for the filetype.

By using `getbufline()` to read from the current buffer instead of file,
Neoformat is able to format your buffer without you having to `:w` your file first.
Also, by using `setline()`, marks, jumps, etc. are all maintained after formatting.

Neoformat supports both sending buffer data to formatters via stdin, and also
writing buffer data to `/tmp/` for formatters to read that do not support input
via stdin.

## Basic Usage

Format the current file using its filetype

```viml
:Neoformat
```

Or specify a certain formatter (must be defined for the current filetype)

```viml
:Neoformat js-beautify
```

Or perhaps run a formatter on save

```viml
augroup fmt
  autocmd!
  autocmd BufWritePre * Neoformat
augroup END
```

## Install

[vim-plug](https://github.com/junegunn/vim-plug)

```viml
Plug 'sbdchd/neoformat'
```

## Current Limitation(s)

If a formatter is either not configured to use `stdin`, or is not able to read
from `stdin`, then buffer data will be written to a file in `/tmp/neoformat/`,
where the formatter will then read from

## Config [Optional]

Define custom formatters.

Options:

| name        | description                                                                                                        | default | optional / required |
| ----------- | ------------------------------------------------------------------------------------------------------------------ | ------- | ------------------- |
| `exe`       | the name the formatter executable in the path                                                                      | n/a     | **required**        |
| `args`      | list of arguments                                                                                                  | \[]     | optional            |
| `replace`   | overwrite the file, instead of updating the buffer                                                                 | 0       | optional            |
| `stdin`     | send data to the stdin of the formatter                                                                            | 0       | optional            |
| `no_append` | do not append the `path` of the file to the formatter command, used when the `path` is in the middle of a command  | 0       | optional            |

Example:

```viml
let g:neoformat_python_autopep8 = {
            \ 'exe': 'autopep8',
            \ 'args': ['-s 4', '-E'],
            \ 'replace': 1 " replace the file, instead of updating buffer (default: 0),
            \ 'stdin': 1, " send data to stdin of formatter (default: 0)
            \ 'no_append': 1,
            \ }

let g:neoformat_enabled_python = ['autopep8']
```

Configure enabled formatters.

```viml
let g:neoformat_enabled_python = ['autopep8', 'yapf']
```

Enable basic formatting when a filetype is not found. Disabled by default.

```viml
" Enable alignment
let g:neoformat_basic_format_align = 1

" Enable tab to spaces conversion
let g:neoformat_basic_format_retab = 1

" Enable trimmming of trailing whitespace
let g:neoformat_basic_format_trim = 1
```

Make Neoformat read from the file instead of the buffer.

```viml
let g:neoformat_read_from_buffer = 0
```

When debugging, you can enable either of following variables for extra logging.

```viml
let g:neoformat_verbose = 1 " only affects the verbosity of Neoformat
" Or
let &verbose            = 1 " also increases verbosity of the editor as a whole
```

## Adding a New Formatter

Note: you should replace everything `{{ }}` accordingly

1. Create a file in `autoload/neoformat/formatters/{{ filetype }}.vim` if it does not
   already exist for your filetype.

2. Follow the following format

See Config above for options

```viml
function! neoformat#formatters#{{ filetype }}#enabled() abort
    return ['{{ formatter name }}', '{{ other formatter name for filetype }}']
endfunction

function! neoformat#formatters#{{ filetype }}#{{ formatter name }}() abort
    return {
        \ 'exe': '{{ formatter name }}',
        \ 'args': ['-s 4', '-q'],
        \ 'stdin': 1
        \ }
endfunction

function! neoformat#formatters#{{ filetype }}#{{ other formatter name }}() abort
  return {'exe': {{ other formatter name }}
endfunction
```

## Supported Filetypes

- Arduino
  - [`uncrustify`](http://uncrustify.sourceforge.net),
    [`clang-format`](http://clang.llvm.org/docs/ClangFormat.html),
    [`astyle`](http://astyle.sourceforge.net)
- C
  - [`uncrustify`](http://uncrustify.sourceforge.net),
    [`clang-format`](http://clang.llvm.org/docs/ClangFormat.html),
    [`astyle`](http://astyle.sourceforge.net)
- C#
  - [`uncrustify`](http://uncrustify.sourceforge.net),
    [`astyle`](http://astyle.sourceforge.net)
- C++
  - [`uncrustify`](http://uncrustify.sourceforge.net),
    [`clang-format`](http://clang.llvm.org/docs/ClangFormat.html),
    [`astyle`](http://astyle.sourceforge.net)
- Crystal
  - `crystal tool format` (ships with [`crystal`](http://crystal-lang.org))
- CSS
  - `css-beautify` (ships with [`js-beautify`](https://github.com/beautify-web/js-beautify)),
    [`prettydiff`](https://github.com/prettydiff/prettydiff),
    [`stylefmt`](https://github.com/morishitter/stylefmt),
    [`csscomb`](http://csscomb.com)
- CSV
  - [`prettydiff`](https://github.com/prettydiff/prettydiff)
- D
  - [`uncrustify`](http://uncrustify.sourceforge.net),
    [`dfmt`](https://github.com/Hackerpilot/dfmt)
- Dart
  - [`dartfmt`](https://www.dartlang.org/tools/)
- Elm
  - [`elm-format`](https://github.com/avh4/elm-format)
- Go
  - [`gofmt`](https://golang.org/cmd/gofmt/),
    [`goimports`](https://godoc.org/golang.org/x/tools/cmd/goimports)
- Haskell
  - [`stylish-haskell`](https://github.com/jaspervdj/stylish-haskell)
- HTML
  - `html-beautify` (ships with [`js-beautify`](https://github.com/beautify-web/js-beautify)),
    [`prettydiff`](https://github.com/prettydiff/prettydiff)
- Jade
  - [`pug-beautifier`](https://github.com/vingorius/pug-beautifier)
- Java
  - [`uncrustify`](http://uncrustify.sourceforge.net),
    [`astyle`](http://astyle.sourceforge.net)
- Javascript
  - [`js-beautify`](https://github.com/beautify-web/js-beautify),
    [`prettydiff`](https://github.com/prettydiff/prettydiff),
    [`clang-format`](http://clang.llvm.org/docs/ClangFormat.html),
    [`esformatter`](https://github.com/millermedeiros/esformatter/)
- JSON
  - [`js-beautify`](https://github.com/beautify-web/js-beautify),
    [`prettydiff`](https://github.com/prettydiff/prettydiff)
- Less
  - [`csscomb`](http://csscomb.com),
    [`prettydiff`](https://github.com/prettydiff/prettydiff)
- Lua
  - [`luaformatter`](https://github.com/LuaDevelopmentTools/luaformatter)
- Markdown
  - [`remark`](https://github.com/wooorm/remark)
- Objective-C
  - [`uncrustify`](http://uncrustify.sourceforge.net),
    [`clang-format`](http://clang.llvm.org/docs/ClangFormat.html),
    [`astyle`](http://astyle.sourceforge.net)
- OCaml
  - [`ocp-indent`](http://www.typerex.org/ocp-indent.html)
- Pandoc Markdown
  - [`pandoc`](https://pandoc.org/MANUAL.html)
- Pawn
  - [`uncrustify`](http://uncrustify.sourceforge.net)
- Perl
  - [`perltidy`](http://perltidy.sourceforge.net)
- PHP
  - [`php_beautifier`](http://pear.php.net/package/PHP_Beautifier)
- Proto
  - [`clang-format`](http://clang.llvm.org/docs/ClangFormat.html)
- Pug (formally Jade)
  - [`pug-beautifier`](https://github.com/vingorius/pug-beautifier)
- Python
  - [`yapf`](https://github.com/google/yapf),
    [`autopep8`](https://github.com/hhatto/autopep8)
- Ruby
  - [`ruby-beautify`](https://github.com/erniebrodeur/ruby-beautify)
- Rust
  - [`rustfmt`](https://github.com/rust-lang-nursery/rustfmt)
- Sass
  - [`sass-convert`](http://sass-lang.com/documentation/#executables),
    [`csscomb`](http://csscomb.com)
- Scala
  - [`scalariform`](https://github.com/scala-ide/scalariform)
- SCSS
  - [`sass-convert`](http://sass-lang.com/documentation/#executables),
    [`stylefmt`](https://github.com/morishitter/stylefmt),
    [`prettydiff`](https://github.com/prettydiff/prettydiff),
    [`csscomb`](http://csscomb.com)
- Shell
  - [`shfmt`](https://github.com/mvdan/sh)
- SQL
  - `sqlformat` (ships with [sqlparse](https://github.com/andialbrecht/sqlparse))
- Typescript
  - [`tsfmt`](https://github.com/vvakame/typescript-formatter)
- VALA
  - [`uncrustify`](http://uncrustify.sourceforge.net)
- XHTML
  - [`tidy`](http://www.html-tidy.org),
    [`prettydiff`](https://github.com/prettydiff/prettydiff)
- XML
  - [`tidy`](http://www.html-tidy.org),
    [`prettydiff`](https://github.com/prettydiff/prettydiff)
