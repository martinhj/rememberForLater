# Vim stuff I want to remember

## Commands


### Redirect output of commands to register
:redir @* | set guifont | redir END

### Open 'all files' in independent tabs
:args file1 file2 | argdo tabe

## Config

### Reconfigure some colors when `colorscheme` is ran
```vim
augroup matchup_matchparen_highlight
autocmd!
autocmd ColorScheme * hi MatchParen guifg=red
augroup END
```
(source: https://github.com/andymass/vim-matchup)
