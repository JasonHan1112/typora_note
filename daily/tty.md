# console，tty
## 终端
终端（termimal）= tty（Teletypewriter， 电传打印机），作用是提供一个命令的输入输出环境，在linux下使用组合键ctrl+alt+T打开的就是终端，可以认为terminal和tty是同义词。
## console
- 在计算机发展的早期，计算机的外表上通常会存在一个面板，面板包含很多按钮和指示灯，可以通过面板来对计算机进行底层的管理，也可以通过指示灯来得知计算机的运行状态，这个面板就叫console。在现代计算机上，在电脑开机时（比如ubuntu）屏幕上会打印出一些日志信息，但在系统启动完成之前，terminal不能连接到主机上，所以为了记录主机的重要日志（比如开关机日志，重要应用程序的日志），系统中就多了一个名为console的设备，这些日志信息就是显示在console上。一台电脑有且只有一个console，但可以有多个terminal。举个例子，电视机上的某个区域一般都会有一些按钮，比如开机，调音量等，这个区域就可以当做console，且这个区域在电视上只有一个，遥控器就可以类比成terminal，terminal可以有多个。
- /dev/console是系统控制台，是与操作系统交互的设备。console有缓冲的概念，为内核提供打印输出。内核把要打印的内容装入缓冲区__log_buff，然后由console来决定打印到哪里（比如是tty0还是ttySn等）。console指向激活的终端。历史上，console指主机本身的屏幕和键盘，而tty指用电缆链接的其它位置的控制台。一般console是tty的符号链接，如果终端设备要实现console功能，必须要内核注册一个struct console结构，一般串口驱动中都会有，如果设备实现tty功能，必须要内核的天通苑子设备注册一个struct tty_driver结构。
## 当前控制台
这是应用程序中的概念，如果当前进程有控制终端（Controlling Terminal），那么/dev/tty就是当前进程控制台的设备文件。对于你登录的shell，/dev/tty就是你使用的控制台，设备号是（5,0）。不过它并不指任何物理意义上的控制台，/dev/tty会映射到当前设备（使用命令“tty”可以查看它具体对应哪个实际物理控制台设备）。你可以输入命令 “tty"，将显示当前映射终端如：/dev/tty1或者/dev/pts/0等。也可以使用命令“ps -ax”来查看其他进程与哪个控制终端相连。在当前终端中输入 echo “tekkaman” > /dev/tty ，都会直接显示在当前的终端中。
## 虚拟控制台 /dev/ttyn
/dev/ttyn是进程虚拟控制台，他们共享同一个真实的物理控制台。如果在进程里打开一个这样的文件且该文件不是其他进程的控制台时，那该文件就是这个进程的控制台。进程的printf会输出到这里。而比较特殊的是/dev/tty0，他代表当前虚拟控制台，是当前所使用虚拟控制台的一个别名。因此不管当前正在使用哪个虚拟控制台（注意：这里是虚拟控制台，不包括伪终端），系统信息都会发送到/dev/tty0上。只有系统或超级用户root可以向/dev/tty0进行写操作。tty0是系统自动打开的，但不用于用户登录。在Framebuffer设备没有启用的系统中，可以使用/dev/tty0访问显卡。
## 伪终端
- 伪终端(Pseudo Terminal)是终端的发展，为满足现在需求（比如网络登陆、xwindow窗口的管理）。它是成对出现的逻辑终端设备(即master和slave设备, 对master的操作会反映到slave上)。它多用于模拟终端程序，是远程登陆(telnet、ssh、xterm等)后创建的控制台设备。
- 使用一个/dev/ptmx作为master设备，在每次打开操作时会得到一个master设备fd，并在/dev/pts/目录下得到一个slave设备（如 /dev/pts/3和/dev/ptmx），这样就避免了逐个尝试的麻烦。由于可能有好几千个用户登陆，所以/dev/pts/*是动态生成的，不象其他设备文件是构建系统时就已经产生的硬盘节点(如果未使用devfs、udev、mdev等) 。第一个用户登陆，设备文件为/dev/pts/0，第二个为/dev/pts/1，以此类推。它们并不与实际物理设备直接相关。现在大多数系统是通过此接口实现pty。
- 我们在X Window下打开的终端或使用telnet 或ssh等方式登录Linux主机，此时均通过pty设备。例如，如果某人在网上使用telnet程序连接到你的计算机上，则telnet程序就可能会打开/dev/ptmx设备获取一个fd。此时一个getty程序就应该运行在对应的/dev/pts/*上。当telnet从远端获取了一个字符时，该字符就会通过ptmx、pts/*传递给 getty程序，而getty程序就会通过pts/*、ptmx和telnet程序往网络上返回“login:”字符串信息。这样，登录程序与telnet程序就通过“伪终端”进行通信。
telnet<--->/dev/ptmx(master)<--->pts/*(slave)<--->getty
