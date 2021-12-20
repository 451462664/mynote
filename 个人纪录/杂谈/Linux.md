### 在 Vim 中优雅地查找和替换

https://harttle.land/2016/08/08/vim-search-in-file.html

### 查看 cpu 使用率
top -bn 1 -i -c

%us: 表示用户空间程序的 cpu 使用率（没有通过 nice 调度）
%sy: 表示系统空间的 cpu 使用率，主要是内核程序。
%ni: 表示用户空间且通过 nice 调度过的程序的 cpu 使用率。
%id: 空闲 cpu
%wa: cpu 运行时在等待 io 的时间
%hi: cpu 处理硬中断的数量
%si: cpu 处理软中断的数量
%st: 被虚拟机偷走的 cpu