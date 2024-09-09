## ArchLinux + KDE 一周使用小结

![Featured Image](https://mozz.ie/assets/8854314f89b9e34d7201c.png)

终于考完了，又来到一个学期末的长草期。没事可干的我在把家里卧室也升级到万兆之后，便蠢蠢欲动想找一个新的什么东西来折腾。

刚好同样也是在半年前，大二上的期末，我安装了基于 GNOME 桌面环境的 Ubuntu 21 进行了「Linux 30天挑战」——我似乎忘了向大家报告这个挑战最后的结果：很遗憾，在坚持了大半个月之后失败了。

而究其失败的根本原因是GNOME的分数缩放有一系列的 BUG，包括但不限于 flameshot 黑屏/崩溃/选取不全、electron 应用全屏会黑屏（Jellyfin Media Player）、部分 QT 应用缩放失效（WPS、网易云音乐）以及 UI 错位等（欧路词典）

这一系列小毛病让我的桌面体验虽然能用，但是极度不爽；更别说一更新就爆炸的 CSGO 了……总之，我没有找到解决这些问题的好办法，最后还是回归了 Windows。

但是这只是一时的妥协，而我依然希望继续使用 Linux 桌面。所以我想，KDE 会不会是一个新的出路？抱着这个念头，我下载了 ArchLinux ISO。因为在我看来，中文生态最好的两个发行版就是 Ubuntu 和 Arch 了；而我这种懒癌晚期是极度讨厌手动打包或者安装应用的。所以第二次 Linux 挑战，在 Arch 上堂堂开幕！

## 安装 & 基本配置

安装方面，没什么好说的，就是跟着 Wiki 的指引下载并且配置自己需要的软件包就好了，我也不是第一次装 Arch 了。

### rEFInd引导配置

```auto
# 只自动搜索外置设备和光驱的 EFI 项目
scanfor external,optical,manual

# 手动配置 Windows 和 Linux 的 entry
menuentry "Arch Linux" { 
  icon /EFI/refind/themes/rEFInd-minimal/icons/os_arch.png
  volume  "Arch Linux" 
  loader  /boot/vmlinuz-linux 
  initrd  /boot/initramfs-linux.img 
  options "root=PARTUUID=c2ae9da6-29ce-c24b-bbbb-b3e33340c681 rw add_efi_memmap" 
  submenuentry "Boot using fallback initramfs" { 
    initrd /boot/initramfs-linux-fallback.img 
  } 
  submenuentry "Boot to terminal" { 
    add_options "systemd.unit=multi-user.target" 
  }
}

menuentry "Windows" { 
   icon /EFI/refind/themes/rEFInd-minimal/icons/os_win.png 
   loader \EFI\Microsoft\Boot\bootmgfw.efi 
}
```

### 挂载目录

得益于新内核里的 ntfs3 驱动，我可以挂载一些 Windows 目录来让 Linux 上的 Wine 微信和 Steam 共享。这样前者可以同步在两个系统下的聊天记录，后者可以用 Proton 玩游戏。本来是用 fstab 挂载的，但是在过了一天之后 fstab 就没办法挂载上了，直接让系统启动不了，我感觉这非常的灵车……于是还是自己写了个 *内裤d* 服务来挂载

```bash
mkdir -p /run/media/archeb/Portable 
mkdir -p /run/media/archeb/Software 
mkdir -p /run/media/archeb/Windows 
mount -t ntfs3 /dev/disk/by-label/Portable /run/media/archeb/Portable -o umask=000
mount -t ntfs3 /dev/disk/by-label/Software /run/media/archeb/Software -o umask=000
mount -t ntfs3 /dev/disk/by-label/Windows /run/media/archeb/Windows -o umask=133
echo Partition Mounted
```

## 分数缩放

令人惊讶的是，这没有什么需要配置的。我就单纯的安装了 nVIDIA 驱动然后在 KDE 设置里面选择 125% 缩放就好了，没有任何应用程序出现任何 BUG……

## 美化

KDE的美化潜力实在是太大了，比GNOME和Windows都要强太多，我花了挺多时间来调教的，最终成果如封面图所示。使用主题是 [Amethyst](https://store.kde.org/p/1799652/)，下面是他的效果图，我自己稍微进行了一些修改。

![](https://mozz.ie/assets/screenshot-20220519-221935-20220628142153728.png.webp)

### 为无边框/自绘窗口添加阴影

有许多 Electron 和 QT App 会选择自绘窗口边框，但是这样会导致它在 KWin 窗口管理器下失去阴影。我们可以利用「配置特殊窗口设置」和「特定窗口优先规则」来给他们加上阴影，效果如下

![img](https://mozz.ie/assets/eajIYX4-20220628142459891-6400744-6400746.png.webp)

具体方法是在「外观」-「窗口装饰元素」里面找到正在使用的主题并且点击设置，然后在「特定窗口优先规则」中指定该窗口，设置「隐藏窗口标题栏」并确定。

![image-20220627020314316](https://mozz.ie/assets/0776430bfdc23418ff715.png.webp)

再在需要阴影的窗口上按 Alt + F3 呼出 KWin 菜单，选择「更多操作」-「配置特殊窗口设置」，设置窗口为强制有标题栏和边框。

![image-20220627020536330](https://mozz.ie/assets/92f5c95b2e4c07ad901f9.png.webp)

然后你的窗口就有了阴影，好看多了！

## 软件

### 输入法

fctix5 真好用，主题是 [FluentDark](https://github.com/Reverier-Xu/FluentDark-fcitx5)

### Wine 微信

感谢 AUR 的维护者大佬们，他们制作的微信功能和 Windows 的几乎一致：聊天、图片预览、文件发送保存、剪贴板等功能都能完美正常使用。虽然小程序和视频播放是意料之中的不能使用，但是 Windows 微信的小程序其实也是几乎残废，很多应用没适配，性能也很差，所以就不奢求 Linux 上也能用了。

在选择用户数据目录上出了一点小问题：整个选择都是失效的，会最终 fallback 到 Wine 环境的桌面。于是我就在 Wine 桌面上创建了一个符号链接到挂载的 NTFS 驱动器的目录里。目前来看数据一切正常，能够在 Windows 和 Linux 的微信之间正常同步

### Microsoft To-Do

日常我比较依赖这个软件来规划记录我的行程，所以它也是必需品。

我下载了 WebSlice 这个 Plasma 小组件，并且设置 URL 为 [https://to-do.live.com/tasks/inbox](https://to-do.live.com/tasks/inbox)，然后通过自定义 JS 功能来重写一下它的外观以适应我的主题。

```CSS
var styles = `:root {
    --bg-primary: transparent;
    --bg-secondary: transparent;
    --bg-tertiary: #202644;
    --bg-hover: rgba(29,53,105,0.3);
    --bg-hover-secondary: rgba(29,53,105,0.3);
    --bg-hover-tertiary: rgba(29,53,105,0.3);
    --bg-active: transparent;
    --bg-active-secondary: transparent;
    --bg-active-tertiary: transparent;
    --bg-separator: transparent;
    --bg-border: transparent;
    --bg-scrollbar: #606278;
    --bg-brand: #2564cf;
    --bg-brand-hover: #215aba;
    --bg-brand-hover-secondary: #8c93a5;
    --bg-brand-secondary: #ecf6fd;
    --bg-brand-disable: #EDEBE9;
    --font-color-primary: #fefefe;
    --font-color-secondary: #999999;
    --font-color-tertiary: #bbbbbb;
    --font-color-warning: #a80000;
    --font-color-brand: #3DB0E3;
    --font-color-disable: #A19F9D;
    --bg-ms-planner: #31752f;
    --bg-ms-one-note: #7719aa;
    --bg-ms-excel: #107c41;
    --bg-ms-word: #185abd;
    --bg-ms-power-point: #c43e1c;
    --bg-ms-teams: #6355a8;
    --list-theme-blue-primary-0: #86aeff;
    --list-theme-blue-primary-10: #86aeff;
    --list-theme-blue-primary-20: #86aeff;
    --list-theme-blue-contrast-highlight-quick-fix: #86aeff;
    --list-theme-blue-top: #86aeff;
    --list-theme-blue-bottom: #86aeff;
    --list-theme-blue-top-minimized: #86aeff;
    --list-theme-blue-bottom-minimized: #86aeff;
}
.addToTop .baseAdd.addTask .baseAdd-input.taskCreation .textPresent, .addToTop .baseAdd.addTask .baseAdd-input.taskCreation.expanded{
box-shadow:none!important;
}
#O365ShellHeader{
display:none;
}
span,a,input{
font-family:sans!important
}
.rightColumn{
background:#202644;
}
`
var styleSheet = document.createElement("style")
styleSheet.innerText = styles
document.head.appendChild(styleSheet)
```

最后把他拖到任务栏的右下角，有需要的时候就可以呼出来看日程啦

![image-20220627015820280](https://mozz.ie/assets/25c934e5c920f5fb74cfb.png.webp)

### 游戏

CSGO 之前在 Ubuntu 上爆炸的原因这次被我找出来了，似乎是新UI的一些视频没办法被正确播放，导致进入游戏之后黑屏一下然后就崩溃了。只需要进入游戏目录然后把 csgo/panorama/videos 目录改个名让他读不到视频，然后再在启动参数里加上 -novid 便可正常启动游戏。

令我惊喜的是在 Windows 下面常年运行在170帧左右的 CSGO（中特效），在 Linux 上竟然可以随便跑到230帧，而且还是高特效。画面丝滑到难以置信，这可是足足 35% 的性能提升……以后 CSGO 就都在 Linux 下玩吧。

其他游戏的话，EAC反作弊官方已经支持 Linux 了，Proton的兼容性也越来越好，听说 Apex 也能玩了。不过其他游戏我玩的不多，就不是很在意这个。

### Android 模拟器

因为有在电脑上抄明日方舟作业的需求，而且我没有其他安卓机器无法 scrcpy，所以 Android 模拟器也是很有必要的。Linux 上的安卓模拟器方案主要就是 Anbox系（比如容器化了的版本reDroid）和虚拟机。

经过多次尝试，我都没办法让 virtualbox、qemu-android-x86 和 reDroid 调用 RTX 2080 的显卡加速，但是 VMWare 却可以。总之在给 Android-X86 打了 vmware 的驱动之后，我在上面流畅地运行了明日方舟。

## 小结

感谢 AUR 和 V 社，让 Linux 生态如此丰富，我现在认为 Arch 已经完全可以成为我日常使用的系统了。总之希望系统不要滚炸然后一直用下去吧！