---
title: scoop的介绍安装及使用
description: 一款windows上的包管理工具
published: true
date: 2022-07-09T18:28:29.080Z
tags: 应用推荐
editor: markdown
dateCreated: 2022-07-09T18:14:48.600Z
---

# 软件介绍
就是让你只需要一行代码就可以自动安装应用的包管理工具，具体可以看看它的[github主页](https://github.com/ScoopInstaller/Scoop)

# 软件安装
使用powershell安装，两行代码解决
```
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser  # 选是
irm get.scoop.sh | iex
```
> 由于很多软件安装源都是github，为了使用体验，建议安装后使用代理{.is-info}
```
scoop config proxy 127.0.0.1:11223
git config --global http.proxy socks5://127.0.0.1:11223
git config --global https.proxy socks5://127.0.0.1:11223
```

> 使用aria2可以很大程度上提升下载速度{.is-info}
```
scoop install aria2
scoop config aria2-warning-enabled false
```

# 前置知识
所有命令行选项如下
```
Command    Summary
-------    -------
alias      Manage scoop aliases
bucket     Manage Scoop buckets
cache      Show or clear the download cache
cat        Show content of specified manifest. If available, `bat` will be used to pretty-print the JSON.
checkup    Check for potential problems
cleanup    Cleanup apps by removing old versions
config     Get or set configuration values
create     Create a custom app manifest
depends    List dependencies for an app, in the order they'll be installed
download   Download apps in the cache folder and verify hashes
export     Exports installed apps, buckets (and optionally configs) in JSON format
help       Show help for a command
hold       Hold an app to disable updates
home       Opens the app homepage
import     Imports apps, buckets and configs from a Scoopfile in JSON format
info       Display information about an app
install    Install apps
list       List installed apps
prefix     Returns the path to the specified app
reset      Reset an app to resolve conflicts
search     Search available apps
shim       Manipulate Scoop shims
status     Show status and check for new app versions
unhold     Unhold an app to enable updates
uninstall  Uninstall an app
update     Update apps, or Scoop itself
virustotal Look for app's hash or url on virustotal.com
which      Locate a shim/executable (similar to 'which' on Linux)
```
在介绍基本的使用前先说说几个基本概念（了解个大概就行了，我也是一知半解，只要知道大体上是个啥东西就不会影响后面的阅读）
## bucket
每个软件都有自己所属的bucket，而scoop可以有很多bucket，每个bucket管理不同类型的软件
也就是 $\text{app} \in \text{bucket} \in \text{scoop}$
以`main bucket`下的应用 `git` 为例，使用`scoop cat git`能看到 `git`这个应用存储在bucket中的`manifest`如下
```json
{
    "version": "2.37.0.windows.1",
    "description": "Distributed version control system",
    "homepage": "https://gitforwindows.org",
    "license": "GPL-2.0-only",
    "notes": "Set Git Credential Manager Core by running: \"git config --global credential.helper manager-core\"",
    "architecture": {
        "64bit": {
            "url": "https://github.com/git-for-windows/git/releases/download/v2.37.0.windows.1/PortableGit-2.37.0-64-bit.7z.exe#/dl.7z",
            "hash": "96808564283669e0129310c14f8ad6ffb55498d3381420bd22200a62585ab2f4"
        },
        "32bit": {
            "url": "https://github.com/git-for-windows/git/releases/download/v2.37.0.windows.1/PortableGit-2.37.0-32-bit.7z.exe#/dl.7z",
            "hash": "45efb4a2c9c3fd11ca7580b0d8da469474c2a6ce1c48ff8a7512541923f0cbdb"
        }
    },
    "bin": [
        "cmd\\git.exe",
        "cmd\\gitk.exe",
        "cmd\\git-gui.exe",
        "usr\\bin\\tig.exe",
        "git-bash.exe"
    ],
    "shortcuts": [
        [
            "git-bash.exe",
            "Git Bash",
            "--cd-to-home"
        ],
        [
            "cmd\\git-gui.exe",
            "Git GUI"
        ]
    ],
    "env_set": {
        "GIT_INSTALL_ROOT": "$dir"
    },
    "checkver": {
        "github": "https://github.com/git-for-windows/git",
        "regex": "v([\\w.]+)/PortableGit-(?<full>[\\w.]+)-64-bit"
    },
    "autoupdate": {
        "architecture": {
            "64bit": {
                "url": "https://github.com/git-for-windows/git/releases/download/v$version/PortableGit-$matchFull-64-bit.7z.exe#/dl.7z"
            },
            "32bit": {
                "url": "https://github.com/git-for-windows/git/releases/download/v$version/PortableGit-$matchFull-32-bit.7z.exe#/dl.7z"
            }
        },
        "hash": {
            "url": "https://github.com/git-for-windows/git/releases/tag/v$version",
            "regex": "<td>$basename</td>\\s*<td>$sha256</td>"
        }
    }
}
```
可以看到里面记录了软件干啥用的，有什么东西，如何安装，从哪安装，从哪更新，等等信息，scoop通过这些信息就可以自动安装和升级软件。
实际上，每个`bucket`都是github上的一个仓库，存储的就是上面的内容，每次更新时，scoop都会同步所有bucket的数据，来获取最新的应用。比如上述的内容就对应 `https://github.com/ScoopInstaller/Main/blob/master/bucket/git.json`。
## shim
`shim`说白了就是个快捷方式，有的软件有GUI用不到shim，但也有很多软件是需要命令行使用的，比如`git python`等。这个时候就需要添加到环境变量，方便使用。但每安装一个软件都要重新配置环境变量，软件一多，十分不方便管理。因此就需要用到`shim`来统一管理，本质上是个指向对应可执行文件的可执行文件。还是以`git`为例，输入 `scoop shim info git`得到以下内容
```
Name     : git
Path     : C:\Users\20577\Scoop\shims\git.exe
Source   : git
Type     : Application
IsGlobal : False
IsHidden : False
```
当你在命令行输入`git`命令时，本质上就是调用了这个exe文件，而这个exe文件再调用真正的git可执行文件。
所有的`shim`全都位于`~\Scoop\shims\`文件夹下，所以只需把这一个文件夹加入环境变量即可，极其方便。
注意到在这个文件夹下每个`*.exe`文件都有一个对应的`*.shim`文件，这是一个文本文件，我们可以用编辑器打开它，例如`git.shim`的结果如下
```
path = "C:\Users\20577\Scoop\apps\git\current\cmd\git.exe"
```
这不就是git的安装目录吗？看到这其实就已经可以大胆猜测了，实际上所有的`*.exe`除了名字不同，内容都是一样的，都是读取对应的同名 `*.shim`文件得到要启动的文件路径，再启动文件。你大可以试着验证一下。
```
cp 7z.exe git.exe
git --version  # 结果可以正常运行
```
## current
current的存在主要是为了解决更新后的数据问题。用户的配置文件，环境变量等，都会因为更新而失效，而有了current后，所有数据全部指向current文件，再由`current`重新定向到最新的版本。详细见下图：
![图 1](https://s2.loli.net/2022/07/10/QM3wLbiDsYcqxEI.png)  
我们随便打开一个应用对应的文件目录，可以看到如下的内容：
```
cd ~/scoop/apps/osulazer
ls

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----          2022/7/1      8:39                2022.630.0
d-----          2022/7/4     12:53                2022.704.0
d-----          2022/7/9     16:28                2022.709.1
d-r--l          2022/7/9     16:28                current
```
可以看见它一共有三个不同的版本，都是文件夹，还有一个`current`，也是文件夹，但最后有个`l`符号，观察`LastWriteTime`可以发现，实际上`current`是和`2022.709.1`同时被创建的，大胆猜测`current`就是一个指向该文件夹的快捷方式，通过如下的方式可以验证：
```
# 在上面的目录下输入
cd .\current
echo 132 > .\test.txt
cat .\test.txt
cd ..\2022.709.1\
cat .\test.txt
```
官方把这个`current`叫做`Directory Junctions`也叫做 `软链接(soft link)`。他和快捷方式是不同的，你可以创建一个快捷方式看看。
```

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----         2022/6/29     11:40                2.37.0.windows.1
d-r--l         2022/6/29     11:40                current-soft
-a----          2022/7/9     23:59           1579 current-shortcut.lnk
```
你会发现，快捷方式本质上是一个文件，他有文件大小，以`.lnk`结尾，但就算你的windows打开了显示文件扩展名，他也不会显示，很奇怪就。同时，快捷方式不能用cd，如果你在终端中运行这个文件，他会用`explorer`打开它指向的文件夹。

而软链接是一个实实在在的文件夹，ls中有一个专门的`l`符号表示，意思就是他是一个链接（link）。与快捷方式不同，软链接是可以cd的，他被认为是一个存在的文件夹。它可以通过 `cmd`中的命令`mklink /J link target`创建。不加`\J`用的是符号链接，这东西，以后再说吧，现在先`TODO`。
mklink的具体用法如下：
```
MKLINK [[/D] | [/H] | [/J]] Link Target

        /D      创建目录符号链接。默认为文件
                符号链接。
        /H      创建硬链接而非符号链接。
        /J      创建目录联接。
        Link    指定新的符号链接名称。
        Target  指定新链接引用的路径
                (相对或绝对)。
```

# 基础用法
## bucket
```
Usage: scoop bucket add|list|known|rm [<args>]
```
通过`scoop bucket known`可以查看已知的bucket，分别是
```

main
extras
versions
nirsoft
php
nerd-fonts
nonportable
java
games
```
通过 `scoop add <name>` 可以添加这个bucket，推荐必装 `extras, dorado`。其中dorado要从github仓库安装。
```
scoop bucket add extras
scoop bucket add dorado https://github.com/h404bi/dorado 
```
## search
```
Usage: scoop search <query>
Searches for apps that are available to install.
```
顾名思义，就是搜索软件，比如我需要下个`chrome`，就运行
```
scoop search chrome

Name               Version                  Source  Binaries
----               -------                  ------  --------
chromium           103.0.5060.66-r1002911   extras  chrome.exe
dartium            1.24.2                   extras  chrome.exe
googlechrome       103.0.5060.114           extras
ungoogled-chromium 103.0.5060.68-1-r1002911 extras  chrome.exe
chromedriver       103.0.5060.53            main
chromecacheview    2.35                     NirSoft
chromecookiesview  1.70                     NirSoft
chromehistoryview  1.50                     NirSoft
chromepass         1.56                     NirSoft
```
在这个结果里找到你想要的name，比如`googlechrome`，然后运行
```
scoop install googlechrome
```
然后等就是了，install有很多选项，但用得都不多，看看就是了。
```
Options:
  -g, --global              Install the app globally
  -i, --independent         Don't install dependencies automatically
  -k, --no-cache            Don't use the download cache
  -u, --no-update-scoop     Don't update Scoop before installing if it's outdated
  -s, --skip                Skip hash validation (use with caution!)
  -a, --arch <32bit|64bit>  Use the specified architecture, if the app supports it
```
## update
这个没啥好说的，运行
```
scoop update *
```
然后等他更新完就好了。如果你只要升级单特定的某个应用（真的会有这样的场景吗？），星号换成应用名就行了。
## hold/unhold
```
Usage: scoop hold <apps>  # 阻止一个应用的升级行为
```
没啥好说的，应用场景的话，对我来说，一个是不想让pycharm更新，另一个是阻止一些更新比较频繁，但用得很少的软件更新，毕竟他们的旧版新版都差不多。

## cleanup/cache
```
scoop cleanup *  # 清除所有旧版本
scoop cache show  # 显示文件的下载缓存
scoop cache rm *  # 清空缓存文件
```
这里的 * 也都可以替换为应用名，不过用的应该都不多。

# 最后
附上一张我自己的应用列表，使用`scoop list`即可得到
```

Name                   Version              Source           Updated             Info
----                   -------              ------           -------             ----
7zip                   22.00                main             2022-06-23 17:35:43
activitywatch          0.11.0               dorado           2022-07-07 15:08:05
anaconda3              2022.05              extras           2022-05-16 11:44:27
arduino                1.8.19               extras           2022-04-14 18:57:49
aria2                  1.36.0-1             main             2022-04-10 13:34:43
assetstudio            0.16.47              games            2022-07-07 10:38:33
chromedriver           103.0.5060.53        main             2022-06-23 17:35:46
clash-for-windows      0.19.23              dorado           2022-07-04 12:55:32
dark                   3.11.2               main             2022-05-26 19:54:03
dingtalk               6.5.20.6139101       dorado           2022-06-16 17:12:48
ditto                  3.24.214.0           extras           2022-04-10 14:21:08
dngrep                 3.0.84.0             extras           2022-06-29 11:40:32
dotnet-desktop-runtime 6.0.4                <auto-generated> 2022-05-26 19:58:22
draw.io                19.0.3               extras           2022-06-11 18:05:38
espanso                0.7.3                main             2022-06-19 20:55:11
fastcopy               4.1.7                extras           2022-06-29 11:40:38
ffmpeg                 5.0.1                main             2022-04-10 13:17:41
firefox                100.0.2                               2022-07-09 18:17:59
git                    2.37.0.windows.1     main             2022-06-29 11:40:57
googlechrome           103.0.5060.114       extras           2022-07-08 15:24:16
hmcl                   3.5.3                dorado           2022-04-10 21:06:23
hugo-extended          0.101.0              main             2022-06-28 13:09:05
innounp                0.50                 main             2022-04-10 22:43:57
irfanview              4.60                 extras           2022-04-10 22:51:08
locale-emulator        2.5.0.1              extras           2022-07-05 00:12:39
mingw                  8.1.0                main             2022-04-10 13:29:06
mobaxterm              22.1                 extras           2022-07-05 18:42:15
mongodb                5.3.2                main             2022-06-23 17:36:59
mongodb-compass        1.32.2               extras           2022-06-11 18:07:29
mongodb-database-tools 100.5.3              main             2022-06-16 17:13:13
neovim                 0.7.2                main             2022-07-07 15:01:43
neteasemusic           2.9.10.200061        dorado           2022-07-08 15:25:24
nvm-windows            1.1.7                dorado           2022-05-29 22:59:18
obs-studio             27.2.4               extras           2022-04-10 20:07:33
ollydbg                1.10                 extras           2022-05-08 02:42:09
openjdk                17.0.2-8             java             2022-04-10 16:51:47
osulazer               2022.709.1           games            2022-07-09 16:28:42
pandoc                 2.18                 main             2022-05-03 23:27:50
potplayer              220706               extras           2022-07-08 15:25:42
pycharm-professional   2022.1.3-221.5921.27 extras           2022-06-23 17:38:27
qbittorrent-enhanced   4.4.1.10             dorado           2022-04-10 15:33:49
r                      4.2.1                main             2022-06-29 11:42:05
real-esrgan            0.2.5.0              dorado           2022-07-08 18:56:29
renpy                  8.0.0                extras           2022-06-29 11:42:27
rstudio                2022.07.0-548        extras           2022-07-08 15:26:29
rtools                 4.2.5168.5107        main             2022-05-11 01:42:10 Held package
scrcpy                 1.24                 main             2022-05-27 13:25:50
screentogif            2.37                 extras           2022-05-03 15:14:24
snipaste               1.16.2               extras           2022-04-10 21:59:19
spacesniffer           1.3.0.2              extras           2022-07-07 19:24:08
sqlitebrowser          3.12.2               extras           2022-05-23 12:15:07
steampp                2.7.2                dorado           2022-05-03 15:14:36
sumatrapdf             3.4.6                extras           2022-06-11 18:07:33
tableplus              4.9.12.200           extras           2022-07-09 16:35:42
telegram               4.0.2                extras           2022-06-29 11:42:42
tesseract              5.2.0.20220708       main             2022-07-09 16:29:17
tesseract-languages    4.1.0                main             2022-06-17 23:41:44
vcredist2022           14.32.31332.0        extras           2022-07-08 15:27:15
vscode                 1.69.0               extras           2022-07-08 15:30:14
waifu2x-ncnn-vulkan    20220419             main             2022-07-08 19:12:57
win-portacle           1.4                  extras           2022-06-01 18:38:45
x64dbg                 2022-07-04_15-40     extras           2022-07-08 15:31:07
```
使用
```
scoop info <name>  # 查看应用信息
scoop home <name>  # 打开应用官网
```
祝您使用愉快！