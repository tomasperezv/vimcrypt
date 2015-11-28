# vimcrypt
Configuration scripts for enable secure encryption in VIM editor

### Native encryption methods

Vim editor has 3 native modes of encryption  
- pkzip based (deprecated)
- blowfish based (vim > 7.3)
- blowfish2 (vim > 7.4.399)

It's recommended to use blowfish2 since the 2 first options have well known vulnerabilities[1].

In order to enable blowfish2, you must set the cryptmethod variable (`cm`)

```vimscript
set cm=blowfish2
```

#### Additional configuration

We also need to take into consideration other vim settings in order to avoid leaving traces of the encrypted files content, specially the swap and backup files.

```vimscript
set noswapfile
set nobackup
set nowritebackup
set viminfo=
```

### GPG support

In addition to the native cryptmethod, vim can be easily integrated with external encryption engines, the most remarkable being GPG. This is more recommended due to obvious reasons.

The following script[2] by Wouter Hanegraaff provides transparent editing of GPG encrypted files.  

1. Avoid writing to ~/.viminfo while editing

  ```vimscript
    autocmd BufReadPre,FileReadPre *.gpg set viminfo=
    autocmd BufReadPre,FileReadPre *.gpg set noswapfile noundofile nobackup
  ```

2. FileReadPre: switch to binary mode when reading

  ```vimscript
    " Switch to binary mode to read the encrypted file
    autocmd BufReadPre,FileReadPre *.gpg set bin
    autocmd BufReadPre,FileReadPre *.gpg let ch_save = &ch|set ch=2
  ```

3. FileReadPost: switch to normal mode for editing

  ```vimscript
    autocmd BufReadPost,FileReadPost *.gpg '[,']!gpg --decrypt 2> /dev/null
    autocmd BufReadPost,FileReadPost *.gpg set nobin
    autocmd BufReadPost,FileReadPost *.gpg let &ch = ch_save|unlet ch_save
    autocmd BufReadPost,FileReadPost *.gpg execute ":doautocmd BufReadPost " . expand("%:r")
  ```

4. FileWritePre: encrypt text before writing

  ```vimscript
    autocmd BufWritePre,FileWritePre *.gpg '[,']!gpg --default-recipient-self -ae 2>/dev/null
    autocmd BufWritePost,FileWritePost *.gpg u
  ```

### Reference

[1] https://dgl.cx/2014/10/vim-blowfish
[2] http://vim.wikia.com/wiki/Encryption
