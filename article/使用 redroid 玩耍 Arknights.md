1.  [使用 redroid 玩耍 Arknights](https://blog.sww.moe/post/redroid-arknights/#%E4%BD%BF%E7%94%A8-redroid-%E7%8E%A9%E8%80%8D-arknights)
    1.  [准备工作](https://blog.sww.moe/post/redroid-arknights/#%E5%87%86%E5%A4%87%E5%B7%A5%E4%BD%9C)
    2.  [启动模拟器容器](https://blog.sww.moe/post/redroid-arknights/#%E5%90%AF%E5%8A%A8%E6%A8%A1%E6%8B%9F%E5%99%A8%E5%AE%B9%E5%99%A8)
        1.  [安装国服](https://blog.sww.moe/post/redroid-arknights/#%E5%AE%89%E8%A3%85%E5%9B%BD%E6%9C%8D)
        2.  [安装日服](https://blog.sww.moe/post/redroid-arknights/#%E5%AE%89%E8%A3%85%E6%97%A5%E6%9C%8D)
    3.  [配置 MAA](https://blog.sww.moe/post/redroid-arknights/#%E9%85%8D%E7%BD%AE-maa)
    4.  [あとがき](https://blog.sww.moe/post/redroid-arknights/#%E3%81%82%E3%81%A8%E3%81%8C%E3%81%8D)
    5.  [评论](https://blog.sww.moe/post/redroid-arknights/#%E8%AF%84%E8%AE%BA)

## 使用 redroid 玩耍 Arknights

至今为止我们已经尝试过 anbox, waydroid 等优秀的安卓模拟器了，但是实际上我还是感觉不太方便。之后我尝试了一下 redroid，并觉得这才是我希望的解决方案。

本文将会介绍在 Arch Linux 上使用 redroid 运行明日方舟国服、日服的教程，以及配置 MAA 进行自动化玩游戏的相关内容。

## 准备工作

首先你需要切换到 `linux-zen` 内核，或者使用其他带有 `binderfs` 和 `memfd` 模块的内核。

其次你需要安装 docker，如果有必要可以配置使用 [sjtu 的镜像](https://mirror.sjtu.edu.cn/docs/docker-registry)。

## 启动模拟器容器

首先随便找个地方创建一个文件夹（例如 `~/vms/redroid11`），当作安卓模拟器的 `/data` 目录。

接下来直接使用 docker 创建并运行一个安卓模拟器。

```bash
docker run -d --privileged \
  -v ~/vms/redroid11:/data \
  -p 5555:5555 \
  --name redroid11 \
  redroid/redroid:11.0.0-latest \
  androidboot.redroid_width=1280 \
  androidboot.redroid_height=720 \
  androidboot.redroid_gpu_mode=host \
  androidboot.use_memfd=1
```

容器启动之后就会直接开启模拟器，可以使用 `adb connect localhost:5555` 连接到模拟器。连接完成之后直接使用 `scrcpy -s localhost:5555` 就可以看到模拟器的界面。声音可能由于编解码器的问题传不过来，如果想要打开声音需要给 `scrcpy` 添加 `--audio-codec=raw` 参数。如果一切正常就可以看到如下界面。

![screenshot](https://blog.sww.moe/assets/image_2024-01-08_16-01-21.png)

之后如果要关闭模拟器，可以使用 `docker stop redroid11`；如果想要再次开启模拟器，就使用 `docker start redroid11`。

如果你不小心配置崩了，可以用 `docker rm redroid11` 删除容器，并使用 `sudo rm -rf ~/vms/redroid11` 清空整个目录，然后从头开始重新来一遍。

### 安装国服

直接去 [ak.hypergryph.com](https://ak.hypergryph.com/) 上下载 apk，然后使用 `adb install arknights-xxx.apk` 安装。

安装完成之后直接启动即可，不会有任何运行的问题。

### 安装日服

去 [apkpure](https://apkpure.com/) 或者 [uptodown](https://com-yostarjp-arknights.en.uptodown.com/android) 等网站下载 apk，然后使用 `adb install xxx.apk` 安装到模拟器中。

这还没完，你还需要一个 `.obb` 文件才可以正常运行，不知道为什么上面的网站都没抓这个 `.obb`，所以你需要自己找到这个 `.obb` 文件然后放到合适的位置。一般来说，你可以从一个已经通过 Google Play 安装完成的设备上拷贝下来，然后复制到 redroid 上面。

这个 `.obb` 文件位于 `/storage/emulated/0/Android/obb/com.YoStarJP.Arknights/main.2000094.com.YoStarJP.Arknights.obb`，大概有 1.7G 大小。

放好之后直接启动就可以了，实际上并不需要使用代理或者 Play Service 就可以运行，帮我们省去了很多麻烦事。就是跑起来有点卡

## 配置 MAA

安装 `maa-assistant-arknights-bin` (AUR)，安装完成之后可以使用 `maa --help` 查看帮助。

根据 MAA 惨不忍睹的[文档](https://maa.plus/docs/%E7%94%A8%E6%88%B7%E6%89%8B%E5%86%8C/CLI%E4%BD%BF%E7%94%A8%E6%8C%87%E5%8D%97.html)，我们进行如下的配置工作。

编辑 `~/.config/maa/asst.toml`，写入如下内容。

```toml
[connection]
type = "ADB"
adb_path = "adb"
device = "localhost:5556"
config = "CompatPOSIXShell"

[instance_options]
touch_mode = "MAATouch"
deployment_with_pause = false
adb_lite_enabled = false
kill_adb_on_exit = false
```

注意 device 一行写入 `adb devices` 连接到的设备名称，一般就是你 connect 的时候的地址。**如果你可能同时跑多个模拟器，那么这里必须要留空（注释掉）。**

之后创建目录 `~/.config/maa/tasks`，里面就可以开始写你的各个任务了。

本人的配置文件如下（具体每个字段可以参考写的稀烂的[集成文档](https://maa.plus/docs/%E5%8D%8F%E8%AE%AE%E6%96%87%E6%A1%A3/%E9%9B%86%E6%88%90%E6%96%87%E6%A1%A3.html)）：

```yaml
# 国服每日任务
# cn_daily.yaml
tasks:
  - type: StartUp
    params:
      client_type: Official
      start_game_enabled: true
  - type: Recruit
    params:
      refresh: true
      select: [4, 5, 6]
      confirm: [4, 5]
      times: 4
  - type: Infrast
    params:
      mode: 0
      facility:
        - Mfg
        - Trade
        - Power
        - Control
        - Reception
        - Office
        - Dorm
      dorm_notstationed_enabled: true
      dorm_trust_enabled: true
      drones: CombatRecord
  - type: Mall
    params:
      shopping: true
      buy_first: ["招聘许可"]
      blacklist: ["加急许可", "家具零件", "碳"]
  - type: Award
```

```yaml
# 清体力
# cn_fight.yaml
tasks:
  - type: StartUp
    params:
      client_type: Official
      start_game_enabled: true
  - type: Fight
    params:
      stage:
        default: ""
        description: 哪一关
      medicine:
        default: 0
        description: 吃多少药
      expiring_medicine: 0
      stone:
        default: 0
        description: 吃多少石头
      report_to_penguin: true
      server: CN
      client_type: Official
```

```yaml
# 肉鸽刷到死
# cn_roguelike.yaml
tasks:
  - type: StartUp
    params:
      client_type: Official
      start_game_enabled: true
  - type: Roguelike
    params:
      theme: Mizuki
      core_char: 焰影苇草
      use_support: false
      refresh_trader_with_dice: true
```

日服的相关配置和国服区别不大，主要是 `client_type` 设置成 `YoStarJP`，以及汇报企鹅物流的 `server` 写成 `JP`。

之后使用 `maa run <task_name>` 运行即可，例如 `maa run cn_daily`。

如果上面没有填写 device 参数的话，就在前面加一个 `-a <device>` 参数指定 `adb` 设备，例如 `maa run -a localhost:5555 cn_daily`。

全部配置完成之后你就可以开始挂机了，甚至可以同时挂两个号。

## あとがき

因为图像是通过视频流传过来的，所以画质可能不会很清楚，这一点上面大概比不过 Waydroid 之类的模拟器，貌似也没有什么很好的解决办法。

如果想要调整分辨率可以使用 `adb shell wm size 1920x1080`。

如果想要调整 DPI 可以使用 `adb shell wm density 240`。

如果你想尝试装上 Google Play，你可以尝试参考这篇 [ReDroid 教學：用 Docker 跑 Android 系統，在 x86 電腦玩 ARM 手機遊戲](https://ivonblog.com/posts/redroid-android-docker/)。经过我的尝试，确实可以装上 Google Play，但是每次安装/更新 arm 架构的 app 都会失败，然后到头来还不如 [Aurora Store](https://auroraoss.com/) 好使。顺便一提 Aurora Store 4.3.5 版本在 redroid 里面会直接崩溃，我也不知道为什么，但是回退到 4.2.5 版本就可以正常运行了。

顺便一提，如果跑日服比较卡顿的话，可以将 `/opt/maa/resource/config.json` 复制到 `/opt/maa/resource/global/YoStarJP/resource/config.json`，然后编辑其中的 `taskDelay` 属性，让 MAA 操作的慢一点，这样可以减少 MAA 识别失败的几率。

Good Luck & Have Fun!

## 评论

少女祈祷中...