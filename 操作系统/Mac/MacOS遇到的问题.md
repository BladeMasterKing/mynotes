# MacOS遇到的问题



## eclipse提示无法打开



前两天刚刚在Mac上安装了eclipse，想要写一个Java，但是却弹出一个错误窗口，提示“应用程序eclipse无法打开”。不知道是什么问题，在访达中的Application文件夹中右键打开也不能用，不知道是什么问题。 

后来查找了很多资料，终于找到了问题所在，是因为权限不足的问题，所以无法打开。

**解决方法**

重新将这个App签一次名即可

```bash
sudo codesign --force --deep --sign - /Applications/Eclipse.app
```

输入完回车以后，会提示`/Applications/Eclipse.app: replacing existing signature`，这里耐心等待它执行完就可以打开了。



## Homebrew 安装的 mysql@5.7 突然不能启动



之前通过 Homebrew 安装的 mysql@5.7 突然不能启动，查找了一下，原来配置的mysql 环境变量是

```bash
PATH=$PATH:/usr/local/mysql/bin
```

发现目录消失了，查了下 Homebrew 的软件安装是如下操作：

1. 通过 `brew install` 安装应用最先是放在 `/usr/local/Cellar/` 目录下。

2. 有些应用会自动创建软链接放在 `/usr/bin` 或者 `/usr/sbin`，同时也会将整个文件夹放在 `/usr/local`

3. 可以使用 `brew list` 软件名确定安装位置。

```bash
$ brew list mysql@5.7
/usr/local/Cellar/mysql@5.7/5.7.42/.bottle/etc/my.cnf
/usr/local/Cellar/mysql@5.7/5.7.42/bin/innochecksum
/usr/local/Cellar/mysql@5.7/5.7.42/bin/lz4_decompress
/usr/local/Cellar/mysql@5.7/5.7.42/bin/my_print_defaults
/usr/local/Cellar/mysql@5.7/5.7.42/bin/myisam_ftdump
```

之前在 `/usr/local/mysql/bin` 中建立的软连接被删掉了，

现在将环境变量中的地址改为 Homebrew 安装的 `mysql@5.7` 所在的真实地址 `/usr/local/Cellar/mysql@5.7/5.7.42/bin`



1.文件权限

sudo命令进入root用户模式后，修改文件以及授权都权限不允许、或者文件系统只读：

原因：使用的sudo是 /usr/local/bin/sudo

解决办法：改用全路径 /usr/bin/sudo 执行即可

设置环境变量

每次需要在mac上设置环境变量时，总是要重新上网搜索该怎么设置，而且只依葫芦画瓢，没搞懂每个步骤，今天痛定思痛，一定要搞清楚，一劳永逸。好，我们开始

为什么要设置环境变量？

背景

在cmd中想要执行net start mysql等操作命令，必须先cd到bin文件所在目录，如D:\mysql\mysql-x.x.xx-winx64\bin，那么每次打开mysql 都要输入那么多指令切换目录是不是很讨厌？怎么弄呢？

原理

当你输入一个指令，比如：net start mysql，那系统怎么知道这个指令有没有呢？系统做了什么事？其实系统是在当前目录和系统环境变量path里面的路径全部查找一遍，找到第一个为准，找不到就报错。所以我们要不每次都切换cmd目录，要不就设置环境变量，以后就不需要再切换cmd路径了。简单的说环境变量里面的path路径这东西，就是cmd系统的查找目录路径

原本必须要到安装目录下才能执行，为了实现在计算机的任意目录下都能执行，才需要配置环境变量。

下面说下几个变量的作用：

(1)、PATH环境变量：作用是指定命令搜索路径，在命令行下面执行命令如javac编译java程序时，它会到PATH变量所指定的路径中查找看是否能找到相应的命令程序。我们需要把jdk安装目录下的bin目录增加到现有的PATH变量中，bin目录中包含经常要用到的可执行文件如javac/java/javadoc等等，设置好PATH变量后，就可以在任何目录下执行javac/java等工具了。

