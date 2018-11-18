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

### Search from pattern to pattern across lines
By example, in an array like this, to match render property and the whole value:

```javascript
const Routes = [
{
    path: `/${lang}/devices`,
    component: DevicesContainer,
    render: (props) => (
      <DevicesContainer {...props} footer={footer} />
    )
  },

  {
    path: `/${lang}/features`
    component: FeaturesContainer,
    render: (props) => (
      <FeaturesContainer {...props} footer={footer} />
      )
  },

  {
    path: `/${lang}/resources`,
    component: ResourcesContainer,
    render: (props)=> (
      <ResourcesContainer {...props} />
      )
  }

]
```
do
```vim
/
" and then
^\s*render:\_.\{-}\ze\(  },\|  }\n\.\{-}]\)
"           ^^ ^^^          ^^^^^^^^^^^^^
"            \   \                \
"             `---\----------------\------------------- any char including \n
"                  `----------------\------------------ repeat as few times as possible (lazy repeating)
"                                    `----------------- setup a match for the `render:` in the last elem in the array
```

quickly delete them all:
```vim
"first
dgn
"then repitedly
.
