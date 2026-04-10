# CKAD Exam Preparation

In the real exam, you'll be given a different instance to be ssh into for each question. So, if we were to use a `~/.bashrc` and `~/.vimrc` file, we'd have to set them up for each question. In addtion, in the real exam, the autocompletion is already set up for us fairly well (i.e., including `k` alias), except no shortcut alias like `--dry-run=client -o yaml` and `--force --grace-period=0`. 

So, my approach is to skip setting up a `~/.bashrc` and `~/.vimrc` file entirely, and use the following vim command when I feel the need while editing in vim:

```vim
:set et ts=2 sw=2 ai nu
```

This will use space for tab, tabstop=2, shiftwidth=2, auto indent, and line number respectively. 