(2)、CLASSPATH环境变量：作用是指定类搜索路径，要使用已经编写好的类，前提当然是能够找到它们了，JVM就是通过CLASSPTH来寻找类的。我们 需要把jdk安装目录下的lib子目录中的dt.jar和tools.jar设置到CLASSPATH中，当然，当前目录“.”也必须加入到该变量中。

javac -c 路径 （可以指定class文件存放目录）

java -cp 路径 （可以指定要执行的class目录）

(3)、JAVA_HOME环境变量：它指向jdk的安装目录，Eclipse/NetBeans/Tomcat等软件就是通过搜索JAVA_HOME变量来找到并使用安装好的jdk。

环境变量的作用是指定Java类所在的目录，我们在写好java文件后，去执行java经常会报错找不到主类。就是没有配置好这个变量的原因 。

规则

Mac 系统的环境变量，加载顺序为：/etc/profile /etc/paths ~/.bash_profile ~/.bash_login ~/.profile ~/.bashrc

其中/etc/profile和/etc/paths是系统级别的，系统启动就会加载，后面几个是当前用户级的环境变量*

后面3个按照从前往后的顺序读取，如果~/.bash_profile文件存在，则后面的几个文件就会被忽略不读了，如果~/.bash_profile文件不存在，才会以此类推读取后面的文件

~/.bashrc没有上述规则，它是bash shell打开的时候载入的。

PATH 的语法为 中间用冒号隔开

export PATH=$PATH:<PATH 1>:<PATH 2>:<PATH 3>:...:<PATH N>

如果要查看 $PATH 的路径

echo $PATH
/etc/paths（如果是全局，建议修改这个文件)
编辑 paths，将环境变量添加到 paths 文件中 ，一行一个路径
/etc/profile（建议不修改这个文件)
全局（公有）配置，不管是哪个用户，登录时都会读取该文件。
/etc/bashrc (一般在这个文件中添加系统级环境变量)
全局（公有）配置，bash shell执行时，不管是何种方式，都会读取此文件

.profile 文件为系统的每个用户设置环境信息,当用户第一次登录时,该文件被执行.并从/etc/profile.d目录的配置文件中搜集shell的设置

如果你有对/etc/profile有修改的话必须得重启你的修改才会生效，此修改对每个用户都生效

./bashrc 每一个运行bash shell的用户执行此文件.当bash shell被打开时,该文件被读取.

对所有的使用bash的用户修改某个配置并在以后打开的bash都生效的话可以修改这个文件，修改这个文件不用重启，重新打开一个bash即可生效。

./bash_profile 该文件包含专用于你的bash shell的bash信息,当登录时以及每次打开新的shell时,该文件被读取.（每个用户都有一个.bashrc文件，在用户目录下）

需要需要重启才会生效，/etc/profile对所有用户生效，~/.bash_profile只对当前用户生效。

JAVA

.bash_profile 为每个用户配置环境变量，用户登录后会自动读取一次

我们切换到当前用户主目录，然后编辑 .bash_profile 文件

cd ～

vim .bash_profile

输入内容

假如我们有两个JAVA环境,需要任意切换 [java15, java1.8]

export JAVA15_HOME=/Library/Java/JavaVirtualMachines/jdk-15.0.1.jdk/Contents/Home

export JAVA8_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_281.jdk/Contents/Home

export JAVA_HOME=$JAVA8_HOME

alias jdk8='export JAVA_HOME=$JAVA8_HOME'

alias jdk15='export JAVA_HOME=$JAVA15_HOME'

要使这段设置立即生效，直接执行下边命令

source ~/.bash_profile

这样当我们登录该用户后，默认使用 java1.8 版本

如果我们输入 jdk15 则可以直接切换 到 java15 版本

Note: alias 是用于设置命令别名，详情请参考： 在Linux中通过alias设置命令别名 | 《Linux就该这么学》

MYSQL

我们切换到当前用户主目录，然后编辑 .bash_profile 文件

cd ~

vim .bash_profile

添加 mysql bin 目录路径到环境变量

export PATH=$PATH:/usr/local/mysql/bin

执行下边命令

source ~/.bash_profile

Python

安装完毕python3以后，敲入命令 which python3，安装路径为：

