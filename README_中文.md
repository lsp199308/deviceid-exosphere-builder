#  Dockerized 工具，用于构建欺骗 DeviceID 的自定义 Exosphere 二进制文件。这可用于在带有移植PRODINFO/PRODINFOF分区的 Nintendo Switch 中启动 Hos

#  如何移植 PRODINFO/PRODINFOF 并从头开始重新创建其他 EMMC 分区
#  警告！！！
- 此过程仅用于在丢失PRODINFO/PRODINFOF分区的内容后将ns引导回 hos，这并不是要取消您的ns禁令。如果您尝试这样做，最有可能的结果是您最终会得到另一个被禁止的ns，从现在开始避免任何形式的盗版，不要在线使用移植的ns

# 要求：
# WINDOWS
- 一个完整的ns EMMC 备份（ns A）
- 能够连接到电脑具有完好的硬件但没有EMMC 备份的ns（ns B）
- [NxNandManager](https://github.com/eliboa/NxNandManager)
- [HackDiskMount](https://files.sshnuke.net/HacDiskMount1055.zip)
- biskey（NSA和NSB）
- Docker
- 最新 [Hekate](https://github.com/CTCaer/hekate/releases)
- 最新 [Atmosphere](https://github.com/Atmosphere-NX/Atmosphere/releases/)
# 教程
1：使用ns A或其备份中打开 EMMC ，并使用 NxNandManager 。

2：记下DeviceID，不带首字母NX和-0末尾的( 或其他任何内容 )，跳过前两位数字。例如，如果它说：NX1122334455667788-0，您需要写下的部分将是：22334455667788。

3：从ns A转储、解密的PRODINFO和PRODINFOF分区。

4：关闭 NxNandManager。

5：使用ns B ，通过 NxNandManager从ns B打开 EMMC 。它可能会说它有BAD CRYPTO。这在未知的 EMMC 上是预料之中。

6：恢复解密的PRODINFO和PRODINFOF从NS A分区到NS B.

7：关闭 NxNandManager。

8：按照本 [91wii](https://bbs.naxgen.cn/forum.php?mod=viewthread&tid=241848&fromuid=2627124)重新创建其余的 EMMC 分区,直到并包括步骤 12。请勿尝试启动NS。

9：在SYSTEM分区上，删除文件夹中以.结尾的文件,save文夹下除8000000000000120外所有文件。不这样做可能会导致启动期间switch冻结或启动 Atmosphere 显示错误。

10：将最新版本的 Hekate 和 Atmosphere 放在您的 SD 卡上。

11：创建自定义 Exosphere 二进制文件以欺骗 DeviceID。

12：使用fusee-primary.bin或从 Hekate FSS0加载它来启动ns。


##  如何使用 Docker 创建自定义 Exosphere 二进制文件

这个工具需要一个挂载到/output容器目录的卷，以及DEVICEID环境变量，用DeviceID来欺骗。


要么在本地构建 docker 镜像，要么使用来自 Dockerhub 的预构建镜像，用DEVICEID您的 DeviceID替换该值（保留00DeviceID 之前的值。如果 NxNandManager 的输出是NX1122334455667788-0，则要使用的值应该是：0x0022334455667788. ）：


（如果您想使用特定的 Atmosphere 版本，请将Dockerfile中Atmosphere版本号，例如0.19.5）



```bash
mkdir -p ./output
docker pull pablozaiden/deviceid-exosphere-builder:latest
docker run -ti --rm -e DEVICEID=0x0022334455667788 -v "$PWD"/output:/output pablozaiden/deviceid-exosphere-builder:latest
```



构建完成后，将output/deviceid_exosphere.bin文件复制到Atmosphere您的 SD 卡目录，并将以下条目添加到BCT.ini：



```ini
[stage2]
exosphere = Atmosphere/deviceid_exosphere.bin


```

如果通过 Hekate（没有fusee-primary.bin）启动，请将其添加到启动配置中以获取自定义 exosphere 二进制文件：



```ini
secmon=Atmosphere/deviceid_exosphere.bin

```
##  如何在没有 Docker 的情况下创建自定义 Exosphere 二进制文件

要在不使用 Docker 映像的情况下构建相同的 Exosphere 自定义二进制文件，您必须首先执行以下手动步骤（有关更多详细信息，只需按照 Dockerfile 中的操作进行操作）：


安装 DevKitPro

安装构建 Atmosphere 所需的库

将 Atmosphere 克隆到你想要的 commit/tag/branch

将deviceid.patch这个 repo 中的文件复制到 Atmosphere 目录中

运行以下命令，修改补丁中的DeviceId值并应用补丁（仅在linux上测试）：
 
 
 ```bash
    export DEVICEID=0x0022334455667788
    sed -i "s/###DEVICEID###/$DEVICEID/g" deviceid.patch

    git am deviceid.patch
 
 
 ```
-
转到exosphere

Build `exosphere`: 
 
 
 ```bash
    make -j$(nproc) exosphere.bin


```


复制exosphere.bin到AtmosphereSD 卡中的目录，然后按照步骤从 dockerized 构建中配置它。
# 注意事项
## 不要共享您的转储。将deviceid_exosphere.bin被绑定到特定的DeviceID不能共享。这同样适用于PRODINFO/PRODINFOF转储。你最终可能会得到一个被禁止的SWITCH。
这样做可能会让您拥有多个具有相同 MAC 地址的SWITCH。尝试将它们同时连接到同一个无线网络可能会导致意外行为。要修改您的 MAC 地址，请编辑解密的 PRODINFO 并修改从 0x210 开始的 0x6 字节，并使用[here](https://switchbrew.org/wiki/Calibration)描述的 CRC16 方法正确填充接下来的 0x2 字节。
##  致谢：

**shchmue**, **Jan4V** and **SciresM** 的耐心回答问题以及有关此问题的所有信息以及完整的 nand 移植选项
