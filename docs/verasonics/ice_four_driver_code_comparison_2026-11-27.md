# ICE 四版驱动逐行对比

2026-11-27。四份脚本均以同一 64 元软件探头定义为基础。B 和 C 把 Profile 1 电压请求写成 35 V，但仍用原版的 1 周期波形和分组间隔。D（ICE_AG）把波形改为 20 周期，并改用均匀 600 μs 间隔。D 将 35 V 写作上限，初始请求为 1.6 V。C 与 D 都是纯发射序列，其中 D 的接收对象没有被调度。以上是源码和 MATLAB 离线构建结论，探头额定、实际功率、声压、温升与固化均为 **unknown**。

## 1. 对照版本与引用规则

下文 `A:43` 表示 A 脚本第 43 行。行号按所列快照计算，包含注释与空行。B/C 使用 `ice_codex` 审计时的 `main` 快照 `778b0c95a22ff15681db7d9a93aac97f802cc8dc`。此前创建 B/C 的提交分别为 `428999d` 和 `ca8f4e2`。

| 代号 | 用户所指版本 | 源码与快照 |
|---|---|---|
| A | 原版驱动 | 本机厂商包 [XJD/SetUpP9_4TSICE_FlashAngles.m](../../VantageNXT-2.3.2-p2/XJD/SetUpP9_4TSICE_FlashAngles.m)，358 行。SHA-256 `f6486733742e376d5f3f8fbe848683789ef3e2f948123523e0e7ef8bcd3a3892`。该厂商包是本机忽略目录，GitHub 上不提供此文件。 |
| B | 高功率驱动，保留成像流程 | [SetUpP9_4TSICE_20mm_35V_offline.m](https://github.com/Zh9426/ice_codex/blob/778b0c95a22ff15681db7d9a93aac97f802cc8dc/verasonics/curing_mode/SetUpP9_4TSICE_20mm_35V_offline.m)，359 行。 |
| C | 单独照射高功率驱动 | [SetUpP9_4TSICE_TXOnly_20mm_35V_offline.m](https://github.com/Zh9426/ice_codex/blob/778b0c95a22ff15681db7d9a93aac97f802cc8dc/verasonics/curing_mode/SetUpP9_4TSICE_TXOnly_20mm_35V_offline.m)，43 行。 |
| D | ICE_AG 高功率驱动 | [SetUpP9_4TSICE_All64_HighPower_P1.m](https://github.com/Zh9426/ICE_AG/blob/88dc3fa2afb2cddc1cfb0f17f0c481b2a048a6b0/verasonics/curing_mode/SetUpP9_4TSICE_All64_HighPower_P1.m)，122 行。`ICE_AG/main` 快照 `88dc3fa2afb2cddc1cfb0f17f0c481b2a048a6b0`。 |

## 2. 探头、电压、波形与聚焦

| 核对项 | A 原版 | B 保留成像 | C 纯照射 | D ICE_AG |
|---|---|---|---|---|
| 系统资源和声速 | `128 TX / 128 RX`，`1540 m/s`，A:36–43 | 相同，B:36–43 | 相同，C:12–19 | 相同，D:20–27 |
| 探头对象 | `P9-4ULts`、`wavelengths`、`computeTrans`、`id=-1`，A:45–50 | 同一定义，B:45–51 | 同一定义，C:20–23 | 同一定义，D:29–34 |
| 电压上限 | `Trans.maxHighVoltage=35`，A:49 | `Trans.maxHighVoltage=35`，B:49 | `Trans.maxHighVoltage=35`，C:22 | `Trans.maxHighVoltage=35` 加 `TPC(1).maxHighVoltage=Trans.maxHighVoltage`，D:33、37 |
| 初始电压请求 | 全脚本未定义 `TPC.hv`。手册的 1.6 V 缺省值无法反推历史实验实设电压。A:45–50、288–291 | `TPC(1).hv=35`，B:50 | `TPC.hv=35`，C:24 | `TPC(1).hv=1.6`，D:36–38 |
| 发射波形 `[MHz,B,C,极性]` | `[6,.67,2,1]`，A:126–127 | `[6,.67,2,1]`，B:127–128 | `[6,.67,2,1]`，C:26–27 | `[6,1,40,1]`，由 D:14–16、67–68 生成 |
| 孔径与 TX 表项 | `Apod=ones(1,64)`，15 个 TX，A:29–34、129–140 | `Apod=ones(1,64)`，15 个相同 TX，B:29–34、130–141 | `Apod=ones(1,64)`，1 个 TX，C:28–31 | `Apod=ones(1,64)`，1 个 TX，D:71–77 |
| 焦点和角度 | `focus=-P.radius`，虚拟顶点发散波。实际请求角度 −22.5° 至 +67.5°，步长 90°/14。A:29–34、52–55、130–140 | `focus=20 mm/λ`，全部转向 0°，逐个调用 `computeTXDelays`。B:131–141 | `focus=20 mm/λ`、`Steer=[0,0]`，C:28–31 | `P.focusMm=20`、`P.steerDeg=0`，焦距和方位单独定义，D:12–13、71–77 |

四版的 `Trans` 在本轮构建中逐字段相同。公共依赖 `Utilities/computeTrans.m:1859–1877` 的执行值包含线阵 `type=0`、64 元、标称 6.5 MHz、`ConnectorES=1:64`、0.13 mm 间距和 0.11 mm 宽度。`connType=4`、阵元几何、50 Ω 和俯仰参数来自现有软件定义，探头实物接口、复阻抗和几何标定仍为 **unknown**。`1540 m/s` 是 **assumed** 介质声速。资源表中的 128 表示软件配置的通道数，64 表示本序列活动阵元数。

B/C/D 使用 `λ = (1540/1000)/6.5 ≈ 0.236923 mm` 将 20 mm 换为 `84.415584416 wavelengths`。构建后，B 的 15 项 `TX` 彼此相等，也逐项等于 C/D 的单项 `TX`。最大编程延时为 `0.269361040 μs`，由 `max(TX.Delay)/Trans.frequency` 换算。ICE_AG 说明中的约 3.5 μs 延时与该默认几何构建值不同。6.5 MHz 用于坐标和延时单位换算，`TW` 请求频率为 6 MHz。20 mm 是 **assumed** 设计目标，实测声压峰位尚未知。定位依据见 B:133、139–140，C:28–31，D:73–77 与 `Utilities/computeTXDelays.m:58–92、125–160`。

## 3. 成像对象、事件图与硬件入口

| 核对项 | A 原版 | B 保留成像 | C 纯照射 | D ICE_AG |
|---|---|---|---|---|
| 接收与图像对象 | 有 `PData/Media`，`RcvBuffer` 30 帧，`Receive` 450 项，有 `TGC/Recon/ReconInfo/Process/UI`。A:57–123、142–216、264–283 | 同一结构，B:58–124、143–217、265–284 | 仅构建 `Resource.Parameters/Trans/TW/TX/TPC/Event/SeqControl`，C:12–43。保留 `numRcvChannels` 系统字段。 | 保留 `PData`，1 帧 `RcvBuffer`（4096×128），1 项未调度 `Receive` 及 `TGC`。D:45–64、79–96。无 `Recon/ReconInfo/Process/Media/UI`。 |
| 一个事件表 | 450 TX+RX、30 重建/显示、1 跳转，共 **481**。30 次 `transferToHost`，每 3 帧 `returnToMatlab`。A:218–261 | 与 A 的 `Event/SeqControl` 逐项相同，B:219–262 | **450** 纯 TX + 1 `stop`，共 **451**。`rcv/recon/process=0`。C:33–43 | **500** 纯 TX + 1 `stop`，共 **501**。`rcv/recon/process=0`。D:98–116 |
| 发射间隔 | 每组 15 发、30 组。前 14 个间隔 400 μs，组末 14,400 μs，名义组周期 20 ms。A:221–224、229–242 | 相同，B:222–225、230–243 | 相同请求间隔，去掉采集后的传输与处理。最后一发仍引用 14,400 μs 的 `timeToNextAcq`，后接 `stop`。C:35–43 | 连续 499 个 600 μs 发射间隔。第 500 发 `seqControl=0`，随后 `stop`。D:17–18、99–116 |
| 序列末尾 | `jump` 回事件 1，A:219–220、256–261 | `jump` 回事件 1，B:220–221、257–262 | `stop`，C:35–36、42–43 | `stop`，D:101–102、110–116 |
| 启动与产物 | `simulateMode=0`，保存原 MAT 后调用 `VSX`。A:43、288–291 | `simulateMode=1`，保存独立 MAT，`VSX` 已注释。B:43、288–292 | `simulateMode=1`，在工作区构建，无 MAT 保存和 VSX 调用。C:1–9、19、35–43 | `simulateMode=1`，保存独立 MAT，`VSX` 已注释。D:27、118–122 |

**A → B 的差异。** 逐行 diff 仅涉及 A/B:21 的注释空格、A/B:43 仿真模式、A/B:49 的注释及 B:50 电压字段、A:132/B:133 焦距、A:138/B:139 转向、A:288–290/B:289–291 输出与入口。原接收与事件图保留。B:187–192 仍按多个角度进行 IQ 累积，但实际累积的是同一 TX 配置产生的回波，原有多角度复合的发射角度多样性已经消失。

**B → C 的差异。** 单个 TX 取代 15 个相同 TX。C:35–43 只保留发射、计时与停止。删除的是接收、DMA、重建和显示的事件及相关对象，而发射请求间隔与单发波形仍相同。事件图更小，实际硬件吞吐是否变化未验证。C 本身不清空 base 工作区，已有旧成像变量的会话可能保留旧变量，因此构建验证使用干净工作区。

**C → D 的差异。** 两者共用同一默认焦点和延时。D 改变单发周期数、半周期有效比例、发射间隔和发射次数。D:45–64、79–96 的占位对象是源码作者为加载器兼容性保留的设计。D:104–116 证明它们不触发任何采集，真实 VSX 兼容性仍未验证。去掉接收事件也不能证明物理接收链已经隔离。按 D00163 Rev. K p. A107，`stop` 是冻结序列，可由解除冻结重新开始，所以 C/D 表内的发数是一次执行的发数。

## 4. 量化比较与功率口径

`TW.Parameters` 的 B 是一个半周期内的有效比例，C 是半周期数。按 `t_burst=C/(2f)` 计算名义包络，按 `t_burst/PRI` 或 `N·t_burst/T` 计算包络占空比。下表暂不计均衡脉冲、时钟量化、实际事件开销及放电压降。波形编译后的 `TW.Bdur` 和时间轨迹本轮没有读到。

| 指标 | A/B/C | D ICE_AG | 推导依据 |
|---|---:|---:|---|
| 完整周期 | 1 | 20 | A:127、B:128、C:27、D:14–16、68 |
| 单发名义包络 | 0.16667 μs | 3.33333 μs | 1/6 和 20/6 μs |
| 局部包络占空比 | 0.04167% | 0.55556% | 原组内 `0.16667/400`，D 为 `3.33333/600` |
| 按名义比较窗口的包络占空比 | 0.01250% | 0.55556% | A/B/C 以 `30×20 ms=600 ms` 为窗口，D 以 `500×600 μs=300 ms` 为窗口 |
| 单表累计名义发射包络 | 75 μs | 1666.67 μs | 450 或 500 次乘以各自包络 |
| 首发至末发请求间隔之和 | 585.6 ms | 299.4 ms | A/B/C 为 `29×20 ms+14×400 μs`，D 为 `499×600 μs` |

在这一比较口径下，D 的**名义平均包络占比约为 A/B/C 的 44.44 倍**，单表包络时间约为 22.22 倍。这两个比值仅描述时序。A/B 的表末可循环，C/D 表尾停止。名义窗口为比较而定义，实际耗时需由设备时序确认。B/C/D 的编程延时相同，但估计声压仍需实际发射波形、电压和负载数据。

ICE_AG 源码 D:17 的约 7.9–8.3 W 来自该仓库说明给出的 **35 V 假设工况、50 Ω 负载及理想电路估算**。D:38 的实际脚本初始请求为 1.6 V。`B=1` 表示半周期有效比例满刻度，脉冲串包络占空比仍只有约 0.556%。D:3、16、66 的 `sysExtendBL=0` 是依据短波形选用 Profile 1 的设计判断，脚本中没有显式赋值。原 ICE_AG 测试在未激活 Java HAL 时跳过 `computeTWWaveform` 的返回码、`sysExtendBL` 和 `fluxHVlimit` 检查。本轮隔离 MATLAB 的 HAL 也不可见。因此不能从 20 周期、理论供电功率或 35 V 上限推出 VSX 可加载、探头安全或 20 mm 处达到 MPa 级。证据见 [ICE_AG 原说明](https://github.com/Zh9426/ICE_AG/blob/88dc3fa2afb2cddc1cfb0f17f0c481b2a048a6b0/docs/verasonics/all64_high_power_driver_2026-11-23.md) 和 [ICE_AG 测试第 47–61 行](https://github.com/Zh9426/ICE_AG/blob/88dc3fa2afb2cddc1cfb0f17f0c481b2a048a6b0/tests/test_all64_high_power_p1.m#L47-L61)。

## 5. 手册依据与实际核验范围

指定 F 盘本机不可访问。本轮核对了本机先前提取的四份指定修订版文本（位于被忽略的 `tmp/pdfs/`，跨电脑可能缺失）。随软件包的产品规格书与 D00108 Rev. L 历史 SHA-256 一致。另三份随包 PDF 是 D00109 Rev. E、D00107 Rev. L 和 D00163 Rev. L，以下指定版章节与页码来自 Rev. D/K 的本机文本。

| 基础手册 | 核对位置和用途 |
|---|---|
| D00108 Rev. L，Product Specification | §1.4 pp. 8–9、§3.1.2–3.1.8 pp. 15–17、§3.2.1 p. 19。系统发射保护与探头/实际声输出是不同约束。解释载荷条件、峰值与 RMS 电流，以及 Profile 5 的配置限制。 |
| D00109 Rev. D，Initial Setup Instructions | §4.5 pp. 9–10。激活后的系统自检及日志流程。本轮没有执行自检。 |
| D00107 Rev. K，System User Manual | §4.2 p. 29、§4.4 pp. 29–30。温度监测需要探头与系统支持，Profile 1–4 属于成像供电，约 10 W 是系统量级描述。Extended Transmit 是独立选件，四份脚本均未调度 Profile 5。 |
| D00163 Rev. K，Sequence Programming Manual | §4.3 pp. 23–25、§12 pp. 95–98、§13.1 pp. 110–111、pp. A107、A109–A110、A154。核对波形与量化、TPC 初值及限制、时序命令和映射。补查厂商随包 `HelpText/TW/sysExtendBL.pdf` 与 `HelpText/TW/Bdur.pdf` 第 1 页。 |

本轮实际用独立 MATLAB R2024b 进程构建了四套结构。A 的临时审计副本强制仿真并移除自动 VSX 和原保存语句。B 的临时副本只省略保存语句。C 直接构建，D 将 MAT 保存到临时目录并重新读回。所有断言通过，核对了四份 `Trans`、A/B/C 的 `TW`、D 的 20 周期波形、B/C/D 的 `TX`、A/B 的接收及事件表相等性、C/D 零成像事件和表尾 `stop`。原始 A 与另外三份源文件均未改动。本轮未执行 VSX、固件/硬件、实际发射、声热传播或材料固化实验。脚本注释及 ICE_AG 的历史许可记录不能代表本次确认了实验室设备的当前配置。