/Library/Frameworks/Python.framework/Versions/3.7/bin/python3.7

我们将这个路径配入环境变量

vim ~/.bash_profile

最后面加入下面这行

PATH=$PATH:Library/Frameworks/Python.framework/Versions/3.7/bin

alias python="/Library/Frameworks/Python.framework/Versions/3.7/bin/python3.7”

保存好后执行 source ~/.bash_profile 或者重启电脑。

然后我们敲入python 即可调用我们的python3了

Allure

#设置allure的PATH

export PATH=/usr/bin:/bin:/usr/sbin:/sbin:/usr/local/bin

export PATH=${PATH}:/Users/yourname/allure-2.17.1/bin

有同学可能会问Mysql 和python的都是PATH 那会不会重复？能不能识别正确？

path里面必然会有很多个路径， ：表示了分割符，使用时软件会从前往后遍历

Python

Easy_install

~/.pydistutils.cfg
[easy_install]
index_url = https://pypi.tuna.tsinghua.edu.cn/simple

Pip

~/.pip/pip.conf
[global]
index-url = https://pypi.tuna.tsinghua.edu.cn/simple

安装homebrew国内镜像

/bin/zsh -c "$(curl -fsSL https://gitee.com/cunkai/HomebrewCN/raw/master/Homebrew.sh)"

安装git

brew install git

配置java环境变量

# 配置JAVA_HOME

export JAVA_HOME=$(/usr/libexec/java_home)
export PATH=$JAVA_HOME/bin:$PATH
export CLASS_PATH=$JAVA_HOME/lib

iterm2设置

1.安装iterm2

