# Editor and git integration

## Any editor

yang is a drop-in replacement for the `yapf` command line. Any editor
integration that runs `yapf` (py-yapf.el for Emacs, PyYapf for Sublime Text,
VS Code formatter extensions, and so on) can run yang instead: point the
integration's executable setting at `yang`. Command line flags are identical.

## Vim

The `vim` plugin reformats a range of code. Copy the `plugin` and `autoload`
directories into your `~/.vim`, use `:packadd`, or use a plugin manager:

```vim
" Plug
Plug 'CarterPerez-dev/yang', { 'rtp': 'plugins/vim', 'for': 'python' }
```

Key bindings for `.vimrc`:

```vim
map <C-Y> :call yang#Yang()<cr>
imap <C-Y> <c-o>:call yang#Yang()<cr>
```

Or call the `Yang` command. Without a range it formats the whole buffer:

```vim
:Yang       " formats whole buffer
:'<,'>Yang  " formats lines selected in visual mode
```

## pre-commit

```yaml
repos:
  - repo: https://github.com/CarterPerez-dev/yang
    rev: v1.0.0
    hooks:
      - id: yang
```

## Plain git pre-commit hook

`plugins/pre-commit.sh` formats staged Python files before they are committed.
Changes it makes stay unstaged so you can review them:

```bash
curl -o pre-commit.sh https://raw.githubusercontent.com/CarterPerez-dev/yang/main/plugins/pre-commit.sh
chmod a+x pre-commit.sh
mv pre-commit.sh .git/hooks/pre-commit
```
