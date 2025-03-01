debian、ubuntu下使用vim键盘错乱，方向键变乱码 退格键不能使用的解决方法
> vi /etc/vim/vimrc.tiny

set compatible 改为 set nocompatible

添加一行：
set backspace=2

变成：

set nocompatible

set backspace=2