[](http://%5Bhttps://gitee.com/mirrors/oh-my-zsh)2. [Gitee的oh-my-zsh仓库](https://gitee.com/mirrors/oh-my-zsh) 下载oh-my-zsh安装包

3.解压后执行 tools目录下的 install.sh 脚本

Mac系统执行命令出现：

chmod: Unable to change file mode on /usr/share/zsh: Read-only file system

出现这个问题是SIP的问题，也就是Mac系统的“系统完整性保护”机制，该机制作为Mac的保护机制，会限制我们进行部分操作，所以我们需要把SIP进行关闭。

➜ jiansheng csrutil status

System Integrity Protection status: disabled.

idea破解

执行重置脚本

reset.sh

执行安装脚本

install.sh

输入activation code

6G5NXCPJZB-eyJsaWNlbnNlSWQiOiI2RzVOWENQSlpCIiwibGljZW5zZWVOYW1lIjoic2lnbnVwIHNjb290ZXIiLCJhc3NpZ25lZU5hbWUiOiIiLCJhc3NpZ25lZUVtYWlsIjoiIiwibGljZW5zZVJlc3RyaWN0aW9uIjoiIiwiY2hlY2tDb25jdXJyZW50VXNlIjpmYWxzZSwicHJvZHVjdHMiOlt7ImNvZGUiOiJQU0kiLCJmYWxsYmFja0RhdGUiOiIyMDI1LTA4LTAxIiwicGFpZFVwVG8iOiIyMDI1LTA4LTAxIiwiZXh0ZW5kZWQiOnRydWV9LHsiY29kZSI6IlBEQiIsImZhbGxiYWNrRGF0ZSI6IjIwMjUtMDgtMDEiLCJwYWlkVXBUbyI6IjIwMjUtMDgtMDEiLCJleHRlbmRlZCI6dHJ1ZX0seyJjb2RlIjoiSUkiLCJmYWxsYmFja0RhdGUiOiIyMDI1LTA4LTAxIiwicGFpZFVwVG8iOiIyMDI1LTA4LTAxIiwiZXh0ZW5kZWQiOmZhbHNlfSx7ImNvZGUiOiJQUEMiLCJmYWxsYmFja0RhdGUiOiIyMDI1LTA4LTAxIiwicGFpZFVwVG8iOiIyMDI1LTA4LTAxIiwiZXh0ZW5kZWQiOnRydWV9LHsiY29kZSI6IlBHTyIsImZhbGxiYWNrRGF0ZSI6IjIwMjUtMDgtMDEiLCJwYWlkVXBUbyI6IjIwMjUtMDgtMDEiLCJleHRlbmRlZCI6dHJ1ZX0seyJjb2RlIjoiUFNXIiwiZmFsbGJhY2tEYXRlIjoiMjAyNS0wOC0wMSIsInBhaWRVcFRvIjoiMjAyNS0wOC0wMSIsImV4dGVuZGVkIjp0cnVlfSx7ImNvZGUiOiJQV1MiLCJmYWxsYmFja0RhdGUiOiIyMDI1LTA4LTAxIiwicGFpZFVwVG8iOiIyMDI1LTA4LTAxIiwiZXh0ZW5kZWQiOnRydWV9LHsiY29kZSI6IlBQUyIsImZhbGxiYWNrRGF0ZSI6IjIwMjUtMDgtMDEiLCJwYWlkVXBUbyI6IjIwMjUtMDgtMDEiLCJleHRlbmRlZCI6dHJ1ZX0seyJjb2RlIjoiUFJCIiwiZmFsbGJhY2tEYXRlIjoiMjAyNS0wOC0wMSIsInBhaWRVcFRvIjoiMjAyNS0wOC0wMSIsImV4dGVuZGVkIjp0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

清理IntellijIdea

在应用程序里删除IntellijIdea.app

cd /Users/xxx/
rm -rf Logs/IntelliJIdeaxxx/
rm -rf Preferences/IntelliJIdeaxxx/
rm -rf Application\ Support/IntelliJIdeaxxx/
rm -rf Caches/IntelliJIdeaxxx

MacOS降级

创建可引导安装器需要满足的条件

USB 闪存驱动器或其他备用宗卷（格式化为 Mac OS 扩展格式），至少有 14 GB 可用储存空间

已下载 macOS Monterey、Big Sur、Catalina、Mojave、High Sierra 或 El Capitan 的安装器

下载 macOS

下载：[macOS Monterey](https://apps.apple.com/cn/app/macos-monterey/id1576738294?mt=12)、[macOS Big Sur](https://apps.apple.com/cn/app/macos-big-sur/id1526878132?mt=12)、[macOS Catalina](https://apps.apple.com/cn/app/macos-catalina/id1466841314?mt=12)、[macOS Mojave](https://apps.apple.com/cn/app/macos-mojave/id1398502828?mt=12) 或 [macOS High Sierra](https://apps.apple.com/cn/app/macos-high-sierra/id1246284741?mt=12)这些内容会作为名为 “安装 macOS [版本名称]” 的 App 下载到您的“应用程序”文件夹。如果安装器在下载后打开，请退出而不要继续安装。要获取正确的安装器，请从运行 macOS Sierra 10.12.5 或更高版本或者 El Capitan 10.11.6 的 Mac 中进行下载。如果您是企业管理员，请通过 Apple 下载，而不要通过本地托管的软件更新服务器进行下载。

下载：[OS X El Capitan](http://updates-http.cdn-apple.com/2019/cert/061-41424-20191024-218af9ec-cf50-4516-9011-228c78eda3d2/InstallMacOSX.dmg)这个内容会作为名为 “InstallMacOSX.dmg” 的磁盘映像下载。在与 El Capitan 兼容的 Mac 上，打开下载的磁盘映像，并运行其中名为“InstallMacOSX.pkg”的安装器。这时会在您的“应用程序”文件夹中安装一个名为“安装 OS X El Capitan”的 App。您将通过这个 App（而不是通过磁盘映像或 .pkg 安装器）创建可引导安装器。

在“终端”中使用“createinstallmedia”命令

1.连接要用于保存可引导安装器的 USB 闪存驱动器或其他宗卷。

2.打开“应用程序”文件夹内“实用工具”文件夹中的“终端”。

3.在“终端”中键入或粘贴以下命令之一。这些命令假设安装器位于您的“应用程序”文件夹中，并且“MyVolume”是您所使用的 USB闪存驱动器或其他宗卷的名称。如果不是这个名称，请将这些命令中的 MyVolume 替换为您的宗卷名称。

sudo /Applications/Install\ macOS\ Monterey.app/Contents/Resources/createinstallmedia --volume /Volumes/MyVolume

Big Sur*：

sudo /Applications/Install\ macOS\ Big\ Sur.app/Contents/Resources/createinstallmedia --volume /Volumes/MyVolume

Catalina*：

sudo /Applications/Install\ macOS\ Catalina.app/Contents/Resources/createinstallmedia --volume /Volumes/MyVolume

Mojave*：

sudo /Applications/Install\ macOS\ Mojave.app/Contents/Resources/createinstallmedia --volume /Volumes/MyVolume

High Sierra*：

sudo /Applications/Install\ macOS\ High\ Sierra.app/Contents/Resources/createinstallmedia --volume /Volumes/MyVolume

El Capitan：

sudo /Applications/Install\ OS\ X\ El\ Capitan.app/Contents/Resources/createinstallmedia --volume /Volumes/MyVolume --applicationpath /Applications/Install\ OS\ X\ El\ Capitan.app

如果您的 Mac 运行的是 macOS Sierra 或更低版本，请使用 --applicationpath 参数和安装器路径，具体方法与在适用于 El Capitan 的命令中完成这个操作的方法类似。

键入命令后：

1.按下 Return 键以输入这个命令。出现提示时，请键入您的管理员密码，然后再次按下 Return 键。在您键入密码时，“终端”不会显示任何字符。

2.出现提示时，请键入 Y 以确认您要抹掉宗卷，然后按下 Return 键。在抹掉宗卷的过程中，“终端”会显示进度。

3.宗卷被抹掉后，您可能会看到一条提醒，提示“终端”要访问可移除宗卷上的文件。点按“好”以允许继续拷贝。

4.当“终端”提示操作已完成时，宗卷的名称将与您下载的安装器名称相同，例如“Install macOS Monterey”。您现在可以退出“终端”并弹出宗卷。

使用可引导安装器

确定您使用的是不是搭载 Apple 芯片的 Mac，然后按照相应的步骤操作：

Apple 芯片

1.将可引导安装器插入已连接到互联网且与您要安装的 macOS 版本兼容的 Mac。

2.将 Mac 开机并继续按住电源按钮，直到看到启动选项窗口，其中会显示可引导宗卷。

3.选择包含可引导安装器的宗卷，然后点按“继续”。

4.macOS 安装器打开后，请按照屏幕上的说明操作。

Intel 处理器

1.将可引导安装器插入已连接到互联网且与您要安装的 macOS 版本兼容的 Mac。

2.将 Mac 开机或重新启动后，立即按住 **Option (Alt) ⌥ ** 键。

3.当您看到显示可引导宗卷的黑屏时，松开 Option 键。

4.选择包含可引导安装器的宗卷。然后点按向上箭头或按下 Return 键。如果您无法从可引导安装器启动，请确保“启动安全性实用工具”中的“外部启动”设置已设为允许从外部介质启动。

5.根据提示选取您的语言。

6.从“实用工具”窗口中选择“安装 macOS”（或“安装 OS X”），然后点按“继续”，并按照屏幕上的说明进行操作。

进一步了解

可引导安装器不会从互联网下载 macOS，但却需要互联网连接才能获取特定于 Mac 机型的固件和其他信息。

要了解 createinstallmedia 命令以及可与它搭配使用的参数，请确保 macOS 安装器位于您的“应用程序”文件夹中，然后在“终端”中输入相应的路径：

/Applications/Install\ macOS\ Monterey.app/Contents/Resources/createinstallmedia
/Applications/Install\ macOS\ Big\ Sur.app/Contents/Resources/createinstallmedia
/Applications/Install\ macOS\ Catalina.app/Contents/Resources/createinstallmedia
/Applications/Install\ macOS\ Mojave.app/Contents/Resources/createinstallmedia
/Applications/Install\ macOS\ High\ Sierra.app/Contents/Resources/createinstallmedia
/Applications/Install\ OS\ X\ El\ Capitan.app/Contents/Resources/createinstallmedia
