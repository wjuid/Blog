Linux控制台浏览器Lynx 简明使用指南

(一) Lynx 简介 
　　 Lynx 是一个字符界面下的全功能的WWW浏览器。Lynx 可以运行在很多种 操作系统下，如VMS, Unix, Windows 95, Windows NT等，当然也包括Linux。 由于没有漂亮的图形界面，所以 Lynx 占用资源极少，而且速度很快。另外 Lynx 还是唯一能在字符终端下运行的 WWW 浏览器。 

　　 Lynx 的主页地址是：http://lynx.browser.org ， 另外 http://www.cc.ukans.edu/lynx_help/Lynx_users_guide.HTML 是 Lynx 的用户指南。 


(二) 运行 Lynx 
　　 可以以 lynx filename 和 lynx PROTOCOL://HOST/PATH/FILENAME 的形式 运行 Lynx ，其中前一种用于浏览本地文件，后一种用于浏览 Internet。 协议(PROTOCOL)，可以是 http, gopher, ftp 和 wais。如： 

HTTP (HyperText Transfer Protocol) 
http://kuhttp.cc.ukans.edu/lynx_help/lynx_help_main.html 
Gopher 
gopher://gopher.micro.umn.edu/11/ 
FTP (File Transfer Protocol) 
ftp://ftp2.cc.ukans.edu/pub/lynx/README 
WAIS (Wide Area Information Service protocol) 
wais://cnidr.org/Directory-of-servers 


如果不带任何参数运行 Lynx，则 Lynx 会先寻找一个叫 WWW_HOME 的环境变量，如果找到的话，就会连接 WWW_HOME 指定的 URL。 WWW_HOME 变量的设置方法是，在bsh 和 ksh下： 

export WWW_HOME=http://www.w3.org/default.html 

csh 下： 

setenv WWW_HOME http://www.w3.org/default.html 

如果 WWW_HOME 变量未指定的话，Lynx 则连接它的主页：http://lynx.browser.org/ 


(三) Lynx 的键盘命令 

移动命令： 
下方向键：页面上的下一个链接(用高亮度显示)。 
上方向键：页面上的前一个链接(用高亮度显示)。 
回车和右方向键： 
跳转到链接指向的地址。 
左方向键：回到上一个页面。 

滚动命令： 
+,Page-Down,Space,Ctrl+f： 
向下翻页。 
-,Page-Up,b,Ctrl+b： 
向上翻页。 
Ctrl+a： 移动到当前页的最前面。 
Ctrl+e： 移动到当前页的最后面。 
Ctrl+n： 向下翻两行。 
Ctrl+p： 往回翻两行。 
)： 向下翻半页。 
(： 往回翻半页。 
#： 回到当前页的 Toolbar 或 Banner。 

文件操作命令： 

c： 建立一个新文件。 
d： 下载选中的文件。 
E： 编辑选中的文件。 
f： 为当前文件显示一个选项菜单。 
m： 修改选中文件的名字或位置。 
r： 删除选中的文件。 
t： Tag highlighted file。 
u： 上载一个文件到当前目录。 

其他命令： 

?,h： 帮助。 
a： 把当前链接加入到一个书签文件里。 
c： 向页面的拥有者发送意见或建议。 
d： 下载当前链接。 
e： 编辑当前文件。 
g： 跳转到一个用户指定的URL或文件。 
G： 编辑当前页的URL，并跳转到这个URL。 
i： 显示文档索引。 
j： 执行预先定义的“短”命令。 
k： 显示键盘命令列表。 
l： 列出当前页上所有链接的地址。 
m： 回到首页。 
o： 设置选项。 
p： 把当前页输出到文件，e-mail，打印机或其他地方。 
q： 退出。 
/： 在当前页内查找字符串。 
s： 在外部搜索输入的字符串。 
n： 搜索下一个。 
v： 查看一个书签文件。 
V： 跳转到访问过的地址。 
x： 不使用缓存。 
z： 停止当前传输。 
[backspace]： 
跳转到历史页(同 V 命令)。 
=： 显示当前页的信息。 
： 查看当前页的源代码。 
!： 回到shell提示符下。 
_： 清除当前任务的所有授权信息。 
*： 图形链接模式的切换开关。 
@： 8位传输模式或CJK模式的切换开关。 
[： pseudo_inlines 模式的切换开关。 
]： 为当前页或当前链接发送一个“HEAD”请求。 
Ctrl+r： 重新装如当前页并且刷新屏幕。 
Ctrl+w： 刷新屏幕。 
Ctrl+u： 删除输入的行。 
Ctrl+g： 取消输入或者传送。 
Ctrl+t： 跟踪模式的切换开关。 
;： 看 Lynx 对当前任务的跟踪记录。 
Ctrl+k： 调用 CookIE Jar 页。 
数字键： 到后面的第 n 个链接。