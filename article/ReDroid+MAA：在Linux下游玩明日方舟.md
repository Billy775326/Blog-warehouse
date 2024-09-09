[![Featured image of post ReDroid+MAA：在Linux下游玩明日方舟](https://lbqaq.top/p/redroid-maa/103123878.webp)](https://lbqaq.top/p/redroid-maa/)

[Linux](https://lbqaq.top/categories/linux/)

## [ReDroid+MAA：在Linux下游玩明日方舟](https://lbqaq.top/p/redroid-maa/)

### 介绍了ReDroid以及MAA安装配置方法，实现在Linux环境下优雅的玩方舟

最近打算将ArchLinux作为主力系统挑战一下，由于平时有用电脑挂明日方舟的习惯，于是打算安装模拟器在Linux上。相较于Windows下各种xx模拟器百花齐放，Linux下就惨不忍睹了。网上搜索到的知名模拟器中，Anbox已经被作者弃坑了，Android Studio模拟器则是面向开发者，想要使用还要安装Android Studio开发环境。

![be like](https://lbqaq.top/p/redroid-maa/IMAGE/FUrYLxA.jpg)

在搜索中，我发现ReDroid可以满足我的需求。作为一个“云手机”的方案，自然是可以部署在本地，摸索明白了部署到服务器上也挺不错(雾。

具体的安装的教程我参考的是Ivon大佬的[这篇文章](https://ivonblog.com/posts/redroid-android-docker/)，在此基础上作了一些小小的修改。

## [#](#redroid%e9%85%8d%e7%bd%ae) ReDroid配置

### [#](#%e5%ae%89%e8%a3%85%e7%8e%af%e5%a2%83) 安装环境

ReDroid需要Linux内核启用binderfs模块，Arch Liunx默认的linux内核并不包含此模块，需要安装linux-zen模块。

这里我强烈建议安装官方的[安装指南](https://github.com/remote-android/redroid-doc/blob/master/deploy/README.md)来，我一开始按照Ivon大佬的教程尝试在默认的Linux内核里启用该模块，结果ReDroid一直启动不了。最后更换内核后就成功运行起来了QAQ。

接着安装docker和docker-compose，用来运行ReDroid的容器

<table class="lntable"><tbody><tr><td class="lntd"><pre tabindex="0" class="chroma"><code><span class="lnt">1
</span></code></pre></td><td class="lntd"><pre tabindex="0" class="chroma"><code class="language-sh" data-lang="sh"><span class="line"><span class="cl">sudo pacman -S docker docker-compose
</span></span></code></pre></td></tr></tbody></table>

还要安装ADB和Scrcpy，用于连接到安卓虚拟机

<table class="lntable"><tbody><tr><td class="lntd"><pre tabindex="0" class="chroma"><code><span class="lnt">1
</span><span class="lnt">2
</span></code></pre></td><td class="lntd"><pre tabindex="0" class="chroma"><code class="language-sh" data-lang="sh"><span class="line"><span class="cl">sudo pacman -S android-tools
</span></span><span class="line"><span class="cl">sudo pacman -S scrcpy
</span></span></code></pre></td></tr></tbody></table>

### [#](#%e5%8f%af%e9%80%89%e9%85%8d%e7%bd%aearm%e8%bd%ac%e8%af%91%e5%99%a8) \[可选\]配置ARM转译器

大部分安卓应用都只有ARM版，这也意味着无法在X86的电脑上运行，这时就需要安装ARM转译器了。

然而，明日方舟居然支持X86，我们可以通过压缩工具解压方舟的安装包，可以看到`lib`下有`x86`目录。这也意味着如果只是用来玩方舟的话，完全不需要配置ARM转译。

同时，根据大佬的教程配置好转译器后，方舟反而会闪退，所以完全不用做此步😥。如果需要安装arm应用，可以看上面大佬的教程来做。

我后面可能会考虑使用libhoudini来作为ReDroid的转译器。

### [#](#%e5%90%af%e5%8a%a8%e9%95%9c%e5%83%8f) 启动镜像

我这里就使用不包含ARM转译器的ReDroid继续了。通过下面的命令即可部署ReDroid：

<table class="lntable"><tbody><tr><td class="lntd"><pre tabindex="0" class="chroma"><code><span class="lnt">1
</span><span class="lnt">2
</span><span class="lnt">3
</span><span class="lnt">4
</span><span class="lnt">5
</span></code></pre></td><td class="lntd"><pre tabindex="0" class="chroma"><code class="language-sh" data-lang="sh"><span class="line"><span class="cl">sudo docker run -itd --privileged <span class="se">\
</span></span></span><span class="line"><span class="cl"><span class="se"></span>    -v /home/luobo/redroid:/data <span class="se">\
</span></span></span><span class="line"><span class="cl"><span class="se"></span>    -p 5555:5555 <span class="se">\
</span></span></span><span class="line"><span class="cl"><span class="se"></span>    --name redroid11 <span class="se">\
</span></span></span><span class="line"><span class="cl"><span class="se"></span>    redroid/redroid:11.0.0-latest
</span></span></code></pre></td></tr></tbody></table>

然后使用adb连接，并执行Scrcpy，显示手机画面：

<table class="lntable"><tbody><tr><td class="lntd"><pre tabindex="0" class="chroma"><code><span class="lnt">1
</span><span class="lnt">2
</span></code></pre></td><td class="lntd"><pre tabindex="0" class="chroma"><code class="language-sh" data-lang="sh"><span class="line"><span class="cl">adb connect localhost:5555
</span></span><span class="line"><span class="cl">scrcpy -s localhost:5555
</span></span></code></pre></td></tr></tbody></table>

对于Google服务框架，我并不是刚需，就不安装了。如果需要安装，可以查看之前提到的教程。

Scrcpy2.0版本，已经自带了声音模块，无需再安装额外的软件。

如果遇到报错`Failed to initialize audio/opus, error 0xfffffffe`

只需要手动指定其他的音频解析器即可，如下面所示：

<table class="lntable"><tbody><tr><td class="lntd"><pre tabindex="0" class="chroma"><code><span class="lnt">1
</span></code></pre></td><td class="lntd"><pre tabindex="0" class="chroma"><code class="language-sh" data-lang="sh"><span class="line"><span class="cl">scrcpy -s localhost:5555 --audio-codec<span class="o">=</span>aac
</span></span></code></pre></td></tr></tbody></table>

如果不需要声音，只需要加上`--no-audio`，如下面所示：

<table class="lntable"><tbody><tr><td class="lntd"><pre tabindex="0" class="chroma"><code><span class="lnt">1
</span></code></pre></td><td class="lntd"><pre tabindex="0" class="chroma"><code class="language-sh" data-lang="sh"><span class="line"><span class="cl">scrcpy -s localhost:5555 --no-audio
</span></span></code></pre></td></tr></tbody></table>

![运行截图](https://lbqaq.top/p/redroid-maa/IMAGE/image-20230617164434447.png)

### [#](#%e5%ae%89%e8%a3%85%e5%ba%94%e7%94%a8%e5%8f%8a%e5%bc%80%e5%85%b3%e6%9c%ba) 安装应用及开关机

向ReDroid安装应用，只需要使用adb即可

<table class="lntable"><tbody><tr><td class="lntd"><pre tabindex="0" class="chroma"><code><span class="lnt">1
</span></code></pre></td><td class="lntd"><pre tabindex="0" class="chroma"><code class="language-sh" data-lang="sh"><span class="line"><span class="cl">adb -s localhost:5555 install <span class="s2">"/mnt/data/Downloads/mrfz_2.0.11_20230605_102450_87eb5.apk"</span>
</span></span></code></pre></td></tr></tbody></table>

开关机也很简单，我们只需对docker容器进行操作。

<table class="lntable"><tbody><tr><td class="lntd"><pre tabindex="0" class="chroma"><code><span class="lnt">1
</span><span class="lnt">2
</span></code></pre></td><td class="lntd"><pre tabindex="0" class="chroma"><code class="language-sh" data-lang="sh"><span class="line"><span class="cl">sudo docker stop redroid11  <span class="c1">#关机</span>
</span></span><span class="line"><span class="cl">sudo docker start redroid11 <span class="c1">#开机</span>
</span></span></code></pre></td></tr></tbody></table>

## [#](#maa%e9%85%8d%e7%bd%ae) MAA配置

MAA作为自动化的挂机长草神器，肯定是要安排上的。

### [#](#%e5%ae%89%e8%a3%85maa) 安装MAA

Arch用户直接aur安装即可

<table class="lntable"><tbody><tr><td class="lntd"><pre tabindex="0" class="chroma"><code><span class="lnt">1
</span></code></pre></td><td class="lntd"><pre tabindex="0" class="chroma"><code class="language-sh" data-lang="sh"><span class="line"><span class="cl">yay -S maa-assistant-arknights
</span></span></code></pre></td></tr></tbody></table>

我使用yay安装时，会安装许多用不到的库，而且会因为依赖错误而无法安装。故还是去[官网](https://maa.plus/)下载压缩包解压使用。

### [#](#%e9%85%8d%e7%bd%aemaa) 配置MAA

我们需要修改`./MAA-v{版本号}-linux-{架构}/Python/` 目录下的 `sample.py` 文件

首先需要配置adb连接，由于我们之前已经安装过adb了，这里直接替换地址即可：

<table class="lntable"><tbody><tr><td class="lntd"><pre tabindex="0" class="chroma"><code><span class="lnt">1
</span></code></pre></td><td class="lntd"><pre tabindex="0" class="chroma"><code class="language-python" data-lang="python"><span class="line"><span class="cl"><span class="k">if</span> <span class="n">asst</span><span class="o">.</span><span class="n">connect</span><span class="p">(</span><span class="s1">'adb'</span><span class="p">,</span> <span class="s1">'127.0.0.1:5555'</span><span class="p">):</span>
</span></span></code></pre></td></tr></tbody></table>

我们可以执行`python sample.py`，如果输出`连接成功`则配置成功

之后我们需要配置MAA的任务，这里可以参考[官方的文档](https://maa.plus/docs/3.1-%E9%9B%86%E6%88%90%E6%96%87%E6%A1%A3.html)进行修改

这里附上我的配置

<table class="lntable"><tbody><tr><td class="lntd"><pre tabindex="0" class="chroma"><code><span class="lnt"> 1
</span><span class="lnt"> 2
</span><span class="lnt"> 3
</span><span class="lnt"> 4
</span><span class="lnt"> 5
</span><span class="lnt"> 6
</span><span class="lnt"> 7
</span><span class="lnt"> 8
</span><span class="lnt"> 9
</span><span class="lnt">10
</span><span class="lnt">11
</span><span class="lnt">12
</span><span class="lnt">13
</span><span class="lnt">14
</span><span class="lnt">15
</span><span class="lnt">16
</span><span class="lnt">17
</span><span class="lnt">18
</span><span class="lnt">19
</span><span class="lnt">20
</span><span class="lnt">21
</span><span class="lnt">22
</span><span class="lnt">23
</span><span class="lnt">24
</span><span class="lnt">25
</span><span class="lnt">26
</span><span class="lnt">27
</span><span class="lnt">28
</span></code></pre></td><td class="lntd"><pre tabindex="0" class="chroma"><code class="language-python" data-lang="python"><span class="line"><span class="cl">    <span class="n">asst</span><span class="o">.</span><span class="n">append_task</span><span class="p">(</span><span class="s1">'StartUp'</span><span class="p">)</span>
</span></span><span class="line"><span class="cl">    <span class="n">asst</span><span class="o">.</span><span class="n">append_task</span><span class="p">(</span><span class="s1">'Fight'</span><span class="p">,</span> <span class="p">{</span>
</span></span><span class="line"><span class="cl">        <span class="s1">'stage'</span><span class="p">:</span> <span class="s1">''</span><span class="p">,</span>                        <span class="c1"># 关卡名，可选，默认为空，识别当前/上次的关卡。</span>
</span></span><span class="line"><span class="cl">        <span class="s1">'report_to_penguin'</span><span class="p">:</span> <span class="kc">True</span><span class="p">,</span>          <span class="c1"># 是否汇报企鹅数据</span>
</span></span><span class="line"><span class="cl">        <span class="s2">"expiring_medicine"</span><span class="p">:</span> <span class="mi">999</span><span class="p">,</span>           <span class="c1"># 最大使用 48 小时内过期理智药数量，可选，默认 0</span>
</span></span><span class="line"><span class="cl">        <span class="c1"># 'penguin_id': '1234567'</span>
</span></span><span class="line"><span class="cl">    <span class="p">})</span>
</span></span><span class="line"><span class="cl">    <span class="n">asst</span><span class="o">.</span><span class="n">append_task</span><span class="p">(</span><span class="s1">'Recruit'</span><span class="p">,</span> <span class="p">{</span>
</span></span><span class="line"><span class="cl">        <span class="s1">'enable'</span><span class="p">:</span> <span class="kc">True</span><span class="p">,</span>                     <span class="c1"># 是否启用本任务，可选，默认为 true</span>
</span></span><span class="line"><span class="cl">        <span class="s1">'select'</span><span class="p">:</span> <span class="p">[</span><span class="mi">4</span><span class="p">,</span> <span class="mi">5</span><span class="p">],</span>                   <span class="c1"># 会去点击标签的 Tag 等级，必选</span>
</span></span><span class="line"><span class="cl">        <span class="s1">'confirm'</span><span class="p">:</span> <span class="p">[</span><span class="mi">3</span><span class="p">,</span> <span class="mi">4</span><span class="p">],</span>
</span></span><span class="line"><span class="cl">        <span class="s1">'times'</span><span class="p">:</span> <span class="mi">4</span>
</span></span><span class="line"><span class="cl">    <span class="p">})</span>
</span></span><span class="line"><span class="cl">    <span class="n">asst</span><span class="o">.</span><span class="n">append_task</span><span class="p">(</span><span class="s1">'Infrast'</span><span class="p">,</span> <span class="p">{</span>
</span></span><span class="line"><span class="cl">        <span class="s1">'facility'</span><span class="p">:</span> <span class="p">[</span>
</span></span><span class="line"><span class="cl">            <span class="s2">"Mfg"</span><span class="p">,</span> <span class="s2">"Trade"</span><span class="p">,</span> <span class="s2">"Control"</span><span class="p">,</span> <span class="s2">"Power"</span><span class="p">,</span> <span class="s2">"Reception"</span><span class="p">,</span> <span class="s2">"Office"</span><span class="p">,</span> <span class="s2">"Dorm"</span>
</span></span><span class="line"><span class="cl">        <span class="p">],</span>
</span></span><span class="line"><span class="cl">        <span class="s1">'drones'</span><span class="p">:</span> <span class="s2">"Money"</span><span class="p">,</span>                  <span class="c1"># 无人机用途 "_NotUse"、"Money"、"SyntheticJade"、"CombatRecord"、"PureGold"、"OriginStone"、"Chip"</span>
</span></span><span class="line"><span class="cl">        <span class="s1">'replenish'</span><span class="p">:</span> <span class="kc">True</span><span class="p">,</span>                  <span class="c1"># 贸易站“源石碎片”是否自动补货,</span>
</span></span><span class="line"><span class="cl">        <span class="s1">'dorm_trust_enabled'</span><span class="p">:</span> <span class="kc">True</span>          <span class="c1"># 是否将宿舍剩余位置填入信赖未满干员</span>
</span></span><span class="line"><span class="cl">    <span class="p">})</span>
</span></span><span class="line"><span class="cl">    <span class="n">asst</span><span class="o">.</span><span class="n">append_task</span><span class="p">(</span><span class="s1">'Visit'</span><span class="p">)</span>
</span></span><span class="line"><span class="cl">    <span class="n">asst</span><span class="o">.</span><span class="n">append_task</span><span class="p">(</span><span class="s1">'Mall'</span><span class="p">,</span> <span class="p">{</span>
</span></span><span class="line"><span class="cl">        <span class="s1">'shopping'</span><span class="p">:</span> <span class="kc">True</span><span class="p">,</span>
</span></span><span class="line"><span class="cl">        <span class="s1">'buy_first'</span><span class="p">:</span> <span class="p">[</span><span class="s1">'招聘许可'</span><span class="p">,</span> <span class="s1">'龙门币'</span><span class="p">],</span>
</span></span><span class="line"><span class="cl">        <span class="s1">'blacklist'</span><span class="p">:</span> <span class="p">[</span><span class="s1">'家具'</span><span class="p">,</span> <span class="s1">'碳'</span><span class="p">],</span>
</span></span><span class="line"><span class="cl">    <span class="p">})</span>
</span></span><span class="line"><span class="cl">    <span class="n">asst</span><span class="o">.</span><span class="n">append_task</span><span class="p">(</span><span class="s1">'Award'</span><span class="p">)</span>
</span></span></code></pre></td></tr></tbody></table>

配置好后只需要一行命令就可以执行脚本了

至此，我们终于可以愉快的挂机了，再也不用担心体力被浪费了（雾

由于ReDroid是基于Dokcer的，这也意味着可以将其配置到服务器上，就可以实现24小时自动挂机了。不过，要想流畅的运行方舟，最好还是有4G以上的内存，目前我的服务器还没有这么大的内存，之后有机会再尝试吧。