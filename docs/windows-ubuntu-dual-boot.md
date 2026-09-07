# Windows 上安装 Ubuntu 双系统：从准备到完成

**目标：保留 Windows，使用 Ventoy 安装 Ubuntu 22.04.4 桌面版。** 本文以这次实际使用的“UEFI + 一块约 1TB 硬盘 + 从 E 盘分出 300GB”为例。

正文按顺序操作，每一步都写明“怎么做”和“完成后看到什么”。**已经装好错误版本的 Ubuntu，请直接看[附录 A：替换旧版本](#reinstall)，不要再次压缩 E 盘。** 其他报错和概念解释统一放在文末。

## 操作顺序

[步骤 1：准备文件和工具](#prepare) → [步骤 2：检查 Windows](#windows) → [步骤 3：分出磁盘空间](#space) → [步骤 4：制作启动 U 盘](#ventoy) → [步骤 5：从 U 盘启动](#boot) → [步骤 6：填写安装选项](#install-options) → [步骤 7：选择分区并安装](#install) → [步骤 8：重启并检查](#verify)

---

<a id="prepare"></a>
## 步骤 1：准备文件和工具

### 1. 下载 Ventoy

1. 打开 [Ventoy 官方下载页](https://www.ventoy.net/en/download.html)。
2. 下载适用于 Windows 的压缩包，文件名类似 `ventoy-版本号-windows.zip`。
3. 解压到一个容易找到的文件夹，后面需要运行里面的 `Ventoy2Disk.exe`。

### 2. 下载 Ubuntu 22.04.4

点击 [Ubuntu 22.04.4 桌面版官方下载](https://old-releases.ubuntu.com/releases/22.04.4/ubuntu-22.04.4-desktop-amd64.iso)。

下载完成后，确认文件名是：

```text
ubuntu-22.04.4-desktop-amd64.iso
```

**这次需要准确的 22.04.4。** 不要选 22.04.5、26 系列或 server 版；ISO 文件不用解压。下载完整性检查方法见[附录 D](#checks)。

### 3. 准备 U 盘和备份

1. 准备一个容量足够的 U 盘，建议 16GB 或以上；本次使用约 64GB U 盘。
2. 把 U 盘内需要的文件复制出来，首次安装 Ventoy 会清除 U 盘。
3. 备份电脑上需要保留的文件，给笔记本接上电源。
4. 用手机打开本指南，方便电脑重启时继续查看。

**完成标志：**有一个已解压的 Ventoy 文件夹、一个准确的 22.04.4 ISO，以及准备好的 U 盘。

<a id="windows"></a>
## 步骤 2：检查 Windows 的启动模式和加密状态

### 1. 确认 UEFI 模式

1. 按键盘 **`Win + R`**，打开“运行”。
2. 输入 **`msinfo32`**，按回车。
3. 在“系统摘要”找到 **“BIOS 模式”**。
4. 本次电脑显示 **UEFI**，后面安装 Ubuntu 时也使用 UEFI 启动。

如果显示“传统/Legacy”，先看[附录 B.6](#legacy)，不要直接照本例修改启动模式。

### 2. 查询 BitLocker

1. 按 **`Win + S`**，输入 `cmd`，先不按回车。
2. 在搜索结果的 **“命令提示符”**上右键。
3. 选 **“以管理员身份运行”**，权限提示点“是”。
4. 在打开的窗口输入下列命令，按回车：

```bat
manage-bde -status
```

查看各盘的 **“转换状态”“已加密百分比”“保护状态”**。本次 C、D、E 都已加密，Ventoy U 盘未加密。具体截图和状态解释见[附录 B.2](#bitlocker-status)。

### 3. 保存恢复密钥

加密已开启时，先在另一台设备上保存对应的恢复密钥：

1. 打开 [微软账户恢复密钥页面](https://account.microsoft.com/devices/recoverykey)。
2. 登录这台电脑使用的微软账户，查找与本机、相应卷对应的记录。
3. 保存在手机或其他设备上，确保电脑无法进入 Windows 时也能查看。

**恢复密钥只自己保存，不发到聊天或 GitHub。** 工作/学校管理的设备，应向所属组织查询。

**完成标志：**已确认 UEFI；知道磁盘是否加密；加密盘的恢复密钥可在其他设备上查看。加密状态不等于安装失败，若后面的安装器要求解密，按[附录 B.4](#bitlocker-block)处理。

<a id="space"></a>
## 步骤 3：从 E 盘分出 300GB 给 Ubuntu

> 本步骤只用于首次安装。已经安装旧 Ubuntu 的情况，使用[附录 A](#reinstall)重用旧分区。

### 1. 打开磁盘管理

1. 按 **`Win + R`**。
2. 输入 **`diskmgmt.msc`**，按回车。
3. 最大化窗口，看下半部分的 **“磁盘 0”“磁盘 1”**和各分区。

![本次压缩前的磁盘布局](assets/windows-ubuntu-dual-boot/01-before-shrink.png)

*压缩前：C、D、E 都在内部磁盘 0；磁盘 1 是 Ventoy U 盘。*

本次 E 盘约 **476.93GB**，空闲约 **344.66GB**，因此选择从 E 盘分出 300GB。C 盘剩余空间少，本次不从 C 盘取空间。

### 2. 压缩 E 盘

1. 右键下半部分的 **E 盘分区**。
2. 选择 **“压缩卷”**，等待查询完成。
3. 确认“可用压缩空间大小”至少为 **307200MB**。
4. 在“输入压缩空间量”填写：

```text
307200
```

5. 点击 **“压缩”**，等待完成。

如果上限不足，先按[附录 B.5](#shrink-limit)处理，不通过删除卷解决。

### 3. 保留“未分配”空间

压缩完成后，E 盘右侧应出现黑色横条标识的 **300.00GB“未分配”**。

![本次压缩后的磁盘布局](assets/windows-ubuntu-dual-boot/02-after-shrink.png)

*压缩后：E 盘约 176.93GB、空闲约 44.67GB；右侧 300GB 未分配用于 Ubuntu。*

**保持“未分配”，不要新建简单卷，也不要格式化。** 原有的 EFI 和恢复分区保持不动。

**完成标志：**磁盘管理里明确出现了约 300GB 未分配空间。

<a id="ventoy"></a>
## 步骤 4：使用 Ventoy 制作启动 U 盘

### 1. 打开 Ventoy

1. 插入 U 盘。
2. 打开步骤 1 解压的 Ventoy 文件夹。
3. 双击 **`Ventoy2Disk.exe`**，权限提示按需点“是”。

### 2. 选择并初始化 U 盘

1. 在“设备”列表选中 U 盘。
2. 根据容量和型号再次核对：本次是约 64GB U 盘，**不能选约 1TB 的内部硬盘**。
3. 点击 **“安装”**，确认清除的是这个 U 盘，等待完成。

**已有可用 Ventoy 的 U 盘，跳过初始化，直接复制 ISO。**

### 3. 复制 Ubuntu ISO

1. 打开资源管理器中的 **Ventoy 盘**，本次盘符为 F。
2. 将 **`ubuntu-22.04.4-desktop-amd64.iso`** 复制进去。
3. 不解压，不改小的 `VTOYEFI` 分区。
4. 等复制完成，再安全弹出。

**完成标志：**Ventoy 数据分区里能看到完整的 22.04.4 ISO 文件。至此只完成启动盘准备，Ubuntu 还没有安装到硬盘。

<a id="boot"></a>
## 步骤 5：重启电脑，从 U 盘启动 Ubuntu

### 1. 进入启动选择页面

1. 重新插好 Ventoy U 盘，保存 Windows 中正在做的工作。
2. 按住 **Shift**，同时点击 **开始 → 电源 → 重启**。
3. 蓝色选项页面出现后松开 Shift。
4. 选择 **“使用设备”**。
5. 选择带有 **UEFI、USB 或 U 盘名称**的正确启动项。

没有该选项或不知道开机按键时，见[附录 B.6](#legacy)。本次保持 UEFI 和现有安全设置，不需要预先关闭 Secure Boot。

### 2. 在 Ventoy 中选择镜像

1. 用方向键选中：

```text
ubuntu-22.04.4-desktop-amd64.iso
```

2. 按回车。
3. 如果出现二级菜单，选择 **“Boot in normal mode”**，再按回车。

若出现蓝色 **Security Violation / MOK** 页面，按[附录 B.1](#secure-boot-error)处理，再回到这里。

### 3. 进入试用桌面

1. Ubuntu 启动菜单中选择 **“Try or Install Ubuntu”**。
2. 等待加载，在欢迎界面选择语言。
3. 点击 **“Try Ubuntu / 试用 Ubuntu”**。
4. 检查显示、键盘、触摸板和 Wi-Fi 能否使用。

**完成标志：**进入 Ubuntu 试用桌面，基本硬件可用。先不要拔 U 盘。

<a id="install-options"></a>
## 步骤 6：打开安装程序，填写基本选项

### 1. 启动安装程序

双击试用桌面的 **“Install Ubuntu 22.04.4 LTS”**。

### 2. 选择语言和键盘

1. 选择 **简体中文**或自己习惯的语言，点“继续”。
2. 根据实际键盘选择布局。
3. 在测试输入框输入几个字符，确认键盘正常，点“继续”。

### 3. 选择安装内容

1. 日常使用可选 **“正常安装”**。
2. 为先得到本次要求的 22.04.4 初始环境，安装阶段先不联网，**不勾选“安装 Ubuntu 时下载更新”**。
3. 第三方显卡/无线驱动按硬件需要处理；离线无法下载的可在安装后使用“附加驱动”安装。
4. 点“继续”，进入安装类型页面。

如果安装第三方驱动时要求设置 Secure Boot/MOK 密码，记录好该密码，之后可能需要登记驱动。

**完成标志：**到达“安装类型”或磁盘分区页面，尚未确认写入硬盘。

<a id="install"></a>
## 步骤 7：选择 Ubuntu 安装分区，开始安装

> 以下是**首次安装到预留空间**的操作。已安装旧 Ubuntu 时，改用[附录 A 的重装分区设置](#reinstall)，不要新建另一套系统。

### 1. 选择“其他选项”

在“安装类型”页面选择 **“其他选项 / Something else”**。

本次已在 Windows 预留空间，使用手动分区明确安装位置。**不要选择“擦除磁盘”，也不要点击“新建分区表”。** 如果安装器明确提示 BitLocker 阻止继续，先按[附录 B.4](#bitlocker-block)处理。

### 2. 在 300GB 空闲空间内建立 Ubuntu 分区

1. 找到内部约 **1TB** 的硬盘，本次设备名为 **`/dev/nvme0n1`**。
2. 找到步骤 3 预留的 **300GiB 左右空闲空间**。安装器可能显示约 322GB，这是计量单位不同。
3. 选中这块空闲，点击 **“+”**建立分区。
4. 使用这块空闲的可用容量；位置如需选择，可放在该空闲空间的起始处。
5. “用于”选择 **“Ext4 日志文件系统”**。
6. “挂载点”选择 **`/`**。
7. 点“确定”。

此方案使用一整个 Ubuntu 系统分区，个人文件也存放在其中；本次无需额外切出 `/home` 或 swap 分区。

### 3. 复用已有 EFI 引导分区

1. 找到内部硬盘已有的 **EFI 系统分区**；本次为 **`/dev/nvme0n1p1`，100MB，FAT/vfat**。
2. 确認它用于 **EFI 系统分区**。
3. **不勾选格式化，不删除它。** 它同时保存 Windows 的启动文件。
4. 如果界面提供挂载点，其用途对应 **`/boot/efi`**；有些界面直接显示“EFI 系统分区”，不用强找挂载点选项。

100MB 是本机已有容量，不是通用的新建规格。若提示 EFI 空间不足，先查看[附录 B.9](#partition-unexpected)。

### 4. 确认启动引导器位置

如底部有 **“安装启动引导器的设备”**，选择内部整盘 **`/dev/nvme0n1`**，不要选 Ventoy U 盘。

同时核对上一步实际使用的 EFI 分区确实在内部硬盘上，不能只看底部设备框。

### 5. 核对修改，确认安装

1. 检查 Windows 的 C/D/E、恢复分区均没有被删除或勾选格式化。
2. 点击 **“现在安装”**。
3. 阅读弹出的磁盘修改摘要，确认只在预留空间建立 Linux 文件系统，**没有格式化 Windows、恢复或 EFI 分区**。
4. 确认正确后点“继续”。

分区名称和本例不同、或摘要看不懂时，先取消确认，保存完整分区列表再核对。

### 6. 设置账户并等待

1. 时区选择 **Shanghai / 上海**。
2. 设置姓名、计算机名、用户名、登录密码。
3. 继续安装，保持电源和 U 盘连接。
4. 等待出现 **“安装完成”**提示。

**完成标志：**安装器明确提示安装完成，并提供“现在重启”。

<a id="verify"></a>
## 步骤 8：重启，分别检查 Ubuntu 和 Windows

### 1. 拔掉安装 U 盘

1. 点击 **“现在重启”**。
2. 看到以下提示后，拔掉 U 盘，再按回车：

```text
Please remove the installation medium, then press ENTER
```

3. 启动菜单出现时选择 **Ubuntu**。

### 2. 确认 Ubuntu 版本和安装位置

进入桌面后按 **`Ctrl + Alt + T`**，依次执行：

```bash
lsb_release -d
findmnt /
df -h /
```

- 离线按指定镜像安装后，应显示 **Ubuntu 22.04.4 LTS**。
- 根目录应来自内部 Linux 分区，而不是 U 盘试用环境的 `overlay`。
- 检查网络、显示、声音、键盘和触摸板。

### 3. 确认 Windows 可用

1. 正常重启。
2. 在启动菜单中选择 **Windows Boot Manager**。
3. 确认 Windows 可进入，C/D/E 的文件正常。
4. 如果要求 BitLocker 恢复密钥，按页面密钥 ID 匹配自己保存的记录并输入。

没有系统选择菜单时看[附录 B.7](#missing-ubuntu)；驱动密钥登记或更新后的版本变化见[附录 B.8](#updates)。

**完成标志：**拔掉 U 盘后，Ubuntu 和 Windows 都能独立启动。之后再开始配置开发环境。

---

## 文末补充：只在需要时查阅

- [附录 A：安装错版本后如何替换为 22.04.4](#reinstall)
- [附录 B：启动、BitLocker、分区和安装排错](#troubleshooting)
- [附录 C：内存、硬盘、ISO 等概念与常见疑问](#concepts)
- [附录 D：只读检查命令与写入前核对清单](#checks)
- [附录 E：官方资料与本次记录说明](#sources)

<a id="reinstall"></a>
## 附录 A：装错版本后，替换成 Ubuntu 22.04.4

### A.1 根据进度选择处理方式

- **只下载/复制错 ISO**：更换 ISO，硬盘系统不用动。
- **只进过 Try Ubuntu**：退出试用，换 ISO 重启。
- **已安装到硬盘**：用正确 ISO 重新安装，替换旧 Ubuntu；换 U 盘文件不会自动完成降级。

本次属于第三种。无须提前在 Windows 删除 Linux 分区，更不能删剩余 E 盘。

### A.2 换 ISO 后怎么进旧 Ubuntu

正常退出并安全弹出 U 盘，重启选内部 **ubuntu** 启动项。直接进 Windows 时可尝试 `Shift + 重启 → 使用设备 → ubuntu`，没有该项就查固件启动菜单。

进 Windows 不表示 Ubuntu 被删。暂时找不到旧启动项，也可以从新 ISO 的试用环境查看分区，不用先修好旧系统才重装。正在运行 U 盘试用系统时不要直接拔 U 盘。

### A.3 查看旧系统的准确分区

在硬盘已安装的 Ubuntu 中按 `Ctrl + Alt + T`，依次执行：

```bash
lsb_release -d
lsblk -o NAME,SIZE,FSTYPE,MOUNTPOINTS
df -h /
```

第一条查版本；第二条查磁盘；第三条查当前根分区。试用环境中的 `df -h /` 可能显示 overlay，不能当作旧系统根分区。

以下按用户终端照片转录，**不是本次整理文档时新执行的检测**：

```text
nvme0n1       953.9G
├─nvme0n1p1     100M  vfat       /boot/efi
├─nvme0n1p2      16M
├─nvme0n1p3    99.1G  BitLocker
├─nvme0n1p4     977M  ntfs
├─nvme0n1p5   375.9G  BitLocker
├─nvme0n1p6   176.9G  BitLocker
└─nvme0n1p7     300G  ext4       /

Filesystem          Size  Used  Avail  Use%  Mounted on
/dev/nvme0n1p7       295G   17G   263G    6%  /
```

因此确认：旧 Ubuntu 位于 **`/dev/nvme0n1p7`，300GiB、ext4，已用约 17GiB**。`df` 中较小的文件系统容量和剩余量涉及文件系统开销、保留空间，不是 E 盘被额外占用。

`loop0`、`loop1` 等 squashfs 通常是 Snap 软件挂载，不是需要逐个删掉的硬盘分区。

### A.4 替换安装镜像

回 Windows 或其他能复制文件的系统，插入 Ventoy：

1. 可删除旧的 `ubuntu-26.04.1-desktop-amd64.iso` 和不需要的 22.04.5 ISO。
2. 复制 `ubuntu-22.04.4-desktop-amd64.iso`，不解压。
3. 等复制完成、安全弹出。
4. 按正文步骤 5从正确镜像进入试用桌面。

Ventoy 不重装，E 盘不再压缩。删除旧 ISO 不会清除旧 Ubuntu。

### A.5 在正确安装器中重用旧分区

先备份旧 Ubuntu 内需要的文件。下面操作会清除 p7 内的旧系统、用户文件、软件和配置。

1. 在 **22.04.4 试用系统**启动安装程序；不要在正在运行的旧 Ubuntu 里格式化自己的根分区。
2. 到“安装类型”选 **其他选项 / Something else**。
3. 核对内部硬盘、分区名、容量、ext4 类型与记录一致。
4. 选 **`/dev/nvme0n1p7` → 更改**。
5. 用于选 **Ext4 日志文件系统**，挂载点 **`/`**。
6. 勾选 **格式化此分区**，不改容量。
7. 复用 **`/dev/nvme0n1p1`** 为 EFI 系统分区，**不格式化**；安装后对应 `/boot/efi`。
8. 如果显示启动引导器设备，选内部整盘 **`/dev/nvme0n1`**，不是 Ventoy。
9. 点“现在安装”后检查摘要，**只格式化已核实的 p7**，确认后写入。

本机需保留的分区：

- **p1**：100MB EFI 引导。安装器可更新 Ubuntu 引导文件，但不能清空/格式化整个分区。
- **p2**：16MB Windows 保留分区。
- **p3**：约 99.1GiB，Windows C 盘。
- **p4**：约 977MiB，Windows 恢复分区。
- **p5**：约 375.9GiB，Windows D 盘。
- **p6**：约 176.9GiB，Windows E 盘。
- **p7**：300GiB，旧 Ubuntu，**本次唯一重新格式化的分区**。

**编号只适用于本机截图中的布局。** 若容量、类型、设备名不同，先确认，不按编号硬套。不要选择“擦除磁盘”，也不要“与现有 Ubuntu 并排安装”，否则可能装出第三个系统。

若提示旧分区已挂载，先退出确认框、关闭访问它的文件管理器，并确认当前是 U 盘试用环境；不要强制卸载正在运行的根目录。若最终摘要显示格式化 EFI 或修改 Windows 分区，取消并重新检查。

本次采用干净重装，不采用“保留新版系统文件、不格式化”的跨版本回退方式，避免新旧系统和配置混杂。

确认写入后，设置时区和账户，等待安装结束。然后按正文[步骤 8](#verify)拔掉 U 盘、验证 22.04.4 和 Windows。无需回到首次安装的“新建分区”步骤。

<a id="troubleshooting"></a>
## 附录 B：操作中遇到差错时再看

<a id="secure-boot-error"></a>
### B.1 Secure Boot / Security Violation / MOK 蓝色页面

用户照片上显示：

```text
Verification failed: (0x1A) Security Violation
```

随后：

```text
Shim UEFI key management
Press any key to perform MOK management
```

这是启动签名验证问题，不能据此判断 Ubuntu 分区坏了。确认 U 盘来自自己准备的官方 Ventoy 后，常见登记步骤是：

1. 错误框选 **OK** 并回车。
2. MOK 倒计时结束前按回车进入菜单。
3. 选 **Enroll key from disk**。
4. 选 **VTOYEFI** 对应分区。
5. 选 **ENROLL_THIS_KEY_IN_MOKMANAGER.cer**。
6. 按页面依次 **Continue → Yes → Reboot**。
7. 重启后重新选择 U 盘。

这是让电脑信任 Ventoy 证书，不是 BitLocker 恢复密钥。证书可能随 Ventoy 版本更新，文件名/菜单不同时按对应版本官方说明核对。不能登记来源不明的证书；Ventoy 默认验证策略与直接启动 Ubuntu 官方介质也不相同，这是一次信任变更。[Ventoy Secure Boot 说明](https://www.ventoy.net/en/doc_secure.html)

Ubuntu 支持 Secure Boot，**禁用它不是固定必做步骤**。[Ubuntu 安全启动说明](https://documentation.ubuntu.com/security/security-features/platform-protections/secure-boot/) 登记仍失败时查 Ventoy 版本、兼容说明和具体错误；旧镜像签名撤销等问题也可能导致启动失败，不能保证所有 0x1A 都靠登记解决。不要清空安全启动密钥。若确需修改固件安全设置，先确保 Windows 恢复密钥可用。


<a id="bitlocker-status"></a>
### B.2 管理员窗口打不开，或者命令没有结果

**管理员入口找错了：**按 `Win + S` 搜索 `cmd`，右键的是搜索结果中的“命令提示符”，再选“以管理员身份运行”。黑色窗口标题栏的属性、设置页面中的终端选项都不是这个入口，不需要开启 sudo 或开发人员模式。

**只有标题、没有卷信息：**先等命令完成；若提示拒绝访问，重新以管理员身份打开。持续无输出也不能当作未加密。应看到每个卷的转换状态、加密百分比、保护状态。

查询成功后的本次结果：C、D、E 都是加密百分比 100%、保护已启用；F 盘 Ventoy 完全解密、0%、保护关闭。

![C、D 盘 BitLocker 状态](assets/windows-ubuntu-dual-boot/03-bitlocker-c-d.png)

*图 1：已成功以管理员身份查询。C、D 的“已解锁”不表示加密已关闭。*

![E 盘与 Ventoy U 盘加密状态](assets/windows-ubuntu-dual-boot/04-bitlocker-e-usb.png)

*图 2：E 盘加密，Ventoy U 盘未加密。截图拍摄于压缩 E 盘之前。*

理解三个区别：

- **已解锁**：当前 Windows 能读取该卷。
- **暂停保护/保护关闭**：不能仅凭此项断定数据已解密。
- **完全解密 + 0%**：该卷解密已完成。

备份恢复密钥也不会关闭加密。


<a id="bitlocker-crash"></a>
### B.3 搜索不到 BitLocker，管理入口闪退

本次在设置中找到了“隐私和安全性 → 设备加密”，其管理入口点击后闪退。仅凭此现象无法确定原因，不能据此判断加密坏了。

可以先通过 [微软账户恢复密钥页面](https://account.microsoft.com/devices/recoverykey) 保存恢复密钥；工作/学校管理的设备向所属组织查询。根据设备和密钥 ID 核对，C/D/E 可能有不同记录。

如果下一步确实需要解密，而相关设置仍不可用，应单独排查该管理问题，不用删除 Windows 分区解决。

<a id="bitlocker-block"></a>
### B.4 安装器要求关闭 BitLocker

如果安装器提示 BitLocker 阻止继续并排安装，应退出，回 Windows 关闭相关加密并等待解密完成，再重新安装。**不要通过“擦除磁盘”绕过提示。** [Ubuntu BitLocker 安装说明](https://ubuntu.com/desktop/docs/en/latest/reference/bitlocker-during-ubuntu-installation/)

对应卷的管理界面可提供“关闭 BitLocker”；“设备加密”也可能有总开关。关闭前确认影响哪些卷、恢复密钥已保存，以及 Windows 版本是否允许以后重新启用。关闭会降低相关数据在设备丢失时的保护，不应把所有卷解密写成无条件必做项。

解密开始后接电等待，再运行 `manage-bde -status`，确认相关卷“完全解密”且 0%。**暂停保护不能代替解密。** 若设置仍闪退、无法操作，应单独排查管理问题，不要因此清空 Windows。[Microsoft BitLocker 操作指南](https://learn.microsoft.com/en-us/windows/security/operating-system-security/data-protection/bitlocker/operations-guide)

本例最终 Linux 截图仍显示 Windows 分区为 BitLocker，Ubuntu 已在独立分区。这只证明本机当时状态，不代表任意安装器都允许相同流程。


<a id="shrink-limit"></a>
### B.5 E 盘空闲足够，却压缩不了 300GB

文件空闲容量不等于可压缩上限，不可移动文件等因素可能限制压缩。

1. 保留“压缩卷”窗口，记录可压缩量。
2. 可以在足够保留 Windows 日常空间的前提下，选择较小的 Ubuntu 容量，或重新规划其他数据分区。
3. 不删卷，不动 EFI/恢复分区，不反复强行操作。

窗口中的常用换算：100GB 填 102400MB；150GB 填 153600MB；200GB 填 204800MB；300GB 填 307200MB。

<a id="legacy"></a>
### B.6 没有“使用设备”，或不知道开机按什么键

**Windows 没有“使用设备”：**尝试“疑难解答 → 高级选项 → UEFI 固件设置 → 重启”，再在固件启动菜单选择 U 盘。也可使用厂商指定的开机启动菜单键。

**快捷键不确定：**查电脑具体型号的官方手册。F2、F12、Esc 等不是通用规则，BIOS 设置键与一次性启动菜单键也可能不同。

**`msinfo32` 显示“传统/Legacy”：**本指南正文按本机 UEFI 编写。先确认现有 Windows 的引导和分区方案，不能直接切换 UEFI、禁用 CSM 后照做。[Ubuntu UEFI 说明](https://help.ubuntu.com/community/UEFI)

**检查 U 盘是否真的以 UEFI 启动：**在 Ubuntu 试用终端执行：

```bash
test -d /sys/firmware/efi && echo UEFI || echo Legacy
```

本案例应输出 UEFI。若为 Legacy，重新选择 UEFI U 盘启动项。

<a id="missing-ubuntu"></a>
### B.7 重启直接进 Windows，找不到 Ubuntu

1. 如果目的是进入硬盘上已安装的系统，正常退出后安全拔掉安装 U 盘。
2. 尝试 `Shift + 重启 → 使用设备 → ubuntu`。
3. 没有 ubuntu 项时查看固件的一次性启动菜单。
4. 仍找不到时，用 U 盘试用环境运行 `lsblk` 查看布局，再排查引导，不先删分区重装。

只更换 U 盘 ISO 不会删除已安装系统。正常进入 Windows 只说明这次启动选择了 Windows，不能证明 Ubuntu 消失。

<a id="updates"></a>
### B.8 驱动登记、安装更新与版本变化

**驱动 MOK 登记：**若安装第三方驱动时设置过登记密码，重启可能出现 Enroll MOK 页面，按提示用该密码完成。它不同于 Ventoy 证书登记、Ubuntu 登录密码和 BitLocker 恢复密钥。未设置过或页面不一致时先核对提示，不猜密码。

**更新后不再显示 22.04.4：**正常的 22.04 软件更新可能使版本显示为较新的 22.04.x，与升级到 24.04/26.04 不同。不要为了保留版本字符串长期停用安全更新。

若课程严格依赖旧内核或软件版本，应另外记录 `uname -r` 和关键软件版本，按课程要求管理环境。仅凭系统版本字符串无法保证整个开发环境一致。

<a id="partition-unexpected"></a>
### B.9 分区、EFI 空间、挂载或硬件提示与正文不同

- **EFI 空间不足：**100MB 是本机已有布局，不是通用建议。先查占用和安装器要求，不能格式化 EFI 来腾空间。
- **出现 LVM、Linux 加密或不同容量/设备名：**不要套用本例 p7 的操作，先识别实际布局。
- **提示分区已挂载：**确认在 U 盘试用环境，关闭正在访问旧分区的文件管理器；不对正在运行的根目录强制卸载。
- **最终摘要要格式化 Windows/恢复/EFI：**取消确认，回分区页面重新检查。
- **试用环境无网络或显示异常：**先排查 22.04.4 对硬件的支持；重新格式化不会修复驱动缺失。

<a id="concepts"></a>
## 附录 C：看不懂的概念与常见疑问

### C.1 运行内存不是硬盘容量

- **RAM（运行内存）**是程序运行时使用的空间。本次 `free -h` 照片中总量约 30GiB、已用约 2.3GiB，是正常系统占用。双系统切换启动时不必把 RAM 永久分成两半。
- **硬盘空间**用于保存系统、软件和文件。本次给 Ubuntu 的 **300GB** 属于这一类。
- **swap（交换空间）**是内存辅助空间。照片中的 8GiB swap 不代表系统分区大小，也不要求新安装照建 8GB swap 分区。

查内存用 `free -h`；查当前 Ubuntu 系统分区容量用 `df -h /`；查磁盘布局用 `lsblk`。

### C.2 C、D、E 不一定是三块硬盘

本次 C/D/E 都位于同一块约 1TB 内部硬盘。Windows 显示“磁盘 0”，Linux 显示 `/dev/nvme0n1`。

从 E 盘压缩出的 300GB 是独立未分配空间。安装后 Ubuntu 使用自己的分区，**不在剩余 E 盘的某个文件夹里**。

### C.3 ISO、Ventoy、试用系统与已安装系统

ISO 是安装镜像；Ventoy 让 U 盘能够启动镜像。复制 ISO 只代表准备好了安装介质，**不代表完成安装**。

“Try Ubuntu”是从 U 盘启动的试用环境。安装到硬盘后，拔掉 U 盘仍能启动的才是已安装系统。删除 U 盘里的旧 ISO 不会卸载硬盘中的 Ubuntu；换 ISO 也不会自动降级旧系统。

<a id="faq"></a>
### C.4 本次常见问题

#### “ISO 已放 U 盘，是不是只剩配置环境？”

还差启动安装器、安装到内部硬盘、拔掉 U 盘验证。试用桌面不等于安装完成。

#### “只能从 C、D 分，不能 E？”

E 也可以。看空间与所在物理硬盘，不看盘符。E 如果在另一块硬盘，要另行规划引导位置。

#### “D/E 各分 200GB，会得到一个 400GB 吗？”

本例两块空间不相邻，普通分区不能直接合并。可分别挂载 `/`、`/home`，但容量分别计算。

#### “管理员终端在哪？终端属性里没有？”

在 Windows 搜索结果上右键“以管理员身份运行”。黑色窗口标题栏的属性不是管理员入口，设置里的 sudo 不用开。

#### “命令没结果是不是没加密？”

不是，等待完成或处理权限错误。一定以实际卷状态为准。“已解锁”也不是“完全解密”。

#### “管理 BitLocker 闪退？”

原因尚未确认。可先从微软账户保存恢复密钥；确需解密而设置不可用时单独排查，不清空 Windows。

#### “Secure Boot 报错，必须关闭吗？”

不是。本次已进入 Ventoy MOK 管理流程，可以按官方说明登记。仍失败则看具体兼容问题，不盲目清空密钥。

#### “换版本，直接删 E 盘吗？”

不。E 是 Windows 数据分区；旧 Ubuntu 在独立 p7。本例在正确安装器内只格式化 p7 来替换。

#### “free -h 用了 2.3GiB，要删掉吗？”

它是 RAM 的正常运行占用。磁盘用 `df -h /` 看，本次约 17GiB；原根分区格式化后旧内容才被清除。

#### “换了 U 盘 ISO，还能进原 Ubuntu？”

可以，U 盘文件与硬盘安装是两回事。正常退出并拔掉安装 U 盘，再选内部 ubuntu 启动项。

#### “重启直接进 Windows，Ubuntu 消失了？”

不一定，可能只是默认启动项。先查固件启动菜单和磁盘布局，再排查引导，不先删分区。

#### “Windows 磁盘管理不显示 Ubuntu 使用量？”

Windows 通常不能原生读 ext4。没有盘符或显示异常空闲比例不代表内容为空，使用 Linux 检查。

#### “300GB 为什么会变 322GB 或 295G？”

分区工具的十进制/二进制单位不同，文件系统还有开销和保留空间。结合分区名、位置、类型判断，不能只比数字。

<a id="checks"></a>
## 附录 D：只读检查命令与最终核对

**Windows 运行窗口（Win + R），每次一条：**

```text
msinfo32
```

```text
diskmgmt.msc
```

**Windows 管理员命令提示符：**

```bat
manage-bde -status
```

**Ubuntu 终端：**

```bash
lsb_release -d
uname -r
lsblk -o NAME,SIZE,FSTYPE,MOUNTPOINTS
findmnt /
df -h /
free -h
```

这些命令本身不格式化。本指南不提供可能误复制的删除分区命令，磁盘写入通过安装器确认。

### 下载和复制后的文件校验

在 Windows PowerShell 中，把路径替换成实际文件位置后执行：

```powershell
Get-FileHash -Algorithm SHA256 -LiteralPath 'D:\Downloads\ubuntu-22.04.4-desktop-amd64.iso'
```

与官方 [SHA256SUMS](https://old-releases.ubuntu.com/releases/22.04.4/SHA256SUMS) 中同一文件名的哈希比较。U 盘中的文件也可以按其路径再检查，确认复制完整。

### 最后一次写入前

- [ ] ISO 确为 `ubuntu-22.04.4-desktop-amd64.iso`。
- [ ] 安装 U 盘以 UEFI 启动。
- [ ] 恢复密钥在其他设备可读，需要的文件已备份。
- [ ] 确定当前属于首次安装还是替换旧版本。
- [ ] 已区分内部硬盘和 Ventoy U 盘。
- [ ] 本例重装仅格式化 p7，不格式化 Windows、恢复、EFI。
- [ ] 已读最终写入摘要，内容符合预期。
- [ ] 完成后拔 U 盘，分别验证 Ubuntu 和 Windows。

<a id="sources"></a>
## 附录 E：资料来源与本次记录说明

### 官方资料

- [Ubuntu 22.04.4 镜像归档](https://old-releases.ubuntu.com/releases/22.04.4/)
- [Ventoy 入门](https://www.ventoy.net/en/doc_start.html)
- [Ventoy Secure Boot 与证书登记](https://www.ventoy.net/en/doc_secure.html)
- [Ubuntu UEFI 社区说明](https://help.ubuntu.com/community/UEFI)
- [Ubuntu 安装时的 BitLocker 问题](https://ubuntu.com/desktop/docs/en/latest/reference/bitlocker-during-ubuntu-installation/)
- [Microsoft manage-bde](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/manage-bde)
- [Microsoft BitLocker 操作指南](https://learn.microsoft.com/en-us/windows/security/operating-system-security/data-protection/bitlocker/operations-guide)
- [Microsoft 设备加密](https://support.microsoft.com/en-us/windows/security/encryption/device-encryption-in-windows)

核对日期为 2026-09-07。当前官方网页可能针对新版安装器；本文目标为 22.04.4，具体按钮文字可能不同。

### 本次已经确认的状态

本文基于实际对话、用户截图和官方资料整理，日期为 2026-09-07。Windows 使用 UEFI，C/D/E 开启 BitLocker；从 E 盘分出 300GB；旧 Ubuntu 已安装在独立 ext4 分区并成功启动。

**尚未收到重装 22.04.4 成功的验收结果。** 附录 A 是基于已核实分区给出的替换方案，不将后续操作写成已经完成。

### 实际截图和记录边界

4 张配图均为本次操作的原始截图，已检查不含恢复密钥和微软账户邮箱。截图展示当时状态，不是所有电脑都应复制的布局。

部分手机照片的微信临时文件已不在原路径，因此未附原照片。Secure Boot 报错和旧 Ubuntu 分区结果按对话可见照片作文字转录，并明确标记，没有生成伪装成实测的截图。详见[图片目录说明](assets/windows-ubuntu-dual-boot/README.md)。
