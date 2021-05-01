# vim

配置文件为`~/.vimrc`，以下是一份简单的配置文件示例：

```shell
set nocompatible
set nu!
filetype on
set history=100
set autoindent
set cindent
set smartindent
set ai!
set showmatch
set guioptions-=T
set vb t_vb=
set ruler
set hlsearch
syntax enable
set tags=./.tags;,.tags
let mapleader = ","
let g:mapleader = ","
set list
set listchars=tab:→\ ,trail:·,space:·
set cul
set tabstop=8
set shiftwidth=8
set cc=0

"plug
call plug#begin('~/.vim/plugged')
Plug 'morhetz/gruvbox'
Plug 'scrooloose/nerdtree', { 'on': 'NERDTreeToggle' }
call plug#end()

"colorscheme
set background=dark
let g:gruvbox_contrast_dark='hard'
colorscheme gruvbox
"set laststatus=2

function C_cc_toggle()
        if &cc == "0"
                set cc=81
        else
                set cc=0
        endif
endfunction

function C_list_toggle()
        if &list == 1
                set nolist
        else
                set list
        endif
endfunction

nmap <leader>w :w!<CR>
nmap <leader>s :set tabstop=4<CR>:set shiftwidth=4<CR>:set expandtab<CR>
nmap <leader>S :set tabstop=8<CR>:set shiftwidth=8<CR>:set expandtab<CR>
nmap <leader>t :set tabstop=4<CR>:set shiftwidth=4<CR>:set noexpandtab<CR>
nmap <leader>T :set tabstop=8<CR>:set shiftwidth=8<CR>:set noexpandtab<CR>
nmap <leader>r :call C_cc_toggle()<CR>
nmap <leader>l :call C_list_toggle()<CR>
nmap <C-]> g<c-]>               "ctags显示所有匹配项
nmap <leader>g <C-w>g]

nmap <F7> :NERDTreeToggle<CR>
"autocmd vimenter * if !argc()|NERDTree|endif
autocmd bufenter * if (winnr("$") == 1 && exists("b:NERDTree") && b:NERDTree.isTabTree()) | q | endif
```
