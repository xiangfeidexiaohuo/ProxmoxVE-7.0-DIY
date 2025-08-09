## PVE 直通+CPU/硬盘/温度显示+换源/去订阅+CPU睿频模式 一键脚本

![png](./pic/55.png)

- 原始脚本来自 [shidahuilang/pve](https://github.com/shidahuilang/pve/blob/main/pve.sh)

- 轻微修改，简化优化、支持 PVE9！

- **修改 PVE 存在风险，请自行备份重要数据；出现任何风险，自行承担！**

- 防止系统没安装curl，使用不了一键脚本，先执行安装curl命令：
```sh
apt-get update -y && apt-get install curl wget sudo bash -y
```

- **一键脚本：**
```
bash -c "$(curl -fsSL https://raw.githubusercontent.com/xiangfeidexiaohuo/pve-diy/master/pve.sh)"
```

![png](./pic/56.png)

***

- **PVE8 升 PVE9：**

<details>
<summary>点击展开，查看详细教程！</summary>

- 1、已换源的情况下，运行以下两行命令：（若未换源，运行上面的一键脚本先换源。）
```
apt update
apt dist-upgrade
```
- 2、运行升级检查命令：（Error 错误多，就放弃，直接镜像重装。）
```
pve8to9
```
- 3、修改为 PVE9 的源：
```
sed -i 's/bookworm/trixie/g' /etc/apt/sources.list
sed -i 's/bookworm/trixie/g' /etc/apt/sources.list.d/*.list
```
- 4、开始更新 PVE9：（升级过程中，若出现弹窗，不懂就一律默认回车。）
```
apt update
apt dist-upgrade
```
- 5、升级完成，重启：（重启完成后，运行上面的一键脚本，修改温度等。）
```
reboot
```

</details>

***
***

## Proxmox VE 7.x 相关教程（7.x 旧版手动修改，有参考价值，弃用！）

* **包括但不限于换源、直通、界面显示温度/频率等。**


***


### Proxmox VE 7.x 换源

* **必须保证pve系统能联网，没联网，没法换源。**

<details>
<summary>点击展开，查看详细教程！</summary>

#### SSH登录到pve后台，然后一条一条的执行命令

#### 1.处理掉企业源
```
rm -rf /etc/apt/sources.list.d/pve-install-repo.list
```

```
echo "#deb https://enterprise.proxmox.com/debian/pve Bullseye pve-enterprise" > /etc/apt/sources.list.d/pve-enterprise.list
```


#### 2.开始换源

```
wget https://mirrors.ustc.edu.cn/proxmox/debian/proxmox-release-bullseye.gpg -O /etc/apt/trusted.gpg.d/proxmox-release-bullseye.gpg
```

```
echo "deb https://mirrors.ustc.edu.cn/proxmox/debian/pve bullseye pve-no-subscription" > /etc/apt/sources.list.d/pve-no-subscription.list
```
```
echo "deb https://mirrors.ustc.edu.cn/proxmox/debian/ceph-pacific bullseye main" > /etc/apt/sources.list.d/ceph.list
```

```
sed -i.bak "s#http://download.proxmox.com/debian#https://mirrors.ustc.edu.cn/proxmox/debian#g" /usr/share/perl5/PVE/CLI/pveceph.pm
```
```
sed -i.bak "s#ftp.debian.org/debian#mirrors.aliyun.com/debian#g" /etc/apt/sources.list
```
```
sed -i "s#security.debian.org#mirrors.aliyun.com/debian-security#g" /etc/apt/sources.list
```
```
echo "deb http://download.proxmox.com/debian/pve bullseye pve-no-subscription" >>  /etc/apt/sources.list
```


#### 3.最后更新
```
apt update && apt dist-upgrade -y
```

</details>



***


###  Proxmox VE 7.x 关订阅提示

<details>
<summary>点击展开，查看详细教程！</summary>

#### 1.WinSCP登录到PVE，编辑打开这个文件：/usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js (防止手贱，建议备份)


#### 2.搜索 data.status.toLowerCase，找到这一段：

![jpg](./pic/01.jpg)

#### 3.直接删掉红框内容，变成如下图，最后保存即可。

![jpg](./pic/02.jpg)

* 如果更新到PVE 7.1-5或者更新，发现改了上面的没效果，那么就要多改一步：

* 继续搜索 data.status.toLowerCase，找到这一段：

![jpg](./pic/03.jpg)

* 直接删掉红框内容，变成如下图，最后保存即可

![jpg](./pic/04.jpg)

#### 4.改完保存，重进PVE界面刷新，去更新点击刷新，就没订阅提示了。

</details>



***


### Proxmox VE 主界面显示CPU温度

<details>
<summary>点击展开，查看详细教程！</summary>

#### 1.登录PVE的SSH，执行命令安装sensors：
```
apt-get install lm-sensors
```

#### 2.探测下温度，执行：`sensors-detect` (一路yes，回车)

#### 3.获取温度信息，执行：`sensors`

![jpg](./pic/1.jpg)

 * 红色箭头：temp1这里是主板温度，可能某些主板还有temp2、temp3等

 * 黄色箭头：Package id 0、core0~5这里是CPU温度，有多少个核心，就显示多少


#### 4.WinSCP登录到PVE，修改这个文件：/usr/share/perl5/PVE/API2/Nodes.pm 

(我习惯于下载到本地用Notepad++修改)

搜索：`$res->{pveversion} = PVE::pvecfg::package()`

在这个定义的下方添加：```$res->{thermalstate} = `sensors`;```

结果如图：

![jpg](./pic/2.jpg)


修改完保存，然后塞回路径。


#### 5.修改这个文件：/usr/share/pve-manager/js/pvemanagerlib.js (建议备份，万一自己改炸就不好了。)

(我习惯于下载到本地用Notepad++修改)

搜索`PVE Manager Version`

![jpg](./pic/3.jpg)

 * 在这个定义的下方添加一个定义：

```
    {
          itemId: 'thermal',
          colspan: 2,
          printBar: false,
          title: gettext('温度'),
          textField: 'thermalstate',
          renderer:function(value){
              const p0 = value.match(/Package id 0.*?\+([\d\.]+)Â/)[1];
              const b0 = value.match(/temp1.*?\+([\d\.]+)?/)[1];
              return `CPU: ${p0} ℃ || 主板: ${b0} ℃ `
            }
    },
```
结果如图：

![jpg](./pic/4.jpg)

* 上述是一种比较简单的万金油做法，有些人可能要把每个核心的温度有写出来，或者说有几个主板温度，也要一起写出来，那么就按照下列的格式：

```
    {
          itemId: 'thermal',
          colspan: 2,
          printBar: false,
          title: gettext('温度'),
          textField: 'thermalstate',
          renderer:function(value){
              const p0 = value.match(/Package id 0.*?\+([\d\.]+)Â/)[1];
              const c0 = value.match(/Core 0.*?\+([\d\.]+)Â/)[1];
              const c1 = value.match(/Core 1.*?\+([\d\.]+)Â/)[1];
              const c2 = value.match(/Core 2.*?\+([\d\.]+)Â/)[1];
              const c3 = value.match(/Core 3.*?\+([\d\.]+)Â/)[1];
              const c4 = value.match(/Core 4.*?\+([\d\.]+)Â/)[1];
              const c5 = value.match(/Core 5.*?\+([\d\.]+)Â/)[1];
              const b0 = value.match(/temp1.*?\+([\d\.]+)?/)[1];
              const b1 = value.match(/temp2.*?\+([\d\.]+)?/)[1];
              return `CPU: ${p0} ℃ || CPU1: ${c0} ℃ CPU2: ${c1} ℃ CPU3: ${c2} ℃ CPU4: ${c3} ℃ CPU5: ${c4} ℃ CPU6: ${c5} ℃ || 主板1: ${b0} ℃ 主板2: ${b1} ℃`
            }
    },
```
结果如图：

![jpg](./pic/5.jpg)

* 红框内就是单独每个核心或者每个主板温度

* 前面的CPU温度是综合温度，后面的CPU1~6是每个核心单独温度

* 可根据实际情况增减CPU温度或者主板温度

* 其实如果核心超过4个，不建议把每个核心温度写出来，不美观

#### 6.改完执行 `systemctl restart pveproxy` 重进PVE主页，就看到温度显示了。

![jpg](./pic/6.jpg)


</details>


***


### Proxmox VE 主界面显示(PCI-E盘位)硬盘温度

**承接上一个显示CPU温度教程。**

* 此教程主要针对PCI-E盘位硬盘(也就是M.2固态)，SATA盘位硬盘显示温度教程在下方。

* 此教程是在上个“Proxmox VE 主界面显示CPU温度”基础上来做。

<details>
<summary>点击展开，查看详细教程！</summary>

#### * 扩展下，主界面添加M.2固态硬盘温度：

#### 1.已经完成“Proxmox VE 主界面显示CPU温度”，然后终端执行：`sensors`

![jpg](./pic/34.jpg)

* nvme-pci-0100此处就是M.2固态温度

#### 2.修改这个文件：/usr/share/pve-manager/js/pvemanagerlib.js

在上个教程改CPU温度的定义里，加入下图红框内的内容：

`const nvme0 = value.match(/Composite.*?\+([\d\.]+)Â/)[1];`

`|| 固态: ${nvme0} ℃ `

![jpg](./pic/35.jpg)

#### 3.改完保存执行`systemctl restart pveproxy`重进PVE主页。

![jpg](./pic/36.jpg)

</details>



***


### Proxmox VE 主界面显示CPU频率

**承接上一个显示CPU温度教程。**

<details>
<summary>点击展开，查看详细教程！</summary>

#### * 扩展下，主界面添加CPU频率：

#### 1.也是修改 /usr/share/perl5/PVE/API2/Nodes.pm 和 /usr/share/pve-manager/js/pvemanagerlib.js 这2个文件

* /usr/share/perl5/PVE/API2/Nodes.pm 刚刚修改CPU温度那里添加：

```
$res->{cpusensors} = `lscpu | grep MHz`;
```
![jpg](./pic/25.jpg)

* /usr/share/pve-manager/js/pvemanagerlib.js 刚刚修改温度的下一行添加：

```
	{
          itemId: 'MHz',
          colspan: 2,
          printBar: false,
          title: gettext('CPU频率'),
          textField: 'cpusensors',
          renderer:function(value){
			  const f0 = value.match(/CPU MHz.*?([\d]+)/)[1];
			  const f1 = value.match(/CPU min MHz.*?([\d]+)/)[1];
			  const f2 = value.match(/CPU max MHz.*?([\d]+)/)[1];
			  return `实时: ${f0} MHz | 最小: ${f1} MHz | 最大: ${f2} MHz `
            }
	},
```
![jpg](./pic/26.jpg)

#### 2.改完执行 `systemctl restart pveproxy` 重进PVE主页，效果如图：

![jpg](./pic/27.jpg)

</details>


***


### Proxmox VE 主界面显示(SATA盘位)硬盘温度

**同样承接上一个显示CPU温度教程。**

* 此教程主要针对SATA盘位硬盘。

<details>
<summary>点击展开，查看详细教程！</summary>

#### * 扩展下，主界面添加硬盘温度：

#### 1.登录PVE的SSH，执行命令安装hddtemp：
```
apt-get install hddtemp
```

* 然后执行 `chmod +s /usr/sbin/hddtemp`

* 执行 `hddtemp /dev/sd?` 就能看到硬盘温度：

![jpg](./pic/33.jpg)


#### 2.然后修改 /usr/share/perl5/PVE/API2/Nodes.pm 和 /usr/share/pve-manager/js/pvemanagerlib.js 这2个文件

* /usr/share/perl5/PVE/API2/Nodes.pm 刚刚修改CPU温度那里添加：

```
 $res->{thermal_hdd} = `hddtemp /dev/sd?`;
```
![jpg](./pic/30.jpg)

* /usr/share/pve-manager/js/pvemanagerlib.js 刚刚修改CPU温度的下一行添加：

```
	{
            itemId: 'thermal-hdd',
            colspan: 2,
            printBar: false,
            title: gettext('硬盘温度'),
            textField: 'thermal_hdd',
            renderer: function(value) {
                value = value.replaceAll('Â', '');
                return value.replaceAll('\n', '<br>');
            }
	},
```
![jpg](./pic/31.jpg)

#### 3.改完执行 `systemctl restart pveproxy` 重进PVE主页，效果如图：

![jpg](./pic/32.jpg)



</details>



***


### Proxmox VE 主界面显示温度的另一种方案: JSON格式

* 因为常规的方案有些局限，但是吧，我又不想推翻整个“温度”相关的教程，所以单独说下。

* 自己本身没插nvme固态，所以呢，我就以普通的CPU温度为例，道理是一样的。

* 适合多个nvme固态等同参数的获取温度，如果不懂是啥，此教程就忽略。

<details>
<summary>点击展开，查看详细教程！</summary>

#### * 比如多个nvme固态显示温度，看如下图：

![jpg](./pic/44.jpg)

可以看到两块nvme固态的温度读取值都是Composite，但是确实有2个固态，用之前常规的方案，就无法显示2个nvme固态的温度，所以需要另外的方案。

#### 1.常理还是必须安装sensors：

```
apt-get install lm-sensors
```

#### 2.然后终端执行`sensors -j`

![jpg](./pic/45.jpg)

coretemp-isa-0000 CPU传感器

temp1_input/temp2_input/temp3_input等就是核心温度


#### 3.也是修改：/usr/share/perl5/PVE/API2/Nodes.pm 同样的位置：

```
$res->{sensinfo} = `sensors -j`;
```
![jpg](./pic/46.jpg)


#### 4.也是修改这个文件：/usr/share/pve-manager/js/pvemanagerlib.js 同样的位置： 

```
	{
          itemId: 'sensinfo',
          colspan: 2,
          printBar: false,
          title: gettext('温度'),
          textField: 'sensinfo',
          renderer:function(value){
			  value = JSON.parse(value.replaceAll('Â', ''));
			  const c0 = value['coretemp-isa-0000']['Core 0']['temp2_input'].toFixed(1);
			  const c1 = value['coretemp-isa-0000']['Core 1']['temp3_input'].toFixed(1);
			  const c2 = value['coretemp-isa-0000']['Core 2']['temp4_input'].toFixed(1);
			  const c3 = value['coretemp-isa-0000']['Core 3']['temp5_input'].toFixed(1);			  
			  return `CPU温度: ${c0}℃ | ${c1}℃ | ${c2}℃ | ${c3}℃ `; 
            }
    },
```

![jpg](./pic/47.jpg)

* coretemp-isa-0000 CPU传感器

* Core 0 到 Core 3 就是CPU核心1~4

* temp2_input 到 temp5_input CPU核心1~4的温度

* 如果有别的传感器温度，比如nvme固态，依葫芦画瓢更改内容。


#### 5.如果用这种方案改写了CPU温度，之前的常规CPU方案代码可以去掉，自行理解，这个教程我写的比较简单。

#### 6.改完保存执行`systemctl restart pveproxy`重进PVE主页。


</details>



***


### Proxmox VE 改显示范围

**如果发现改上面的温度/CPU频率/硬盘温度，步骤又没错，但是主界面不显示，就需要下列教程。**

<details>
<summary>点击展开，查看详细教程！</summary>

#### 改布局：

* 还是这个文件：pvemanagerlib.js，搜索：`widget.pveNodeStatus`

![jpg](./pic/29.jpg)

将 height: 300 改大为400，或者更大，然后保存。



* 搜索：`gettext('Status') + ': ' + zpool` (这一处不一定有，搜不到，就不用管了。)

![jpg](./pic/28.jpg)

将 height: 600 改大为700，或者更大，然后保存。


</details>



***


### 核显直通(intel)

<details>
<summary>点击展开，查看详细教程！</summary>

#### 1.编辑GRUB配置文件：/etc/default/grub

```
sed -i "s/quiet/quiet intel_iommu=on iommu=pt video=efifb:off,vesafb:off/g" /etc/default/grub
```

改好结果：

![jpg](./pic/7.jpg)


然后执行：
```
update-grub
```

#### 2.添加所需的系统模块(驱动)：/etc/modules

```
echo "vfio" >> /etc/modules

echo "vfio_iommu_type1" >> /etc/modules

echo "vfio_pci" >> /etc/modules

echo "vfio_virqfd" >> /etc/modules
```

改好结果：

![jpg](./pic/8.jpg)


* #### PVE7.x 直接跳过3、4、5，直接到第6步即可；PVE6.x就一步步看。


#### 3.添加模块(驱动)黑名单：/etc/modprobe.d/pve-blacklist.conf

```
echo "blacklist snd_hda_intel" >> /etc/modprobe.d/pve-blacklist.conf

echo "blacklist snd_hda_codec_hdmi" >> /etc/modprobe.d/pve-blacklist.conf

echo "blacklist i915" >> /etc/modprobe.d/pve-blacklist.conf
```

改好结果：

![jpg](./pic/9.jpg)


#### 4.查看GPU的ID：
```
lspci -nn | grep VGA
```

比如我的：
```
00:02.0 VGA compatible controller [0300]: Intel Corporation HD Graphics [8086:1606] (rev 08)
```
![jpg](./pic/10.jpg)

 * 8086:1606 就是核显的ID

 * 00:02.0 是核显的编号

接着执行：(ids=xxxx:xxxx，xxxx:xxxx替换成你获取的ID)
```
echo "options vfio-pci ids=8086:1606" >> /etc/modprobe.d/vfio.conf
```

#### 5.如果要音频直通，就搜索音频设备的ID
```
lspci -nn | grep Audio
```
比如我的：
```
00:03.0 Audio device [0403]: Intel Corporation Broadwell-U Audio Controller [8086:160c] (rev 08)
00:1b.0 Audio device [0403]: Intel Corporation Wildcat Point-LP High Definition Audio Controller [8086:9ca0] (rev 03)
```
![jpg](./pic/11.jpg)

 * 8086:160c/8086:9ca0  就是音频设备ID (一个是板载，一个是单独的音频孔，所以是2个)

 * 00:03.0/00:1b.0 是音频设备编号

接着执行：(ids=xxxx:xxxx，xxxx:xxxx替换成你获取的GPU/音频设备ID，用英文逗号隔开)
```
echo "options vfio-pci ids=8086:1606,8086:160c,8086:9ca0" >> /etc/modprobe.d/vfio.conf
```

#### 6.更新内核并重启：

执行：
```
update-initramfs -u

reboot
```

#### 7.验证是否开启iommu：

终端输入：

```
dmesg | grep 'remapping'
```

要出现：DMAR-IR: Enabled IRQ remapping in x2apic mode


接着用下面的命令：
```
find /sys/kernel/iommu_groups/ -type l  
```
出现很多数据，每一行最后的xx:xx.x是设备编号，查看要直通的设备的编号是否在里面。

#### 8.添加PCI设备即可：(我只添加核显，音频设备看设备编号按需添加)


![jpg](./pic/12.jpg)


![jpg](./pic/13.jpg)


#### 9.验证核显直通成功没？

 * 在直通后的系统的终端执行：
```
ls /dev/dri
```
 * 输出如图，出现“renderD128”就成功了：

![jpg](./pic/21.jpg)

</details>


***


### 黑群晖7.x人脸识别(DS918+)

<details>
<summary>点击展开，查看详细教程！</summary>

#### 1.已经按照上面的步骤，核显直通黑裙7；

#### 2.编辑"/etc/pve/qemu-server/102.conf"文件：

* 为啥是102.conf，因为演示的黑裙7的虚拟机ID是102，根据你自己的来。

显卡hostpci0: 这一行的最后，添加：
```
,legacy-igd=1
```

![jpg](./pic/22.jpg)

#### 3.回到PVE管理界面，更改黑裙7的硬件——显示为无，然后重启黑裙7。

![jpg](./pic/23.jpg)

#### 4.然后等待Photos索引，就能看到人脸已经识别了。

![jpg](./pic/24.jpg)

</details>



***


### GVT-G直通(intel)

**此方式适合桌面级别的U(差不多5代起步)，小主机(J4125/N5105等)不支持！！！**

* 这种直通和上面的直通方法，二选一，不能同时选2种！！！

* 好处是显卡拆成好多个分别给不同虚拟机使用。


<details>
<summary>点击展开，查看详细教程！</summary>

#### 1.首先在主板BIOS里面启用GTD，GTX等选项，若有aperture size选项，建议512M，没有就不管吧；

#### 2.编辑GRUB配置文件：/etc/default/grub

```
sed -i "s/quiet/quiet intel_iommu=on i915.enable_gvt=1/g" /etc/default/grub
```

然后执行：
```
update-grub
```

#### 3.添加所需的系统模块(驱动)：/etc/modules

```
echo "vfio" >> /etc/modules

echo "vfio_iommu_type1" >> /etc/modules

echo "vfio_pci" >> /etc/modules

echo "vfio_virqfd" >> /etc/modules

echo "kvmgt" >> /etc/modules
```
#### 4.更新内核并重启：

执行：
```
update-initramfs -u

reboot
```

#### 5.验证是否开启GVT：

0000:00:02.0  将00:02.0换成自己的GPU的编号 (lspci -nn | grep VGA 查看，最前面的就是)

```
ls /sys/bus/pci/devices/0000:00:02.0/mdev_supported_types/
```

出现如下即为成功(教程机核显为UHD630)：

i915-GVTg_V5_4 i915-GVTg_V5_8

#### 6.配置直通：

cpu类型设置成HOST，将机器设置成q35，将虚拟机显卡设置成无，添加PCIE设备：勾选高级里的ROM-Bar和PCI-E，主GPU不勾选，MDev类型选择合适"显卡"。

![jpg](./pic/20.jpg)

* “可用”显示的多少，就可以添加多个“显卡”。

比如上图就是说：只能添加1个v5_4的“显卡”或者添加2个v5_8的“显卡”，只能用一种类型的显卡。

</details>


***



### GVT-G改显存，增加"显卡"

**承接上一个GVT-G直通教程。**

**此教程有风险，请知悉，若操作，请自行承担风险。**

<details>
<summary>点击展开，查看详细教程！</summary>

#### 为了分配更多显卡，需给核显分配更多显存。但一般主板的BIOS并没有给调节显存的选项"aperture size"，只给了类似DVMT(共享显存)选项，最大1024MB，当把DVMT改成1024MB后，并没有用。所以本教程强行改aperture size。

通过命令 `lspci -vs 00:02.0` 查看，一般默认是256M。

![jpg](./pic/gvt/1.jpg)

128M的显卡只有一个，所以需要改大到512M。

![jpg](./pic/20.jpg)

#### 1.自行去主板官网下载当前主板的BIOS(注意版本)，然后下载教程提供的工具包。

[工具包下载地址](https://raw.githubusercontent.com/xiangfeidexiaohuo/ProxmoxVE-7.0-DIY/master/%E6%94%B9GVT%E5%B7%A5%E5%85%B7%E5%8C%85.zip)

#### 2.使用工具包里的UEFITool0270工具，打开BIOS文件，提取模块；

![jpg](./pic/gvt/2.jpg)

* 按Ctrl+F打开搜索页面，切换到text选项卡，搜索aperture size：

![jpg](./pic/gvt/3.jpg)

* 点击搜索出来的结果，会跳转到对应模块位置；

![jpg](./pic/gvt/4.jpg)

* 然后导出模块，并另存为。

![jpg](./pic/gvt/5.jpg)

![jpg](./pic/gvt/6.jpg)

#### 3.使用工具包里的IRFExtractor.exe打开另存为的文件，找偏移量。

* 打开，并解析出文本，另存为。

![jpg](./pic/gvt/7.jpg)

![jpg](./pic/gvt/8.jpg)

* 打开解析文本，搜索aperture size，红框内的 `0x2E8` 就是我们要找的偏移量。每个主板的BIOS偏移量不一样。

![jpg](./pic/gvt/9.jpg)

* 通过图可以看出，默认是0x1，也就是256M。若要改512M，就得默认0x3，改1G，就得默认0x7。

![jpg](./pic/gvt/10.jpg)

#### 4.准备一个U盘，格式化为FAT32，然后把工具包内的EFI文件夹放U盘根目录，然后电脑重启进U盘引导。

![jpg](./pic/gvt/11.jpg)

* U盘引导进入grub命令行模式，直接输入命令：`setup_var 0x2E8 0x3` ，意思就是把aperture size的偏移量默认改成0x3，也就是aperture size为512M。

* 建议只改512M，经验告诉我们改1G，可能会出问题。

![jpg](./pic/gvt/12.jpg)

#### 5.改完成功后，开机PVE，就会看到"显卡"多了很多。

通过命令 `lspci -vs 00:02.0` 查看，已经变成512M。

![jpg](./pic/gvt/16.jpg)

![jpg](./pic/gvt/13.jpg)


</details>


***


### GVT-G黑群辉核显直通+人脸

<details>
<summary>点击展开，查看详细教程！</summary>

#### 1.先把群辉虚拟机关机，然后硬件——显示，设为无； 然后修改虚拟机配置文件；

/etc/pve/qemu-server/102.conf (教程演示群辉虚拟机ID是102，所以是102.conf)

在配置文件第一行写入以下代码：

```
args: -device vfio-pci,sysfsdev=/sys/bus/mdev/devices/604e42e4-2e90-11ec-8861-037c58d42915,addr=02.0,x-igd-opregion=on,driver=vfio-pci-nohotplug
```
![jpg](./pic/gvt/14.jpg)

#### 2.然后PVE终端，运行命令：

```
mkdir /var/lib/vz/snippets

cp /usr/share/pve-docs/examples/guest-example-hookscript.pl /var/lib/vz/snippets/102-autocreate.pl
```

尾部的102和虚拟机ID对应；然后修改/var/lib/vz/snippets/102-autocreate.pl，在如图位置添加下列2行代码：

```
system("echo 604e42e4-2e90-11ec-8861-037c58d42915 > /sys/bus/pci/devices/0000:00:02.0/mdev_supported_types/i915-GVTg_V5_4/create");

上一行代码中的i915-GVTg_V5_4，取决于你要什么类型的显卡，V5_2/8都行。


system("echo 1 > /sys/bus/mdev/devices/604e42e4-2e90-11ec-8861-037c58d42915/remove");
```
![jpg](./pic/gvt/15.jpg)


#### 3.最后终端执行：`qm set 102 --hookscript local:snippets/102-autocreate.pl` (102和虚拟机ID对应)

#### 4.群辉开机，核显已经直通，且photo人脸ok。


</details>


***



### 直通硬盘(全盘映射)

<details>
<summary>点击展开，查看详细教程！</summary>

#### 上面说了核显直通，接着说硬盘直通。前面步骤完成了，现在很简单了。

#### 1.查看读取存储设备序列号：
```
ls /dev/disk/by-id
```

![jpg](./pic/18.jpg)


找出自己的硬盘序列号。比如我的就是：

ata-ST1000XXXXXXXXXXXXXXX

#### 2.执行命令：

 * 102：改成自己要直通硬盘的的虚拟机ID。

 * sata1：已有sata0，所以往后排，为sata1，按需修改。

```
qm set 102 -sata1 /dev/disk/by-id/ata-ST1000XXXXXXXXXXXXXXX
```

返回下面信息就说明成功挂载：

update VM 102: -sata1 /dev/disk/by-id/ata-ST1000XXXXXXXXXXXXXXX


#### 3.返回PVE查看，已经挂载，重启即可完成。

![jpg](./pic/19.jpg)

</details>


***


### PVE 直接安装Docker

<details>
<summary>点击展开，查看详细教程！</summary>

#### 1.登录PVE的SSH，输入以下命令，安装Docker：
```
curl -sSL https://get.docker.com/ | sh

chmod 777 /var/run/docker.sock

systemctl start docker

systemctl enable docker.service
```

上面的命令一条一条执行完毕后，docker就安装好了。

![jpg](./pic/14.jpg)


#### 2.接着终端继续输入命令，安装Portainer-CE汉化版：

```
docker run -d --restart=always --name="portainer" -p 9000:9000 -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data 6053537/portainer-ce
```

#### 3.等待上述安装命令执行完毕，然后打开PVE的IP地址+9000端口，进入Portainer-CE。

ps：例如我的pve的ip是192.168.100.1，那么就是进入http://192.168.100.1:9000。

![jpg](./pic/17.jpg)

</details>



***


### 配置PVE的性能/省电模式

* 常理来说Linux的系统CPU模式是性能模式，而PVE是基于debian的系统，所以PVE默认的也是性能模式。

* 可能有些小伙伴的PVE主机性能很强，功耗也高，可以通过设置成省电模式，来降低功耗。

* 具体能降低多少，请自测。

<details>
<summary>点击展开，查看详细教程！</summary>


#### 1.当前PVE主机是否支持多种模式切换，终端执行：
```
cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_available_governors
```

![jpg](./pic/48.jpg)

能返回 `performance` (性能模式) 和 `powersave` (省电模式)等，就接着往下看，如果只有 `performance` 就关闭这个教程。

* 有些机器CPU比较新，可能还有conservative、schedutil、ondemand等，都可以切换。

#### 2.安装cpupower，终端执行：
```
apt-get install linux-cpupower -y
```

![jpg](./pic/49.jpg)


#### 3.切换成省电模式，终端执行：
```
cpupower -c all frequency-set -g powersave
```

![jpg](./pic/50.jpg)

有多少核，就会返回多少cpu。


* 如果想切换到其他模式：conservative、schedutil、ondemand等

```
cpupower -c all frequency-set -g conservative
```
```
cpupower -c all frequency-set -g schedutil
```
```
cpupower -c all frequency-set -g ondemand
```


* 然后执行： `cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor` ，查看当前CPU处于什么模式。

![jpg](./pic/51.jpg)

可以看到已经切成`powersave`省电模式。

* 如果要切回性能模式，就执行： `cpupower -c all frequency-set -g performance` 。


</details>



***


### PVE7.x升级第三方内核6.0

* PVE7.x目前官方默认内核是5.15，找到个第三方的6.0内核，也可以玩玩。

* 仅适用于PVE 7.x，PVE6.x绕道。

* 仅适用于动手能力强的，小白绕道!

* 升级内核可能有翻车的风险，风险自行承担！

<details>
<summary>点击展开，查看详细教程！</summary>

#### 0.升级6.0内核之前，请先牢记当前pve的内核版本号，后面回退要用。

![jpg](./pic/54.jpg)

如图所示，当前内核版本号就为：5.15.64-1-pve


#### 1.添加存储库的GPG密钥，终端执行：
```
curl -1sLf 'https://dl.cloudsmith.io/public/pve-edge/kernel/gpg.8EC01CCF309B98E7.key' | gpg --dearmor -o /usr/share/keyrings/pve-edge-kernel.gpg
```

#### 2.设置pve-edge-kernel存储库，终端执行：
```
echo "deb [signed-by=/usr/share/keyrings/pve-edge-kernel.gpg] https://dl.cloudsmith.io/public/pve-edge/kernel/deb/debian bullseye main" > /etc/apt/sources.list.d/pve-edge-kernel.list
```

#### 3.安装内核，终端执行：
```
apt update
```

```
apt install pve-kernel-6.0-edge
```

#### 4.重启以后，内核就升级到6.0了。

![jpg](./pic/52.jpg)


#### 5.查看内核更新没，可去下面的网址看看，若有新版，重复第三步的操作。

[查看内核更新](https://github.com/fabianishere/pve-edge-kernel/releases)


#### 6.已经升级到第三方内核，如果你在wed界面“更新”，看到更新，就不要胡乱更新了。


#### 7.如果需要回退内核，利用proxmox-boot-tool回退，终端执行：

```
proxmox-boot-tool kernel pin 升级前的内核版本号

```
升级前我的内核版本号是：5.15.64-1-pve，所以命令如下：

```
proxmox-boot-tool kernel pin 5.15.64-1-pve

```

```
proxmox-boot-tool refresh
```

#### 8.执行完第7步的两条命令后，系统重启，内核就回退了。

![jpg](./pic/53.jpg)


然后看到内核切回了，然后终端运行：

```
rm -rf /etc/default/grub.d/proxmox-kernel-pin.cfg
```

以后就能正常使用“升级”来更新PVE。

</details>


