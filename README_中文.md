DeviceID Exosphere Builder
Dockerized 工具，用于构建欺骗 DeviceID 的自定义 Exosphere 二进制文件。这可用于在带有移植PRODINFO/PRODINFOF分区的 Nintendo Switch 中启动 Horizo​​n with Atmosphere

如何移植 PRODINFO/PRODINFOF 并从头开始重新创建其他 EMMC 分区
警告
此过程仅用于在丢失PRODINFO/PRODINFOF分区的内容后将控制台引导回 Horizo​​n 。这并不是要取消您的控制台。如果您尝试这样做，最有可能的结果是您最终会得到另一个被禁止的控制台。从现在开始避免任何形式的盗版，不要在线使用移植的控制台

要求：
视窗
一个工作控制台完整的 EMMC 备份（控制台 A），或运行最新 Hekate 的控制台，连接到 PC
具有工作硬件但没有工作 EMMC 备份的控制台（控制台 B）
NxNandManager
黑客磁盘挂载
控制台 A 和 B 的 BIS 密钥
码头工人
最新的赫卡特
最新气氛
脚步
使用控制台 A BIS 密钥从控制台 A或其备份中打开 EMMC ，并使用 NxNandManager 。
记下DeviceID，不带首字母NX和-0末尾的( 或其他任何内容 )，跳过前两位数字。例如，如果它说：NX1122334455667788-0，您需要写下的部分将是：22334455667788。
从控制台 A转储、解密PRODINFO和PRODINFOF分区。
关闭 NxNandManager。
使用控制台 B BIS 密钥，通过 NxNandManager从控制台 B打开 EMMC 。它可能会说它有BAD CRYPTO。这在 nuked EMMC 上是预料之中的。
恢复解密PRODINFO和PRODINFO从控制台甲分区到控制台B.
关闭 NxNandManager。
按照本指南重新创建其余的 EMMC 分区，BOOT0并BOOT1在控制台 B 上使用控制台 B BIS 密钥，直到并包括步骤 12。请勿尝试启动控制台。
在SYSTEM分区上，删除文件夹中除以.结尾的文件/文件save夹外的120所有文件/文件夹。不这样做可能会导致启动期间控制台冻结或启动时 Atmosphere 显示错误。
将最新版本的 Hekate 和 Atmosphere 放在您的 SD 卡上。
创建自定义 Exosphere 二进制文件以欺骗 DeviceID。
使用fusee-primary.binHekate 或从 Hekate 链式加载它来启动控制台。
如何使用 Docker 创建自定义 Exosphere 二进制文件
这个工具需要一个挂载到/output容器目录的卷，以及DEVICEID环境变量，用DeviceID来欺骗。

要么在本地构建 docker 镜像，要么使用来自 Dockerhub 的预构建镜像，用DEVICEID您的 DeviceID替换该值（保留00DeviceID 之前的值。如果 NxNandManager 的输出是NX1122334455667788-0，则要使用的值应该是：0x0022334455667788. ）：

（如果您想使用特定的 Atmosphere 版本，请latest使用 Atmosphere 版本号更改docker 标签，例如0.14.4）

mkdir -p ./输出
码头工人拉 pablozaiden/deviceid-exosphere-builder:latest
docker run -ti --rm -e DEVICEID=0x0022334455667788 -v " $PWD " /output:/output pablozaiden/deviceid-exosphere-builder:latest
构建完成后，将output/deviceid_exosphere.bin文件复制到Atmosphere您的 SD 卡目录，并将以下条目添加到BCT.ini：

[stage2] 
exosphere = Atmosphere/deviceid_exosphere.bin
如果通过 Hekate（没有fusee-primary.bin）启动，请将其添加到启动配置中以获取自定义 exosphere 二进制文件：

secmon =大气/deviceid_exosphere.bin
如何在没有 Docker 的情况下创建自定义 Exosphere 二进制文件
要在不使用 Docker 映像的情况下构建相同的 Exosphere 自定义二进制文件，您必须首先执行以下手动步骤（有关更多详细信息，只需按照 Dockerfile 中的操作进行操作）：

安装 DevKitPro
安装构建 Atmosphere 所需的库
将 Atmosphere 克隆到你想要的 commit/tag/branch
将deviceid.patch这个 repo 中的文件复制到 Atmosphere 目录中
运行以下命令，修改补丁中的DeviceId值并应用补丁（仅在linux上测试）：
导出设备ID=0x0022334455667788
sed -i " s/###DEVICEID###/ $DEVICEID /g " deviceid.patch

git am deviceid.patch
转到exosphererepo 内的目录
构建exosphere：
make -j $( nproc ) exosphere.bin
复制exosphere.bin到AtmosphereSD 卡中的目录，然后按照步骤从 dockerized 构建中配置它。
注意事项
重要的。不要共享您的转储和个性化构建。将deviceid_exosphere.bin被绑定到特定的DeviceID和不能共享。这同样适用于PRODINFO/PRODINFOF转储。你最终可能会得到一个被禁止的控制台。
这样做可能会让您拥有多个具有相同 MAC 地址的控制台。尝试将它们同时连接到同一个无线网络可能会导致意外行为。要修改您的 MAC 地址，请编辑解密的 PRODINFO 并修改从 0x210 开始的 0x6 字节，并使用此处描述的 CRC16 方法正确填充接下来的 0x2 字节。
致谢：
shchmue、Jan4V和SciresM耐心回答问题以及有关此问题的所有信息以及完整的 nand 移植选项。
