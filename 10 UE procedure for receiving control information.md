# 🔖 章节：10 UE procedure for receiving control information *(终端接收控制信息的流程)*

---

## 📖 章节导读：5G 调度的“中枢神经”与“盲检游戏”

**背景信息**：
欢迎来到 3GPP 38.213 协议中最核心、最庞大的章节之一——**第 10 章**。

在 5G 网络中，基站和手机之间每秒钟都在进行成千上万次的数据收发。但手机怎么知道什么时候该收数据？什么时候该发数据？该用什么频率？该用多大功率？
所有的这一切指挥信息，都写在下行控制信息（**DCI**, Downlink Control Information）里。而 DCI 是通过物理下行控制信道（**PDCCH**）发送的。

第 10 章，就是一本写给手机的**《看黑板指南》**。它规定了手机每天日常的“寻宝游戏”规则：
1. **去哪找黑板？（CORESET 控制资源集）**：基站不是在所有频率上都发通知的，只会划定几块特定区域。
2. **什么时候看？怎么找？（Search Space 搜索空间）**：手机不能24小时死盯着，那样太费电。它要在特定的时间点，去特定的位置去“盲猜（Blind Decoding）”有没有自己的通知。
3. **通知长什么样？（DCI Formats）**：不同的任务（比如调度下行数据、调度上行数据、系统广播）对应不同格式的通知单。

### 📚 基础术语预热 (本章核心骨架)
*   **PDCCH (Physical Downlink Control Channel)**：物理下行控制信道。**一句话解释**：基站用来发布各种调度命令的“公共大黑板”。
*   **DCI (Downlink Control Information)**：下行控制信息。**一句话解释**：贴在黑板上的“具体通知单”，每张单子上写着分配给某个手机的具体任务。
*   **CORESET (Control Resource Set)**：控制资源集。**一句话解释**：黑板在物理频率和时间上的具体“边框大小”。
*   **Search Space**：搜索空间。**一句话解释**：手机去看黑板的“排班表”和“注意力聚焦范围”。分为公共搜索空间（CSS，看大喇叭广播）和终端专属搜索空间（USS，看私人信件）。
*   **Blind Detection / Decoding**：盲检 / 盲解码。**一句话解释**：因为基站不会提前告诉你通知贴在黑板的哪个确切角落，手机只能拿着自己的密码（RNTI）把黑板上的纸条一张张撕下来试着解密，解开了就是自己的。

---

## 📖 协议原文拆解 (一)：双连接 (DC) 下的“两套班子”与“名词替换”

> **协议原文**
> If the UE is configured with a SCG, the UE shall apply the procedures described in this clause for both MCG and SCG except for PDCCH monitoring in Type0/0A/0B/2/2A -PDCCH CSS sets where the UE is not required to apply the procedures in this clause for the SCG
> - When the procedures are applied for MCG, the terms 'secondary cell', 'secondary cells' , 'serving cell', 'serving cells' in this clause refer to secondary cell, secondary cells, serving cell, serving cells belonging to the MCG respectively.
> - When the procedures are applied for SCG, the terms 'secondary cell', 'secondary cells', 'serving cell', 'serving cells' in this clause refer to secondary cell, secondary cells (not including PSCell), serving cell, serving cells belonging to the SCG respectively. The term 'primary cell' in this clause refers to the PSCell of the SCG.

### 📚 增量术语表
* **MCG / SCG (Master/Secondary Cell Group)**：主小区组 / 辅小区组。**一句话解释**：5G 的“双连接”技术（手机同时连两个基站提升网速）。MCG 就像是你的“全职主业”（管核心控制），SCG 就像是你的“兼职副业”（纯为了多赚钱/提网速）。
* **PSCell (Primary SCG Cell)**：辅小区组主小区。**一句话解释**：兼职公司里的“部门经理”。虽然是副业，但也得有个带头人来管这个辅小区组。
* **Type0/0A/0B/2/2A -PDCCH CSS**：特定类型的公共搜索空间。**一句话解释**：基站用来发布全网大喇叭广播（如系统消息 SIB、寻呼 Paging 叫醒服务）的公告栏。

### 🚪 第一关：两套班子的“平行管理”
当手机开启了双连接（同时连着主基站 MCG 和辅基站 SCG），手机大脑里需要运行两套独立的调度管理系统。
协议规定：本第 10 章里写的所有关于“怎么看黑板收通知”的规矩，手机必须**在 MCG 和 SCG 里各自照做一遍**，两边互不干涉。

### 🎯 第二关：兼职公司不管“闲事” (CSS 豁免)
虽然两边规矩一样，但有个重要的**例外（Except）**：
手机**不需要**在辅小区组（SCG）里去监听 `Type0/0A/0B/2/2A` 这些公共公告栏。
**为什么？（物理意义）**：
这些特定的公告栏是用来发“全网系统广播”和“电话呼入唤醒（寻呼）”的。在双连接架构中，主基站（MCG）负责所有的核心信令和控制面工作，辅基站（SCG）是个纯粹的“数据打工人”。所以，听广播、接电话这种“行政类闲事”，手机只听主基站的就行了，辅基站只管埋头传数据，不用去看这些公告。

### 🧩 第三关：协议名词的“动态角色扮演” (Context 隔离)
3GPP 协议的条文是通用的，为了不在每一句话里都啰嗦地写“MCG的主小区”或“SCG的主小区”，协议在这里定义了一个**“语境替换法则”**：
当你在读本章协议时：
* **带入 MCG 语境**：所有的“辅小区、服务小区”，指的都是主业公司里的部门。
* **带入 SCG 语境**：所有的“辅小区、服务小区”，指的都是兼职公司里的部门。**最关键的是：如果在协议里看到“主小区 (Primary cell)”这个词，在 SCG 语境下，你要自动在脑子里把它替换成“辅基站的带头大哥 (PSCell)”。**

### 🔄 双连接下的规章制度映射图
```text
【第 10 章：下行控制信息接收规章制度】
                   │
    手机判断：是否配置了 SCG (开启双连接)？
                   │
      ┌────────────┴────────────┐
      ▼                         ▼
【应用于 MCG (主业公司)】   【应用于 SCG (兼职公司)】
      │                         │
      ├─ 监听所有配置的 DCI      ├─ 监听大部分 DCI
      │                         │
      ├─ 听全网广播和寻呼        ├─ ⛔ 豁免！不用听 Type0/0A/0B/2/2A 这些系统广播/寻呼
      │  (Type 0/0A/2 等 CSS)   │
      │                         │
      └─ 协议里的 Primary Cell  └─ 协议里的 Primary Cell 
         就是绝对的主小区 (PCell)     自动替换翻译为 PSCell (辅组主小区)
```

### 💡 章节硬核总结

**本段协议确立了 Chapter 10 在 Dual Connectivity (EN-DC, NR-DC 等) 架构下的宏观适用域（Scope）与上下文解析规则（Context Mapping）。在控制面层面，协议定义了 MAC 实体与 PHY 实体在 MCG 与 SCG 的实例平行性：PDCCH 盲检流程在两个 Cell Group 中独立并发执行。在业务面层面，协议明确了 SCG 纯粹的用户面 (User-plane) 定位，通过显式豁免（Exempt）UE 在 SCG 监听系统信息（SIB1/OSI）和寻呼（Paging）相关的 CSS 搜索空间，极大地降低了 UE 的盲检复杂度与功耗。最后，通过对象重载（Overloading）声明，统一了后续行文中 Cell Index 的语境逻辑，使得同一套 L1 调度算法可以无缝复用于 PCell 与 PSCell。**

## 📖 协议原文拆解 (二)：看黑板的总纲领——“寻宝游戏”的三个套娃

> **协议原文**
> A UE monitors a set of PDCCH candidates in one or more CORESETs on the active DL BWP on each activated serving cell configured with PDCCH monitoring according to corresponding search space sets where monitoring implies receiving each PDCCH candidate and decoding according to the monitored DCI formats.

### 📚 增量术语表
* **PDCCH candidates**：PDCCH 候选者。**一句话解释**：黑板上贴着的一张张“可疑的小纸条”。手机不确定哪张是给自己的，只能一张张拿下来试。
* **Active DL BWP**：激活的下行部分带宽。**一句话解释**：手机当前真正在工作的一段下行频率范围（不需要整个大宽带全开，省电）。
* **Monitor (监听)**：**一句话解释**：物理层的一个复合动作，包含了“把纸条拿下来（接收）”和“用自己的密码试着解开（解码）”两个步骤。

### 🚪 第一关：找黑板的“四层套娃”定位法
手机每天要看基站发来的调度命令，但不能瞎看，必须按照层层递进的坐标去寻找。这段话给出了一个清晰的**“四层套娃”物理寻址逻辑**：

* **第一层（哪栋楼？）**：`activated serving cell`（已激活的服务小区）。基站可能给你分了好几个载波（辅小区），你得在处于“激活”状态的那个载波上去找。
* **第二层（哪层楼？）**：`active DL BWP`（激活的下行带宽）。在那个载波里，你只看当前正在工作的那个小频段（BWP）。
* **第三层（哪面墙？）**：`CORESETs`（控制资源集）。在那个小频段里，基站只在几块特定的“墙面”上挂黑板，这些墙面就是 CORESET。
* **第四层（什么时候看？）**：`Search space sets`（搜索空间集）。光知道墙在哪还不行，你还得按照基站给的“排班表”（搜索空间），在指定的时间点去瞄一眼。

### 🎯 第二关：什么叫“Monitor (监听)”？
普通人理解的监听是“用耳朵听”，但 3GPP 对 `Monitor` 下了极其严格的数学定义。
在 5G 物理层，Monitor 是一个包含试错成本的暴力运算过程：
1. **Receiving (接收提取)**：按照规则，把黑板上的某一个“可疑纸条 (PDCCH candidate)”的电磁波信号提取出来，变成数字比特。
2. **Decoding (盲解码)**：拿着自己关心的那几种**通知单格式 (monitored DCI formats)**，套用自己的私有密码 (RNTI)，尝试去解密这串比特。如果解开后 CRC 校验通过，恭喜你中奖了，这张纸条就是发给你的指令！如果解不开，就扔掉换下一张纸条继续试。

### 🔄 PDCCH 盲检空间拓扑图
```text
【一层层缩小寻找范围】

 🌍 1. 激活的服务小区 (Activated Serving Cell)
   │
   └──► 📶 2. 当前激活的频段 (Active DL BWP)
         │
         ├──► 🟩 3. 控制资源集 (CORESET 0) ──► 墙面物理大小 (比如 24个PRB宽，2个符号长)
         │       │
         │       └─► 🕒 4. 搜索空间 (Search Space #1) ──► 排班表 (比如每隔 10个时隙看一次)
         │               │
         │               └─► 📝 盲检候选者 (PDCCH Candidates)
         │                   试纸条1 ─(用 DCI 1_0 解码)─► ❌失败
         │                   试纸条2 ─(用 DCI 1_0 解码)─► ✅成功！提取调度指令！
         │
         └──► 🟦 3. 控制资源集 (CORESET 1) ──► 另一块墙面...
```

### 💡 章节硬核总结

**本段协议是整个第 10 章 L1 调度接收的“总纲领 (General Principle)”。它用一句话严谨地勾勒出了 PDCCH 盲检 (Blind Decoding) 的多维拓扑边界：跨载波域 (`serving cell`) $\rightarrow$ 带宽域 (`active DL BWP`) $\rightarrow$ 物理资源域 (`CORESET`) $\rightarrow$ 时域行为 (`search space`)。同时，明确给出了 `Monitor` 这一核心动词的物理层 L1 算子定义：即从时频网格中提取特定的 CCE (Control Channel Element) 聚合实体（Candidate），并针对预期的 DCI 负荷大小（Payload sizes）与对应的 RNTI 执行基于 Viterbi/Polar 算法的 CRC 校验解码过程。这一盲检机制既是 5G 调度灵活性的源泉，也是终端基带算力功耗消耗的绝对大头。**

## 📖 协议原文拆解 (三)：把两张碎纸条拼成一张大通知 (PDCCH 缝合术)

> **协议原文**
> In the remaining of this clause, when a PDCCH reception by a UE includes two PDCCH candidates from corresponding search space sets, as described in clause 10.1
> - a PDCCH monitoring occasion is the union of the PDCCH monitoring occasions for the two PDCCH candidates
> - the start of the PDCCH reception is the start of the earlier PDCCH candidate
> - the end of the PDCCH reception is the end of the PDCCH candidate that ends later

### 📚 增量术语表
* **Two PDCCH candidates from corresponding search space sets**：两个候选纸条。**一句话解释**：这是 5G 后期版本（比如多发射点 mTRP 场景）引入的高级玩法。基站怕一张纸条传丢了，把同一个调度通知抄了两遍，分别贴在不同的黑板（CORESET / Search Space）上。手机需要把这两张纸条“拼起来（或联合解码）”增加成功率。
* **PDCCH monitoring occasion**：PDCCH 监听时刻。**一句话解释**：手机真正集中注意力去看黑板的那一小段连续的时间。

### 🚪 第一关：“双保险”模式下的时间跨度合并
以前，基站一次只发一个通知，手机在一个时刻看完就结束了。
现在，基站开启了“双保险”模式，把同一个通知通过两条路径（两个不同的候选者）发给手机。这两个纸条可能不是完全同时到达的，有的贴在第一块黑板的早上，有的贴在第二块黑板的下午。
**协议在这里规定了这种“缝合怪”的时间界定法则：**
当手机要把这两张纸条当成一个整体（一次 PDCCH 接收）来对待时，它的“监听时刻”怎么算？
* **答案是取并集 (Union)**：这两张纸条占据的所有时间段，都属于这次监听任务。

### 🎯 第二关：重新定义“开头”和“结尾”
既然是两张可能存在时间差的纸条，那么这整个接收过程的“起点”和“终点”该怎么界定呢？（这非常重要，因为后面算各种 $N_{gap}$ 等待时间都要用到终点锚点）。
协议给出了最符合物理逻辑的极限框选法：
* **起点界定**：以两张纸条中**最先到达（earlier）**的那张纸条的起点为准。
* **终点界定**：以两张纸条中**最后结束（ends later）**的那张纸条的终点为准。

**生活类比**：
你在等一个被分成两包寄出的“双拼快递”。
快递 A 早上 9点 到，9点半 卸完货；
快递 B 早上 10点 到，10点半 卸完货。
你整个“收快递任务”的时间跨度怎么算？
* 起点：早上 9点（最先到的时间）。
* 终点：早上 10点半（最后卸完的时间）。
从 9点 到 10点半，这整个大区间，在协议里被视为“一次完整的 PDCCH Reception”。

### 🔄 两个 PDCCH Candidate 的时间框选图解
```text
【双路 PDCCH 联合接收的时间轴定义】

频段 A:  [纸条 1 (Candidate A)]
         │◄──── 时长 ────►│
         │                │
频段 B:  │           [纸条 2 (Candidate B)]
         │           │◄──── 时长 ────►│
         │           │                │
         ▼           ▼                ▼
时间 ────┴───────────┼────────────────┴───────────►
      (9:00)       (9:30)           (10:30)

【合成后的单个逻辑接收动作】
起点 (Start) = 9:00 (以最早开始的纸条1为准)
终点 (End)   = 10:30 (以最晚结束的纸条2为准)
联合监听时刻 = 纸条1 和 纸条2 占据的所有时间块 (Union)
```

### 💡 章节硬核总结

**本段协议在 L1 时间轴上，为多站协同/多路分集（如 Rel-16/17 引入的 PDCCH repetition 或 mTRP 基于多 Search Space 的软合并）接收提供了一致性的物理边界定义。当 UE 执行多 PDCCH Candidate 的联合盲检（Joint decoding / Soft combining）时，这两个物理实体在逻辑上被收敛为一个统一的宏观 PDCCH 接收事件。通过取时间域的并集（Union），并应用 Min(Start_time) 和 Max(End_time) 的极值包络法则，协议优雅地消除了后续调度时序（如 PDSCH 准备时间 $N_{T,1}$、PUCCH 反馈时间 $N_{T,2}$ 以及各种 Gap 的起点计算）在面临多径延迟或时域跨度不一致时可能产生的锚点歧义。**

## 📖 协议原文拆解 (四)：就算“漏看了一半”，时间账本也不能乱 (异常情况界定)

> **协议原文**
> The PDCCH reception includes the two PDCCH candidates also when the UE is not required to monitor one of the two PDCCH candidates as described in clauses 10 (except clause 10.4), 11.1, 11.1.1 and 17.2.

### 📚 增量术语表
* **Not required to monitor**：不被要求监听（免听豁免）。**一句话解释**：协议里规定的某些特殊情况，手机允许“偷懒”，对原本应该看的某些纸条（PDCCH candidate）视而不见。例如：算力不够了（超过了最大盲检次数）、遇到 TDD 半双工红绿灯冲突等。

### 🚪 第一关：双份文件，只看了一份怎么办？
上一段规定了，基站给你发了“双保险”的两张纸条（两个 PDCCH candidates），你要把它们的时间跨度“合并包络”起来，算作一个大事件的起点和终点。
但这产生了一个新问题：
如果因为某些原因（比如刚好那段时间基站在发大广播 SSB 冲突了，或者手机处理能力爆表了），按照其他协议条款的规定，手机被允许**只看其中一张纸条，不看另一张纸条**，那这个时候，前面那个“时间合并框选”的规矩还算数吗？

### 🎯 第二关：时间包络框的“绝对刚性”
协议给出了斩钉截铁的答案：**即使你因为种种合法原因，只看了其中一张纸条（也就是 `not required to monitor one of the two`），你在算这整个事件的时间起点和终点时，依然必须把没看的那张纸条也算进去！**

**为什么要这么不讲理？（物理意义）**
这是为了保证基站和手机对“时间锚点”的理解保持绝对一致（Alignment）。
基站是按照“双份发送”的时间底线去计算你该什么时候回复的。如果手机因为私自少看了一份，就擅自把整个接收任务的“终点”提前了，然后提前给基站发数据，基站就会觉得你在抢跑，导致数据接收失败。
因此，不管你实操中“看没看全”，在排时间表算账的时候，必须假装两者都在，维持原来的包络边界不变。

**生活类比**：
老板（基站）通知你：“明早 9点到 10点开会（纸条1），10点半到 11点还有个补充会（纸条2）。不管你开哪个，会后半小时把报告交给我！”
正常情况：你开完 11点的会，11点半交报告。
特殊情况：你有特批假条，10点半的补充会你不用参加（not required to monitor）。那你可以 10点半就提前交报告吗？
**不行！** 老板的时间轴是锁死的，就算你没参加后半场，你也必须等到 11点（原本两场会最晚结束的时间）之后，也就是 11点半再交报告。因为老板是卡着这个总时钟在等你的。

### 🔄 缺席状态下的时间轴刚性法则
```text
【双保险 PDCCH 接收，其中一个被合法豁免】

频段 A:  [纸条 1 (正常监听)] ✅
         │◄──── 时长 ────►│
         │                │
频段 B:  │           [纸条 2 (因冲突/超载被合法跳过，没看)] ❌
         │           │◄──── 时长 ────►│
         │           │                │
         ▼           ▼                ▼
时间 ────┴───────────┼────────────────┴───────────►
      (9:00)       (9:30)           (10:30)

【计算时间边界时，没看的部分也要计入！】
起点 (Start) = 9:00 (纸条1起点)
终点 (End)   = 10:30 (尽管纸条2没看，终点依然必须以纸条2为准！)
(基站和手机都使用这个刚性的 10:30 作为后续排期的锚点)
```

### 💡 章节硬核总结

**本段协议是多重 PDCCH 盲检机制（如 PDCCH Repetition）中的一项极为关键的“防脱轨（Failsafe）”时序约束设计。当引入多重候选者联合传输时，终端可能因为满足特定条款（如超越 Overbooking limit，或是与 SSB/RateMatching pattern 时域冲突）而发生物理上的丢弃监听（Dropping）。但此时，基站侧并不知道（或为了调度简化不考虑）UE 丢弃了哪一个分支。为了消除 L1 层下行接收终点模糊性，协议强行将物理的“接收操作行为”与逻辑的“时间轴包络计算”解耦：不论 UE 是否实际物理盲检了晚到达的那个 PDCCH Candidate，只要它被高层配置关联在同一个联合接收组内，该次整体 PDCCH Reception 的宏观终结边界（End boundary）就不可篡改地锚定在两者中绝对时域靠后的那一个。这是维护 DCI 到 PDSCH/PUCCH 严格时序（K0/K1/K2 delay）准确性的基石。**

## 📖 协议原文拆解 (五)：手机算力的天花板——“最多能看多少黑板？”

> **协议原文**
> If a UE is provided monitoringCapabilityConfig for a serving cell, the UE obtains an indication to monitor PDCCH on the active DL BWP of the serving cell for a maximum number of PDCCH candidates and non-overlapping CCEs
> - per slot, as in Tables 10.1-2 and 10.1-3, if monitoringCapabilityConfig = r15monitoringcapability, or
> - per span, as in Tables 10.1-2A and 10.1-3A, if monitoringCapabilityConfig = r16monitoringcapability, or
> - per group of $X_s$ slots according to combination ($X_s$,$Y_s$), as in Tables 10.1-2B and 10.1-3B, if monitoringCapabilityConfig = r17monitoringcapability

### 📚 增量术语表
* **Maximum number of PDCCH candidates / non-overlapping CCEs**：最大盲检次数 / 最大不重叠 CCE 数量。**一句话解释**：手机解码芯片的“算力天花板”。基站不能在黑板上贴一万张纸条让手机去猜，手机算不过来会死机，必须有个最高上限。
* **monitoringCapabilityConfig**：监听能力配置。**一句话解释**：代表手机按哪种“时代的规矩”来结算算力。分为 R15（基础版）、R16（超低延迟加强版）和 R17（极简低功耗版）。
* **per span**：按“跨度”结算。**一句话解释**：R16 引入的概念，把一个时隙（Slot）切成更碎的几段（比如 2个或4个符号），在这一小段时间内算总账。
* **per group of $X_s$ slots**：按“多时隙组”结算。**一句话解释**：R17 为 RedCap (轻量级 5G 物联网终端) 引入的概念，放宽要求，允许好几天（几个时隙）加起来才算一次总账。

### 🚪 第一关：解码是有成本的 (算力限制)
手机每一次“瞄一眼黑板（PDCCH盲检）”，都在疯狂燃烧芯片算力（跑 Viterbi 或 Polar 译码器）。
如果基站配置的搜索空间太密集，手机就“烧屏”了。所以协议规定了两个“紧箍咒”（物理底线）：
1. **纸条总数限制 (Max Candidates)**：一段时间内，最多只能试解几张纸条。
2. **眼动范围限制 (Max non-overlapping CCEs)**：一段时间内，手机的眼睛需要在黑板上扫过的不同区域（不重叠的格子）总数不能超过上限（考察信道估计和数据搬运的带宽压力）。

### 🎯 第二关：三种时代，三种算账的“周期”
这部分上限是怎么算的？多久算一次账？基站会根据手机的能力和业务需求下发配置：

* **【经典基准版】(Rel-15)：按“天 (Slot)”算账**
  如果配置的是 `r15monitoringcapability`，那就以一个完整的时隙（通常 14 个符号，相当于一天）为单位。不管你这一天怎么排班，只要一天内的盲检总次数和看黑板的总格数不超过 Tables 10.1-2/3 的规定就行。

* **【极速内卷版】(Rel-16)：按“小时 (Span)”算账**
  如果配置的是 `r16monitoringcapability`（主要用于 URLLC，比如工业控制、自动驾驶）。这种业务要求极速响应，等不了一天。基站可能每隔 2 个符号（一两小时）就贴一次纸条。这时候，算账周期缩小到一个“Span（跨度）”。保证在这么短的时间内，算力也不超标（查 Table -2A/-3A）。

* **【低碳养生版】(Rel-17)：按“周 ($X_s$ Slots)”算账**
  如果配置的是 `r17monitoringcapability`（主要用于手表现表、传感器等低功耗 RedCap 设备）。这种设备算力极弱。算账周期被拉长到“一组连续的 $X_s$ 个时隙”。也就是这设备只要保证“一周（比如 5 个时隙）加起来的劳动量”不超过 Table -2B/-3B 就行，允许它慢慢悠悠地解。

### 🔄 算力天花板的三种结算周期
```text
【假设单次最大盲检能力为 44 次】

【R15：普通打工人 (按日结)】
 ├────── 1 个 Slot (14个符号) ──────┤
 │ 此区间内总盲检次数 <= 44 次        │ 

【R16：计件特种兵 (按小时结)】
 ├─ Span 1 ─┼─ Span 2 ─┼─ Span 3 ─...┤ (每个 Span 可能只有 2~7 个符号)
 │<= XX 次  │<= XX 次  │<= XX 次     │ (要求芯片有极强的短时爆发爆发力)

【R17：佛系临时工 (按周结)】
 ├──── Slot 1 ────┼──── Slot 2 ────┼──── Slot 3 ────┤ (共 X_s=3 个 Slot)
 │ 这整整 3 个 Slot 加起来，总盲检次数 <= 44 次            │ (极大幅度降低了芯片性能要求)
```

### 💡 章节硬核总结

**本段协议完整勾勒了 5G 从 eMBB 到 URLLC 再到 RedCap 的 PDCCH 盲检能力（Blind Decoding Capability）演进史。在 L1 层，终端资源分配存在严格的 Overbooking 限制，即在规定的时间窗内，PDCCH candidates 数量与 Non-overlapping CCEs 聚合数量双重受限。协议通过 `monitoringCapabilityConfig` 参数巧妙地将这套算力核算框架在时域尺度上进行了动态伸缩：R15 锚定传统的 per-slot 粒度；R16 引入了基于 (X, Y) pattern 的 per-span 亚时隙（Sub-slot）粒度，以支撑毫秒级 URLLC 调度而不引发极短突发内的算力超载；R17 则反向降维，引入了 per group of slots 粒度，允许将原本分配给一个时隙的算力预算平摊到多个时隙上，这是从根本上削减 IoT 终端基带芯片面积与成本的关键协议级支撑。**

## 📖 协议原文拆解 (六)：没给算力配置时，默认按什么规矩干活？(缺省算力模型)

> **协议原文**
> The remaining of this clause, including clause 10.1, considers that a UE is provided monitoringCapabilityConfig for a serving cell. If the UE is not provided monitoringCapabilityConfig for the serving cell, corresponding statements that the UE is provided monitoringCapabilityConfig for the serving cell are substituted as follows
> - for SCS configuration $\mu \in \{0,1,2,3\}$, the UE monitors PDCCH on the active DL BWP of the serving cell for maximum numbers of PDCCH candidates and non-overlapping CCEs per slot as in Tables 10.1-2 and 10.1-3.
> - for SCS configuration $\mu \in \{5,6\}$, the UE monitors PDCCH on the active DL BWP of the serving cell for maximum numbers of PDCCH candidates and non-overlapping CCEs per group of $X_s$ slots according to combination ($X_s$,$Y_s$)=(4,1) for $\mu=5$ and ($X_s$,$Y_s$)=(8,1) for $\mu=6$ as in Tables 10.1-2B and 10.1-3B.

### 📚 增量术语表
* **SCS configuration ($\mu$)**：子载波间隔配置。**一句话解释**：代表了 5G 无线电波的频率刻度，$\mu$ 越大，频率越高，每个符号（Symbol）和时隙（Slot）在物理时间上就越短。
* **$\mu \in \{0,1,2,3\}$**：常规频段。对应 15, 30, 60, 120 kHz 的子载波间隔（覆盖普通的 Sub-6G 和基础毫米波）。
* **$\mu \in \{5,6\}$**：极限高频段。对应 480, 960 kHz 的子载波间隔（覆盖极高频毫米波或太赫兹频段）。

### 🚪 第一关：不教我怎么干，我就按“出厂设置”干
上一段讲了基站会发 `monitoringCapabilityConfig` 告诉手机是用 R15、R16 还是 R17 的方式来计算最大盲检能力。
但如果手机在某些极早期阶段（比如刚接入还没收到专用配置），或者基站偷懒**没有下发这个配置**怎么办？
协议说：别慌，如果没收到配置，我们就根据你当前所处的频率环境（SCS），启动一套**雷打不动的“缺省（Default）”法则**。

### 🎯 第二关：常规频段的“经典日结”
如果手机工作在 $\mu = 0, 1, 2, 3$ 这些常规频段下：
* 缺省法则：直接假装你收到了 **R15 的基础版配置**。所有的算力天花板（最大盲检次数、最大 CCE 数）统统按照**每个时隙 (per slot)** 为单位来结算，去查 Table 10.1-2 和 10.1-3。这非常符合我们日常 eMBB (大带宽) 业务的习惯。

### 🧩 第三关：极高频段的“抱团降载”
如果手机工作在 $\mu = 5, 6$ 这类超高频（480kHz, 960kHz）的毫米波频段下，情况就严重了。
要知道，$\mu$ 越大，时隙（Slot）在物理时间上就变得极其短暂。比如 $\mu=6$ 时，一个 Slot 只有十几微秒长！如果还要求手机按照“每个时隙”去算账，那对基带芯片的瞬时处理压力是毁灭性的。
* 缺省法则：强制假装你收到了类似 **R17 的“养生版配置”**。
  * 当 $\mu=5$ 时，把 **4 个时隙** 打包成一组 (group of $X_s=4$) 算总账。
  * 当 $\mu=6$ 时，把 **8 个时隙** 打包成一组 (group of $X_s=8$) 算总账。
这样，哪怕单个时隙在物理时间上飞快地闪过，手机也可以名正言顺地把本来分配给一个时隙的繁重解码任务，摊平到 4 个或 8 个时隙的宽裕绝对时间里慢慢消化。

### 🔄 算力缺省模型的频段分支树
```text
【未收到 monitoringCapabilityConfig 时的算力降级模型】

 检查当前的 SCS 频率配置 (μ)
        │
        ├─► [常规频段: μ = 0, 1, 2, 3 (<= 120kHz)]
        │      └─► 采用 R15 经典模式
        │          结算周期 = 1 个 Slot
        │          (时隙长度尚可，芯片压力可控，按时隙结算)
        │
        └─► [极限高频: μ = 5, 6 (>= 480kHz)]
               └─► 采用降载“抱团”模式 (per group of slots)
                   ├─ μ = 5 ─► 4 个 Slot 绑一起算一次账
                   └─ μ = 6 ─► 8 个 Slot 绑一起算一次账
                   (物理时间跑得太快，必须把多个时隙捏在一起来稀释算力密度！)
```

### 💡 章节硬核总结

**本段协议确立了 5G PHY 层极具工程智慧的 PDCCH 盲检能力缺省映射（Default Mapping）逻辑。在缺失 RRC 显式指示时，系统并非简单地回退到 R15 Baseline，而是引入了 SCS（Numerology）强相关的维度。对于 Sub-6G 及基础 FR2 频段（$\mu \le 3$），缺省退退回 per-slot 的常规算力约束；而对于 Rel-17 引入的 FR2-2 极高频毫米波（$\mu=5, 6$），由于时隙绝对时长呈指数级缩短（至 sub-0.1ms 级别），若仍维持 per-slot 的盲检密度将导致 UE 解码管线崩溃。因此，协议在此处强制将高频缺省算力挂钩至 group of slots (即时间尺度拉伸)，通过 4 倍或 8 倍的跨时隙资源池化（Pooling），将极限高频下的瞬时计算峰值硬性抹平，保证了基带实现的物理可行性。**

## 📖 协议原文拆解 (七)：极高频段 ($\mu=6$) 的“新手村禁令”

> **协议原文**
> The UE does not expect to monitor PDCCH with SCS configuration $\mu=6$ before the UE is provided dedicated higher layer parameters.

### 📚 增量术语表
* **$\mu=6$ (SCS configuration 6)**：子载波间隔配置 6。**一句话解释**：高达 **960 kHz** 的子载波间隔！这是 5G (Rel-17以后) 在极高频毫米波频段才会启用的“狂飙模式”。在这个模式下，一个物理时隙 (Slot) 的时间长度不到 0.016 毫秒（常规 15kHz 时隙的六十四分之一），时间过得飞快。
* **Dedicated higher layer parameters**：专属高层参数。**一句话解释**：手机和基站建立好 RRC 连接后，基站专门为这台手机量身定制下发的一整套高级配置清单（不再是公共大喇叭广播的通用参数）。

### 🚪 第一关：超速驾驶不能在新手村练习
我们在前面的协议（比如第 8 章）里经常看到，手机在刚开机、还处于初始接入阶段时，会大量依赖系统广播和出厂默认配置来收发信号。
但是，对于 $\mu=6$ (960kHz) 这种极致的“狂飙模式”，协议直接下达了一道极其强硬的禁令：
**在手机拿到“专属的高层定制配置（Dedicated RRC）”之前，绝对不预期（does not expect）去监听使用 $\mu=6$ 的 PDCCH 通知！**

**为什么要下这道禁令？（物理意义）**
1. **硬件容错率为零**：$\mu=6$ 的时隙极短，对手机的定时同步精度、硬件切换速度、波束追踪能力要求极为苛刻。如果没有基站经过精确测算后下发的闭环专属参数（如精细的 TCI 波束状态、特殊的盲检打组规则），手机依靠模糊的公共参数根本无法在这个速度下成功捕获 PDCCH。
2. **规避标准复杂化**：如果允许在公共初始接入阶段（比如发 Msg2、收 SIB1）就使用 $\mu=6$，3GPP 就必须去写一大堆针对极短时隙的公共搜表、默认映射等极其冗余的 fallback 规则。禁止它，不仅保护了手机，也简化了协议。

**生活类比**：
刚出驾校的新手（还没拿到专属高层参数的 UE），只能在限速 60 的城市道路或限速 120 的普通高速（$\mu \le 3$）上开。
至于限速 400 公里的 F1 极速赛道（$\mu=6$），由于对底盘调校、轮胎温度要求太高，你**绝对不准上去开**。必须等你考取了高级赛照，并且技师为你量身调校好全套赛车参数（拿到 Dedicated higher layer parameters）后，才允许你上赛道。

### 🔄 早期接入阶段的频率封锁状态
```text
【手机开机，处于初始接入阶段 (未建立专属 RRC 连接)】
          │
          ├─► 试图监听 PDCCH (μ=0, 15kHz) ──► ✅ 允许 (经典 eMBB 接入)
          │
          ├─► 试图监听 PDCCH (μ=3, 120kHz) ─► ✅ 允许 (毫米波 FR2 常规接入)
          │
          └─► 试图监听 PDCCH (μ=6, 960kHz) ─► ❌【强制封印】
                                              "does not expect"
                                              由于时隙太短，容错率极低，
                                              在没有专属配置加持前，禁止开启！
```

### 💡 章节硬核总结

**本段协议通过一个高层断言（L1/L2 assertion: UE does not expect）彻底切断了 $\mu=6$ (960kHz SCS) 在初始接入阶段（Initial Access Phase, RRC Idle/Inactive）的合法性。作为 Rel-17 NR 延伸至 71GHz 频谱（FR2-2）的产物，$\mu=6$ 的物理特征是极端的符号时长压缩。若允许 UE 在缺乏专属 RRC 配置（尤其是精细化的 Beam Management 和跨时隙调度限制配置）的开环状态下监听 $\mu=6$ 的 PDCCH，将导致不可控的盲检失锁与功耗灾难。因此，协议在此处画下硬边界，规定 960kHz 的极速调度仅作为一种 Connected-mode（连接态）下的高阶增强特性存在。**

## 📖 协议原文拆解 (八)：从“按天结”到“按小时结”的超低延迟盲检 (Span 的诞生)

> **协议原文**
> A UE can indicate a capability to monitor PDCCH according to one or more of the combinations (X,Y) = (2, 2), (4, 3), and (7, 3) per SCS configuration of $\mu=0$ and $\mu=1$. A span is a number of consecutive symbols in a slot where the UE is configured to monitor PDCCH. Each PDCCH monitoring occasion is within one span. If a UE monitors PDCCH on a cell according to combination (X,Y), the UE supports PDCCH monitoring occasions in any symbol of a slot with minimum time separation of $X$ symbols between the first symbol of two consecutive spans, including across slots. A span starts at a first symbol where a PDCCH monitoring occasion starts and ends at a last symbol where a PDCCH monitoring occasion ends, where the number of symbols of the span is up to $Y$.

### 📚 增量术语表
* **Span**：跨度。**一句话解释**：这是为了 URLLC（低延迟高可靠）业务专门发明的时间单位。它比一个时隙（Slot，14个符号）小得多，是由几个连续符号组成的一小段极短的时间（比如只包含 2 个符号）。
* **Combination (X, Y)**：(X, Y) 组合能力。**一句话解释**：手机向上报的“瞬时算力体检报告”。$X$ 代表“两次高强度脑力劳动之间至少需要休息几个符号（最小发车间隔）”；$Y$ 代表“一次高强度劳动最多能持续几个符号（最长加班时间）”。
* **(2, 2)**：极其顶级的算力配置！代表最多工作 2 个符号，然后只需休息到第 2 个符号就能再次开工（几乎无缝衔接）。

### 🚪 第一关：为什么要发明 Span 这个概念？
在传统的 eMBB（刷剧、下载）里，手机只要在一天（一个 Slot 的最开头）看一眼黑板就行了，剩下的十几个符号时间都可以低功耗处理数据（这叫 per-slot 监听）。
但是，对于**工业机械臂、自动驾驶等 URLLC 业务**，绝对不能等！基站可能随时（在 Slot 的中间任意时刻）突发下达紧急指令。
因此，手机必须在一天内**频繁地看多次黑板**。为了在这种频繁看黑板的压力下核算手机的解码算力，协议发明了 **Span（亚时隙跨度）**，把一天切成了好几个短平快的工作块。

### 🎯 第二关：(X, Y) 组合的“工作与休息”法则
手机会告诉基站：“我能承受的 (X, Y) 压力是这几挡。”（通常只在 $\mu=0, 1$ 即 15/30kHz 下生效，因为高频下符号本来就很短了）。
基站在排班（排布 Search Space）时，必须严格遵守这个能力：
1. **$Y$ 限制（Span 的长度上限）**：一个 Span 从你开始看黑板的第一个符号算起，到你离开黑板的最后一个符号结束。这段时间不能超过 $Y$ 个符号。
2. **$X$ 限制（Span 之间的冷却期）**：相邻的两个 Span 的起点之间，必须隔开至少 $X$ 个符号。这保证了手机在极高强度的盲检后，有足够的绝对时间去消化（清空译码器内存）。**关键点：这个 $X$ 的间隔，哪怕是跨过了半夜零点（跨 Slot），也必须严格遵守！**
3. **不出界**：每一次单独的 PDCCH 监听动作（Monitoring occasion），必须被完完整整地包在这个 Span 里面。

**生活类比**：
手机是个特种兵。他告诉司令部（基站）自己的体能是 (X, Y) = (4, 3)。
* $Y=3$：每次执行特种突击任务（看黑板），时间绝对不能超过 3 小时。
* $X=4$：两次突击任务的**发起点**之间，必须至少间隔 4 小时。
比如：早上 8:00 开始任务（最多干到 11:00结束）。那么下一次任务最早也只能在 12:00（8点+4小时）才能发起。即使到了第二天（跨 Slot），这个 4 小时的 CD 时间也依然有效。

### 🔄 (X, Y) Span 机制时域拓扑图
```text
【以 (X,Y) = (7, 3) 为例，在同一个 Slot 内的排班】

符号: 0  1  2  3  4  5  6  7  8  9  10 11 12 13
     ├─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┤ (一个 Slot 14 个符号)

     [Span #1]             [Span #2]
     |← Y=3 →|             |←Y=2→|
     [看黑板1]              [看黑板2]
     ▲                     ▲
     │                     │
     └───── 间隔 X=7 ──────┘
     (从Span1起点到Span2起点，必须 >= 7个符号) ✅合法排班！

如果 Span#2 在符号 5 就开始了，那 X 间隔只有 5，小于 7，❌非法排班！手机会死机报错。
```

### 💡 章节硬核总结

**本段协议是 Rel-16 为 URLLC (Ultra-Reliable Low-Latency Communication) 引入的核心 L1 增强机制：基于 (X, Y) 范式的 per-span PDCCH 盲检架构。为了打破传统 per-slot 调度带来的 ms 级等待时延界限，协议允许将 PDCCH monitoring occasions 放置在 Slot 的任意符号内。然而，高频次的盲检会瞬间打爆基带芯片的 Trellis/Polar Decoder 吞吐量。因此，协议巧妙地抽象出了 `Span` 的概念，并通过 $X$ (连续 span 起点的最小符号间距) 和 $Y$ (单个 span 的最大符号跨度) 两个正交维度，为 UE 的瞬时峰值算力（Peak Processing Capability）加上了刚性的数学保护套。特别是强调了“including across slots”，弥补了跨时隙边界时可能出现的算力击穿漏洞。**

## 📖 协议原文拆解 (九)：当遇到多种排班能力都满足时，选“算力天花板”最高的那套

> **协议原文**
> If a UE indicates a capability to monitor PDCCH according to multiple (X,Y) combinations and a configuration of search space sets to the UE for PDCCH monitoring on a cell results to a separation of every two consecutive PDCCH monitoring spans that is equal to or larger than the value of $X$ for more than one of the multiple combinations (X,Y), the UE monitors PDCCH on the cell according to the combination (X,Y), from the more than one combinations (X,Y), that is associated with the largest maximum number of $M_{PDCCH}^{max,(X,Y),\mu}$ and $C_{PDCCH}^{max,(X,Y),\mu}$ defined in Table 10.1-2A and Table 10.1-3A. The UE expects to monitor PDCCH according to the same combination (X,Y) in every slot on the active DL BWP of a cell.

### 📚 增量术语表
* **Multiple (X,Y) combinations**：多种(X,Y)组合。**一句话解释**：高端手机会向基站显摆：“我既能做 (2,2) 这种极高压的突击任务，也能做 (4,3) 和 (7,3) 这种偏长线的任务”。
* **$M_{PDCCH}^{max}$ / $C_{PDCCH}^{max}$**：分别是 Table -2A 和 -3A 里规定的每次 Span 内允许的【最大盲检次数】和【最大不重叠 CCE 数】。**一句话解释**：这是与特定 (X,Y) 挂钩的算力配额。通常，要求你干活越累的组合（X越小，休息越短），单次分配给你的算力配额就越小，防止你瞬间过载。

### 🚪 第一关：基站排的班，到底算哪一种组合？(重叠判定)
当一个高端手机上报了自己支持 (2,2)、(4,3)、(7,3) 三种能力时。
基站实际下发了一套盲检排班表（Search Space sets）。
此时，手机一看这个排班表：两个任务的间隔居然是 **8 个符号**！
* 间隔 8 >= 7，满足了 (7,3) 的要求。
* 间隔 8 >= 4，满足了 (4,3) 的要求。
* 间隔 8 >= 2，也满足了 (2,2) 的要求。

那么问题来了：既然三种规矩都满足，手机去查表算盲检次数上限时，到底该套用哪个 (X,Y) 对应的上限值来结算？

### 🎯 第二关：算力“就高不就低”原则
协议给出了一个非常明确的决断法则：**选择拥有最高算力配额上限（$M^{max}$ 和 $C^{max}$ 最大）的那一个 (X,Y) 组合来作为结算标准。**

**为什么要这么选？（物理意义）**
通常来说，(7,3) 因为休息时间最长，所以协议分给它每个 Span 的“干活量上限”是最大的（能解更多的纸条）。而 (2,2) 因为高频连轴转，分配的单次干活上限最小。
既然基站现在的实际排班间隔长达 8 个符号，说明基站并没有压榨你频繁干活。既然给了你充足的休息时间，那你单次看黑板的能力理应按**最强的那一档（通常是 7,3）**来释放，让你在单次能处理最多的调度指令，从而提升系统的下行调度容量。

### 📅 第三关：排班规矩不能朝令夕改
协议的最后一句话补充了一个稳定态原则：
对于某个激活的下行带宽（Active DL BWP），一旦手机通过上面的逻辑确定了当前使用的是哪一套 (X,Y) 结算规则，那么**手机预期在每一个时隙 (every slot) 里，基站都会严格遵照同一套 (X,Y) 组合来排班。**
绝对不允许今天用 (7,3) 结算，明天突然变成 (2,2) 结算。这极大降低了手机底层状态机疯狂切换的复杂度。

### 🔄 (X,Y) 组合的多重决议树
```text
【手机具备多种能力: (2,2), (4,3), (7,3)】

[基站下发的实际搜索空间配置，提取出真实的间隔 X_real]
             │
             ▼ (假设 X_real = 5)
             │
[步骤1：筛选符合条件的组合]
  ├─ 满足 (2,2) 吗？ (5>=2) ✅
  ├─ 满足 (4,3) 吗？ (5>=4) ✅
  └─ 满足 (7,3) 吗？ (5>=7) ❌ (排除)
             │
[步骤2：在符合条件的组合中选“算力上限最大”的]
  候选：(2,2) 和 (4,3)
             │
  去查表，发现 (4,3) 的 M_max 和 C_max 值更大！
             │
             ▼
【最终决议】：当前这个 BWP 下的所有时隙，统一按 (4,3) 的结算标准来执行！
```

### 💡 章节硬核总结

**本段协议解决了 URLLC per-span 盲检框架下的“参数集歧义（Parameter Ambiguity）”问题。由于较长间隔的 (X,Y) pattern 必定在数学上包容较短间隔的 (X,Y) pattern，当基站的实际时域搜索空间配置相对稀疏时，将引发 UE 在核算 L1 盲检能力上限（Overbooking budget）时的匹配重叠。协议采用“最大化调度容量（Maximize Scheduling Capacity）”原则，明确指示 UE 在所有兼容的 (X,Y) 候选集中，挑选出对应 $M_{PDCCH}^{max}$ 和 $C_{PDCCH}^{max}$ 绝对值最大的那组配置作为当前 BWP 的准则。同时，通过 `UE expects` 语义强制约束了该 (X,Y) 决议在时域上的全局一致性（Per-slot consistency），严禁基站跨时隙动态切换 Span pattern 粒度。**

## 📖 协议原文拆解 (十)：太赫兹/毫米波下的“养生排班法” ($(X_s, Y_s)$ 多时隙分组)

> **协议原文**
> For SCS configuration $\mu=5$ or $\mu=6$, a UE can indicate a capability to monitor PDCCH according to one or more combinations ($X_s$,$Y_s$), where $X_s$ and $Y_s$ are numbers of consecutive slots. Groups of $X_s$ slots are consecutive and non-overlapping and the $Y_s$ slots are within the $X_s$ slots. The first group of $X_s$ slots starts from the beginning of a subframe. The start of two consecutive groups of $Y_s$ slots is separated by $X_s$ slots.

### 📚 增量术语表
* **$\mu=5$ 或 $\mu=6$**：极高频毫米波 (FR2-2)。**一句话解释**：子载波间隔达到 480kHz 甚至 960kHz，在这个频率下，一个时隙（Slot）短得只有十几微秒。
* **$(X_s, Y_s)$ 多时隙分组**：极高频下的特殊算力统筹方式。**一句话解释**：因为单个时隙太短了，手机处理不过来。所以把 $X_s$ 个时隙打包成一个“大周期（比如一周）”，然后只要求手机在这个周期内的其中 $Y_s$ 个时隙（比如工作日）里干活（看黑板），剩下的时间全用来休息和处理刚才看黑板得到的数据。

### 🚪 第一关：极高频倒逼出的“宏观跨度”
前面讲 $(X,Y)$ 时，是在**一个时隙内**按“符号（Symbol）”去切碎。
现在到了 $\mu=5$ 和 $\mu=6$，玩法完全反过来了：因为时隙本身已经碎成了渣，协议只能把**多个时隙**拼起来当成一个大单位来算账，这也就是 $X_s$ 和 $Y_s$ 里的后缀 $s$ (Slots) 的含义。

### 🎯 第二关：($X_s$, $Y_s$) 的作息制度
* **$X_s$ (总考勤周期)**：相当于基站把连续的 $X_s$ 个时隙圈起来，画成一个互不重叠的格子（Group）。这是算盲检能力的总账本范围。
* **$Y_s$ (实际干活区间)**：在这个大格子（$X_s$）里面，基站最多只能安排 $Y_s$ 个连续的时隙来给你下发 PDCCH 通知。
* **物理要求**：这两个干活区间（$Y_s$）的起始点之间，必须严格隔开 $X_s$ 个时隙。这就保证了周期性，绝不让手机处于连轴转的过载状态。

**生活类比**：
在极高频下，老板（基站）发通知的速度太快了，你（手机）处理不过来。
你跟老板商定了一个 $(X_s, Y_s) = (4, 1)$ 的能力协议：
* $X_s=4$：我们以 4 天为一个考勤大周期。
* $Y_s=1$：在这 4 天的大周期里，你最多只能集中在 1 天给我派发调度指令，剩下的 3 天我不看黑板，纯用来慢慢消化你那 1 天派发的数据。
下一周期的派发，必须等到前一个周期的起点过去 4 天后才能再次开始。这就是典型的“休养生息”排班法。

### 📅 第三关：排版的源头对齐 (从子帧开头算起)
这么多个时隙怎么划分呢？从哪里算作第一个 $X_s$ 分组的开头？
协议给了一个绝对的物理时间锚点：**从一个子帧（Subframe，恒定为 1毫秒）的绝对起点开始算。**
不管里面的 Slot 有多短（比如 $\mu=6$ 时，1毫秒内有 64 个 Slot），大家划分 $X_s$ 组的起跑线都是 1毫秒 的边界，确保了基站和手机的日历绝对不跑偏。

### 🔄 (X_s, Y_s) 极高频多时隙打包逻辑图
```text
【以 μ=6，(X_s, Y_s) = (8, 2) 为例】
(1个子帧 Subframe = 1ms = 64个极短的 Slot)

1ms 边界起点 (Subframe start)
  ▼
  ├─ Slot 0 ─ Slot 1 ─ Slot 2 ─ Slot 3 ─ Slot 4 ─ Slot 5 ─ Slot 6 ─ Slot 7 ─┤
  │                                                                         │
  │◄───────────────────── 一个考勤大周期 X_s = 8 个时隙 ─────────────────────►│
  │                                                                         │
  │◄── Y_s = 2 ──►│◄────────────── 纯休息/处理数据的宽裕时间 ────────────────►│
   [看黑板][看黑板] [不看黑板][不看黑板][不看黑板][不看黑板][不看黑板][不看黑板]

  (下一个 X_s=8 的周期，必须和这个周期的起点严格相差 8 个时隙)
```

### 💡 章节硬核总结

**本段协议是 3GPP Rel-17 为了支撑 FR2-2 极高频（480/960kHz SCS）特性而创造的一种防算力击穿机制：基于 $(X_s, Y_s)$ 的跨时隙 PDCCH 监控分布模型。由于极高频下物理符号周期被极度缩短，维持传统的 per-slot 盲检结算会导致终端基带芯片的译码器瞬时吞吐量与功耗指数级爆炸。因此，协议在时域维度引入了一层宏观封装：将 $X_s$ 个时隙打包为一个解耦的结算组（Group），并限制该组内的有效调度窗口不得超过 $Y_s$ 个连续时隙。这一机制在物理层创造了人为的“监听间隙（Monitoring Gap）”，利用时间换取空间，使得基带可以平滑地处理极短突发的高密度 DCI。同时，通过将其与 1ms Subframe 边界对齐，确保了跨载波与跨 numerology 调度的定时网格（Timing Grid）一致性。**

## 📖 协议原文拆解 (十一)：极高频下“系统大喇叭”与“私人信件”的区别对待

> **协议原文**
> If a UE monitors PDCCH on a cell according to combination ($X_s$,$Y_s$), the UE can monitor PDCCH for Type1-PDCCH CSS set provided by dedicated higher layer signalling, Type3-PDCCH CSS sets, and USS sets in any slot of the $Y_s$ slots, and the UE can monitor PDCCH for Type0/0A/2-PDCCH CSS set and Type1-PDCCH CSS set provided in SIB1 in any slot of the $X_s$ slots. The UE determines the number of monitored PDCCH candidates and the number of non-overlapped CCEs for combination ($X_s$,$Y_s$) based on all search space sets within the $X_s$ slots, as applicable according to the search space set configurations, and maximum corresponding values are provided in Table 10.1-2B and Table 10.1-3B, respectively.

### 📚 增量术语表
* **USS sets (UE-Specific Search space sets)**：终端专属搜索空间集。**一句话解释**：这是只发给你这个手机的“私人信件”，别人解不开，通常用来调度你平时的刷剧、打游戏等业务数据。
* **Type0/0A/2-PDCCH CSS / SIB1配置的Type1**：系统级公共搜索空间。**一句话解释**：这是全网“系统大喇叭”，广播所有人都需要听的系统消息、寻呼叫醒、初始随机接入回执。非常重要，优先级极高。
* **专属高层配置的Type1/Type3 CSS**：非核心公共搜索空间。**一句话解释**：虽然也是公共的，但是是给特定一群已经连上网的用户看的（比如群发节电信号、群发上行功率调整指令），不如“系统大喇叭”那么致命。

### 🚪 第一关：谁受 $(X_s, Y_s)$ 的“假期限制”？(私人信件/非核心公共消息)
上一段我们学到，在极高频下，手机通过 $(X_s, Y_s)$ 制度来休息。比如在一个 $X_s$ 的大周期里，只在 $Y_s$ 这么短的时间内干活看黑板。
这段协议明确了：**这种“强制休息”的规矩，只针对普通的业务（私人信件 USS、非核心公共消息 Type3/专属Type1）有效！**
基站如果想给你发个人业务调度，**必须老老实实地把你安排在 $Y_s$ 这个指定的工作时间内**。如果你在 $Y_s$ 时间之外收到了私人业务的黑板排班，你可以理直气壮地拒绝看（不在这几天工作）。

### 🎯 第二关：系统大喇叭无视“假期”，全年无休！
如果是极其致命的“全网系统大喇叭”（Type0/0A/2 CSS 等）：
基站**完全不需要遵守 $Y_s$ 的干活时间限制**！
协议说：`the UE can monitor PDCCH for Type0... in any slot of the Xs slots.`
意思是：哪怕你现在正处于 $(X_s, Y_s)$ 排班表里的“休息日”，只要基站拉响了全网系统大喇叭（比如全网寻呼、系统关键参数变更），你都必须爬起来在整个 $X_s$ 周期的任何一个时隙去听。**系统公共安全的优先级大于私人休假规矩！**

### 🧩 第三关：不管看什么，算总账的天花板不变
虽然大喇叭随时随地都可能响，但手机的物理算力是有限的。
协议最后一句兜底：不管是你在 $Y_s$ 里看的私人信件，还是你在整个 $X_s$ 里随时跳起来看的大喇叭，**只要落在这个 $X_s$ 的大周期内，所有的盲检纸条数和看黑板的总面积（CCEs），必须通通加在一起算一个总账！**
这个总账的天花板，依然要死死卡住 Table 10.1-2B 和 10.1-3B 规定的绝对上限。如果基站排班导致加起来的总量超标了，手机底层的 Drop（丢弃）机制就会触发。

### 🔄 (X_s, Y_s) 下的优先级盲检豁免图
```text
【一个 X_s 大周期内的排班情况】

     [ 必须干活的 Y_s 时间 ]    [       法定的休息时间 (X_s 减去 Y_s)       ]
  ├───────────────────────┼─────────────────────────────────────────────┤

💼 私人信件 (USS/Type3)
   ✅ 正常盲检解码            ❌ 基站不能在这里排班，排了手机也不看！

📢 系统大喇叭 (Type0/0A/2)
   ✅ 正常盲检解码            🚨 只要基站排了班，手机休息日也必须强制爬起来看！

📊 算力总账结算 (Overbooking 检查)
   <━━━━━━━━━ 把这整条时间线上的所有看黑板动作加起来算一次总账 ━━━━━━━━━>
   (总 Candidates 数 / 总 CCE 数，绝不能超过 Table 10.1-2B/3B 的上限)
```

### 💡 章节硬核总结

**本段协议揭示了极高频 (FR2-2) 下 PDCCH 监控机制的精细化优先级解耦逻辑。虽然 $(X_s, Y_s)$ 结构本质上是为了通过时域稀疏化来降低 UE 基带的处理负荷，但 3GPP 绝不允许这种降载机制阻断核心系统广播（SI）、寻呼（Paging）以及初始接入（Initial Access）的生命线。因此，协议在时域监听权限上实施了双轨制（Dual-track）：USS 与非关键 CSS 必须被严格收敛约束在 $Y_s$ 时隙窗口内；而核心系统级 CSS (如 Type0/0A/2) 则享有最高豁免权，能够穿透 $Y_s$ 的封锁，在整个 $X_s$ 组的任意时隙中被合法调度。为了防止这种越界调度彻底摧毁 UE 的算力防线，协议在数学层面补齐了闭环：强行要求将全量搜索空间（不论其是否在 $Y_s$ 内）统一归口至 $X_s$ 尺度的统一算力预算池（Budget Pool）中进行 Overbooking 联合核算。**

## 📖 协议原文拆解 (十二)：极高频 ($\mu=6$) 下的“最强打工人”选择法则

> **协议原文**
> For $\mu=6$, if the UE indicates a capability to monitor PDCCH according to multiple combinations ($X_s$,$Y_s$) and a configuration of search space sets to the UE for PDCCH monitoring on a serving cell results to a separation of every two consecutive groups of $Y_s$ slots that is not smaller than $X_s$ for more than one combinations ($X_s$,$Y_s$), of the multiple combinations ($X_s$,$Y_s$), the UE monitors PDCCH on the cell according to the combination ($X_s$,$Y_s$), from the more than one combinations ($X_s$,$Y_s$), that is associated with the largest maximum number of $M_{PDCCH}^{max,X_s,\mu}$ and $C_{PDCCH}^{max,X_s,\mu}$ defined in Table 10.1-2B and Table 10.1-3B.

### 📚 增量术语表
* **$\mu=6$**：子载波间隔配置 6 (960 kHz)。极高频狂飙模式。
* **multiple combinations ($X_s,Y_s$)**：多种多时隙分组组合能力。**一句话解释**：手机向基站报告说：“我既能接受‘4天一结’的作息，也能接受‘8天一结’的作息。”
* **$M_{PDCCH}^{max}$ / $C_{PDCCH}^{max}$**：最大盲检次数 / 最大 CCE 数量。分配给某一种特定作息的算力总配额。

### 🚪 第一关：似曾相识的“多选一”迷局
这段话看起来非常眼熟。在前面拆解“按小时结”的 $(X,Y)$ 时（拆解第九段），我们遇到过一模一样的逻辑。
区别在于，这里是专门针对 **$\mu=6$ 且“按多天结”的 $(X_s,Y_s)$** 模式。
当手机向基站上报了多种 $(X_s, Y_s)$ 能力，而基站实际排出来的搜索空间非常稀疏，导致这些任务之间的间隔大到**同时满足了好几种 $(X_s, Y_s)$ 的最小休息时间要求**时，手机该套用哪一套标准来计算自己的盲检天花板？

### 🎯 第二关：能力越强，责任越大 (选择算力上限最高的)
协议的裁决方式和前面如出一辙：**挑那个算力上限（配额）最大的组合！**
* 既然基站给的间隔足够宽裕（满足了多个要求），那就意味着此时网络并没有对手机进行高频压榨。
* 在这种宽裕的环境下，手机理应释放出自己最大的处理潜能。
* 手机会去查 Table 10.1-2B 和 10.1-3B，对比那些符合当前间隔条件的 $(X_s,Y_s)$ 组合，找出哪个组合对应的 $M^{max}$ (纸条数) 和 $C^{max}$ (格子数) 最大，就正式把系统锁定在这个结算标准上。

**生活类比**：
你是外包员工，你同时拥有两种接单能力：
能力 A：干 1 天休 3 天 (X=4)，每次最多能抗 50 包水泥。
能力 B：干 2 天休 6 天 (X=8)，每次最多能抗 120 包水泥。
现在工头（基站）通知你，最近货少，每次送完货后保证你连休 8 天！
你一算：休 8 天，既满足了能力 A 的底线（3天），也满足了能力 B 的底线（6天）。
工头问你：那你这次能抗多少包？
按照协议规定：既然休息时间这么充足，你必须以**最大能力（120包，能力 B 的标准）**来备战！

### 🔄 μ=6 场景下 (X_s, Y_s) 能力回退择优树
```text
【手机上报多种极高频能力，例如组合 A 和 组合 B】

[分析当前基站排班表的实际间隔 Gap]
             │
[步骤1：筛选符合条件的休假模式]
  ├─ 实际 Gap >= 组合 A 的 X_s 吗？ ──► (是) 留作备选
  └─ 实际 Gap >= 组合 B 的 X_s 吗？ ──► (是) 留作备选
             │
[步骤2：在备选中挑“干活配额”最多的]
  去查极高频专属表格 (Table 10.1-2B / 3B)
  假设 组合 B 对应的 M_max 和 C_max 最大！
             │
             ▼
【最终判定】: 手机锁定使用组合 B 作为接下来的算力天花板。
```

### 💡 章节硬核总结

**本段协议在数学逻辑上与前述的 Span (X,Y) 选择机制互为镜像，但其作用域专门限定在了 FR2-2 ($\mu=6$) 的极高频组时隙 $(X_s, Y_s)$ 场景中。这是因为 3GPP 协议体系的严谨性要求消除任何潜在的 L1 参数多解性（Ambiguity）。当搜索空间的时域稀疏度超过了多个终端能力的下限时，系统自动执行“Max-Capacity Fallback”策略。通过显式地将终端绑定到预算最大的合法 $(X_s, Y_s)$ 状态上，协议确保了基站在较宽松的调度周期内，能够向 UE 下发尽可能多的 PDCCH candidates（例如聚合更多的分量载波或配置更多的 Fallback DCI），从而在不击穿终端瞬时算力的前提下，最大化系统下行调度的稳健性与吞吐潜力。**

## 📖 协议原文拆解 (十三)：到底什么是“看黑板的能力”？(盲检能力的统一定义)

> **协议原文**
> A UE capability for PDCCH monitoring per slot, or per group of $X_s$ slots according to combination ($X_s$,$Y_s$), or per span on an active DL BWP of a serving cell is defined by a maximum number of PDCCH candidates and non-overlapped CCEs the UE can monitor per slot, or per group of $X_s$ slots according to combination ($X_s$,$Y_s$), or per span, respectively, on the active DL BWP of the serving cell.

### 📚 增量术语表
* **UE capability for PDCCH monitoring**：PDCCH 盲检能力。**一句话解释**：手机处理下行控制信令的“算力体检指标”。
* **CCE (Control Channel Element)**：控制信道元素。**一句话解释**：组成 PDCCH 通知单的基本“积木块”。基带芯片需要消耗内存和算力去对这些积木块进行信道估计（Channel Estimation）。

### 🚪 第一关：统一度量衡（万变不离其宗）
在前面的拆解中，我们看到了协议为了适应不同的业务（eMBB、URLLC、RedCap），发明了三种截然不同的时间结算周期：
1. **Per slot**（按天结，R15 基准）
2. **Per span**（按小时结，R16 低延迟）
3. **Per group of $X_s$ slots**（按周结，R17 极高频/低功耗）

这段协议是一句**全局的总结定义**：它宣告，不管你用上面哪一种时间周期来算账，手机“盲检能力”这个词的物理内涵是绝对统一的。它永远只由**两个核心指标**来定义。

### 🎯 第二关：能力的“两把标尺” (Candidates 与 CCEs)
基站要衡量一个手机在一个时间段内会不会“过载”，必须同时卡死以下两个物理极限：
1. **最多能猜几次密码？ (Maximum number of PDCCH candidates)**：
   这就好比限制了手机芯片里的**解码器（Polar Decoder）**的最高运转次数。哪怕每个纸条都很小，如果你让手机在短时间内解密 1000 次，解码芯片也会瞬间烧爆。
2. **最多能看多大面积？ (Maximum number of non-overlapped CCEs)**：
   这就好比限制了手机芯片的**内存和信道估计（Channel Estimation）模块**。因为在解密之前，手机必须把黑板上这块区域的电磁波信号先存下来，并过滤掉杂音。即使你只让手机解 1 次密码，但如果这张纸条占据了黑板上巨大的面积（比如聚合等级 AL=16，用了 16 个 CCE），对手机的前端信号处理压力也是极大的。协议强调了 **non-overlapped（不重叠）**，因为如果两张纸条贴在同一个格子上，手机只需要对这个格子做一次信道过滤就可以了，算力可以复用。

### 🔄 盲检能力定义的逻辑闭环
```text
【PDCCH 盲检能力 (UE Capability) 的通用定义公式】

 不管时间周期是什么：
 ├── (1) 经典模式: 每 Slot
 ├── (2) 极速模式: 每 Span
 └── (3) 降载模式: 每 X_s 个 Slot
        │
        ▼
 其能力的天花板，永远被以下两个物理维度的极限所定义：
   ┌────────────────────────────────────────────────────────┐
   │ 维度 A (逻辑解密负荷): 该周期内，最多能试解多少个 Candidates (纸条) │
   │ 维度 B (物理处理负荷): 该周期内，最多能提取多少个不重叠的 CCE (面积) │
   └────────────────────────────────────────────────────────┘
 (基站排班时，这两个指标只要有一个超标，就会触发手机的丢弃/降级保护机制)
```

### 💡 章节硬核总结

**本段协议在 L1 层面对 `PDCCH monitoring capability` 进行了严密的公理化统一定义（Axiomatic Definition）。尽管 3GPP 在不同的 Release 中引入了多维度的时域粒度（Slot, Span, Group of slots），但衡量终端控制信道处理能力的 Cost Metrics（成本度量）始终解耦为两个正交的物理维度：1) 盲检候选数（PDCCH Candidates），直接约束 L1 译码引擎（如 Polar Decoder）的算力开销（Logic complexity）；2) 不重叠控制信道元素（Non-overlapped CCEs），直接约束前端基带内存缓冲（Buffer size）与信道估计（Channel Estimation/Equalization）的面积负荷。这种在任何时域粒度下都保持度量衡一致性的设计，为后续所有复杂的 Overbooking 丢弃算法（如基于 Search Space ID 的降级规则）提供了一个绝对统一的数学判定基准。**

## 📖 协议原文拆解 (十四)：资源冲突第一弹——当“私人小纸条”撞上“系统大广播”

> **协议原文**
> For monitoring of a PDCCH candidate by a UE, if the UE
> - has received ssb-PositionsInBurst in SIB1 and has not received ssb-PositionsInBurst in ServingCellConfigCommon for a serving cell, and
> - does not monitor PDCCH candidates in a Type0-PDCCH CSS set, and
> - at least one RE for a PDCCH candidate overlaps with at least one RE of a candidate SS/PBCH block, after puncturing if applicable, corresponding to a SS/PBCH block index provided by ssb-PositionsInBurst in SIB1,
> the UE is not required to monitor the PDCCH candidate.

### 📚 增量术语表
* **SS/PBCH block (SSB)**：同步信号与物理广播信道块。**一句话解释**：基站向全网广播的最基础、最致命的“救护车警笛”信号，手机靠它找到基站并对准时间。
* **RE (Resource Element)**：资源粒子。**一句话解释**：5G 物理层最小的资源单位（频域上1个子载波，时域上1个符号），就像屏幕上的一个像素点。
* **Puncturing**：打孔。**一句话解释**：基站在发数据时，如果遇到了更重要的数据（比如大喇叭），就把原本数据所在的那几个 RE 直接挖空（丢弃不发），用来给大喇叭腾位置。

### 🚪 第一关：基站也会“画错线” (资源重叠)
基站在分配资源时有时候会发生冲突：它可能把一张调度纸条（PDCCH candidate）贴在了黑板上，但此时恰好是一辆“救护车（系统大广播 SSB）”预定要通过的时间和频率。
于是，这张小纸条在最底层的物理像素（RE）上，和 SSB 撞车了（Overlap）。

### 🎯 第二关：手机可以“装瞎”的三个前提条件
当纸条（PDCCH）和救护车（SSB）撞车时，手机能不能直接丢掉这张纸条不看？协议给出了极其严谨的三个前置条件，必须**同时满足**：

1. **信息差期（依赖 SIB1，还没拿到专属配置）**：手机只收到了最基础的 SIB1 广播里的 SSB 位置信息，还没有收到专属信令（`ServingCellConfigCommon`）里更精确的更新信息。这意味着手机对当前环境的认知还处于“大锅饭”阶段。
2. **看的不是最核心的黑板**：手机当前看这张纸条，不是在 `Type0-PDCCH CSS`（也就是用来调度 SIB1 自身的最高优公告栏）里看的。这说明这张撞车的纸条优先级不够高（可能是寻呼，也可能是普通业务）。
3. **物理上确实撞车了**：即便考虑了基站可能的“打孔（Puncturing）”操作，这张纸条至少还有一个像素点（RE）和 SIB1 里预告的那个 SSB 撞上了。

### 🛡️ 第三关：启动免死金牌 (Not required to monitor)
只要上述三个条件同时满足，协议赋予手机一项合法豁免权：
**`the UE is not required to monitor the PDCCH candidate.`（终端不被要求监听这个 PDCCH 候选者）。**

**物理意义**：
SSB（同步广播）在 5G 里的优先级是不可撼动的。如果基站粗心大意，把普通纸条压在了 SSB 上，手机强行去解密这张纸条，大概率会因为信号被 SSB 的强能量覆盖而解码失败（全是乱码）。既然注定失败，协议干脆允许手机“装瞎”，直接扔掉这张纸条，节省宝贵的解码算力和电量。

### 🔄 资源重叠冲突判决树
```text
【PDCCH 纸条与 SSB 广播车相撞时的处理流程】

 发现一张 PDCCH 纸条的至少一个像素点 (RE) 和 SSB 撞了！
        │
        ├─ 检查条件1：目前只有 SIB1 的 SSB 位置信息吗？ ──► (是) 往下看
        │
        ├─ 检查条件2：这张纸条不在最核心的 Type0 公告栏里吗？ ──► (是) 往下看
        │
        ▼
 【三个条件全中！】
        │
        └─► 【合法弃权】：手机理直气壮地丢弃这张纸条 (不消耗盲检算力)。
            (内心独白：基站你安排的资源冲突了，这锅我不背，我不瞎解了！)
```

### 💡 章节硬核总结

**本段协议在 L1 物理层 RE (Resource Element) 粒度上，定义了 PDCCH 与下行核心参考信号（SSB）发生资源碰撞（Collision）时的终端异常处理行为。这体现了 5G NR 调度架构中的严格优先级原则：SS/PBCH Block 拥有绝对的物理层豁免权与优先级。当基站由于调度冲突或打孔策略（Puncturing）失误，导致非 Type0-CSS（即非调度 SIB1 自身）的 PDCCH candidate 在物理时频网格上与 SIB1 宣告的 SSB 发生哪怕只有一个 RE 的交叠时，协议通过 `not required to monitor` 原语显式赋予了 UE 免除盲检的权利。这一机制不仅避免了因 SNR 恶化导致的无谓译码尝试，有效降低了 UE 的动态功耗，同时也倒逼网络侧（eNodeB/gNB）的调度器实现必须在时频资源网格上严格规避 SSB 碰撞区带内嵌区域。**

## 📖 协议原文拆解 (十五)：资源冲突第二弹——拿到专属配置后，依然避让“大广播”

> **协议原文**
> For monitoring of a PDCCH candidate by a UE, if the UE
> - has received ssb-PositionsInBurst in ServingCellConfigCommon for a serving cell, and
> - does not monitor PDCCH candidates in a Type0-PDCCH CSS set, and
> - at least one RE for a PDCCH candidate overlaps with at least one RE of a candidate SS/PBCH block, after puncturing if applicable, corresponding to a SS/PBCH block index provided by ssb-PositionsInBurst in ServingCellConfigCommon,
> the UE is not required to monitor the PDCCH candidate.

### 📚 增量术语表
* **ServingCellConfigCommon**：服务小区公共配置。**一句话解释**：手机在度过了“只靠听 SIB1 广播度日”的极早期阶段后，基站下发的一份更精确、更详细的本小区公共参数说明书。它里面的信息优先级高于 SIB1。

### 🚪 第一关：似曾相识的条款，只是“说明书”升级了
如果你仔细看，这一段和上一段（第十四段）长得几乎一模一样。
它们处理的都是同一个核心冲突：**基站安排的小纸条（PDCCH）和系统大广播（SSB）在物理像素（RE）上撞车了。**

**区别在哪里？**
区别在于手机手中拿着的“时刻表”更新了：
* **上一段**：手机只拿到了最原始的 `SIB1` 时刻表。
* **这一段**：手机已经收到了更权威的 `ServingCellConfigCommon` 时刻表。

### 🎯 第二关：升级后的免听逻辑
当手机拿到了 `ServingCellConfigCommon` 之后，它脑子里的 SSB（救护车）的发车时刻表就被刷新了。
此时，如果再次发生资源撞车，判定条件变成了：
1. **参考依据更新**：目前是以 `ServingCellConfigCommon` 里的 SSB 位置信息为准。
2. **非最核心黑板**：看的依然不是用来调度最高优系统消息的 `Type0-PDCCH CSS`。
3. **物理重叠**：哪怕考虑了打孔（Puncturing），这张纸条还是和 `ServingCellConfigCommon` 里指示的 SSB 像素点撞上了。

**结果也是一样的霸道**：
只要满足这三个条件，手机依然享有合法豁免权：**直接扔掉这张纸条，不用去解它（not required to monitor）！**

### 🔄 更新时刻表后的免检逻辑
```text
【PDCCH 纸条与 SSB 广播车相撞】

手机翻开手中的时刻表：
发现不再是初始的 SIB1，而是已经拿到了更新的 ServingCellConfigCommon！
        │
        ├─► 检查：这张纸条在 Type0 公告栏吗？ ──(不在)──► 往下走
        │
        ├─► 检查：和新时刻表上的 SSB 物理像素(RE)撞了吗？ ──(撞了哪怕1个点)──► 往下走
        │
        ▼
 【符合豁免条件】
 手机直接放弃盲检这张纸条，算力省下，防止解出一堆乱码。
```

### 💡 章节硬核总结

**本段协议是前述 SSB 碰撞规避法则在连接态或 RRC 配置更新后的自然延续（Forward Extension）。在 5G RRC 状态机中，终端对 SS/PBCH block 位置分布的认知会随着系统信息的获取而发生从 `SIB1` 到 `ServingCellConfigCommon` 的平滑过渡。本条款严密地覆盖了终端在获取到 `ServingCellConfigCommon` 后的状态，确保无论在哪一个 RRC 配置阶段，L1 物理层对于“非 Type0 CSS 的 PDCCH Candidate 一旦与信令宣告的 SSB 发生 RE 级交叠即可合法 Drop（丢弃）”这一最高优先级避让准则的判定逻辑保持绝对的连贯性与自洽性。这在底层软件实现中，意味着碰撞检测模块（Collision Detection Engine）的输入源由 SIB1 数据库无缝切换到了 ServingCellConfigCommon 数据库，而核心丢弃逻辑保持不变。**

## 📖 协议原文拆解 (十六)：资源冲突第三弹——在多天线塔(mTRP)场景下的精准避让

> **协议原文**
> For monitoring of a PDCCH candidate by a UE, if the UE
> - has received ssb-PositionsInBurst in SSB-MTCAdditionalPCI for a serving cell, and
> - at least one RE for a PDCCH candidate overlaps with at least one RE of a candidate SS/PBCH block, after puncturing if applicable, corresponding to a SS/PBCH block index provided by ssb-PositionsInBurst in SSB-MTCAdditionalPCI with same physical cell identity as the one associated with a RS having same quasi-collocation properties as a CORESET for the PDCCH candidate,
> the UE is not required to monitor the PDCCH candidate.

### 📚 增量术语表
* **SSB-MTCAdditionalPCI**：附加物理小区ID的 SSB 测量时间配置。**一句话解释**：这是 5G 后期版本（比如支持多发射点 mTRP 时）引入的高级配置。基站不仅让你测本小区的天线塔信号，还让你去测另外一个有着“不同物理小区ID（Additional PCI）”的天线塔发射的 SSB 信号。
* **Physical Cell Identity (PCI)**：物理小区ID。**一句话解释**：每个基站天线塔在空气中发送的无线电“身份证号”。

### 🚪 第一关：复杂的双塔环境 (mTRP)
前两段（14和15段）处理的都是“本小区的调度纸条”碰上了“本小区的救护车（SSB）”的简单情况。
但现在，网络变得极其复杂：手机连着一个服务小区，但这个服务小区是由**两个不同的小区ID的天线塔**（比如天线塔 A 和天线塔 B）一起协作来给你发数据的。
这时候，如果遇到资源重叠，手机该不该扔掉纸条？协议给出了极其严谨的**交叉印证逻辑**。

### 🎯 第二关：精准避让的“同源鉴定法”
协议说，只有满足以下连环条件，你才能合法扔掉纸条：
1. **拿到了高级时刻表**：手机收到了 `SSB-MTCAdditionalPCI`，也就是知道了天线塔 B（附加 PCI）的发车时刻表。
2. **发生物理重叠**：一张小纸条（PDCCH）和天线塔 B 发出的 SSB 在像素点（RE）上撞车了。
3. **【核心条件】波束与天线塔同源 (同 PCI)**：
   这张纸条贴在哪个黑板（CORESET）上？这个黑板的波束方向（QCL参考信号）属于哪个天线塔的身份证号（PCI）？
   如果这个黑板的身份证号，**恰好等于**发生撞车的那个天线塔 B 的身份证号，那么恭喜你，这俩是一个塔发出来的，可以触发避让机制。

**生活类比**：
你在广场上，同时有两个大喇叭（塔A和塔B）在喊你。
* 塔B 的大喇叭（SSB）正在播报紧急广播。
* 这时候，塔B 也顺便在它的专属黑板（同 PCI 的 CORESET）上贴了一张小纸条给你。
小纸条和紧急广播的时间地点撞车了。因为它们**同属于塔B（同源干扰）**，紧急广播的声音会直接盖掉小纸条，小纸条肯定没法看，你可以直接扔掉不看（not required to monitor）。

**反向思考（如果不同源呢？）**：
如果这张纸条是贴在**塔A**的黑板上，而撞车的是**塔B**的大喇叭。因为它们在物理空间上来自两个截然不同的方向（不同的 QCL，不同的 PCI），手机高级的接收天线也许可以通过空间抗干扰技术（波束赋形）在屏蔽塔B大喇叭的同时，清晰地看清塔A的纸条。这时候，协议**就不允许你随意扔掉纸条**了，你还得尝试去解！

### 🔄 多塔(mTRP)场景下的精准避让逻辑链
```text
发现一张纸条(PDCCH) 与 【附加塔(Additional PCI)】的 SSB 撞车了！
        │
        ├─► 追踪这张纸条的背景：
        │    纸条所在黑板(CORESET)的波束来源是哪个 PCI？
        │
        ├──► 【是同一个附加塔的 PCI (同源)】
        │       └─► 属于塔内严重互扰，信号被彻底盖死。
        │           ✅ 合法弃权：手机不解这张纸条！
        │
        └──► 【是另一个塔的 PCI (异源空间复用)】
                └─► 可以通过多天线空间滤波隔离干扰！
                    ❌ 不准弃权：手机必须尝试去解这张纸条！
```

### 💡 章节硬核总结

**本段协议将 PDCCH 与 SSB 的冲突避让（Collision Avoidance）机制升维至 Rel-17 引入的多发送接收点（mTRP）及异构 PCI 协同调度场景。在多 PCI 架构下，终端（UE）会同时维持针对 Serving Cell PCI 以及 Additional PCI 的 SSB 测量窗口（MTC）。协议在此处实施了极为精密的“空间维同源性（Spatial-Source Homogeneity）”判定法则：只有当发生碰撞的 PDCCH Candidate 其寄宿的 CORESET 的 TCI-State (QCL属性) 溯源出的物理小区标识（PCI），与导致物理 RE 重叠的那个干扰源 SSB 所属的 Additional PCI 完全一致时，终端才被允许丢弃该 PDCCH 的盲检任务。这一设定从底层 L1 避免了跨 TRP 空间解耦调度时被错误地触发本地丢弃机制，最大限度地保护了 mTRP 架构下的调度容量。**

## 📖 协议原文拆解 (十七)：公共大喇叭不讲“外地方言” (核心 CSS 的 PCI 约束)

> **协议原文**
> A UE is not required to monitor PDCCH candidates for a Type0/0A/0B/1/1A /2/2A -PDCCH CSS set when the active TCI state for a corresponding CORESET is not associated with physCellId in ServingCellConfigCommon.

### 📚 增量术语表
* **Type0/0A/0B/1/1A /2/2A -PDCCH CSS set**：核心公共搜索空间大家族。**一句话解释**：这些都是基站用来发全网广播、初始随机接入回执、寻呼找人等极其重要的“公共大喇叭”。
* **active TCI state**：激活的传输配置指示状态。**一句话解释**：当前这个黑板（CORESET）正在使用的“波束方向”和“天线对准参数”。
* **physCellId (Physical Cell ID)**：物理小区ID。**一句话解释**：基站天线塔的唯一“身份证号”。写在 `ServingCellConfigCommon` 里的是你的“正牌服务小区（大本营）”的身份证号。

### 🚪 第一关：“公共大喇叭”的绝对归属权
在 5G 复杂的网络部署里，基站为了提高网速，可能会把个人业务的数据（USS 里的纸条）通过其他协作的天线塔（不同的小区 ID）发送给你。这就叫**跨塔调度**。
但是！这段协议给那些极其重要的**公共消息（所有的 Type CSS 大家族）**划定了一条神圣不可侵犯的底线：
**公共大喇叭，必须且只能由你的“正牌服务小区（本尊）”亲自播报！**

### 🎯 第二关：如果大喇叭里传出“外地方言”怎么办？
协议规定，手机在听这些核心公共公告栏（CSS）时，必须检查当前用来接收信号的天线波束参数（active TCI state）。
如果手机发现：**“咦？这个公告栏的波束源头，竟然不是指向我的正牌小区 ID（physCellId），而是指向了某个外来的、协作的小区 ID ？”**
只要发生这种情况，手机拥有绝对的豁免权：
**`not required to monitor`（直接拒绝监听这个公共黑板上的所有纸条！）**

**生活类比**：
你在“A村”（你的服务小区）当村民。村里有个大广播喇叭（CSS），专门播报村委会通知（Type0）、洪水警报（Type2）等。
为了让你听广播更清楚，村长说：“大家听广播时，要把耳朵朝向村口那棵大树（TCI state 关联到 A村的物理ID）。”这很正常。
但如果有一天，村长突然修改了广播站的配置，让你“把耳朵朝向隔壁的 B村去听 A村的洪水警报（TCI state 关联到非本村的物理ID）”。
面对这种荒谬的指令，协议赋予了你底气：**你可以直接不听（不监听盲检），因为这违反了公共信息必须由本村亲自发布的根本法！**

### 🔄 公共搜索空间 (CSS) 的波束血统核查
```text
【手机准备监听极其关键的 Type0/0A/0B/1/1A/2/2A CSS 公共黑板】

检查该黑板当前激活的波束配置 (Active TCI state)
        │
        ├─► 追溯其波束的源头 (Source RS) 属于哪个物理小区 ID (PCI)？
        │
        ├──► 【情况 1：PCI == 我自己的服务小区 ID】
        │       └─► 血统纯正。✅ 必须老老实实监听盲解！
        │
        └──► 【情况 2：PCI != 我自己的服务小区 ID (外来 PCI)】
                └─► 荒谬配置！系统级公共消息不该由外地塔代发。
                    ✅ 合法罢工：手机直接无视该黑板上的所有候选者，拒绝监听！
```

### 💡 章节硬核总结

**本段协议在 L1 层面硬性规范了核心公共搜索空间（Common Search Space, CSS）组的空间域波束溯源（Spatial Beam Traceability）边界。随着 Rel-16/17 中 mTRP（多发送接收点）及 Inter-cell multi-TRP 架构的引入，网络侧可以利用具备异构 PCI 的射频节点（Additional/Non-serving PCI）来为终端下发控制信令（TCI-State 关联到异构 PCI）。然而，协议在此处画下红线：对于承载系统信息、随机接入、寻呼等生命线广播控制信道的 Type0~Type2A CSS 簇，其宿主 CORESET 的激活 TCI-State 必须具备绝对的本地同源性。一旦其 TCI 状态关联到非 `ServingCellConfigCommon` 宣告的 `physCellId`，终端有权拒绝盲检。这在极大地简化了底层物理层状态机抗错能力的同时，也从规范层面封堵了网络侧在关键系统广播上滥用异地协同调度可能带来的协议紊乱。**

## 📖 协议原文拆解 (十八)：看最重要黑板时的“清场假设” (Type0-PDCCH 的特权)

> **协议原文**
> If a UE monitors the PDCCH candidate for a Type0-PDCCH CSS set on the serving cell according to the procedure described in clause 13, the UE may assume that no SS/PBCH block is transmitted in REs used for monitoring the PDCCH candidate on the serving cell.

### 📚 增量术语表
* **Type0-PDCCH CSS set**：Type0 公共搜索空间。**一句话解释**：这是整个 5G 系统里**优先级最高**的一块黑板！它上面贴的纸条，是指引手机去接收 `SIB1`（系统信息块1，这是手机入网必看的总规章制度）的。
* **clause 13**：第 13 章。**一句话解释**：专门讲手机如何接收 SIB1 等基础系统广播的章节。
* **may assume**：可以假设。**一句话解释**：物理层标准用语。意味着“基站保证一定不会这么干，所以手机在底层电路设计时，直接不用去防备这种可能性，放心大胆地干”。

### 🚪 第一关：最高优先级的黑板，绝不能被碰瓷
前面拆解时（第十四、十五段），我们讲过：如果普通的黑板（PDCCH）和救护车（SSB）撞车了，普通黑板直接作废，手机可以装瞎不看。
但是，这里提到了一个极其特殊的存在：**Type0-PDCCH CSS**。
这块黑板是用来调度 `SIB1` 的，是整个 5G 网络能够运转的基石。如果连这块黑板也被随便撞车作废了，手机就连网都入不了了！

### 🎯 第二关：基站的底层承诺（清场保障）
为了保护这块最神圣的黑板，协议从**基站发射侧**给出了一个绝对承诺，并允许手机在接收侧**大胆假设**：
当手机去监听这块 Type0 公共黑板上的小纸条时，手机**完全可以假设（may assume）**，基站**绝对不可能**把任何一个救护车（SSB）的像素点（RE）安排在这个黑板的位置上。

**物理意义**：
基站在排兵布阵时，必须对 Type0-PDCCH 和 SSB 进行完美的正交隔离（要么在时间上错开，要么在频率上错开）。这就像是在公路上划出了一条不可侵犯的应急车道，即使是救护车（SSB），在此时此刻也绝不允许驶入这条特定的车道，从而为 Type0 营造了一个绝对干净、无干扰的“清场”环境。

**生活类比**：
（前面讲的）：你作为一个普通快递员（看普通纸条），路上看到有救护车（SSB）过来，你必须靠边停车（扔掉纸条不看）。
（这段讲的）：现在你手里拿的是**“国家最高机密文件”（Type0-PDCCH）**。交管局（协议）向你保证：“当你跑这条专线去取文件时，我们已经封锁了整条道路。你放心大胆地闭着眼睛往前冲，在这条路上**绝对不可能**有任何救护车突然窜出来干扰你！”

### 🔄 Type0 特权与普通空间的差异树
```text
【PDCCH 盲检时的资源碰撞防御策略】

手机开始看黑板 (Monitor PDCCH candidates)
        │
        ├─► 【普通业务 / 其他公共黑板 (如 Type1, Type2, USS)】
        │       └─► 必须防备！如果和 SSB 撞车了，启动自保机制 ──► (放弃解密)
        │
        └─► 【国家机密黑板 (Type0-PDCCH CSS，找 SIB1 专用)】
                └─► 放弃防备！直接在电路层面假设“本区域绝对清场”。
                    基站若敢在这里排 SSB 就是基站违规，手机只管全速解密，无需执行碰撞判定。
```

### 💡 章节硬核总结

**本段协议确立了初始接入阶段控制信道网络拓扑的最高优先级边界：Type0-PDCCH CSS 区域的“绝对射频洁净度（Absolute RF Cleanliness）”假设。对于承载 RMSI (Remaining Minimum System Information, 即 SIB1) 调度任务的 CORESET 0 及相关的 Type0 Search Space，基站侧在资源映射网格（Resource Grid）调度算法上负有强制排他义务。协议通过 `may assume` 原语，豁免了 UE 侧在盲检 Type0 PDCCH 时需要执行的 RE 级 SSB 碰撞避让判定（Collision Detection Overhead）。这种正交化设计确保了在最恶劣的未同步或冷启动开环阶段，最底层的信令管道不被同信道的任何高能导频或同步信号所吞噬，极大提升了盲检的成功率并简化了 L1 ASIC 的状态机逻辑。**

## 📖 协议原文拆解 (十九)：当 5G 小纸条被 4G 的“老钉子”扎破 (LTE CRS 冲突)

> **协议原文**
> If at least one RE of a PDCCH candidate for a UE on the serving cell overlaps with at least one RE of lte-CRS-ToMatchAround or of LTE-CRS-PatternList, the UE
> - is not required to monitor the PDCCH candidate if the UE is not provided pdcchCandidateReception-WithCRSOverlap,
> - monitors the PDCCH candidate if the UE is provided pdcch-CandidateReceptionWithCRSOverlap and the UE indicates an associated capability corresponding to the configuration of lte-CRS-ToMatchAround or of LTE-CRS-PatternList [18, TS 38.306].

### 📚 增量术语表
* **LTE CRS (lte-CRS-ToMatchAround / LTE-CRS-PatternList)**：4G 小区特定参考信号及其避让图纸。**一句话解释**：在 4G/5G 共享基站（DSS 动态频谱共享）时，4G 信号里那些去不掉的、像地钉一样散布在各个角落的“常驻信号”。
* **pdcchCandidateReception-WithCRSOverlap**：允许在 CRS 重叠下接收 PDCCH 的配置。**一句话解释**：基站发给手机的一本“高级补洞技能书”，告诉手机：“如果小纸条被 4G 的地钉扎破了，你不要扔，绕开那些破洞拼凑一下继续看！”

### 🚪 第一关：历史遗留问题——与 4G 前辈合租的烦恼
现在的 5G 网络，为了节省频段资源，经常会和 4G LTE “合租”在同一个频段上（这叫 DSS 技术）。
但是，4G LTE 有一个极具标志性的“老大难”问题：它有一套叫 CRS 的参考信号，像满地撒的钉子一样，密密麻麻地扎在时频网格里，去都去不掉。
如果基站在贴 5G 的调度小纸条（PDCCH）时，不小心把纸条贴在了 4G 的 CRS 钉子上（RE 发生物理重叠），这张 5G 纸条就被扎破了（被 4G 信号干扰覆盖）。

### 🎯 第二关：没学过“缝补术”的普通人 (直接弃权)
遇到这种被 4G 钉子扎破的纸条，手机该怎么办？协议给出了两种截然不同的态度：
* **情况 A**：如果手机**没有**收到基站下发的 `pdcchCandidateReception-WithCRSOverlap`（补洞技能书）。
手机看到纸条破了字看不清，会理直气壮地说：“这题超纲了，我没这技术！”
于是启动合法豁免权：**`not required to monitor`（直接把纸条扔垃圾桶，不解了）**。

### 🧩 第三关：持证上岗的“高级裁缝” (必须绕开钉子解密)
如果基站想在这么恶劣的环境下硬传控制信息，它必须启用高级玩家模式：
* **情况 B**：基站给手机发了“补洞技能书”（配置了参数），同时，手机在出厂时也向基站拍过胸脯：“老板，我拥有极其强大的运算能力，给我 4G CRS 的位置图纸，我能搞定！”（UE indicates an associated capability）。
在这种强强联合下，协议态度大变：
手机**必须去监听（monitors the PDCCH candidate）**！
它会在底层物理电路里，精准地把那些被 4G 钉子扎破的像素点（RE）抠掉，假装它们不存在，然后用剩下的残缺数据，通过强大的纠错算法（Rate Matching，速率匹配）硬生生地把原本的 DCI 调度指令还原出来！

### 🔄 5G 纸条碰上 4G 钉子的判决树
```text
【5G PDCCH 候选者 与 4G CRS 发生 RE 级物理重叠】
        │
        ├─► 手机检查自己有没有收到“允许在冲突下接收”的配置证书？
        │
        ├──► 【没有配置】 (普通打工人)
        │       └─► 字被扎破了看不了！
        │           ✅ 合法弃权：扔掉纸条，不消耗算力盲检。
        │
        └──► 【有配置 + 自身能力支持】 (高级缝补裁缝)
                └─► 必须干活！
                    根据基站给的 4G 钉子分布图 (LTE-CRS-PatternList)，
                    精准抠掉破损像素，用剩余有效像素硬算还原出 DCI 指令！
```

### 💡 章节硬核总结

**本段协议是 5G 物理层为解决 DSS（Dynamic Spectrum Sharing，动态频谱共享）场景下跨 RAT (Radio Access Technology) 共存干扰而设计的经典 L1 Rate Matching 避让机制。由于 LTE CRS 在时频网格中具有全带宽、高密度、不可消除的特性，5G PDCCH 极易与其发生 RE 级碰撞。为避免 SNR 灾难导致的盲检失败，协议解耦了 UE 的行为：对于基础 UE，遇到此类碰撞直接视为非法映射并丢弃盲检（Drop）；而对于支持高级 Rate Matching 能力的高阶 UE，网络可以通过 RRC 下发显式的冲突接收使能参数，强制终端在提取 PDCCH LLR (Log-Likelihood Ratio) 软信息时，精准打孔（Puncture/Rate match around）掉落入 LTE CRS 的 RE 资源。这极大提升了在严重碎片化的 DSS 频谱上部署 5G 控制信道的容量与鲁棒性与调度自由度。**

## 📖 协议原文拆解 (二十)：共享频段里的“红绿灯”与“默认放行” (RB Set 状态避让)

> **协议原文**
> If a UE is provided availableRB-SetsPerCell, the UE is not required to monitor PDCCH candidates that overlap with any RB from RB sets that are indicated as unavailable for receptions by an available RB set indicator field in DCI format 2_0 as described in clause 11.1.1. If the UE does not obtain the available RB set indicator for a symbol, the UE monitors PDCCH candidates on all RB sets in the symbol.

### 📚 增量术语表
* **availableRB-SetsPerCell**：单小区可用 RB 集合配置。**一句话解释**：开启了一种“动态地盘汇报”功能，通常用于共享频谱（NR-U），因为那里的频段是被大家抢来抢去的，某些地盘可能随时被别人占走。
* **DCI format 2_0 (Available RB set indicator)**：包含可用地盘指示的 DCI 2_0。**一句话解释**：基站发出的一个全网“红绿灯”交通广播，告诉你：“兄弟们，1号地盘我抢到了（绿灯可用），2号地盘被 Wi-Fi 抢走了（红灯不可用）。”
* **Unavailable for receptions**：不可用于接收。**一句话解释**：由于基站没抢到这个地盘的发送权，导致这个地盘里根本没有 5G 信号，只有白噪音或别人的干扰。

### 🚪 第一关：交通广播说“前方封路” (红灯避让)
在免授权共享频段（NR-U）里，基站发数据之前也得“先听后发（LBT）”。如果基站没抢到信道，那块频率（RB Set）就废了。
为了不让手机像傻子一样在废频率上苦等，基站如果成功抢到了其他频段，就会用 `DCI format 2_0` 赶紧广播一个“红绿灯指示”。
协议规定：如果手机**成功收到了交通广播**，并且广播里明确说某个 RB Set 是**“红灯（Unavailable）”**。
那么，任何试图贴在这个“封路地盘”上的小纸条（PDCCH candidates），手机统统享有合法豁免权：**`not required to monitor`（直接无视，根本不要去浪费算力解它！）**。

**生活类比**：
交管局（基站）在群里发广播（DCI 2_0）：“注意！东直门大街（某个 RB Set）刚刚被 Wi-Fi 施工队挖断了，禁止通行（Unavailable）！”
这时，如果你看到你的派件单（搜索空间排班表）上原本写着要去东直门大街的公告栏看纸条。既然交管局都说路断了，你去了肯定也是白跑一趟（只有噪音）。所以你理直气壮地直接坐在原地不去了，节省汽油（算力）。

### 🎯 第二关：没听清广播怎么办？ (默认绿灯前行)
但是，这个“交通广播（DCI 2_0）”本身也是在空气中传播的，万一手机恰好没收到、漏听了怎么办？
这时，手机面临一个状态盲区：我不知道前面的路到底是断了还是通的。
协议给出了极为硬核的“盲动”底线：**如果你没收到红绿灯指示（does not obtain），你绝对不能原地躺平！你必须默认所有的路都是通的（all RB sets），硬着头皮去所有的公告栏把纸条全看一遍！**

**物理意义**：
“宁可错杀一千（白费算力去解噪音），不可放过一个（漏解关键调度）。”
如果没收到指示就瞎假设不可用，会导致手机错过基站好不容易抢到信道发出的调度指令。所以，在信息缺失时，物理层必须执行最悲观（工作量最大）的假设：全量盲检！

### 🔄 RB Set 动态可用性状态机
```text
【手机准备在某个符号的多个 RB Set 上进行 PDCCH 盲检】
        │
        ├─► 检查：我刚才收到过 DCI 2_0 的交通广播吗？
        │
        ├──► 【收到了】 (有明确的路况信息)
        │       │
        │       ├─ 地盘 A：显示为 Unavailable (红灯) ──► ❌ 放弃看地盘 A 的黑板！
        │       └─ 地盘 B：显示为 Available (绿灯) ────► ✅ 正常去看地盘 B 的黑板！
        │
        └──► 【没收到 / 错过了】 (两眼一抹黑)
                └─► 协议底线机制启动！
                    假装所有的地盘都是绿灯！
                    ✅ 地盘 A 的黑板去解！ ✅ 地盘 B 的黑板也去解！ (算力全开硬抗)
```

### 💡 章节硬核总结

**本段协议定义了 NR-U（Shared Spectrum）场景下，基于动态信道可用性指示（DCI format 2_0）的 PDCCH 盲检动态裁剪（Dynamic Pruning）机制。由于 LBT（Listen-Before-Talk）结果的不可预知性，基站的实际发射带宽在频域上是以 RB Set 为粒度动态变化的。为了避免终端在 LBT 失败（基站未发射）的空资源上进行毫无意义的能量积分与极化译码从而导致无谓的功耗与 False Alarm，协议允许 UE 利用 DCI 2_0 的前馈信息对盲检候选集进行实时剪枝。但更为关键的是，为了防止 DCI 2_0 自身的漏检导致的 Error Propagation（错误蔓延），协议设计了极其保守的 Failsafe（防故障）准则：在缺乏前馈指示的状态盲区，UE 必须回退至静态 RRC 配置的全集状态，对所有 RB Sets 强行进行遍历盲检。**

## 📖 协议原文拆解 (二十一)：算账的高级数学题——多基站协同 (mTRP) 下的盲检能力折算

> **协议原文**
> If a UE can support
> - a first set of $N_{cells,0}^{DL}$ serving cells where the UE is either not provided coresetPoolIndex or is provided coresetPoolIndex with a single value for all CORESETs on all DL BWPs of each scheduling cell from the first set of serving cells, and
> - a second set of $N_{cells,1}^{DL}$ serving cells where the UE is not provided coresetPoolIndex or is provided coresetPoolIndex with a value 0 for a first CORESET, and with a value 1 for a second CORESET on any DL BWP of each scheduling cell from the second set of serving cells
> the UE determines, for the purpose of reporting pdcch-BlindDetectionCA, pdcch-BlindDetectionCA1, and pdcch-BlindDetectionCA3, a number of serving cells as $N_{cells,0}^{DL} + R \cdot N_{cells,1}^{DL}$ where $R$ is a value reported by the UE.

### 📚 增量术语表
* **Carrier Aggregation (CA) / DL serving cells**：载波聚合 / 下行服务小区。**一句话解释**：手机同时连了好几个频段（比如一边连着主频段，一边连着几个扩展频段加速），$N_{cells}^{DL}$ 就是统计你同时收几个频段的数据。
* **coresetPoolIndex**：CORESET 池索引。**一句话解释**：这是多基站塔协作（mTRP）的核心标志！如果一个频段里的黑板（CORESET）被标上了不同的 Pool Index（比如一部分是 0，一部分是 1），说明这个频段是由两个不同的物理天线塔同时在给你发信号的。
* **$N_{cells,0}^{DL}$**：第一类小区数量（单塔小区）。**一句话解释**：在这个频段里，所有的黑板都只有同一个 Pool Index（或者干脆没标），说明只有一个塔在干活。
* **$N_{cells,1}^{DL}$**：第二类小区数量（双塔/mTRP小区）。**一句话解释**：在这个频段里，黑板被明确分成了 Pool 0 和 Pool 1 两拨，说明有两个塔在同时干活，手机的接收复杂度直接翻倍。

### 🚪 第一关：手机向上级汇报“总承载力”
手机能同时并行处理几个载波（服务小区）的盲检任务，是有物理极限的。手机必须把自己的这个能力（比如 `pdcch-BlindDetectionCA`，载波聚合下的盲检能力上限）汇报给基站。
但在 5G 引入 mTRP（多天线塔）技术后，计算“你到底扛了几个小区”的方式变了！因为有些小区里藏着两个塔在同时给你发数据。

### 🎯 第二关：双塔小区的“膨胀折算率 (R)”
为了公平地核算手机的脑力消耗，协议发明了一个**折算公式**：
总承载小区数 = 单塔小区数 ($N_{cells,0}^{DL}$) + 折算系数 ($R$) × 双塔小区数 ($N_{cells,1}^{DL}$)

* 如果一个小区是**单塔单发（第一类小区）**，它的权重就是 1，算 1 个小区的工作量。
* 如果一个小区是**双塔协作（第二类小区，存在 Pool 0 和 1）**，手机要在同一个频段里同时盯着两个天线塔发来的不同纸条空间（非理想回程或高强度复用），这对芯片算力的挤压非常可怕。
* 此时，手机会根据自己的硬件架构，向基站上报一个**折算系数 $R$**（通常大于 1）。
* 这句话的意思是：**“老板，我处理这 1 个双塔小区的工作量，顶得上处理 $R$ 个普通的单塔小区！你给我算工资（分配盲检预算）的时候，得按 $R$ 倍给我膨胀着算！”**

### 🔄 载波聚合盲检能力的折算公式图解
```text
【手机向上报自己的总调度承载能力】

 假设手机同时连了 5 个频段 (小区)：
 ├─ 频段 A (只有单塔在发) ───► $N_{cells,0}$ = 3 个这种小区
 ├─ 频段 B (只有单塔在发)      (权重为 1)
 ├─ 频段 C (只有单塔在发)

 ├─ 频段 D (塔1和塔2同时发!) ─► $N_{cells,1}$ = 2 个这种小区
 └─ 频段 E (塔1和塔2同时发!)    (难度翻倍！手机上报折算系数 R = 2)

 【最终能力基站侧核算】：
 手机上报的能力必须覆盖： 3 + 2 × 2 = 7 个“当量单塔小区”的工作量预算！
 (如果基站乱排班超过了这个当量折算上限，手机底层的安全丝就会熔断丢弃任务)
```

### 💡 章节硬核总结

**本段协议是 5G L1 层为了适配 Rel-16 mTRP (Multi-TRP) NCJT (Non-Coherent Joint Transmission) 架构，而在 Carrier Aggregation (CA) 盲检能力预算 (Blind Decoding Budget) 核算体系中打入的一块关键补丁。在 NCJT 中，UE 会在一个 Component Carrier 上收到属于两组完全正交的 CORESET Pools (通过 `coresetPoolIndex` 区分) 的 PDCCH candidates，这在基带底层的接收前端（Receiver Front-end）和译码管线上等效于多增加了一路逻辑载波的负担。为了在不推翻原有 CA 盲检上限框架的前提下真实反映 UE 的物理层压力，协议引入了基于 UE Capability 上报的线性折算因子 $R$。通过将 mTRP 载波等效膨胀为 $R$ 个普通载波的计算方式，优雅地完成了复杂空间域多工调度到一维算力预算模型的数学映射。**

## 📖 协议原文拆解 (二十二)：超大带宽玩家的“算力声明” (CA>4 小区时的能力上报)

> **协议原文**
> If a UE indicates in UE-NR-Capability a carrier aggregation capability larger than 4 serving cells and the UE is not provided monitoringCapabilityConfig for any downlink cell or if the UE is provided monitoringCapabilityConfig = r15monitoringcapability for all downlink cells where the UE monitors PDCCH, the UE includes in UE-NR-Capability an indication for a maximum number of PDCCH candidates and for a maximum number of non-overlapped CCEs the UE can monitor per slot when the UE is configured for carrier aggregation operation over more than 4 cells. When a UE is not configured for NR-DC operation, the UE determines a capability to monitor a maximum number of PDCCH candidates and a maximum number of non-overlapped CCEs per slot that corresponds to $N_{cells}^{cap}$ downlink cells, where
> - $N_{cells}^{cap}$ is $N_{cells,0}^{DL}+R \cdot N_{cells,1}^{DL}$ if the UE does not provide pdcch-BlindDetectionCA where $N_{cells,0}^{DL}+N_{cells,1}^{DL}$ is the number of configured downlink serving cells
> - otherwise, $N_{cells}^{cap}$ is the value of pdcch-BlindDetectionCA

### 📚 增量术语表
* **UE-NR-Capability**：UE的NR能力报告。**一句话解释**：手机入网时递交给基站的一份详细“个人简历”，里面写明了自己支持几个载波聚合、芯片最高算力是多少等极其底层的硬件参数。
* **Carrier aggregation capability larger than 4**：载波聚合能力大于 4 个小区。**一句话解释**：只有旗舰级的 5G 手机才能同时聚合 5 个甚至更多的高速频段。这是通信里的高端玩家。
* **$N_{cells}^{cap}$**：手机声明支持的载波盲检容量上限。**一句话解释**：手机最终算下来，自己这颗芯片到底能抗住几个“当量小区”的总盲检工作量。
* **pdcch-BlindDetectionCA**：PDCCH 载波聚合盲检能力参数。**一句话解释**：手机在简历里自己主动填报的一个数字，告诉基站“我的总盲检承载力就这么大了”。

### 🚪 第一关：高端玩家必须提交“专项体检报告”
协议首先划了一条线：**如果手机在简历里吹牛说自己能同时处理超过 4 个下行频段（CA > 4）**，且处于传统的“按天结”模式（R15 能力或缺省配置）。
那么，协议要求这个手机必须在“简历 (UE-NR-Capability)”里，明明白白、老老实实地交一份**专项体检报告**。
报告里必须写清楚：**“当网络真的给我挂上超过 4 个频段时，我在每一个时隙（per slot）里，全网所有频段加起来，最多能看多少张纸条（PDCCH candidates），最多能扫多少面积的黑板（non-overlapped CCEs）。”**
*为什么要写明？* 因为大于 4 载波后，底层的总线带宽和解调芯片极容易瞬间爆表，必须要有全网封顶的数字约束。

### 🎯 第二关：如果手机没主动填报上限，基站怎么算？
如果手机处于普通组网模式（非 NR-DC 双连接），基站需要知道这台手机的总承载能力 $N_{cells}^{cap}$。
这里分两种情况：
1. **手机填了能力值 (otherwise 分支)**：
   如果手机在简历里明确上报了 `pdcch-BlindDetectionCA`（比如填了 6）。那就按手机报的算，$N_{cells}^{cap} = 6$。基站在排班时，总任务量绝不能超过 6 个小区的基准值。
2. **手机没填能力值 (第一分支)**：
   如果手机只说了支持几个频段，但没明确报出总盲检能力数字，那基站就**强行用上一段学到的折算公式来算总账**！
   把当前配置给你的单塔小区数 ($N_{cells,0}^{DL}$) 和 双塔小区数 ($N_{cells,1}^{DL}$) 加起来，并且给双塔小区乘上膨胀系数 $R$。
   算出来的这个值，就是你的天花板 $N_{cells}^{cap}$。

### 🔄 旗舰机盲检能力核算树
```text
【一台高端手机入网，声明支持聚合 5 个下行载波】

 必须向基站上交《PDCCH盲检能力专项报告》 (限制了全网 Candidates 和 CCEs 的总和)

 基站开始核算它的总承载能力 N_cells_cap：
        │
        ├─► 手机在报告里主动填了 pdcch-BlindDetectionCA 吗？
        │
        ├──► 【填了，值为 7】
        │       └─► 算力天花板 N_cells_cap = 7。基站照此排班。
        │
        └──► 【没填】
                └─► 启动公式核算：你当前配置了 3 个单塔，2 个双塔，折算系数 R=2。
                    算力天花板 N_cells_cap = 3 + 2 × 2 = 7。
                    虽然你连了 5 个实体频段，但你必须承担 7 个频段的算力压力！
```

### 💡 章节硬核总结

**本段协议定义了在超大载波聚合（Large-scale CA, $N_{cells} > 4$）场景下的 L1 PDCCH 盲检能力（Blind Decoding Capacity）标定机制。当载波数超过 4 时，基带的聚合盲检压力不再是一个可以无脑线性累加的安全值。为此，协议强制要求声明大 CA 能力的 UE 显式上报其在时隙级（per slot）的全局 Candidates 和 CCEs 处理极限。在确定基带能支撑的等效调度容量 $N_{cells}^{cap}$ 时，协议给出了明确的 Fallback 树：优先使用 UE 显式上报的 `pdcch-BlindDetectionCA` 标量值；若 UE 未做显式能力解耦，则退化至根据 RRC 配置的载波类型（单 TRP 还是多 TRP）辅以系数 $R$ 进行基于物理配置的等效数学硬算。这套体系确保了基站在跨载波统一调度时的 Overbooking 预算不至于击穿旗舰手机的物理层前端。**

## 📖 协议原文拆解 (二十三)：NR-DC 双连接下的“分灶吃饭”与“绝对红线”

> **协议原文**
> When a UE is configured for NR-DC operation, the UE determines a capability to monitor a maximum number of PDCCH candidates and a maximum number of non-overlapped CCEs per slot that corresponds to $N_{cells}^{cap}=N_{cells}^{MCG}$ downlink cells for the MCG where $N_{cells}^{MCG}$ is provided by pdcch-BlindDetection for the MCG and determines a capability to monitor a maximum number of PDCCH candidates and a maximum number of non-overlapped CCEs per slot that corresponds to $N_{cells}^{cap}=N_{cells}^{SCG}$ downlink cells for the SCG where $N_{cells}^{SCG}$ is provided by pdcch-BlindDetection for the SCG. When the UE is configured for carrier aggregation operation over more than 4 cells, or for a cell group when the UE is configured for NR-DC operation, the UE does not expect to monitor per slot a number of PDCCH candidates or a number of non-overlapped CCEs that is larger than the maximum number as derived from the corresponding value of $N_{cells}^{cap}$.

### 📚 增量术语表
* **NR-DC operation**：NR-DC (新空口双连接) 操作。**一句话解释**：这是 5G 里最顶级的组网方式，手机同时连接两个纯正的 5G 独立基站（比如一个主基站 MCG 提供广覆盖，一个辅基站 SCG 提供超高网速）。
* **pdcch-BlindDetection**：专门下发的 PDCCH 盲检能力分配值。**一句话解释**：既然连了两个基站，算力就得“分家”。这是明确分给各自主机站和辅基站的“盲检预算配额”。

### 🚪 第一关：双连接必须“分灶吃饭” (独立核算)
在上一段我们讲了单基站超多载波（CA > 4）的情况，天花板是统一的一个 $N_{cells}^{cap}$。
但当手机进入 **NR-DC（双 5G 基站连接）**模式后，因为是两个不同的基站在同时给你下发指令，为了防止两边抢算力打架，协议规定必须**严格“分家核算”**：
* 对于主基站群（MCG），它的天花板独立计算，其当量的承载小区数等于网络配置给它的专用配额，即 $N_{cells}^{cap} = N_{cells}^{MCG}$。
* 对于辅基站群（SCG），它的天花板也是独立计算的，即 $N_{cells}^{cap} = N_{cells}^{SCG}$。
两边的配额都是由各自 RRC 参数里的 `pdcch-BlindDetection` 字段明确划拨的。谁也不能占用谁的算力预算。

### 🎯 第二关：最高级别的防线 (Does not expect 警告)
在把算力天花板怎么算（单基站折算、或者双基站分家）都讲得明明白白之后，协议给出了 L1 物理层对基站调度器的**最高级别警告（Red Line）**：
无论你是一个连了超多频段（>4）的高端玩家，还是一个开了双连接（NR-DC）的玩家，对于任何一个计算好的小区组天花板 $N_{cells}^{cap}$：
**手机绝对不预期（does not expect）在一个时隙内，实际需要盲检的纸条数量（PDCCH candidates）或扫过的黑板面积（non-overlapped CCEs）会超过由这个 $N_{cells}^{cap}$ 换算出来的绝对上限值！**

**物理意义**：
这是保护手机芯片不被“烧毁”的最后一道物理防火墙。
`does not expect` 在 3GPP 语境里意味着：如果基站的排班系统（Scheduler）出 bug 了，排出来的任务总量超出了手机申报的这个天花板，基站属于**严重违规**。面对这种违规排班，手机底层的译码芯片不需要尝试去强行解密，直接罢工丢弃（Drop）多出来的任务是合法的，并且丢包的责任完全在基站。

### 🔄 NR-DC 下的算力分家与红线图
```text
【手机开启 NR-DC，同时连接 5G 主基站和 5G 辅基站】
         │
         ├─► [MCG 主组] 拿到配额 N_cells_MCG = 3
         │     └─► 换算出 MCG 的盲检算力上限。
         │     ⛔ 警告基站A：这边的实际派单量如果超过上限，我直接拒收！
         │
         └─► [SCG 辅组] 拿到配额 N_cells_SCG = 2
               └─► 换算出 SCG 的盲检算力上限。
               ⛔ 警告基站B：这边的实际派单量如果超过上限，我同样拒收！

(两者在底层的内存和解码器资源上严格物理/逻辑隔离，互不透支)
```

### 💡 章节硬核总结

**本段协议完成了 5G 高阶多连接架构（NR-DC 及 Large CA）下 PDCCH 盲检能力预算的顶层分配与收敛。在 NR-DC 模式下，由于 MCG 与 SCG 的 MAC 调度器在物理节点上解耦，无法实时进行算力协商。因此，协议在 RRC 配置层面对 UE 整体基带算力实施了强制的静态切片（Static Partitioning），即通过显式的 `pdcch-BlindDetection` 为两个 Cell Group 各自锚定了等效小区上限 $N_{cells}^{MCG}$ 和 $N_{cells}^{SCG}$。随后，协议使用强制排误原语 `UE does not expect` 画下了算力红线。这不仅赋予了 UE 物理层在遭遇网络侧 Overbooking 调度（如 DCI 洪峰）时进行防卫性 Drop 的权利，更是从 3GPP 标准层面划定了网络设备（gNB）调度器算法的安全责任边界。**

## 📖 协议原文拆解 (二十四)：双连接“分家产”的数学红线 (算力守恒定律)

> **协议原文**
> When a UE is configured for NR-DC operation with a total of $N_{cells}^{DL,NR-DC}$ downlink cells on both the MCG and the SCG, the UE expects to be provided pdcch-BlindDetection for the MCG and pdcch-BlindDetection for the SCG with values that satisfy
> - pdcch-BlindDetection for the MCG + pdcch-BlindDetection for the SCG <= pdcch-BlindDetectionCA, if the UE reports pdcch-BlindDetectionCA, or
> - pdcch-BlindDetection for the MCG + pdcch-BlindDetection for the SCG <= $N_{cells}^{DL,NR-DC}$, if the UE does not report pdcch-BlindDetectionCA.

### 📚 增量术语表
* **$N_{cells}^{DL,NR-DC}$**：NR-DC 下行小区总数。**一句话解释**：手机同时连着主辅两个基站，所有参与下行传输的频段（载波）加在一起的物理总个数。
* **pdcch-BlindDetectionCA**：终端上报的载波聚合盲检总能力。**一句话解释**：手机出厂时在能力报告里声明的“我这颗芯片到底能扛几个当量的频段”。

### 🚪 第一关：分家可以，但“家产总额”不能变
上一段讲了，NR-DC 模式下，主基站（MCG）和辅基站（SCG）要“分灶吃饭”，各自拿一个叫 `pdcch-BlindDetection` 的配额去算账。
但这引出了一个新问题：这两个配额是谁给的？（网络侧配置的）。网络会不会瞎配？比如手机明明只能扛 6 个小区的算力，网络却给主基站配了 4 个，辅基站配了 4 个（加起来等于 8）？

协议为了防止网络侧“乱开空头支票”，在这里设立了极其严格的**“算力能量守恒定律”**。
网络侧下发给 MCG 的配额加上下发给 SCG 的配额，**加起来绝对不能超过手机的物理底线！**

### 🎯 第二关：判定“算力天花板”的两条分支
这个物理底线（天花板）到底是多少？这里分两种情况：

1. **“写了简历”的手机 (If UE reports...)**：
   如果手机在能力报告里明确填了 `pdcch-BlindDetectionCA`（比如手机说我能扛 8 个当量的算力）。
   那么，`MCG 拿到的份额 + SCG 拿到的份额 必须 <= 8`。

2. **“没写简历”的普通手机 (If UE does not report...)**：
   如果手机没填这个总上限，那么天花板就直接等于**当前配置给你的实际物理频段总数（$N_{cells}^{DL,NR-DC}$）**。
   比如你现在总共连了 5 个频段，那么 `MCG份额 + SCG份额 必须 <= 5`。

**生活类比**：
你是包工头（UE 基带），手下有两支施工队（MCG 处理线程和 SCG 处理线程）。
甲方（网络侧）每天给两支施工队分别派任务量（pdcch-BlindDetection 配额）。
* 如果你签合同前声明过：“我总共只有 80 个人”（上报了能力值）。那甲方给两边派的任务量加起来，绝对不能超过 80 人的工作量。
* 如果你没声明总人数，甲方就看今天总共给你开了几个工地（实际物理频段数，比如 5 个），那派给你的总任务配额，加起来绝对不能超过 5 个标准工地的工作量。

### 🔄 NR-DC 配额下发的合规性检查图
```text
【网络给 NR-DC 手机下发 MCG 和 SCG 的算力配额】
         │
         ├─► [MCG 配额] = X
         ├─► [SCG 配额] = Y
         │
【手机收到配置后的底层自检 (UE Expects)】
         │
         ├─ 算出总和: Sum = X + Y
         │
         ├─ 寻找我的天花板 Limit：
         │    ├── 我上报过总能力值 C 吗？ ──► 天花板就是 C
         │    └── 没上报过？ ────────────► 天花板就是当前的物理小区总数 N_total
         │
         └─► 【进行判定】: Sum 必须 <= Limit！
             (如果违反此定律，基站配置严重非法，手机物理层状态机将产生异常甚至拒配)
```

### 💡 章节硬核总结

**本段协议在 RRC 配置层面对 NR-DC 架构下的 PDCCH 盲检预算切片（Budget Partitioning）施加了全局的代数约束（Algebraic Constraint）。它通过 `UE expects` 原语建立了闭环的 L1 合规性校验机制，确保网络侧配置给 MCG 和 SCG 的逻辑算力配额之和（$\Sigma$ pdcch-BlindDetection），绝对不溢出终端底层的物理算力水池（Budget Pool）。该水池的容量取决于 UE 能力上报状态：显式上报场景下受控于 `pdcch-BlindDetectionCA` 标量；隐式场景下受控于实际激活的 DL Cells 总数。这一守恒定律从系统设计源头切断了因 MCG 和 SCG 独立 RRC 实体缺乏算力统筹而导致配置出 UE 基带“不可解（Infeasible）”状态的风险。**

## 📖 协议原文拆解 (二十五)：手机主动要求“偏心”分配算力 (非对称双连接预算上报)

> **协议原文**
> For NR-DC operation, the UE may indicate, through pdcch-BlindDetectionMCG-UE and pdcch-BlindDetectionSCG-UE, respective maximum values for pdcch-BlindDetection for the MCG and pdcch-BlindDetection for the SCG.

### 📚 增量术语表
* **pdcch-BlindDetectionMCG-UE**：终端期望的 MCG 盲检能力上限。**一句话解释**：手机主动告诉基站：“我这颗芯片的结构，分给主基站（MCG）这边的处理能力最多只有这么大。”
* **pdcch-BlindDetectionSCG-UE**：终端期望的 SCG 盲检能力上限。**一句话解释**：同理，手机告诉基站：“我分给辅基站（SCG）那边的处理能力最多只有那么大。”

### 🚪 第一关：手机也有硬件设计的“偏科”
前面讲过，在 NR-DC 双连接下，网络需要给 MCG 和 SCG 分配盲检算力配额，并且总和不能超标。
但如果网络把所有的算力都砸给 MCG，或者全砸给 SCG，行不行？
**对某些手机来说，不行！**

因为有的手机在设计底层基带芯片时，负责处理 MCG 和 SCG 信号的是两块物理上独立的硬件模块（或者资源池划分不均衡）。
比如手机的总算力是 10，但 MCG 模块的硬件上限是 7，SCG 模块的硬件上限是 5。
这时候，如果网络非常极端地给 MCG 强行分配了 9 的配额（9 + 1 = 10，虽然总和没超标），手机的 MCG 硬件模块也会瞬间被撑爆。

### 🎯 第二关：主动亮明“木桶的短板”
为了防止网络这种“极端偏心”的分配导致局部硬件崩溃，协议允许手机（`may indicate`）主动向网络亮出自己两块硬件的局部底牌：
* 通过上报 `pdcch-BlindDetectionMCG-UE`，锁死主基站这边能接单的绝对上限。
* 通过上报 `pdcch-BlindDetectionSCG-UE`，锁死辅基站那边能接单的绝对上限。

**生活类比**：
你（UE）承包了一个项目（NR-DC），手下有 A、B 两个施工队。
你上报总公司：“我这包工头总共能抗 100 人的工作量（总能力 `pdcch-BlindDetectionCA`）。”
但为了防止总公司瞎派活，你补充了一个极其重要的声明：“请注意，虽然我总共能抗 100 人，但我 A队 最多只能塞 70 人进去（`MCG-UE` 上限），B队 地方小最多只能塞 50 人进去（`SCG-UE` 上限）。你给我分配两边任务的时候，不仅总和不能超 100，各自分配的数量也不能超过它们各自的容量上限！”

### 🔄 NR-DC 算力双向挤压图
```text
【基站给手机下发 MCG 和 SCG 的算力配额】

必须同时满足以下所有红线：

  红线 1 (上一段讲的全局总账):
  [MCG配额] + [SCG配额] <= 手机的总承载力 (如 10)

  红线 2 (本段补充的局部木桶效应):
  [MCG配额] 必须 <= 手机上报的 MCG 局部上限 (如 7)
  [SCG配额] 必须 <= 手机上报的 SCG 局部上限 (如 5)

(只有在这重重数学约束下算出来的配额，才不会烧毁基带)
```

### 💡 章节硬核总结

**本段协议在 L1 盲检能力上报机制中引入了“局部硬边界（Local Hard Boundaries）”的概念。这主要针对部分 UE 基带架构在实现 NR-DC 时，可能采用了非对称的双 DSP/FPGA 核心簇，导致 MCG 和 SCG 的译码资源池并非完全动态共享的情况。通过允许 UE 显式暴露 `pdcch-BlindDetectionMCG-UE` 和 `SCG-UE` 这两个子上限标量，协议在之前 $X+Y \le Z$ 的全局代数约束之上，补充了 $X \le X_{max}$ 与 $Y \le Y_{max}$ 的局部非对称极值约束。这一反馈链路使得基站的 RRC 统筹实体在执行跨节点预算切片时，能够精准避开终端硬件管线的物理瓶颈区域。**

## 📖 协议原文拆解 (二十六)：局部上限不仅要有，还得“加起来溢出”才能容错

> **协议原文**
> If the UE reports pdcch-BlindDetectionCA,
> - the value range of pdcch-BlindDetectionMCG-UE or of pdcch-BlindDetectionSCG-UE is [1, …, pdcch-BlindDetectionCA-1], and
> - pdcch-BlindDetectionMCG-UE + pdcch-BlindDetectionSCG-UE >= pdcch-BlindDetectionCA.
>
> Otherwise, if $N_{cells}^{DL,NR-DC,max}$ is a maximum total number of downlink cells that the UE can be configured on both the MCG and the SCG for NR-DC as indicated in UE-NR-Capability,
> - the value range of pdcch-BlindDetectionMCG-UE or of pdcch-BlindDetectionSCG-UE is [1, 2, 3], and
> - pdcch-BlindDetectionMCG-UE + pdcch-BlindDetectionSCG-UE >= $N_{cells}^{DL,NR-DC,max}$.

### 📚 增量术语表
* **$N_{cells}^{DL,NR-DC,max}$**：最大支持双连接下行小区总数。**一句话解释**：如果没有上报专门的盲检总数参数，这代表手机硬件最多能同时吃下几个下载通道（频段），通常高端机可能是 4 个甚至更多。

### 🚪 第一关：手机在填报“局部上限”时的数值范围约束
上一段讲了手机可以主动向基站上报 MCG 和 SCG 各自的局部算力天花板。
这段协议直接对手机填报的**数值**做出了严格的代数约束，分为两类手机：

**1. 上报了全局总算力 (`pdcch-BlindDetectionCA`) 的手机：**
* **个体范围**：MCG 或 SCG 任何一边的局部上限，必须在 `1` 到 `总算力-1` 之间。不能填 0（说明不支持双连接了），也不能填满总算力（得给另一边留至少 1 个小区的活路）。

**2. 没上报全局总算力，只报了支持几个物理载波的手机：**
* **个体范围**：被死死限制在 `[1, 2, 3]` 这三个数值里。为什么？因为对于这种没有专门填算力表的手机，协议默认它的双连接总承载力极有可能只有小几个小区，直接物理封顶在 3 个以内，避免基站被误导。

### 🎯 第二关：极其反直觉的“大于等于 (>=)”定式！
这是本段协议的精髓！请看这两个公式：
* 局部上限M + 局部上限S **>=** 总算力！

等一下，我们之前不是说“分配的配额加起来不能超过总算力（**<=**）”吗？
**注意区分：**
* 之前讲的 **<=**，是**基站实际给你派活（配置参数）**时，加起来不能超过你的极限。
* 这里讲的 **>=**，是**手机在汇报自身硬件能力上限（Capability）**时，两个局部上限加起来，必须**溢出**你的全局总算力！

**为什么要“溢出”？（物理意义与资源池共享）**
这是为了给基站的调度器提供**极大的动态弹性空间**！
假设手机的总算力是 10。如果手机上报 MCG 上限是 5，SCG 上限是 5（5+5=10）。那么基站**唯一**的分配方式只能是：MCG 5 个，SCG 5 个。基站想多给 MCG 分配一点都不行。
如果手机上报 MCG 上限是 8，SCG 上限是 7（8+7=15 >= 10，成功溢出！）。
这就意味着手机的两个硬件模块实际上是有很多**共享资源池**的。这种汇报方式告诉基站：“我总共只能接 10 个活，但你可以**非常灵活**地调配！你可以给主基站派 8 个，辅基站派 2 个；也可以给主基站 3 个，辅基站 7 个。只要你保证最后总和是 10 就行！”

**生活类比**：
一个大型宴会厅（UE 总能力，可容纳 100 席）。
它有东厅（MCG 硬件）和西厅（SCG 硬件），中间有**活动屏风**。
你向公安局（基站）报备安全容量：
* 你绝不能说：“东厅最多容纳 50 席，西厅最多容纳 50 席。”（太死板，客人来了没法动态调整）。
* 协议要求你必须这么报：“我的东厅拉开屏风最多能容纳 80 席，我的西厅拉开屏风最多能容纳 70 席。**但是（总控）无论你怎么摆，整个宴会厅总数绝对不能超过 100 席！**”
这里的 $80+70 \ge 100$，就是本段协议里极其高明的动态余量设计。

### 🔄 局部上限“溢出汇报”逻辑图
```text
【手机向网络提交的能力清单】

  全局总能力 (pdcch-BlindDetectionCA) = 10 

  ├─ 填报 MCG 局部能力 (pdcch-BlindDetectionMCG-UE) = 8
  └─ 填报 SCG 局部能力 (pdcch-BlindDetectionSCG-UE) = 7

【协议核查机制】
  检查一：都在 [1 到 10-1=9] 的范围内吗？ ──► (8和7都符合) ✅
  检查二：8 + 7 >= 10 成立吗？ ──────────────► (15 >= 10) ✅

(手机合规！这种“加起来大于总数”的上报，为基站跨基站资源分配赋予了极强的弹性)
```

### 💡 章节硬核总结

**本段协议在 UE 能力上报规则中，植入了极具动态统筹哲学的 L1 约束不等式：$\Sigma (SubCapacity_{max}) \ge Capacity_{total}$。在 NR-DC 架构下，终端的基带处理引擎往往存在跨 DSP core/Thread 的软硬件资源复用池（Resource Pooling）。如果硬性规定局部能力上限之和等于全局总能力，将导致 RRC 配置被锁死在唯一的切分比例上，抹杀了调度器的拓扑弹性。协议通过强制要求两个 Sub-Capacity 的代数和必须大于或等于全局 Capacity，在信令层面揭示并映射了基带硬件底层的软共享（Soft-sharing）边界。这给予了网络侧在分配 $N_{cells}^{MCG}$ 和 $N_{cells}^{SCG}$ 权重时充足的游走空间（Sliding window），只要最终下发的实际配置之和满足上一节的 $\le$ 总容量红线即可。**

## 📖 协议原文拆解 (二十七)：极速模式下的“双黄线”与严格体检 (R16 算力申报)

> **协议原文**
> If a UE indicates in UE-NR-Capability a carrier aggregation capability larger than two downlink cells, the UE includes in UE-NR-Capability an indication for a maximum number of PDCCH candidates and a maximum number of non-overlapped CCEs that the UE can monitor per span when the UE is configured for carrier aggregation operation over more than two downlink cells with monitoringCapabilityConfig = r16monitoringcapability. When a UE is not configured for NR-DC operation and the UE is provided monitoringCapabilityConfig = r16monitoringcapability for all downlink cells where the UE monitors PDCCH, the UE determines a capability to monitor a maximum number of PDCCH candidates and a maximum number of non-overlapped CCEs per span that corresponds to $N_{cells}^{cap-r16}$ downlink cells, where
> - $N_{cells}^{cap-r16}$ is the number of configured downlink cells if the UE does not provide pdcch-MonitoringCA
> - otherwise, $N_{cells}^{cap-r16}$ is the value of pdcch-MonitoringCA

### 📚 增量术语表
* **r16monitoringcapability**：R16 盲检能力。**一句话解释**：之前提过的 URLLC 超低延迟模式，算账周期从“按天结 (per slot)”变成了“按极短的小时结 (per span)”。
* **pdcch-MonitoringCA**：R16 专属载波聚合能力值。**一句话解释**：手机针对极速 Span 模式专门上报的“瞬时爆发算力上限”，对应常规模式下的 `pdcch-BlindDetectionCA`。
* **$N_{cells}^{cap-r16}$**：R16 模式下的总盲检天花板。**一句话解释**：在这个极短的 Span 时间内，手机芯片最多能同时抗住几个当量小区的并发解密任务。

### 🚪 第一关：警戒线大幅收紧！从“超 4”降到“超 2”
还记得我们在（二十二）段拆解的常规模式吗？在传统的“按天结 (per slot, R15)”模式下，手机同时连 **4 个**载波以上，协议才要求它必须上报算力专项体检报告。

但这段协议专门针对**极速模式 (R16, per span)** 制定了新规矩：警戒线直接对半砍！
**只要手机吹牛说自己能同时支持超过 2 个下行小区 (larger than two downlink cells)**，它就必须立刻在简历里附上它在 Span 极短时间内的极限能力（最多解几张纸条、看几个格子）。

**为什么要收紧警戒线？（物理意义）**
因为 R16 的 per span 是针对工业控制、自动驾驶等高压业务设计的。在一个极短的 Span（比如只有 2 个 OFDM 符号，几十微秒）内，如果要同时并发处理 3 个小区传来的 DCI，对底层解码芯片（Polar译码器）瞬时吞吐量的压榨是极其恐怖的，瞬间功耗极大，极易导致硅片“热熔断”。所以，警戒线必须极其严苛。

**生活类比**：
常规业务（R15 per slot）就像是**工地搬砖**，要求一天交一次工。老板（基站）给你派 4 个工地的活，你还可以自己统筹安排、偷偷摸鱼，超过 4 个工地才需要专项审批。
极速业务（R16 per span）就像是**高危拆弹**，要求几分钟内必须搞定！只要老板让你同时盯**超过 2 个**定时炸弹，你就必须立刻向上级提交书面的《脑力极限测试报告》，明确说出你在几分钟内到底能剪断几根线，绝不能盲目接单！

### 🎯 第二关：核定极速模式下的算力天花板
如果手机运行在普通组网（非双连接），基站该怎么确定它在 Span 模式下的算力天花板 $N_{cells}^{cap-r16}$ 呢？
协议给出了和 R15 非常相似的判定树，但专属参数换了：
1. **老实填了专项报告的**：如果手机明确上报了 `pdcch-MonitoringCA`，那太好了，基站直接把这个值作为排班的天花板（otherwise 分支）。
2. **没填专项报告的**：基站直接把“当前给你配置了几个实际下行物理小区”，作为你的天花板。比如给你配了 3 个小区，那你就是 3。基站在排班时不能超过这个量。

### 🔄 R16 极速模式下的算力核算树
```text
【一台手机入网，声明支持 3 个下行载波 (CA > 2)，且激活了 per-span 极速模式】

 触发高危警戒线！手机必须在 UE-NR-Capability 中上报极速盲检上限！

 基站开始核算它的极速承载能力 N_cells_cap_r16：
        │
        ├─► 手机在报告里主动填了专属参数 pdcch-MonitoringCA 吗？
        │
        ├──► 【填了，值为 4】
        │       └─► 算力天花板锁定 = 4！基站在单次 Span 派发的任务量绝不能超标。
        │
        └──► 【没填】
                └─► 启动保守回退机制：当前给你实际配了几个小区？
                    配了 3 个，那天花板锁定 = 3！
```

### 💡 章节硬核总结

**本段协议是 3GPP 针对 Rel-16 URLLC 特性（`monitoringCapabilityConfig = r16monitoringcapability`）在 L1 物理层引入的专门化 Overbooking 防御机制。相较于 eMBB per-slot 盲检（其 CA 门限为 >4 cells），URLLC per-span 盲检的 L1 基带瞬时峰值处理压力（Peak Processing Payload）呈指数级上升。因此，协议在 RRC 能力信令中采取了更为严苛的预防性截断（Preventive Truncation）：强制将显式能力上报门限降低至 `CA > 2`。同时，独立开辟了专门用于衡量亚时隙（Sub-slot）算力深度的标量 $N_{cells}^{cap-r16}$ 及其对应的参数 `pdcch-MonitoringCA`。这种在协议规格上对业务类型物理特性的极致隔离，是 5G 能够在同一套空口协议上同时完美支撑大带宽与极低延迟的前提。**

## 📖 协议原文拆解 (二十八)：极速双连接下的“专项资产分割” (R16 NR-DC 分配)

> **协议原文**
> When a UE is configured for NR-DC operation and the UE is provided monitoringCapabilityConfig = r16monitoringcapability for all downlink cells where the UE monitors PDCCH, the UE determines a capability to monitor a maximum number of PDCCH candidates and a maximum number of non-overlapped CCEs per span that corresponds to
> - $N_{cells}^{cap-r16} = N_{cells,r16}^{MCG}$ downlink cells for the MCG where $N_{cells,r16}^{MCG}$ is provided by pdcch-BlindDetection2 for the MCG, and
> - $N_{cells}^{cap-r16} = N_{cells,r16}^{SCG}$ downlink cells for the SCG where $N_{cells,r16}^{SCG}$ is provided by pdcch-BlindDetection2 for the SCG

### 📚 增量术语表
* **pdcch-BlindDetection2**：R16 第二代 PDCCH 盲检能力分配值。**一句话解释**：这是为了极速模式（per span）专门发明的新参数。和之前普通模式用的 `pdcch-BlindDetection` 彻底分开，专款专用。
* **$N_{cells,r16}^{MCG}$ / $N_{cells,r16}^{SCG}$**：极速模式下主/辅基站各自的配额。**一句话解释**：主基站和辅基站各自拿到的一笔“高爆发算力预算”。

### 🚪 第一关：新业务必须用“新账本”
在前面的（二十三）段，我们学过手机如果在双连接（NR-DC）模式下，算力必须在 MCG 和 SCG 之间分家，当时用的是 `pdcch-BlindDetection` 这个老参数。
但现在手机接的是**极速、超低延迟的 URLLC 业务**（`r16monitoringcapability`，按 Span 极短时间结账）。
协议的原则是：“专款专用，账本解耦”。
对于极速模式的分家，网络不能再用老参数来分了，必须下发一个全新的参数——**`pdcch-BlindDetection2`**（注意后面的那个数字 "2"，它标志着这是专属于 R16 Span 模式的第二代分配指令）。

### 🎯 第二关：双基站“极速算力”的物理隔离
手机根据基站下发的这个“第二代账本”，明确算出自己在极短的瞬时时间内（Span），对两边基站的极限抗压能力：
* **主基站 (MCG) 侧**：天花板 $N_{cells}^{cap-r16}$ 锁定为网络通过 `pdcch-BlindDetection2` 专门批给 MCG 的配额。
* **辅基站 (SCG) 侧**：天花板 $N_{cells}^{cap-r16}$ 锁定为网络通过 `pdcch-BlindDetection2` 专门批给 SCG 的配额。

**为什么要分出 "第二代账本"？（物理意义）**
这是由于硬件架构的异构性决定的。一部 5G 手机，它用来处理常规业务（一天处理一回，长时延）的译码器集群，和它用来处理极速业务（几十微秒必须出结果，高爆发）的硬件加速器（Accelerators）甚至 SRAM 缓存大小是完全不同的。
因此，手机必须在软件信令层面（RRC）拥有**两套完全独立切分的算力水池**。网络可以给 MCG 的常规业务分配很大，但给 MCG 的极速业务分配得很小，互不干扰，防止底层调度出现逻辑灾难。

### 🔄 R16 极速双连接算力切分图
```text
【手机处于 NR-DC (双基站) 且全网开启 URLLC 极速盲检 (per Span)】

      网络下发第二代算力切分方案 (pdcch-BlindDetection2)
                     │
         ┌───────────┴───────────┐
         ▼                       ▼
   [拨给 主基站 MCG]         [拨给 辅基站 SCG]
         │                       │
 锁定为 MCG 的极速天花板     锁定为 SCG 的极速天花板
 (MCG 瞬时盲检绝不可越线!)  (SCG 瞬时盲检绝不可越线!)

*(注：这套极速天花板与之前的常规天花板并行存在，手机底层硬件同时监控两套红线)*
```

### 💡 章节硬核总结

**本段协议在 5G NR-DC (Multi-Radio Dual Connectivity) 架构下，针对 Rel-16 的亚时隙级（Sub-slot/per-span）PDCCH 盲检特性，完成了一次教科书级的参数解耦（Parameter Decoupling）。为了在双节点并发调度的恶劣环境中保护 UE 基带中极高功耗、极低延迟的专属硬件加速器流水线不被瞬间冲垮，协议在此引入了全新域名的 IE 字段 `pdcch-BlindDetection2`。该字段独立于传统的 per-slot 切分参数，专门用于在 RRC 层静态划分 MCG 与 SCG 在微秒级 Span 时间窗内的 Peak Decoding Capacity 预算。这一分离设计极大地增强了网络规划的前向兼容性，使得运营商可以对不同 Cell Group 承载 eMBB 与 URLLC 业务的能力进行完全正交的精细化配置。**

## 📖 协议原文拆解 (二十九)：极速模式下的最后通牒 (R16 算力红线警告)

> **协议原文**
> When the UE is configured for carrier aggregation operation over more than 2 cells, or for a cell group when the UE is configured for NR-DC operation, the UE does not expect to monitor per span a number of PDCCH candidates or a number of non-overlapped CCEs that is larger than the maximum number as derived from the corresponding value of $N_{cells}^{cap-r16}$.

### 📚 增量术语表
* **$N_{cells}^{cap-r16}$**：R16 极速模式算力天花板。**一句话解释**：手机在几十微秒的瞬间（一个 Span），最多能同时抗住几个小区的盲检任务量。
* **does not expect**：不预期（最高警告）。**一句话解释**：物理层给基站排班系统画的“高压电网”。基站若敢越过此线，手机直接罢工。

### 🚪 第一关：收网！所有极速算力合并入“红线”判决
前两段（27段和28段），协议分别教了在**单基站多载波（CA>2）**和**双基站（NR-DC）**两种复杂情况下，手机该怎么算出这个极其金贵的极速天花板参数 $N_{cells}^{cap-r16}$。
这段话是对这两部分内容的一个**终极法律收口**。

### 🎯 第二关：“微秒级”的罢工权 (Per Span 保护)
和常规模式（Per Slot，一天算一次账）不同，这里强调的是在**极短的瞬间（per span）**内的绝对保护。
协议向基站下达最后通牒：
不论你是通过载波聚合（CA）给手机塞了超多频段，还是通过双基站（NR-DC）给手机两头派活。只要你配置的是极速模式（R16），**在任何一个极短的 Span 瞬间内**，你实际丢给手机去解密的纸条总数（Candidates）和扫视的黑板总面积（CCEs），**绝对不允许**超过从 $N_{cells}^{cap-r16}$ 算出来的理论物理极限值！

**物理意义**：
在微秒级的时间刻度上，电子芯片是没有“喘息”空间的。如果基站排班出错，导致某个瞬间让手机解码的次数超过了其内部译码器阵列（Decoder Array）的物理并行上限，会导致数据堆积在寄存器里溢出，直接造成整个 URLLC 链路的灾难性断裂。
`does not expect` 赋予了手机在这种极端压榨下**直接抛弃超额任务**的自卫权利。

### 🔄 极速模式防过载终极逻辑环
```text
【URLLC 极速盲检 (Per Span) 过载防御机制】

 算出手机的极速天花板 N_cells_cap_r16 (无论是算出来的，还是网络下发的配额)
        │
        ▼
 【进入实战盲检：在一个仅包含比如 2 个符号的极短 Span 内】
        │
        ├─► 基站实际派发的 纸条总数(Candidates) 超标了吗？
        │      ├──► 超标 ──► 🚨 触发 [does not expect] 罢工异常！
        │      └──► 没超 ──► 继续检查
        │
        └─► 基站实际派发的 黑板面积(CCEs) 超标了吗？
               ├──► 超标 ──► 🚨 触发 [does not expect] 罢工异常！
               └──► 没超 ──► ✅ 安全！手机芯片火力全开，成功解析本次极速调度。
```

### 💡 章节硬核总结

**本段协议是整个 5G Rel-16 URLLC (Ultra-Reliable Low-Latency Communication) 物理层在控制信道维度的最终防线（Ultimate Failsafe Boundary）。在将时域颗粒度从毫秒级的 Slot 暴力压缩至亚毫秒（甚至是几十微秒）级的 Span 后，基站调度器（Scheduler）极易因为多载波（CA > 2）或跨节点（NR-DC）调度的非完美协同，引发瞬时盲检负荷越限（Instantaneous Overbooking）。在此，3GPP 再次亮出了无敌的 `UE does not expect` L1 原语，强行要求基站侧的调度算法必须具备前向资源预算（Forward Resource Budgeting）能力，确保在任何物理 Span 切片内的 Candidates 与 CCEs 积分值严守 $N_{cells}^{cap-r16}$ 衍生出的数学极限。这不仅免去了终端侧实现复杂丢弃逻辑的成本，更从根本上保障了低延迟业务不可妥协的可靠性要求。**

## 📖 协议原文拆解 (三十)：极速双连接的“算力守恒定律” (R16 配额合规检查)

> **协议原文**
> When a UE is configured for NR-DC operation with a total of $N_{cells}^{DL,NR-DC}$ downlink cells on both the MCG and the SCG and the UE is provided monitoringCapabilityConfig = r16monitoringcapability for all downlink cells where the UE monitors PDCCH, the UE expects to be provided pdcch-BlindDetection2 for the MCG and pdcch-BlindDetection2 for the SCG with values that satisfy
> - pdcch-BlindDetection2 for the MCG + pdcch-BlindDetection2 for the SCG <= pdcch-MonitoringCA, if the UE reports pdcch-MonitoringCA, or
> - pdcch-BlindDetection2 for the MCG + pdcch-BlindDetection2 for the SCG <= $N_{cells}^{DL,NR-DC}$, if the UE does not report pdcch-MonitoringCA

### 📚 增量术语表
* **monitoringCapabilityConfig = r16monitoringcapability**：全网极速盲检模式。**一句话解释**：手机同时连着主辅两个基站，而且基站明确要求手机在所有下行通道上，全部挂上“按小时结（per span）”的高爆发档位。

### 🚪 第一关：旧壶装新酒，极速模式下的“家产分割法”
在第（二十四）段，我们拆解过普通模式（per slot）下双基站如何分算力：主辅基站的份额加起来不能超过总天花板。
这段协议的逻辑和（二十四）段**完全一模一样**，唯一的区别是：**参数全部换成了针对极速模式（R16 URLLC）的“第二代专属参数”。**

协议再次强调了基站下发极速算力配额时的**“能量守恒定律”**：
基站给主基站（MCG）发的极速配额 (`pdcch-BlindDetection2`) 加上给辅基站（SCG）发的极速配额 (`pdcch-BlindDetection2`)，**加在一起的总和，绝对不能把手机撑爆！**

### 🎯 第二关：极速天花板到底多高？(两条分支)
这个防爆的天花板怎么定？依然是看手机入网时递交的简历：
1. **写了极速专项体检报告的** (If UE reports `pdcch-MonitoringCA`)：
   如果手机明确填了专属的极速模式天花板参数，那么：
   `MCG 第二代配额 + SCG 第二代配额 必须 <= 这个专项上报值`。

2. **没写极速专项体检报告的** (If UE does not report...)：
   如果手机偷懒没填，那基站就用最简单粗暴的方法：当前手机总共连了几个物理频段（$N_{cells}^{DL,NR-DC}$），天花板就是几。
   `MCG 第二代配额 + SCG 第二代配额 必须 <= 总物理频段数`。

**生活类比**：
你在双轨制赛车俱乐部打工。
以前开普通赛车（普通模式 pdcch-BlindDetection），老板给你主辅两个赛道分的任务量总和不能超过你的普通驾照级别。
现在你升级到了**F1极速赛车**（极速模式 pdcch-BlindDetection2）。老板给你主辅两个极速赛道分的任务量，总和同样绝对不能超过你的**F1高级驾照级别（pdcch-MonitoringCA）**！如果你没考高级驾照，那老板就按你现在租了几条赛道来死板地给你设上限。

### 🔄 R16 极速模式配额合规性检查图
```text
【全网开启 R16 URLLC 极速模式下的双连接 (NR-DC) 配额下发】

         ├─► [MCG 第二代配额] = A
         ├─► [SCG 第二代配额] = B
         │
【手机物理层底层拦截与自检 (UE Expects)】
         │
         ├─ 算出极速总负荷: Sum = A + B
         │
         ├─ 寻找我的极速天花板 Limit：
         │    ├── 我上报过专属极速能力 pdcch-MonitoringCA 吗？ ──► 有，天花板就是它
         │    └── 没上报过？ ───────────────────────────────► 没，天花板就是实际总频段数
         │
         └─► 【进行生死判定】: Sum 必须 <= Limit！
             (基站若违反此极速守恒定律，手机基带将拒绝应用该配置)
```

### 💡 章节硬核总结

**本段协议在数学逻辑上完美复刻了基础 eMBB (per-slot) 场景下的双连接资源守恒方程，但其作用域被严格且精细地隔离在了 Rel-16 的 Sub-slot (per-span) 高维空间内。在异构网络调度中，物理层参数的名称空间隔离（Namespace Isolation）至关重要。通过使用带有 `2` 后缀或专属 `r16` 标记的新 IE (Information Element) 字段，3GPP 确保了基站侧的 RRC 配置器 (Configurator) 与终端侧的基带核算状态机 (Accounting State Machine) 能够在 eMBB 和 URLLC 两种完全不同特征的时域尺度上，分别运行互不干扰的 Overbooking 校验准则。这种将宏观拓扑逻辑与微观物理参量正交解耦的设计，是 5G 协议能够像搭积木一样无限平滑演进的底层密码。**

## 📖 协议原文拆解 (三十一)：极速模式下的“偏科”上报 (R16 局部上限声明)

> **协议原文**
> When a UE is configured for NR-DC operation and the UE is provided monitoringCapabilityConfig = r16monitoringcapability for all downlink cells where the UE monitors PDCCH, the UE may indicate, through pdcch-BlindDetectionMCG-UE-r16 and pdcch-BlindDetectionSCG-UE-r16, respective maximum values for pdcch-BlindDetection for the MCG and pdcch-BlindDetection for the SCG.

### 📚 增量术语表
* **pdcch-BlindDetectionMCG-UE-r16 / pdcch-BlindDetectionSCG-UE-r16**：R16 版终端期望的主/辅基站盲检能力上限。**一句话解释**：手机针对极速 Span 模式，专门向基站亮出的“我分给主基站的极速处理模块最大有多强”、“我分给辅基站的极速处理模块最大有多强”的两张底牌。

### 🚪 第一关：旧套路，新马甲 (极速硬件的局部瓶颈)
这段话读起来也非常眼熟。在第（二十五）段中，我们见过普通模式下手机上报局部瓶颈的机制（MCG-UE 和 SCG-UE）。
现在到了极速模式（R16 per span），逻辑是完全映射的，只是加上了 **`-r16`** 的专属后缀。
由于处理极速 URLLC 业务的硬件加速器（Accelerators）在手机芯片内部往往是独立划分资源的。有些手机可能给主基站（MCG）分配了大量的极速译码核心，而给辅基站（SCG）只留了一点点。
为了防止网络在派发极速任务时“偏心偏到短板上”，手机必须（`may indicate`）主动交代这些极速硬件模块的真实局部上限。

### 🎯 第二关：双轨制并行上报的严谨性
请注意协议在这里的一个小瑕疵/细节：
原文后半句写的是 `respective maximum values for pdcch-BlindDetection...`。按照严谨的协议演进逻辑，这里其实应该是指约束前文刚刚定义的第二代参数 `pdcch-BlindDetection2`。
但核心精神是明确的：**只要是在极速 `r16` 语境下，手机上报的所有局部瓶颈约束，都是通过带有 `-r16` 后缀的专属 RRC 信令来完成的。**
这保证了普通业务和极速业务在上报自己“能吃几碗饭”时，用的是完全不同的两张表，互不干扰。

**生活类比**：
你在双轨制公司（NR-DC）打工。
以前送普通快递（普通盲检），你报过一次工时底线：“我最多给东区（MCG）跑 7趟，西区（SCG）跑 5趟。”
现在公司开通了“10分钟加急闪送”（R16 极速盲检）。
你不能用以前的底线来糊弄，你必须专门递交一份《闪送体力极限量表（带 -r16 后缀）》：“报告老板，如果是跑这种要命的加急件，我给东区最多跑 4趟，西区最多跑 2趟。请按这个新的上限给我派加急件！”

### 🔄 极速模式双基站配额的局部枷锁图
```text
【手机的 URLLC (极速) 硬件资源池划分】

   [极速总算力池 pdcch-MonitoringCA]

   ├─► [MCG 极速硬件切片] ── 上报局部瓶颈 pdcch-BlindDetectionMCG-UE-r16
   │
   └─► [SCG 极速硬件切片] ── 上报局部瓶颈 pdcch-BlindDetectionSCG-UE-r16

【基站派发极速任务时的合规审查】
  基站给 MCG 派的极速活，绝不能超过手机上报的 MCG-r16 瓶颈！
  基站给 SCG 派的极速活，绝不能超过手机上报的 SCG-r16 瓶颈！
```

### 💡 章节硬核总结

**本段协议在 RRC 信令层面彻底补齐了 Rel-16 极速盲检架构（per-span PDCCH monitoring）的能力上报闭环。正如常规模式下的 `MCG-UE` 和 `SCG-UE` 保护了 eMBB 译码流水线，带 `-r16` 后缀的全新 capability indicators 保护了 UE 内部极其昂贵的低延迟硬件资源（如专门用于极速极化译码的 Fast-Polar Decoder 实例）。这种彻底的 Namespace（命名空间）分离，使得 UE 可以向网络如实反映其底层硅片在处理常态流量与短脉冲流量时完全异构的瓶颈分布（Bottleneck distribution），从而避免了基站在 NR-DC 这种缺乏毫秒级协同的松耦合架构下，意外打爆某一个特定无线电承载的瞬时算力。**

## 📖 协议原文拆解 (三十二)：极速模式下的“溢出汇报”与极其严苛的缺省惩罚

> **协议原文**
> If the UE reports pdcch-MonitoringCA,
> - the value range of pdcch-BlindDetectionMCG-UE-r16 or of pdcch-BlindDetectionSCG-UE-r16 is [1, …, pdcch-MonitoringCA-1], and
> - pdcch-BlindDetectionMCG-UE-r16 + pdcch-BlindDetectionSCG-UE-r16 >= pdcch-MonitoringCA.
>
> Otherwise, if $N_{cells}^{DL,NR-DC,max,r16}$ is a maximum total number of downlink cells for which the UE is provided monitoringCapabilityConfig = r16monitoringcapability and the UE is configured on both the MCG and the SCG for NR-DC as indicated in UE-NR-Capability
> - the value of pdcch-BlindDetectionMCG-UE-r16 or of pdcch-BlindDetectionSCG-UE-r16 is 1,
> - pdcch-BlindDetectionMCG-UE-r16 + pdcch-BlindDetectionSCG-UE-r16 >= $N_{cells}^{DL,NR-DC,max,r16}$.

### 📚 增量术语表
* **$N_{cells}^{DL,NR-DC,max,r16}$**：极速双连接最大下行物理小区总数。**一句话解释**：如果没有上报极速算力总值，这就是手机在极速模式（R16 per span）下，双连接所能挂载的物理天线频段总数。

### 🚪 第一关：旧瓶装新酒，极速资源池的“动态溢出”
这段话的前半部分，和我们在第（二十六）段讲的常规模式上报规则在数学形式上一模一样。
对于那些主动上报了“极速总算力（`pdcch-MonitoringCA`）”的好学生：
* 个体配额不能是 0，也不能独占全部（给另一边留条活路）。
* **最核心的溢出定理**：`MCG极速上限 + SCG极速上限 >= 极速总算力`！
这个大于等于（$\ge$）再次证明了：即使是极速硬件，手机芯片内部的主辅基站模块依然存在**资源池共享（Resource Pooling）**。通过溢出上报，赋予了基站跨基站动态倾斜分配极速任务的灵活性。

### 🎯 第二关：极其严苛的“缺省惩罚” (数值被死死锁在 1)
协议的后半部分（Otherwise分支）露出了獠牙！请对比一下常规模式：
* 常规模式下（第26段），如果没报总能力，局部上限的范围好歹还是 `[1, 2, 3]`。
* **在极速模式下（本段），如果没报总能力，局部上限的值直接被死死锁死在 `1`！没得选！**

**为什么要施加这么惨烈的惩罚？（物理意义）**
因为 `r16monitoringcapability`（极速模式）实在太消耗底层硬件资源了！在几十微秒的 Span 时间窗内解密控制信道，对芯片的热量和吞吐量压榨到了极限。
如果你连一份完整的《极速总算力评估报告》都不交，协议绝对不敢对你有任何高估。为了防止你的基带瞬间烧毁，协议直接对你启动**最底线的自我保护**：无论主基站还是辅基站，你各自局部的最大承载力**强制核定为 1 个当量小区**。哪怕你物理上连了两个频段（1+1 $\ge$ 2 满足不等式），单个基站也绝不能给你派发超过 1 个小区的极速并发任务！

**生活类比**：
（常规货运）：你不报总装载量，交警看你的车型，允许你在 [1吨, 2吨, 3吨] 里挑一个局部上限来报备。
（运输烈性炸药 - R16 极速模式）：如果你不提交《特种运输车抗爆总容量认证》，交警直接一刀切：前车厢（MCG）最多只能装 1 箱，后车厢（SCG）最多只能装 1 箱。绝对不允许你报 2 甚至 3，宁可一趟趟跑，也绝对不承担爆炸（芯片烧毁）的风险！

### 🔄 极速模式下的局部算力上报决议树
```text
【手机准备上报极速模式 (R16) 的双连接局部算力 MCG-r16 和 SCG-r16】

        ├─► 有没有上报总的极速算力 pdcch-MonitoringCA？
        │
        ├──► 【有报 (比如报了 4)】
        │       └─► 范围宽松：MCG和SCG可以在 [1, 2, 3] 里选。
        │           条件：两者相加 >= 4 (比如报 2+3 或者 3+2 都可以，提供弹性)
        │
        └──► 【没报】
                └─► 触发底线极速惩罚！
                    范围锁死：MCG-r16 只能报 1，SCG-r16 只能报 1！
                    条件：1 + 1 >= 物理小区总数 (意味着此时总物理小区最多只能是 2 个)
```

### 💡 章节硬核总结

**本段协议在 5G URLLC 极速盲检框架下，完美闭环了 NR-DC 架构的局部能力上报机制，并展示了 3GPP 协议极为谨慎的“Failsafe (防故障)”降级设计。前半段的不等式 $\Sigma (SubCapacity_{max}) \ge Capacity_{total}$ 继续发扬了软资源池共享的调度弹性。而后半段的 Fallback 机制堪称严厉：在缺失明确的极速全局预算 `pdcch-MonitoringCA` 时，协议直接褫夺了 UE 局部能力上报的灵活性，将其强行收敛常量 `1`。这从 L1 数学模型的根源上，物理切断了网络侧在信息不对称情况下向未知极速算力的终端下发高密度并发 DCI 的路径，以最粗暴但最有效的方式保全了基带芯片在极限时序下的稳定性。**

## 📖 协议原文拆解 (三十三)：极高频/低功耗模式下的“摸鱼上限” (R17 算力申报)

> **协议原文**
> If a UE indicates in UE-NR-Capability a carrier aggregation capability larger than four downlink cells, the UE includes in UE-NR-Capability an indication for a maximum number of PDCCH candidates and a maximum number of non-overlapped CCEs that the UE can monitor per group of $X_s$ slots when the UE is configured for carrier aggregation operation over more than four downlink cells for which the UE is provided monitoringCapabilityConfig = r17monitoringcapability. When a UE is not configured for NR-DC operation for all downlink cells where the UE monitors PDCCH, the UE determines a capability to monitor a maximum number of PDCCH candidates and a maximum number of non-overlapped CCEs per group of $X_s$ slots that corresponds to $N_{cells}^{cap-r17}$ downlink cells, where
> - $N_{cells}^{cap-r17}$ is $N_{cells,0}^{DL}+R \cdot N_{cells,1}^{DL}$ if the UE does not provide pdcch-MonitoringCA-r17 where $N_{cells,0}^{DL}+N_{cells,1}^{DL}$ is the number of configured downlink serving cells
> - otherwise, $N_{cells}^{cap-r16}$ is the value of pdcch-MonitoringCA-r17

*(注：协议原文最后一行的 `$N_{cells}^{cap-r16}$` 应为笔误，逻辑上及根据前文推断，此处必然为 `$N_{cells}^{cap-r17}$`。)*

### 📚 增量术语表
* **r17monitoringcapability**：R17 盲检能力。**一句话解释**：之前提过的“低碳养生版/极高频版”模式，算账周期变成了“按周结 (per group of $X_s$ slots)”，允许把活摊在好几天里慢慢干。
* **pdcch-MonitoringCA-r17**：R17 专属载波聚合能力值。**一句话解释**：手机针对“按周结”模式专门上报的算力上限（区别于普通版的按天结和 R16 的按小时结）。
* **$N_{cells}^{cap-r17}$**：R17 模式下的总盲检天花板。**一句话解释**：在这个“养生”大周期内，手机最多能揽下几个当量小区的合并解密任务。

### 🚪 第一关：警戒线回归！重新变回“超 4 申报”
还记得在 R16 的极速模式（按小时结，第27段）里，协议由于担心芯片爆表，极其严厉地把申报红线收紧到了“大于 2 个小区”吗？
现在到了 R17 的“养生模式（按周结）”，由于工作节奏大幅放缓，芯片有充足的时间慢慢消化数据，所以**协议的警戒线重新放宽，回到了常规水平：“大于 4 个小区”。**

只有当手机吹牛说自己能同时支持超过 4 个下行小区，且配置了 R17 “按周结”的模式时，才强制要求你在简历里附上这份专属的《低频次长周期脑力评估报告》。

### 🎯 第二关：核算“按周结”模式的天花板
如果在非双连接的普通聚合网络下，基站怎么算这个 $N_{cells}^{cap-r17}$ 呢？
这里的逻辑和普通 R15 模式（第22段）**完全一模一样，连折算公式都没变**，只是套了个 `-r17` 的马甲：
1. **老实填了专属报告的** (otherwise分支)：按你填的 `pdcch-MonitoringCA-r17` 算！
2. **没填专属报告的** (第一分支)：启用祖传公式！用 单塔小区数 ($N_{cells,0}$) 加上 折算系数 $R$ 乘以 双塔小区数 ($N_{cells,1}$) 硬算出来！

**生活类比**：
（R16 极速计件工）：因为工作强度大到离谱，超过 2 个厂子的活就得严查极限体力，算工资也是独立一套苛刻算法。
（R17 佛系长工）：因为是按周交工，压力小，又恢复了“超过 4 个厂子才需要专项审批”的宽松政策。而且如果不主动填表，老板核算你“一周能干多少活”的公式，和核算普通员工（R15）的公式完全一样（单倍工资 + 协作双倍工资），依然考虑了双塔协作的膨胀折算系数 R。

### 🔄 R17 养生模式算力核算树
```text
【手机入网，声明支持 5 个下行载波 (CA > 4)，且激活了 per group of Xs slots 模式】

 触发警戒线！手机必须在简历中上报 R17 专属的跨时隙盲检上限！

 基站开始核算它的宏观承载能力 N_cells_cap_r17：
        │
        ├─► 手机填了专属参数 pdcch-MonitoringCA-r17 吗？
        │
        ├──► 【填了】 ──► 天花板直接锁定为这个填报值！
        │
        └──► 【没填】 ──► 启用折算公式：
                          单塔频段数 + (R系数 × 双塔频段数)
                          算出的结果就是接下来基站在每个 Xs 周期内的排班天花板。
```

### 💡 章节硬核总结

**本段协议补齐了 3GPP 盲检体系的第三块拼图：Rel-17 针对 FR2-2 (极高频) 或 RedCap (低复杂度) 引入的跨时隙组（per group of $X_s$ slots）过载防御机制。与极其严苛的 URLLC (R16 per-span) 不同，R17 机制的物理本质是通过时域拉伸来平摊算力（Time-averaging of computational load），因此其硬件压榨程度远低于 R16。反映在协议规章上，就是其强制显式上报的 CA 门限回退至了与基础 eMBB 相同的宽容度（`CA > 4`），且在其缺省的隐式回退（Fallback）算法中，完美复用了 eMBB 的 $N_{cells,0}^{DL}+R \cdot N_{cells,1}^{DL}$ 线性膨胀公式。这不仅从侧面印证了通信底层物理压力的强弱，也展示了协议在设计“平行宇宙（R15/R16/R17 Namespace）”时保持一致性与特异性平衡的工程美学。**

## 📖 协议原文拆解 (三十四)：养生模式下的最后通牒 (R17 算力红线警告)

> **协议原文**
> When the UE is configured for carrier aggregation operation over more than 4 cells, the UE does not expect to monitor per group of $X_s$ slots a number of PDCCH candidates or a number of non-overlapped CCEs that is larger than the maximum number as derived from the corresponding value of $N_{cells}^{cap-r17}$.

### 📚 增量术语表
* **per group of $X_s$ slots**：每 $X_s$ 个时隙的周期内。**一句话解释**：R17 模式下的算账大周期（按周结）。
* **does not expect**：不预期。**一句话解释**：代表底层 L1 的最高级“罢工权”和防越线保护。

### 🚪 第一关：收网！R17 专属的绝对防线
在之前的（二十三）段和（二十九）段，我们分别看到了 R15（按天结）和 R16（按小时结）模式下的终极“不预期（does not expect）”红线。
毫无悬念，到了 R17（按周结）模式，协议同样要在最后进行**法理收口**。这段话就是那道不容踩踏的红线边界。

### 🎯 第二关：算总账的尺度变了，但底线依然刚硬
当手机被塞了超过 4 个载波，并且开启了 R17 这种“佛系/养生”的跨时隙组盲检模式时。
虽然基站可以把通知单（PDCCH）散布在这连续的 $X_s$ 个时隙（比如一周）里慢慢发。
但是！协议对基站的排班系统下达了最后通牒：
**把这 $X_s$ 个时隙（一整周）里，所有要看的纸条数量（Candidates）和扫过的黑板面积（CCEs）全加起来算个总账，这个总账数字，绝对不允许超过由 $N_{cells}^{cap-r17}$ 算出来的理论物理极限值！**

**物理意义**：
虽然 R17 模式通过“拉长周期”缓解了手机芯片的瞬时并发压力，但这绝不意味着手机的“总内存缓冲池（Buffer Limit）”是无限的。如果基站在这个大周期内塞入的总任务量越限，会导致手机缓存爆满、解码任务无法在周期末尾前清空，进而引发整条基带流水线的崩溃。`does not expect` 原则在此赋予了手机在宏观周期过载时**整体丢弃（Drop）超额任务**的合法权利。

### 🔄 三大模式防过载逻辑的“大一统”
```text
【5G PDCCH 盲检的三大防越权基石】

┌─────────────────┬───────────────┬─────────────────┬──────────────┐
│ 模式 (能力配置) │ 门限 (超限才查) │ 核算周期 (算账频次)│ 终极红线参数 │
├─────────────────┼───────────────┼─────────────────┼──────────────┤
│ R15 (基础 eMBB) │ CA > 4 cells  │ per slot (按天结) │ N_cells_cap  │
├─────────────────┼───────────────┼─────────────────┼──────────────┤
│ R16 (极速 URLLC)│ CA > 2 cells  │ per span (按时结) │ N_cells_cap-r16│
├─────────────────┼───────────────┼─────────────────┼──────────────┤
│ R17 (高频/低功耗)│ CA > 4 cells  │ per Xs slots (按周)│ N_cells_cap-r17│
└─────────────────┴───────────────┴─────────────────┴──────────────┘

共同底线：在各自的核算周期内，实际派发的任务量若超过对应红线，
UE 均启动 [does not expect] 防御机制，拒收超载控制信令！
```

### 💡 章节硬核总结

**本段协议是对 R17 `per group of Xs slots` 盲检架构在 L1 层的最后闭环。它重申了 3GPP 协议设计中的“契约精神（Contractual Principle）”：终端上报的 Capability（即 $N_{cells}^{cap-r17}$）不仅是网络侧资源分配的参考，更是 L1 物理状态机执行（Execution）的强制阻断边界。通过这句原语，协议免除了 UE 基带实现极其复杂的“过载队列积压处理（Overload Queue Management）”逻辑。一旦 gNB 调度器的前向积分算力（Forward Integrating Budget）在特定的 $X_s$ 宏观时域窗内发生溢出，终端 L1 将被合法赋权直接实施物理层信道丢弃（Channel Dropping）。这构成了 eMBB、URLLC、RedCap 三大业务演进在控制面容量控制上的逻辑大一统。**

## 📖 协议原文拆解 (三十五)：佛系双连接的“第四代账本” (R17 NR-DC 分配)

> **协议原文**
> When a UE is configured for NR-DC operation and the UE is provided monitoringCapabilityConfig = r17monitoringcapability for all downlink cells where the UE monitors PDCCH, the UE determines a capability to monitor a maximum number of PDCCH candidates and a maximum number of non-overlapped CCEs per group of $X_s$ slots that corresponds to
> - $N_{cells}^{cap-r17} = N_{cells,r17}^{MCG}$ downlink cells for the MCG where $N_{cells,r17}^{MCG}$ is provided by pdcch-BlindDetection4 for the MCG, and
> - $N_{cells}^{cap-r17} = N_{cells,r17}^{SCG}$ downlink cells for the SCG where $N_{cells,r17}^{SCG}$ is provided by pdcch-BlindDetection4 for the SCG

### 📚 增量术语表
* **pdcch-BlindDetection4**：第四代 PDCCH 盲检配额下发参数。**一句话解释**：专门为了 R17 “按周结（per group of Xs slots）”模式发明的全新 RRC 资源切分账本。
* **$N_{cells,r17}^{MCG}$ / $N_{cells,r17}^{SCG}$**：R17 模式下主/辅基站各自切分到的物理配额。

### 🚪 第一关：换汤不换药的“隔离法则”
这段协议描述的场景是：手机既开启了 NR-DC（双连接，连着俩基站），同时又在这俩基站的所有链路上启用了 `r17monitoringcapability`（R17 的佛系养生模式）。
还记得 R16 极速双连接时（第28段），协议专门发明了一个叫 `pdcch-BlindDetection2` 的“第二代账本”来切分算力吗？

这里的设计哲学**完全一致**。对于 R17 的新业务模型，协议绝不复用以前的旧参数，而是彻底打了个隔离补丁，引入了代号为 **"4"** 的全新参数 `pdcch-BlindDetection4`！
主基站（MCG）和辅基站（SCG）各自拿着这个专属于 R17 的新账本去分配自己的天花板限额，互相物理隔离，互不透支。

### 🎯 第二关：为什么直接跳到了 "4"？ (揭秘 3GPP 的命名彩蛋)
细心的你可能会问：第一代叫 `pdcch-BlindDetection` (R15)，第二代叫 `pdcch-BlindDetection2` (R16极速)，第三代去哪了？为什么这里直接叫 `pdcch-BlindDetection4` (R17养生)？

*这源于 3GPP 协议极为错综复杂的平行版本演进：*
* 第一代 (没有数字)：给 R15 基础的 per slot 用的。
* 第二代 (`2`)：给 R16 的 per span 用的。
* 第三代 (`3`)：其实存在于其他协议扩展里，主要是给 R16/R17 针对 **CA 下不同 $\mu$ (子载波间隔) 的混合调度能力**用的（用来区分高低频混用的折算）。
* 第四代 (`4`)：也就是这里，专门被定义来分配给 R17 的 `per group of Xs slots`（跨时隙组）使用。
**这种把参数名数字序号彻底拉开的写法，展现了极其硬核的命名空间隔离（Namespace Isolation），让设备在解析长达几千行的 ASN.1 信令字典时，绝不会串台！**

### 🔄 NR-DC 下的三代算力切分账本对比图
```text
【双基站 NR-DC 下，基站下发算力配额的“专款专用”体系】

 1. 如果你在跑常规业务 (Per Slot, R15) 
    ──► 基站使用参数 [pdcch-BlindDetection] 进行主辅切分。

 2. 如果你在跑极速业务 (Per Span, R16)
    ──► 基站使用参数 [pdcch-BlindDetection2] 进行主辅切分。

 3. 如果你在跑大周期/佛系业务 (Per group of Xs slots, R17)
    ──► 基站使用参数 [pdcch-BlindDetection4] 进行主辅切分！✅ (本段核心)

(三套账本在 RRC 信令里各自并行，手机底层的三套状态机各自读取，互不干扰)
```

### 💡 章节硬核总结

**本段协议完成了 5G Rel-17 跨时隙组盲检特性（per group of $X_s$ slots）在 NR-DC 网络架构下的最终映射。它再次重申了 3GPP 协议设计中“正交化物理变量必须具有正交化信令容器（Orthogonal Signaling Containers for Orthogonal Physical Variables）”的金科玉律。为了防止 R17 的长时域积分算力被错误地混入 R15 的单日算力或 R16 的微秒算力池中，协议强行启用了 RRC 新元组 `pdcch-BlindDetection4`。通过该独占标识，MCG 与 SCG 分别锚定自身在 R17 时域尺度上的等效调度上限（$N_{cells,r17}^{MCG}$ 和 $N_{cells,r17}^{SCG}$），从而彻底隔绝了非对称双连接部署下潜在的参数污染与跨层调度灾难。**

## 📖 协议原文拆解 (三十六)：R17 双连接配额发放的“总能量守恒”

> **协议原文**
> When a UE is configured for NR-DC operation with a total of $N_{cells}^{DL,NR-DC}$ downlink cells on both the MCG and the SCG and the UE is provided monitoringCapabilityConfig = r17monitoringcapability for all downlink cells where the UE monitors PDCCH, the UE expects to be provided pdcch-BlindDetection4 for the MCG and pdcch-BlindDetection4 for the SCG with values that satisfy
> - pdcch-BlindDetection4 for the MCG + pdcch-BlindDetection4 for the SCG <= pdcch-MonitoringCA-r17, if the UE reports pdcch-MonitoringCA-r17, or
> - pdcch-BlindDetection4 for the MCG + pdcch-BlindDetection4 for the SCG <= $N_{cells}^{DL,NR-DC}$, if the UE does not report pdcch-MonitoringCA-r17

### 📚 增量术语表
* 本段无新增术语。均为 R17 体系下相关参数的组合应用。

### 🚪 第一关：似曾相识的“防爆数学题” (跨代逻辑复用)
读到这里，您应该已经完全掌握了 3GPP 协议的编写套路了！
这一段在数学形式上，与我们在第（二十四）段学过的 R15 常规双连接配额检查、以及在第（三十）段学过的 R16 极速双连接配额检查，**完完全全一模一样**！

唯一的变化是，所有的变量名统统换上了 **R17 的专属套装**：
* 账本名换成了：`pdcch-BlindDetection4`
* 手机上报的总预算表换成了：`pdcch-MonitoringCA-r17`

### 🎯 第二关：守恒定律在 R17 模式下的二次宣示
只要基站全网开启了 R17 的 `per group of Xs slots` (跨时隙大周期) 模式，且手机挂载了 NR-DC 双基站。
协议再次赋予手机 `UE expects`（不容侵犯的底层自检权）：基站发给主辅基站的 4 号账本配额总和，必须严守物理上限。
1. **乖学生路线**：如果你老老实实交了 R17 专属体检报告 (`pdcch-MonitoringCA-r17`)，那两边分配的配额加起来，绝对不能超过你体检报告上的总额。
2. **缺省路线**：如果你偷懒没交体检报告，基站依然用最保守的算法——看你物理上连了几个天线频段（$N_{cells}^{DL,NR-DC}$）。两边的配额加起来绝不能超过这个物理频段总数。

**生活类比**：
这就像是跨国公司（3GPP）财务部的铁律，无论你用的是哪种货币（R15，R16，还是 R17 体系）：
* 分公司 A（MCG）领到的预算，加上 分公司 B（SCG）领到的预算，**永远不可能、也绝不允许**大于总公司账户里上报的总金额。如果财务总监（基站配置）算错了账，下属（手机基带）有权拒不执行。

### 🔄 盲检配置的“三位一体”校验大结局
```text
【5G 盲检配置合规性法则矩阵】 (总结版)

当处于 NR-DC 双基站架构下：

(R15 常规体系) -> 使用 1代账本
[MCG_1] + [SCG_1] <= pdcch-BlindDetectionCA (或 物理小区总数)

(R16 极速体系) -> 使用 2代账本
[MCG_2] + [SCG_2] <= pdcch-MonitoringCA (或 物理小区总数)

(R17 跨组体系) -> 使用 4代账本  ✅ (本段定音)
[MCG_4] + [SCG_4] <= pdcch-MonitoringCA-r17 (或 物理小区总数)

*(任何一套体系的总账对不上，都会导致相应的调度配置失效)*
```

### 💡 章节硬核总结

**本段协议是 3GPP Overbooking 防御机制在 R17 维度的“对称复制（Symmetric Replication）”。至此，5G 物理层针对 eMBB、URLLC 以及 RedCap/FR2-2 的三大 PDCCH monitoring 范式，均已完成了各自独立的闭环能力握手（Capability Handshake）与预算收敛（Budget Convergence）。通过在数学约束中严格替换所有的 IE Prefix/Suffix，协议杜绝了在复杂多模式混合运行时的交叉污染（Cross-contamination）。这段带有 `UE expects` 的硬限准则，构成了 38.213 第 10 章算力核算部分的最后一块基石，保证了无论网络下发的 RRC 树如何庞大、参数组合多么异构，L1 底层调度总能在代数总和上被死死摁在一个安全的边界内。**

## 📖 协议原文拆解 (三十七)：R17 模式下的局部上限“底牌明示” (偏科上报机制)

> **协议原文**
> When a UE is configured for NR-DC operation and the UE is provided monitoringCapabilityConfig = r17monitoringcapability for all downlink cells where the UE monitors PDCCH, the UE may indicate, through pdcch-BlindDetectionMCG-UE-r17 and pdcch-BlindDetectionSCG-UE-r17, respective maximum values for pdcch-BlindDetection4 for the MCG and pdcch-BlindDetection4 for the SCG.

### 📚 增量术语表
* **pdcch-BlindDetectionMCG-UE-r17 / pdcch-BlindDetectionSCG-UE-r17**：R17 版终端期望的主/辅基站盲检能力上限。**一句话解释**：这是手机专门针对“按周结”（R17）的大周期模式，向基站提交的局部模块瓶颈说明：“我处理 R17 任务时，主基站模块最多能扛多少，辅基站模块最多能扛多少”。

### 🚪 第一关：保持队形！专属马甲下的双向瓶颈沟通
一模一样的配方，一模一样的味道！
这是继第（二十五）段的 R15 常规版、第（三十一）段的 R16 极速版之后，协议第三次赋予手机展示“局部偏科底牌”的权利（`may indicate`）。
这次穿上的是 **`-r17`** 的专属马甲。

如果手机的硬件设计导致其在处理大跨度、跨时隙周期的盲检时，主基站和辅基站模块的内存缓存（Buffer）极其不均衡（比如 MCG 分配了巨大的 SRAM 缓存可以存很多天的数据，而 SCG 缓存很小）。
手机就可以通过这两个带着 `-r17` 后缀的高层参数，把这种局部的物理短板如实汇报给网络侧，防止网络按照“一刀切”的比例把某个弱势硬件模块给撑爆。

### 🎯 第二关：协议行文的严谨性闭环 (修正前文小瑕疵)
我们在第（三十一）段分析 R16 的局部上报时，曾提到协议行文里有一个小瑕疵（原文只写了约束第一代老参数 `pdcch-BlindDetection`）。
请注意，在这个 R17 的最新段落中，3GPP 协议展现了严谨的自我修正能力！
这里的原文非常精确地写明了：**“...respective maximum values for pdcch-BlindDetection4...”**
这句话的意思是，手机上报的这两个 `-r17` 局部上限，**是为了死死锁住基站未来下发的 `pdcch-BlindDetection4`（第四代专属账本）的具体配额数值的。** 这种变量名的一一对应，让底层的软件代码实现不再有任何歧义。

### 🔄 R17 局部能力上限的嵌套约束图解
```text
【R17 跨时隙 (按周结) 双基站调度防护网】

手机主动上报：
  ├─ 整体天花板：pdcch-MonitoringCA-r17 (例如总算力 = 8)
  ├─ 局部短板 M：pdcch-BlindDetectionMCG-UE-r17 (主基站最多抗 6)
  └─ 局部短板 S：pdcch-BlindDetectionSCG-UE-r17 (辅基站最多抗 4)

基站下发实际配额 (第4代账本)：
  ├─ 派给主基站的配额：pdcch-BlindDetection4 for MCG
  └─ 派给辅基站的配额：pdcch-BlindDetection4 for SCG

协议强校验 (三把锁)：
  锁 1 (上段规定)：主配额 + 辅配额 <= 8 (总算力不爆)
  锁 2 (本段约束)：主配额 <= 6 (主基站不爆)
  锁 3 (本段约束)：辅配额 <= 4 (辅基站不爆)
```

### 💡 章节硬核总结

**本段协议完成了 5G 物理层 R17 PDCCH 监控特性（`r17monitoringcapability`）在能力协商侧的最后一块马赛克拼图。针对跨时隙组盲检这种可能引发深度内存堆积（Deep Memory Buffering）的特殊场景，协议允许 UE 暴露出其处理 MCG 和 SCG 信令时的非对称物理瓶颈（Asymmetric Physical Bottlenecks）。尤为值得称道的是，本条款在行文上实现了极致的闭环映射，将 UE 侧上报的能力标量（带有 `-r17` 后缀的 UE 能力参数）直接且唯一地挂钩到了基站侧对应的网络配置标量（带有 `4` 后缀的系统分配参数）上，彻底消灭了标准实现中的跨域强耦合风险，体现了通信标准中顶级的“正交化解耦（Orthogonal Decoupling）”设计艺术。**

## 📖 协议原文拆解 (三十八)：养生模式下的“溢出汇报”与“宽大处理” (R17 数值约束)

> **协议原文**
> If the UE reports pdcch-MonitoringCA-r17,
> - the value range of pdcch-BlindDetectionMCG-UE-r17 or of pdcch-BlindDetectionSCG-UE-r17 is [1, …, pdcch-MonitoringCA-r17-1], and
> - pdcch-BlindDetectionMCG-UE-r17 + pdcch-BlindDetectionSCG-UE-r17 >= pdcch-MonitoringCA-r17.
>
> Otherwise, if $N_{cells}^{DL,NR-DC,max,r17}$ is a maximum total number of downlink cells for which the UE is provided monitoringCapabilityConfig = r17monitoringcapability and the UE is configured on both the MCG and the SCG for NR-DC as indicated in UE-NR-Capability
> - the value range of pdcch-BlindDetectionMCG-UE-r17 or of pdcch-BlindDetectionSCG-UE-r17 is [1, 2, 3], and
> - pdcch-BlindDetectionMCG-UE-r17 + pdcch-BlindDetectionSCG-UE-r17 >= $N_{cells}^{DL,NR-DC,max,r17}$.

### 📚 增量术语表
* **$N_{cells}^{DL,NR-DC,max,r17}$**：R17 缺省情况下的双连接最大物理小区总数。**一句话解释**：如果手机没交“R17总算力体检表”，基站用来作为天花板保底计算的那个物理频段连接数。

### 🚪 第一关：雷打不动的“资源池溢出定理” (>=)
一看到这个排版，就知道 3GPP 的老规矩又来了！
对于乖乖交了体检报告（`pdcch-MonitoringCA-r17`）的手机：
* 局部上限（MCG 或 SCG）不能是 0，也不能全占（给对方留至少 1 个）。
* **核心定理重现**：`主基站局部上限 + 辅基站局部上限 >= 总算力上限`！
这个大于等于（$\ge$）再次赋予了基站极大的**动态调度弹性**。它意味着手机底层处理 R17 任务的硬件模块也是“软共享”的，只要基站派发的任务总和没超标，基站可以随意在主辅基站之间进行算力的来回倾斜。

### 🎯 第二关：从“一刀切”到“宽大处理” (对比 R16 的缺省惩罚)
这段协议真正的亮点，隐藏在后半段（Otherwise 分支，即手机偷懒没交总能力报告时）。
请回忆一下我们在第（三十二）段讲过的 **R16 极速模式（按小时结）**：如果没交报告，协议给出了最严厉的惩罚，把局部上限死死**锁在 1**！

但是，现在是 **R17 养生模式（按周结）**！协议的态度发生了一百八十度大转弯：
* **局部上限的范围放宽到了 `[1, 2, 3]`！**
* 为什么？因为 R17 是“跨时隙组 (per group of Xs slots)”计算的！物理时间被大幅度拉长了。哪怕你没交底细，基站也知道你芯片有充足的好几天（好几个时隙）时间去慢慢消化这些数据。所以根本不需要用“最高只能为 1”的极刑来防范芯片瞬间烧毁。协议“宽大处理”，允许你在缺省状态下依然上报 2 甚至 3 的局部上限。

**生活类比**：
（R16 极速高压）：你要送“10分钟生死时速加急件”。没交体检报告？老板绝对不敢冒险，严格限制你东区只能跑 1 趟，西区只能跑 1 趟。
（R17 佛系长单）：你要送“一周内送达的不粘锅”。没交体检报告？老板觉得无所谓，反正有一周时间给你慢慢送，所以政策放宽，允许你申报东区可以接 3 趟，西区也可以接 3 趟。

### 🔄 三代缺省惩罚模式的对比树
```text
【当手机未上报“总盲检算力”时，协议对局部上限的缺省限制】

 ├─ R15 常规模式 (Per slot) ────► 允许在 [1, 2, 3] 中灵活选择
 │
 ├─ R16 极速模式 (Per span) ────► 🚨 危险！死死锁死在 [ 1 ] ！（防芯片瞬间过载）
 │
 └─ R17 佛系模式 (Per group) ───► ☕ 放宽！恢复到 [1, 2, 3] 中灵活选择 ✅ (本段精髓)
```

### 💡 章节硬核总结

**本段协议在微观数值域完美体现了 5G 物理层设计的“时域弛豫（Time-domain Dilution）”效应。在对 R17 `r17monitoringcapability` 的能力参数边界进行界定时，协议不仅复刻了表征软资源共享的 $\ge$ 溢出不等式，更在其隐式回退（Fallback）路径中放弃了 R16 的一元极值（$Value=1$）惩罚，退回到了与 R15 eMBB 基线相同的 `[1, 2, 3]` 宽容度区间。这一关键差异在底层信令上印证了物理本质：通过引入 $X_s$ slots 的时域分组滑窗，极高频/低功耗终端的 L1 译码瞬时峰均比（Peak-to-Average Ratio of computational load）被成功拉平。因此，即便处于未知算力深度的开环盲区，网络侧在局部小区赋予 UE 多载波（$\le 3$）并发调度的安全性依然得到了物理层面的保障。**

## 📖 协议原文拆解 (三十九)：冰火两重天！常规与极速的“混合双打套餐” (R15+R16 混配)

> **协议原文**
> If a UE indicates in UE-NR-Capability a carrier aggregation capability larger than one downlink cell with monitoringCapabilityConfig = r15monitoringcapability or larger than one downlink cell with monitoringCapabilityConfig = r16monitoringcapability, the UE includes in UE-NR-Capability an indication for a maximum number of PDCCH candidates and a maximum number of non-overlapped CCEs the UE can monitor for downlink cells with monitoringCapabilityConfig = r15monitoringcapability or for downlink cells with monitoringCapabilityConfig = r16monitoringcapability when the UE is configured for carrier aggregation operation over more than two downlink cells with at least one downlink cell with monitoringCapabilityConfig = r15monitoringcapability and at least one downlink cell with monitoringCapabilityConfig = r16monitoringcapability. When a UE is not configured for NR-DC operation, the UE determines a capability to monitor a maximum number of PDCCH candidates and a maximum number of non-overlapped CCEs per slot or per span that corresponds to $N_{cells,r15}^{cap-r16}$ downlink cells or to $N_{cells,r16}^{cap-r16}$ downlink cells, respectively, where
> - $N_{cells,r15}^{cap-r16}$ is the number of configured downlink cells if the UE does not provide pdcch-BlindDetectionCA1
> - otherwise,
>   - if the UE reports only one combination of (pdcch-BlindDetectionCA1, pdcch-BlindDetectionCA2), $N_{cells,r15}^{cap-r16}$ is the value of pdcch-BlindDetectionCA1
>   - else, $N_{cells,r15}^{cap-r16}$ is the value of pdcch-BlindDetectionCA1 from a combination of (pdcch-BlindDetectionCA1, pdcch-BlindDetectionCA2) that is provided by pdcch-BlindDetectionCA-CombIndicator
>
> and
> - $N_{cells,r16}^{cap-r16}$ is the number of configured downlink cells if the UE does not provide pdcch-BlindDetectionCA2
> - otherwise,
>   - if the UE reports only one combination of (pdcch-BlindDetectionCA1, pdcch-BlindDetectionCA2), $N_{cells,r16}^{cap-r16}$ is the value of pdcch-BlindDetectionCA2
>   - else, $N_{cells,r16}^{cap-r16}$ is the value of pdcch-BlindDetectionCA2 from a combination of (pdcch-BlindDetectionCA1, pdcch-BlindDetectionCA2) that is provided by pdcch-BlindDetectionCA-CombIndicator

### 📚 增量术语表
* **Mixed CA (at least one r15 and at least one r16)**：混合载波聚合。**一句话解释**：手机同时连着好几个频段，其中有些频段跑的是普通业务（R15，按天结），有些频段跑的是极速业务（R16，按小时结），冰火两重天同时运行。
* **pdcch-BlindDetectionCA1 / CA2**：混合模式下的专用算力指标。**一句话解释**：**CA1** 专门代表在这种混合模式下，你能分给“普通业务”的算力；**CA2** 专门代表你能分给“极速业务”的算力。
* **Combination (CA1, CA2)**：算力套餐组合。**一句话解释**：因为手机总算力有限，普通业务干多了，极速业务就得少干。所以手机会向上报几组“套餐”（比如 套餐A: 普通3+极速1；套餐B: 普通1+极速2）。
* **CombIndicator (Combination Indicator)**：套餐选择指示。**一句话解释**：基站看完了你提交的套餐列表后，用这个参数告诉你：“我今天翻牌子选套餐B！”

### 🚪 第一关：迎接“冰火两重天”的挑战
前面的章节，我们要么全网都是 R15，要么全网都是 R16。
这段协议描述了现实中最复杂的情况：**手机连了超过 2 个频段，而且是“混搭”的！**（至少有一个跑 R15 慢速，至少有一个跑 R16 极速）。
由于极速业务极其吃算力，如果强行把这两种业务的算力割裂开，会导致极大的资源浪费。所以，协议发明了**“套餐制 (Combination)”**。

### 🎯 第二关：向基站提交《算力自选套餐菜单》
一旦进入混搭模式，手机在向基站上报能力时，不再是报一个死板的数字，而是**报一组或多组 $(CA1, CA2)$ 的向量组合**。
* **生活类比**：
  你是一个大厨（UE），总公司（基站）要求你同时接“普通快餐（R15）”和“极速国宴（R16）”的订单。
  你向总公司提交了一份菜单：
  * 套餐 1 (CA1=4, CA2=1)：如果普通快餐做 4 份，那我极速国宴最多只能做 1 份。
  * 套餐 2 (CA1=2, CA2=2)：如果普通快餐减到 2 份，那我极速国宴可以多做 1 份，达到 2 份。

### 🧩 第三关：基站的“翻牌子”与最终天花板确认
手机提交完菜单后，在实际排班时，两边的天花板（常规上限 $N_{cells,r15}$ 和 极速上限 $N_{cells,r16}$）到底该怎么算？
协议给出了极为严密的判定树：
1. **没交菜单的** (does not provide)：基站不跟你废话，直接看你连了几个常规频段、几个极速频段，物理上有几个就算几个（极其保守的缺省底线）。
2. **只交了一个单一套餐的** (only one combination)：那就没得选，直接把套餐里的 CA1 赋值给常规上限，CA2 赋值给极速上限。
3. **交了多个套餐供挑选的** (else 分支)：基站会根据当前全网的业务侧重点（今天是快餐需求多还是国宴需求多），下发一个 **`CombIndicator`（套餐选择指示）**。手机收到这个指示后，立刻从自己的菜单里提取对应的 CA1 和 CA2，锁死为当前的算力天花板！

### 🔄 混搭模式 (R15+R16) 天花板判定流程树
```text
【手机进入混合 CA 模式 (常规频段 + 极速频段)】

        ├─► 手机有没有提交 (CA1, CA2) 的套餐菜单？
        │
        ├──► 【没提交】
        │       └─► 按照物理频段数硬算。
        │
        └──► 【提交了】
                │
                ├─► 菜单里只有 1 个套餐吗？
                │      └─► 直接使用该套餐的 CA1 作为常规上限，CA2 作为极速上限。
                │
                └─► 菜单里有 多个 套餐？
                       └─► 手机等待基站下发 CombIndicator！
                           (比如基站下发: "启用你的套餐 2")
                           手机立刻提取套餐 2 里的 CA1 和 CA2，作为双轨防爆红线！
```

### 💡 章节硬核总结

**本段协议在 5G L1 层面上解决了一个极其棘手的硬件资源调度难题：Mixed Numerology/Mixed Capability 场景下的基带算力动态分配（Dynamic Computing Power Allocation）。当 UE 同时处于 per-slot (R15) 和 per-span (R16) 的混合监听状态时，基带内部的译码资源池实际上是处于一种高度非线性的竞争状态。为了使调度器能够实现全局最优（Global Optimum），协议从单纯的标量（Scalar）能力上报演进为了向量组合（Vector Combinations）上报。终端通过 $(CA1, CA2)$ 的元组向网络暴露了其内部硬件在处理常规 DCI 和低延迟 DCI 时的 Pareto 边界（帕累托前沿）。网络通过下发 `CombIndicator` 可以在这条边界上自由滑动，从而在 eMBB 和 URLLC 业务的调度容量之间实现灵活的 Trade-off（折衷），这极大提升了协议向后兼容性与复杂场景下的硬件压榨效率。**

## 📖 协议原文拆解 (四十)：常规与养生的“混合双打套餐 2.0” (R15+R17 混配)

> **协议原文**
> If a UE indicates in UE-NR-Capability a carrier aggregation capability larger than one downlink cell with monitoringCapabilityConfig = r15monitoringcapability or larger than one downlink cell with monitoringCapabilityConfig = r17monitoringcapability, the UE includes in UE-NR-Capability an indication for a maximum number of PDCCH candidates and a maximum number of non-overlapped CCEs the UE can monitor for downlink cells with monitoringCapabilityConfig = r15monitoringcapability or for downlink cells with monitoringCapabilityConfig = r17monitoringcapability when the UE is configured for carrier aggregation operation over more than two downlink cells with at least one downlink cell with monitoringCapabilityConfig = r15monitoringcapability and at least one downlink cell with monitoringCapabilityConfig = r17monitoringcapability. When a UE is not configured for NR-DC operation, the UE determines a capability to monitor a maximum number of PDCCH candidates and a maximum number of non-overlapped CCEs per slot or per group of $X_s$ slots that corresponds to $N_{cells,r15/r17}^{cap-r17}$ downlink cells or to $N_{cells,r17/r15}^{cap-r17}$ downlink cells, respectively, where
> - $N_{cells,r15/r17}^{cap-r17}$ is $N_{cells,0}^{DL}+R \cdot N_{cells,1}^{DL}$ if the UE does not provide pdcch-BlindDetectionCA1 in pdcch-BlindDetectionMixedList1, where $N_{cells,0}^{DL}+N_{cells,1}^{DL}$ is the number of configured downlink serving cells
> - otherwise,
>   - if the UE reports only one combination of (pdcch-BlindDetectionCA1, pdcch-BlindDetectionCA2) in pdcch-BlindDetectionMixedList1, $N_{cells,r15/r17}^{cap-r17}$ is the value of pdcch-BlindDetectionCA1
>   - else, $N_{cells,r15/r17}^{cap-r17}$ is the value of pdcch-BlindDetectionCA1 from a combination of (pdcch-BlindDetectionCA1, pdcch-BlindDetectionCA3) that is provided by pdcch-BlindDetectionCA-CombIndicator-r17
>
> and
> - $N_{cells,r17/r15}^{cap-r17}$ is $N_{cells,0}^{DL}+R \cdot N_{cells,1}^{DL}$ if the UE does not provide pdcch-BlindDetectionCA2 in pdcch-BlindDetectionMixedList1, where $N_{cells,0}^{DL}+N_{cells,1}^{DL}$ is the number of configured downlink serving cells
> - otherwise,
>   - if the UE reports only one combination of (pdcch-BlindDetectionCA1, pdcch-BlindDetectionCA2) in pdcch-BlindDetectionMixedList1, $N_{cells,r17/r15}^{cap-r17}$ is the value of pdcch-BlindDetectionCA2
>   - else, $N_{cells,r17/r15}^{cap-r17}$ is the value of pdcch-BlindDetectionCA3 from a combination of (pdcch-BlindDetectionCA1, pdcch-BlindDetectionCA3) that is provided by pdcch-BlindDetectionCA-CombIndicator-r17

*(注：协议原文中出现 `(pdcch-BlindDetectionCA1, pdcch-BlindDetectionCA2)` 与 `(pdcch-BlindDetectionCA1, pdcch-BlindDetectionCA3)` 的混用，这通常是 3GPP 协议定稿时的笔误，实际应统一为 `CA1, CA3` 或新命名以匹配 r17，但逻辑完全不受影响。)*

### 📚 增量术语表
* **Mixed CA (r15 and r17)**：R15与R17混合载波聚合。**一句话解释**：手机同时连着好几个频段，既有跑普通业务的（R15，按天结），也有跑极高频养生业务的（R17，按周结）。
* **pdcch-BlindDetectionMixedList1**：混合算力套餐列表。**一句话解释**：专门给 R15+R17 混搭模式提供的一张“菜单”，里面罗列了手机支持的各种算力折中组合。
* **pdcch-BlindDetectionCA-CombIndicator-r17**：R17版的套餐选择指示。**一句话解释**：基站用来“点菜”的那个勾选器，告诉手机这次挑菜单上的第几个套餐。

### 🚪 第一关：旧瓶装新酒，混搭模式的排列组合扩展
这一大段长到令人窒息的协议，其实如果理解了上一段（三十九段：R15+R16 混搭），就会觉得异常亲切。
因为 3GPP 协议就像是在写 C++ 的重载函数（Overload）！
上一段解决了“常规 + 极速”的混搭算力该怎么分。
这一段解决的是**“常规 (R15) + 大周期养生 (R17)”**的混搭算力该怎么分。

同样是因为手机的硬件处理总能力有限，普通时隙处理得多了，大周期处理的配额就得压缩。所以它们依然需要**提交《算力自选套餐菜单》**。

### 🎯 第二关：完美复刻的套餐抉择判定树
这里的逻辑判定树和上一段简直是连标点符号都透着熟悉的味道：
1. **如果不交菜单 (缺省)**：如果没提供组合列表，那就启动最无脑也最保守的“祖传公式” $\rightarrow$ 单塔数量加上（R系数乘双塔数量），算出来是多少，天花板就是多少。
2. **如果交了菜单但只有一个选项**：那基站没得选，直接套用套餐里唯一的一组 $(CA1, CA2)$，分配给常规和 R17 模式。
3. **如果菜单里有多个选项 (多套餐)**：基站就会根据当前全网的负载倾向，下发一个带有 `-r17` 后缀的专属点菜器（`CombIndicator-r17`）。手机被点到哪个套餐，就严格从那个套餐里提取 CA1 和 CA3 作为两边的算力天花板。

**生活类比**：
你在同一个大集团打两份工，既要去跑普通的按天结单子（R15），也要去跑按周结的长线单子（R17）。
你给老板递交了工作能力分配菜单（`MixedList1`）：
方案A：每天干 3 单，每周那边的单子最多 2 个。
方案B：每天干 1 单，每周那边的单子可以接 5 个。
老板权衡了一下，发给你一个指令（`CombIndicator-r17` = 方案B）。你立刻按照方案B的配额锁死自己的接单上限。

### 🔄 R15 + R17 混搭模式算力判定缩略图
```text
【混合模式 R15 (Per slot) + R17 (Per group of Xs slots)】

手机提交混合能力清单 (pdcch-BlindDetectionMixedList1)
        │
        ├─► 有多个套餐供选！
        │
        ▼
基站下发 R17 点菜器 (CombIndicator-r17)
        │
        ├─► 提取出该套餐的 CA1 值 ──► 设定为 N_{cells,r15/r17}^{cap-r17} (常规天花板)
        │
        └─► 提取出该套餐的 CA3 值 ──► 设定为 N_{cells,r17/r15}^{cap-r17} (R17天花板)
```

### 💡 章节硬核总结

**本段协议是 3GPP 物理层调度模型在多版本（Multi-Release）特性叠加时的标准防御式编程（Defensive Programming in Protocol）。它严丝合缝地复制了 Rel-16 中解决 Mixed Numerology/Capability 的 Pareto 前沿协商机制，将其平移到了 R15 与 R17 混用的场景。在引入 R17 FR2-2 极高频组时隙调度的同时，不可避免地会遇到 UE 同时连接 sub-6GHz eMBB 载波的异构聚合场景。为防止两者在争夺基带内部共享 SRAM 或 MAC 层处理队列时发生不可预测的拥塞，协议启用了 `pdcch-BlindDetectionMixedList1`。通过这一列表，UE 将其非线性的算力 trade-off 离散化为若干组向量配置；基站侧则通过 `CombIndicator-r17` 进行动态裁决。这种机制从宏观信令上，保障了异构调度体系下 L1 软硬件资源的绝对安全边界。**

## 📖 协议原文拆解 (四十一)：冰火交锋！极速与佛系的“混搭套餐 3.0” (R16+R17 混配)

> **协议原文**
> If a UE indicates in UE-NR-Capability a carrier aggregation capability larger than one downlink cell with monitoringCapabilityConfig = r16monitoringcapability or larger than one downlink cell with monitoringCapabilityConfig = r17monitoringcapability, the UE includes in UE-NR-Capability an indication for a maximum number of PDCCH candidates and a maximum number of non-overlapped CCEs the UE can monitor for downlink cells with monitoringCapabilityConfig = r16monitoringcapability or for downlink cells with monitoringCapabilityConfig = r17monitoringcapability when the UE is configured for carrier aggregation operation over more than two downlink cells with at least one downlink cell with monitoringCapabilityConfig = r16monitoringcapability and with at least one downlink cell with monitoringCapabilityConfig = r17monitoringcapability. When a UE is not configured for NR-DC operation, the UE determines a capability to monitor a maximum number of PDCCH candidates and a maximum number of non-overlapped CCEs per span or per group of $X_s$ slots that corresponds to $N_{cells,r16/r17}^{cap-r17}$ downlink cells or to $N_{cells,r17/r16}^{cap-r17}$ downlink cells, respectively, where
> - $N_{cells,r16/r17}^{cap-r17}$ is the number of configured downlink cells if the UE does not provide pdcch-BlindDetectionCA1 in pdcch-BlindDetectionMixedList2
> - otherwise,
>   - if the UE reports only one combination of (pdcch-BlindDetectionCA1, pdcch-BlindDetectionCA2) in pdcch-BlindDetectionMixedList2, $N_{cells,r16/r17}^{cap-r17}$ is the value of pdcch-BlindDetectionCA1
>   - else, $N_{cells,r16/r17}^{cap-r17}$ is the value of pdcch-BlindDetectionCA2 from a combination of (pdcch-BlindDetectionCA2, pdcch-BlindDetectionCA3) that is provided by pdcch-BlindDetectionCA-CombIndicator-r17
>
> and
> - $N_{cells,r17/r16}^{cap-r17}$ is the number of configured downlink cells if the UE does not provide pdcch-BlindDetectionCA2 in pdcch-BlindDetectionMixedList2
> - otherwise,
>   - if the UE reports only one combination of (pdcch-BlindDetectionCA1, pdcch-BlindDetectionCA2) in pdcch-BlindDetectionMixedList2, $N_{cells,r17/r16}^{cap-r17}$ is the value of pdcch-BlindDetectionCA2
>   - else, $N_{cells,r17/r16}^{cap-r17}$ is the value of pdcch-BlindDetectionCA3 from a combination of (pdcch-BlindDetectionCA2, pdcch-BlindDetectionCA3) that is provided by pdcch-BlindDetectionCA-CombIndicator-r17

### 📚 增量术语表
* **Mixed CA (R16 and R17)**：极速与极高频混合载波聚合。**一句话解释**：这是对手机芯片压榨最残忍的一种业务组合。一边是要求几微秒内必须出结果的极速业务（R16 per span），另一边是要求连续横跨好几个时隙大周期的极高频业务（R17 per group）。
* **pdcch-BlindDetectionMixedList2**：混合算力套餐列表第 2 版。**一句话解释**：专门给 R16+R17 这种极端混搭模式定制的一张“菜单”（请回忆上一段的 `MixedList1` 是给 R15+R17 用的，3GPP 再次进行了绝对的命名空间隔离）。

### 🚪 第一关：左手“颠勺爆炒”，右手“文火慢炖”
阅读到这里，我们已经彻底打通了 5G 盲检调度在多频段聚合时的“任督二脉”。
这是混搭场景的最后一块拼图：**R16 + R17 同时运行**。

* **R16 极速模式** 对芯片的要求是：**瞬间爆发力极强**，要求底层极化译码器（Polar Decoder）极速运转。
* **R17 佛系模式** 对芯片的要求是：**缓存肚子极大**，因为要跨越好几天（好几个时隙）算总账，要求 SRAM 内存能囤积海量的射频采样点。

这两种能力在芯片底层抢占的是完全不同维度的资源。如果手机同时连着这两种频段（CA > 2 cells 且触发混搭），它必须向基站提交一份全新的专属能力评估菜单——`MixedList2`。

### 🎯 第二关：连代码格式都复刻的“查表逻辑”
协议关于怎么选套餐、怎么定天花板的决策树，和前面的 R15+R16、R15+R17 简直是一比一复制粘贴的：
1. **没交菜单 (缺省)**：直接按分配的物理频段数锁死（极其保守，甚至去掉了 $R$ 倍的折算，说明在这个极端模式下协议不敢有任何乐观估计）。
2. **菜单只有单选**：直接取元组里的 CA1 和 CA2 赋值给两边。
3. **菜单有多选 (点菜模式)**：基站下发 `CombIndicator-r17`。手机收到后，从列表中抽出对应的套餐，将套餐里的两项配额分别锁死为主/辅两种业务的物理算力天花板。

*(注：再次注意 3GPP 协议定稿时对 CA1/CA2/CA3 下标编号产生的轻微笔误混用，但这在 ASN.1 字典解析时并不影响逻辑：点菜器点中哪个元组，就机械地提取元组的第一项给 R16，第二项给 R17。)*

### 🔄 混搭模式 3.0 (R16 + R17) 最终判定树
```text
【最极端并发：R16 (按微秒极速结) + R17 (按周佛系结)】

手机提交 极端混合能力清单 (pdcch-BlindDetectionMixedList2)
        │
        ├─► 有多个妥协套餐可供选择！
        │    (比如: 套餐A 极速爆炒多一点；套餐B 慢炖多一点)
        │
        ▼
基站根据当前全网负载，下发点菜器 (CombIndicator-r17)
        │
        ├─► 提取出该套餐的前置变量 ──► 设定为 N_{cells,r16/r17}^{cap-r17} (R16天花板)
        │
        └─► 提取出该套餐的后置变量 ──► 设定为 N_{cells,r17/r16}^{cap-r17} (R17天花板)
```

### 💡 章节硬核总结

**本段协议完成了 5G 物理层对于 PDCCH 盲检机制在不同时域粒度特征组合（Features Combinations）下的全局穷举（Global Enumeration）。通过 `MixedList2` 的引入，3GPP 成功对 R16 (Sub-slot URLLC) 与 R17 (Multi-slot RedCap/FR2) 这一对硬件压榨特征截然相反的技术进行了 L1 资源隔离规划。由于极低延迟（需极速译码）与大尺度跨度（需极深缓冲）在 ASIC/DSP 硬件层面极难达到双峰值，协议允许终端通过元组向量上报其 Pareto 最优边界。并且，在 Failsafe 缺省路由中去除了之前 eMBB 混配中允许的 $R \cdot N$ 线性膨胀系数，强行退化至物理小区数量原值，体现了底层架构在面对未知极寒工况时“安全至上（Safety-Critical）”的设计哲学。至此，5G 下行控制信道的容量约束框架已经彻底闭环。**

## 📖 协议原文拆解 (四十二)：终极三合一大缝合！常规、极速与养生的“全家桶套餐”

> **协议原文**
> If a UE indicates in UE-NR-Capability a carrier aggregation capability larger than one downlink cell with monitoringCapabilityConfig = r15monitoringcapability, or larger than one downlink cell with monitoringCapabilityConfig = r16monitoringcapability, or larger than one downlink cell with monitoringCapabilityConfig = r17monitoringcapability, the UE includes in UE-NR-Capability an indication for a maximum number of PDCCH candidates and a maximum number of non-overlapped CCEs the UE can monitor for downlink cells with monitoringCapabilityConfig = r15monitoringcapability, or for downlink cells with monitoringCapabilityConfig = r16monitoringcapability, or for downlink cells with monitoringCapabilityConfig = r17monitoringcapability when the UE is configured for carrier aggregation operation over more than three downlink cells with at least one downlink cell with monitoringCapabilityConfig = r15monitoringcapability, at least one downlink cell with monitoringCapabilityConfig = r16monitoringcapability and at least one downlink cell with monitoringCapabilityConfig = r17monitoringcapability. When a UE is not configured for NR-DC operation, the UE determines a capability to monitor a maximum number of PDCCH candidates and a maximum number of non-overlapped CCEs per slot or per span or per group of 𝑋𝑠 slots that corresponds to 𝑁𝑐𝑒𝑙𝑙𝑠,𝑟15/{𝑟16,𝑟17}𝑐𝑎𝑝−𝑟17 downlink cells or to 𝑁𝑐𝑒𝑙𝑙𝑠,𝑟16/{𝑟15,𝑟17}𝑐𝑎𝑝−𝑟17 downlink cells or to 𝑁𝑐𝑒𝑙𝑙𝑠,𝑟17/{𝑟15,𝑟16}𝑐𝑎𝑝−𝑟17 downlink cells, respectively, where
>  - 𝑁𝑐𝑒𝑙𝑙𝑠,𝑟15/{𝑟16,𝑟17}𝑐𝑎𝑝−𝑟17 is the number of configured downlink cells if the UE does not provide pdcch-BlindDetectionCA1 in pdcch-BlindDetectionMixedList3
>  - otherwise,
>    - if the UE reports only one combination of (pdcch-BlindDetectionCA1, pdcch-BlindDetectionCA2, pdcch-BlindDetectionCA3) in pdcch-BlindDetectionMixedList3, 𝑁𝑐𝑒𝑙𝑙𝑠,𝑟15/{𝑟16,𝑟17}𝑐𝑎𝑝−𝑟17 is the value of pdcch-BlindDetectionCA1
>    - else, 𝑁𝑐𝑒𝑙𝑙𝑠,𝑟15/{𝑟16,𝑟17}𝑐𝑎𝑝−𝑟17 is the value of pdcch-BlindDetectionCA1 from a combination of (pdcch-BlindDetectionCA1, pdcch-BlindDetectionCA2, pdcch-BlindDetectionCA3) that is provided by pdcch-BlindDetectionCA-CombIndicator-r17
>  - 𝑁𝑐𝑒𝑙𝑙𝑠,𝑟16/{𝑟15,𝑟17}𝑐𝑎𝑝−𝑟17 is the number of configured downlink cells if the UE does not provide pdcch-BlindDetectionCA2 in pdcch-BlindDetectionMixedList3
>  - otherwise,
>    - if the UE reports only one combination of (pdcch-BlindDetectionCA1, pdcch-BlindDetectionCA2, pdcch-BlindDetectionCA3) in pdcch-BlindDetectionMixedList3, 𝑁𝑐𝑒𝑙𝑙𝑠,𝑟16/{𝑟15,𝑟17}𝑐𝑎𝑝−𝑟17 is the value of pdcch-BlindDetectionCA2
>    - else, 𝑁𝑐𝑒𝑙𝑙𝑠,𝑟16/{𝑟15,𝑟17}𝑐𝑎𝑝−𝑟17 is the value of pdcch-BlindDetectionCA2 from a combination of (pdcch-BlindDetectionCA1, pdcch-BlindDetectionCA2, pdcch-BlindDetectionCA3) that is provided by pdcch-BlindDetectionCA-CombIndicator-r17
>  and
>  - 𝑁𝑐𝑒𝑙𝑙𝑠,𝑟17/{𝑟15,𝑟16}𝑐𝑎𝑝−𝑟17 is the number of configured downlink cells if the UE does not provide pdcch-BlindDetectionCA3 in pdcch-BlindDetectionMixedList3
>  - otherwise,
>    - if the UE reports only one combination of (pdcch-BlindDetectionCA1, pdcch-BlindDetectionCA2, pdcch-BlindDetectionCA3) in pdcch-BlindDetectionMixedList3, 𝑁𝑐𝑒𝑙𝑙𝑠,𝑟17/{𝑟15,𝑟16}𝑐𝑎𝑝−𝑟17 is the value of pdcch-BlindDetectionCA3
>    - else, 𝑁𝑐𝑒𝑙𝑙𝑠,𝑟17/{𝑟15,𝑟16}𝑐𝑎𝑝−𝑟17 is the value of pdcch-BlindDetectionCA3 from a combination of (pdcch-BlindDetectionCA1, pdcch-BlindDetectionCA2, pdcch-BlindDetectionCA3) that is provided by pdcch-BlindDetectionCA-CombIndicator-r17

### 📚 增量术语表
* **Triple Mixed CA (R15 + R16 + R17)**：终极大混搭载波聚合。**一句话解释**：这是 5G 手机性能面临的最高考验！手机同时挂着三类频段：有的频段要求“按天结”（R15 常规），有的频段要求“按微秒结”（R16 极速），还有的频段要求“按周结”（R17 养生）。三线操作，同时并行！
* **pdcch-BlindDetectionMixedList3**：混合算力套餐列表第 3 版 (终极版)。**一句话解释**：专门给 R15+R16+R17 三合一模式定制的菜单。前面的 List1 是两项组合，List2 也是两项组合，这个 List3 直接进化成了 **三维向量组合**。
* **(CA1, CA2, CA3)**：三维算力套餐组合。**一句话解释**：手机向基站申报的算力配比。CA1 留给常规，CA2 留给极速，CA3 留给养生。

### 🚪 第一关：逼出手机的极限潜能 (三线并行的门槛)
前三段协议（39、40、41段）像是分别在教你怎么玩“红+蓝”、“红+绿”、“蓝+绿”的组合。
这段协议直接迎来了最终 Boss 战：“**红+蓝+绿 三色全开**”。

只要手机吹牛说它能在以上任何一种模式下支持聚合 1 个以上的小区，并且现在基站真的给它配置了 **大于 2 个小区（即至少 3 个），且这三个小区刚好把 R15、R16、R17 这三种调度模式全占齐了**！
此时，手机必须交出终极防爆说明书——也就是基于 `MixedList3` 的三维能力套餐列表。

### 🎯 第二关：万变不离其宗的“套餐提取逻辑”
虽然文字长得像是在念咒语，参数角标长得像乱码（比如 `$N_{cells,r15/\{r16,r17\}}^{cap-r17}$`，意思是在混搭了 R16和R17 的情况下，专门为 R15 分配的天花板），但判定逻辑**完全没有任何创新，依旧是祖传三板斧**：

1. **缺省惩罚 (不交菜单)**：直接看物理上分配了几个这种类型的小区，有几个算几个。没有任何多余的折算，用最死板、最不吃性能的底线运行。
2. **唯一选项 (交了单一套餐)**：直接把三维向量 `(CA1, CA2, CA3)` 原封不动地分别赋值给 R15、R16、R17 这三套底层译码流水线，作为它们各自死守的算力上限。
3. **多套餐点菜 (交了组合菜单)**：基站下发终极点菜器 `CombIndicator-r17`。手机顺着编号找到那个被点中的三维向量套餐。
   * 把套餐里的 **第 1 项 (CA1)** 抽出来 $\rightarrow$ 锁死为 R15 的天花板。
   * 把套餐里的 **第 2 项 (CA2)** 抽出来 $\rightarrow$ 锁死为 R16 的天花板。
   * 把套餐里的 **第 3 项 (CA3)** 抽出来 $\rightarrow$ 锁死为 R17 的天花板。

**生活类比**：
你在经营一家超级外卖驿站（终端基带）。
业务 1：普通的日结外卖（R15）。
业务 2：10 分钟必达的加急药单（R16）。
业务 3：一周内送达的大件家电（R17）。
这三种业务需要的交通工具（硬件资源）完全不同。你给平台（基站）提交了一套《三栖混合作业运力分配方案 (MixedList3)》。
平台今天看大件多，下令激活“方案C”。你立马翻出方案C的运力表，把你的员工严格按表上的数字切分成三拨人，各自守着各自的最高接单上限，雷池半步不可越。

### 🔄 终极三合一算力分发架构图
```text
【PDCCH 盲检能力三维度并发分配机制 (R15 + R16 + R17)】

  基站统筹全局，下发点菜指令 [CombIndicator-r17] 
               │
               ▼
  手机在 MixedList3 菜单中精准定位到那一组三维配额向量：(CA1, CA2, CA3)
               │
      ┌────────┼────────┐
      ▼        ▼        ▼
 [分配给 R15] [分配给 R16] [分配给 R17]
  (按天结)   (按微秒结)   (按周结)
   │        │        │
 🔒限额CA1  🔒限额CA2  🔒限额CA3
 (三套相对独立的底层状态机，拿着各自的天花板去应对基站发来的混合 DCI 洪流)
```

### 💡 章节硬核总结

**本段协议是 5G 下行控制信令盲检算力模型（PDCCH Blind Decoding Capability Model）走向大一统的最终乐章。它构建了一个覆盖 eMBB(R15)、URLLC(R16) 和 RedCap/FR2(R17) 的全矩阵三维配置空间。在底层硬件架构上，这意味着基带芯片不仅要支持时域颗粒度从微秒到几十毫秒的跨度，还要能在不同的调度切片之间进行毫秒级的上下文切换（Context Switching）与内存隔离。3GPP 协议通过极其严密的数学下标定义（如 `$N_{cells,r15/\{r16,r17\}}^{cap-r17}$`）和三维元组（3-tuple `CA1,CA2,CA3`）的硬编码寻址路由，把这种对硬件近乎变态的并发要求，优雅地收敛成了一个查表操作（Look-up Table）。这种参数字典树（ASN.1 RRC Information Elements）的设计，确保了基站和终端在任何疯狂的组合调度下，其算力账本都绝对平衡、不可击穿。**

## 📖 协议原文拆解 (四十三)：双连接 + 冰火两重天！终极切糕大法 (NR-DC下的 R15+R16 混配)

> **协议原文**
> When a UE is configured for NR-DC operation and is provided monitoringCapabilityConfig = r15monitoringcapability for at least one downlink cell and monitoringCapabilityConfig = r16monitoringcapability for at least one downlink cell where the UE monitors PDCCH, the UE determines a capability to monitor a maximum number of PDCCH candidates and a maximum number of non-overlapped CCEs that corresponds to
> - $N_{cells,r15}^{cap-r16} = N_{cells,r15}^{MCG}$ downlink cells for the MCG where $N_{cells,r15}^{MCG}$ is provided by pdcch-BlindDetection3 for the MCG,
> - $N_{cells,r15}^{cap-r16} = N_{cells,r15}^{SCG}$ downlink cells for the SCG where $N_{cells,r15}^{SCG}$ is provided by pdcch-BlindDetection3 for the SCG, and
> - $N_{cells,r16}^{cap-r16} = N_{cells,r16}^{MCG}$ downlink cells for the MCG where $N_{cells,r16}^{MCG}$ is provided by pdcch-BlindDetection2 for the MCG,
> - $N_{cells,r16}^{cap-r16} = N_{cells,r16}^{SCG}$ downlink cells for the SCG where $N_{cells,r16}^{SCG}$ is provided by pdcch-BlindDetection2 for the SCG

### 📚 增量术语表
* **NR-DC + Mixed CA**：双连接叠加混合载波。**一句话解释**：手机连着两个基站（主辅），同时这两个基站又各自或交叉地给你派发普通业务（R15）和极速业务（R16）。这是拓扑结构最复杂的通信场景。
* **pdcch-BlindDetection3**：第三代 PDCCH 盲检能力分配值。**一句话解释**：终于破案了！在第（三十五）段中消失的那个神秘的“第 3 号账本”出现了。它是专门用来在“混搭模式”下，用来给双连接的主辅基站切分 **R15 (常规业务)** 算力配额的。

### 🚪 第一关：找回失落的“3号账本”
之前的协议里，我们见过了 1代、2代、4代账本。
* 1代：全网只有 R15 时，切分主辅基站的常规算力。
* 2代：全网只有 R16 时，切分主辅基站的极速算力。
* 4代：全网只有 R17 时，切分主辅基站的养生算力。

但现在，全网既有 R15 又有 R16（混搭模式）。
极速部分的切分，协议继续沿用了专款专用的 **2代账本 (`pdcch-BlindDetection2`)**。
常规部分的切分怎么办？能用 1代账本 吗？**不行！**
因为在混搭模式下，常规算力已经被极速算力抢走了一部分，它的大小和纯 R15 时截然不同。为了防止参数污染，3GPP 发明了这本专门在混搭模式下给 R15 用的 **3代账本 (`pdcch-BlindDetection3`)**。

### 🎯 第二关：在两个维度上同时“切蛋糕”
在这个极其复杂的场景下，手机收到的不是一个天花板数值，而是**四个绝对物理天花板的数值**。手机基带必须在内部划出四个严格隔离的处理特区：

1. **主基站 (MCG) 的常规业务区**：从 3代账本 领预算，干 R15 的活。
2. **辅基站 (SCG) 的常规业务区**：从 3代账本 领预算，干 R15 的活。
3. **主基站 (MCG) 的极速业务区**：从 2代账本 领预算，干 R16 的活。
4. **辅基站 (SCG) 的极速业务区**：从 2代账本 领预算，干 R16 的活。

**生活类比**：
你在两家外卖平台接单（主平台MCG、辅平台SCG），同时这两个平台都有“普通单(R15)”和“加急单(R16)”。
公司为了防止你猝死，给你下发了四个不同的接单上限配额本：
* “3号本子”规定了你在两个平台分别最多能接几单普通件。
* “2号本子”规定了你在两个平台分别最多能接几单加急件。
你每天干活，就看着这四个配额，哪个配额满了，就拒接对应平台对应类型的单子。

### 🔄 四象限物理算力切割矩阵
```text
【NR-DC 双连接】 + 【R15常规/R16极速混搭】

            [主基站 MCG 资源池]            [辅基站 SCG 资源池]
         ┌─────────────────────┐      ┌─────────────────────┐
[R15 业务]│ 3代账本 (BlindDet-3)│      │ 3代账本 (BlindDet-3)│
         ├─────────────────────┤      ├─────────────────────┤
[R16 业务]│ 2代账本 (BlindDet-2)│      │ 2代账本 (BlindDet-2)│
         └─────────────────────┘      └─────────────────────┘

*(基站的 RRC 层将通过信令分别向这四个物理格子填入上限数值，彻底锁死终端处理的边界)*
```

### 💡 章节硬核总结

**本段协议是 3GPP 命名空间隔离设计（Namespace Isolation Design）的极致体现。它在逻辑最繁杂的网形（Multi-TRP NR-DC）与最错乱的时域（Mixed R15/R16 Numerology）叠加态下，构建了一个清晰的 $2 \times 2$ 算力正交分配矩阵。这里最大的亮点是填补了协议字典中缺失的 `pdcch-BlindDetection3`。为什么在混合模式下需要一个新的 R15 参数？因为在独立的 eMBB 模式中，R15 的算力天花板是无竞争的（Uncontested）；而在混合模式下，R15 的算力配额已经被 R16 动态压缩过了。使用专门的 `BlindDetection3` 而非复用 `BlindDetection1`，从 ASN.1 信令结构上彻底避免了终端在 RRC 重配置过程中状态机回滚时可能发生的新旧参数交叉感染。这赋予了 5G 系统无与伦比的调度稳定性。**

## 📖 协议原文拆解 (四十四)：混搭模式的最后通牒——常规与极速的“双重红线”

> **协议原文**
> When the UE is configured for carrier aggregation operation over more than two downlink cells with at least one downlink cell with monitoringCapabilityConfig = r15monitoringcapability, at least one downlink cell with monitoringCapabilityConfig = r16monitoringcapability, and no downlink cell has SCS configuration $\mu \in \{5,6\}$, or for a cell group when the UE is configured for NR-DC operation, the UE does not expect to
> - monitor per slot a number of PDCCH candidates or a number of non-overlapped CCEs that is larger than the maximum number as derived from the corresponding value of $N_{cells,r15}^{cap-r16}$, and
> - monitor per span a number of PDCCH candidates or a number of non-overlapped CCEs that is larger than the maximum number as derived from the corresponding value of $N_{cells,r16}^{cap-r16}$

### 📚 增量术语表
* **no downlink cell has SCS configuration $\mu \in \{5,6\}$**：不包含极高频段。**一句话解释**：协议在这里专门加了一个限定条件，把 480kHz 和 960kHz 的太赫兹/极高频毫米波排除在外。因为那些频段要用 R17 的“按周结”特殊规则。这里我们只讨论纯粹的 R15(常规) 和 R16(极速) 混搭。
* **does not expect**：不预期。**一句话解释**：物理层给基站排班系统画的“高压电网”，越线即罢工。

### 🚪 第一关：收网！混合模式下的双线防御
就像我们之前在单独的 R15 模式、单独的 R16 模式下看到的最后通牒一样，现在轮到 **R15+R16 混搭模式** 结账了。
既然在前面（三十九段和四十三段）基站已经把两种业务的天花板（$N_{cells,r15}^{cap-r16}$ 和 $N_{cells,r16}^{cap-r16}$）算得清清楚楚了，那么在实战执行时，手机就会同时拉起**两道防御红线**。

### 🎯 第二关：独立运转的“两套违章抓拍系统”
手机底层会同时运行两个算力监控探头，分别盯着两种业务：
1. **盯住“按天结”的常规业务 (Per Slot)**：
   探头在每一天（一个 Slot）结束时算总账。如果你基站今天派给我的常规解密任务，超过了 $N_{cells,r15}$ 换算出来的天花板，手机绝不惯着（`does not expect`），直接罢工拒收多余的常规纸条。
2. **盯住“按瞬时结”的极速业务 (Per Span)**：
   探头在每一个极短的微秒瞬间（一个 Span）里算总账。如果你基站在这个瞬间塞给我的加急任务，超过了 $N_{cells,r16}$ 换算出来的天花板，手机同样瞬间罢工（`does not expect`），直接扔掉超载的极速纸条。

**生活类比**：
你是一个外卖骑手，同时接常规单（R15）和闪送单（R16）。
平台给你定下的物理极限是：【每天最多送 50 个常规单】，且【每小时最多送 5 个闪送单】。
* 场景A：平台今天给你派了 4 个闪送单（没超），但一天下来给你派了 60 个常规单（超了）。对不起，多出的 10 个常规单我直接拒接！
* 场景B：平台今天给你派了 30 个常规单（没超），但在中午12点那一个小时里，疯狂给你派了 8 个闪送单（超了）。对不起，多出的 3 个闪送单我当场拒接！
两套规矩，两套账本，互相独立，任何一个越界都会触发底层的保护机制。

### 🔄 混搭模式下的双路过载监控矩阵
```text
【手机处于 R15(常规) + R16(极速) 混合并行盲检状态】

            [基站下发的实际 DCI 排班洪流]
                       │
       ┌───────────────┴───────────────┐
       ▼                               ▼
[监控探头 1: 时隙级 Per Slot]     [监控探头 2: 亚时隙级 Per Span]
  (统计所有 R15 纸条和面积)         (统计所有 R16 纸条和面积)
       │                               │
       ├─► 超过 N_r15 算出的上限？      ├─► 超过 N_r16 算出的上限？
       │                               │
   (是) 🚨 触发罢工！              (是) 🚨 触发罢工！
   (否) ✅ 正常解密                (否) ✅ 正常解密

*(两个探头并行工作，确保底层硬件的长时间功耗与瞬时峰值功耗均处于安全区)*
```

### 💡 章节硬核总结

**本段协议确立了 5G 物理层在 Mixed Numerology/Capability 场景下的“正交越限保护（Orthogonal Overbooking Protection）”法则。在去除了 FR2-2 极高频（$\mu \in \{5,6\}$）的干扰后，协议明确指示了在 R15 与 R16 混合调度时，UE 的物理层防线也是双轨并行的。`does not expect` 原语被同时挂载到了宏观的 per-slot 累加器和微观的 per-span 累加器上。这种设计说明了基带内部的资源竞争并非简单的“一锅粥”，而是长时序译码池与极低延迟译码池的物理/逻辑隔离。调度器（Scheduler）必须同时求解两个维度的约束方程，任何一个维度的超载都将被底层状态机无情拦截，从而在根本上保障了复杂异构载波聚合环境下的系统调度收敛性。**

## 📖 协议原文拆解 (四十五)：冰火双规的再延续——常规与养生的“双重红线”

> **协议原文**
> When the UE is configured for carrier aggregation operation over more than two downlink cells with at least one downlink cell with monitoringCapabilityConfig = r15monitoringcapability, at least one downlink cell with monitoringCapabilityConfig = r17monitoringcapability, and no downlink cell with monitoringCapabilityConfig = r16monitoringcapability, the UE does not expect to
> - monitor per slot a number of PDCCH candidates or a number of non-overlapped CCEs that is larger than the maximum number as derived from the corresponding value of $N_{cells,r15/r17}^{cap-r17}$, and
> - monitor per group of $X_s$ slots a number of PDCCH candidates or a number of non-overlapped CCEs that is larger than the maximum number as derived from the corresponding value of $N_{cells,r17/r15}$

*(注：协议最后缺了上标，按照全文逻辑一致性，此处应为 `$N_{cells,r17/r15}^{cap-r17}$`)*

### 📚 增量术语表
* **no downlink cell with monitoringCapabilityConfig = r16monitoringcapability**：不包含极速频段。**一句话解释**：排除了要求“按微秒结”的 R16 URLLC 业务。这里只探讨“按天结”（R15）和“按周结”（R17）两类长周期业务的混搭。

### 🚪 第一关：换汤不换药的“过载保护法”
这完全是上一段（四十四段）的镜像条款。
上一段处理的是 **R15 (常规) + R16 (极速)**。
这一段处理的是 **R15 (常规) + R17 (佛系/极高频)**。

既然在第（四十）段中，我们已经教会了基站怎么在这两种模式之间通过“菜单点菜（MixedList1）”切分出了各自的天花板配额：
* 常规天花板：$N_{cells,r15/r17}^{cap-r17}$
* 养生天花板：$N_{cells,r17/r15}^{cap-r17}$

那么在实际跑业务时，依然是启动**两套独立的违章抓拍系统**，严格执法！

### 🎯 第二关：双重账期的严格清算 (Per Slot vs Per Group)
手机底层的两个探头，各自按照不同的“查账周期”在运转：
1. **日结探头 (Per Slot)**：盯着 R15 业务。每天日落时（Slot 结束）结算一次，如果你这一天派的 R15 纸条数或总面积超过了算好的常规天花板，手机直接罢工拒收多余部分（`does not expect`）。
2. **周结探头 (Per group of $X_s$ slots)**：盯着 R17 业务。这可能需要好几天（$X_s$ 个时隙）才结算一次。在这一整个大周期结束时，如果你这段时间累积塞给我的 R17 纸条总数超过了算好的养生天花板，手机同样行使否决权，罢工拒收（`does not expect`）。

**生活类比**：
你是外包公司（手机），接了甲方（基站）的两个项目。
项目A（R15）签的是**日薪合同**，每天工作上限 8 小时。
项目B（R17）签的是**包工合同**，每 7 天工作上限总计 20 小时。
* 只要甲方敢让你今天干 10 小时项目A，你立刻罢工（日结账单越线）。
* 只要甲方这 7 天内累计让你干了 25 小时项目B，你同样在满 20 小时的那一刻起直接罢工（包工账单越线）。
两份合同，互不干涉，红线分明。

### 🔄 R15 + R17 混搭模式双防线拓扑
```text
【手机处于 R15(常规) + R17(养生/极高频) 混合并行盲检状态】

            [基站下发的实际 DCI 排班洪流]
                       │
       ┌───────────────┴───────────────┐
       ▼                               ▼
[探头 1: 时隙级 Per Slot]         [探头 2: 宏观周期级 Per Group]
  (统计单日 R15 任务总量)            (统计整个Xs周期内 R17 任务总量)
       │                               │
       ├─► 超过 N_r15 上限？             ├─► 超过 N_r17 上限？
       │                               │
   (是) 🚨 触发日结罢工！            (是) 🚨 触发周期罢工！
   (否) ✅ 正常解密                 (否) ✅ 正常解密

*(防呆设计确保了常规内存池与深层大周期缓存池各自独立安全)*
```

### 💡 章节硬核总结

**本段协议完成了混合载波聚合中“异构时域积分器（Heterogeneous Time-domain Integrators）”的合法性断言。当 UE 面临 eMBB（per-slot）与 FR2-2/RedCap（per group of slots）并发的调度时，底层的物理现象是缓存生命周期的巨大割裂：R15 的信道估计缓存每日刷新，而 R17 的缓存可能需要维持数日。因此，协议在此明确了双重 `UE does not expect` 检查准则。它将宏观过载判定逻辑解耦为两个拥有不同积分时间窗（Integration Window）的状态机。这种时域完全正交的防御红线，使得基站在对全网资源进行联合调度时，必须同时满足两个约束方程，从根本上防止了长周期任务挤占短周期内存，或短周期高发任务打断长周期累积过程的底层硬件灾难。**

## 📖 协议原文拆解 (四十六)：极速与养生的终极防线——冰火两重天的“双重红线”

> **协议原文**
> When the UE is configured for carrier aggregation operation over more than two downlink cells with at least one downlink cell with monitoringCapabilityConfig = r16monitoringcapability, at least one downlink cell with monitoringCapabilityConfig = r17monitoringcapability, and no downlink cell with monitoringCapabilityConfig = r15monitoringcapability, the UE does not expect to
> - monitor per span a number of PDCCH candidates or a number of non-overlapped CCEs that is larger than the maximum number as derived from the corresponding value of $N_{cells,r16/r17}^{cap-r17}$, and
> - monitor per group of $X_s$ slots a number of PDCCH candidates or a number of non-overlapped CCEs that is larger than the maximum number as derived from the corresponding value of $N_{cells,r17/r16}^{cap-r17}$

### 📚 增量术语表
* **no downlink cell with monitoringCapabilityConfig = r15monitoringcapability**：不包含常规频段。**一句话解释**：排除了要求“按天结”的普通 R15 业务。本段只讨论最变态的组合：**“按微秒结”的极速 R16** 加上 **“按周结”的长周期 R17**。

### 🚪 第一关：补齐混搭防御矩阵的最后一块拼图
经过（四十四段）和（四十五段）的洗礼，这段协议的结构已经对我们毫无秘密可言了。
协议在不厌其烦地穷举所有可能的混搭场景，并为每一种场景下达**违规拦截令（`does not expect`）**。

这一段处理的是手机底层硬件**撕裂感最强**的一种调度态：
* 一部分电路在疯狂运转，处理那些可能几微秒后就要出结果的 URLLC 极速任务（R16）。
* 另一部分电路在慢吞吞地积攒数据，处理好几天才算一笔总账的极高频/低功耗任务（R17）。

### 🎯 第二关：微观瞬时与宏观长周期的“独立执法”
既然在第（四十一）段，手机和基站已经通过点菜菜单（`MixedList2`）商量好了这两项极端业务各自的算力天花板配额，那么实战执法就必须无条件生效：

1. **瞬时抓拍探头 (Per Span)**：死死盯住 R16 业务。只要在这几十微秒的一个极短瞬间，基站塞进来的急件数量或面积超过了换算出来的极速天花板，手机瞬间判定**超载罢工**！
2. **长效审计探头 (Per group of $X_s$ slots)**：默默记录 R17 业务。在这个长达几天的周期快结束时一算账，如果累计的慢性件总数或总面积超过了换算出来的养生天花板，手机同样判定**超载罢工**！

**生活类比**：
你在核电站控制室上班。
左手控制面板是**核反应堆紧急急停按钮（R16极速）**，要求毫秒级反应；右手面板是**冷却水温长期监控表（R17大周期）**，要求观察几天内的走势。
为了防止你手忙脚乱出人命，安监局（协议）立下死规矩：
* 只要一小时内响起的急停警报超过了你申报的极限次数，你直接拒绝操作剩下的警报（瞬时超载罢工）。
* 只要一周内累积需要填写的温度报表超过了你申报的最高本数，你在超额那一刻直接拒写（长周期超载罢工）。

### 🔄 R16 + R17 冰火两重天防过载拓扑
```text
【手机处于 R16(微秒极速) + R17(跨天长线) 混合并行盲检状态】

            [基站下发的实际 DCI 极端排班洪流]
                       │
       ┌───────────────┴───────────────┐
       ▼                               ▼
[探头 A: 亚时隙级 Per Span]       [探头 B: 宏观周期级 Per Group]
  (统计微秒瞬间的 R16 任务峰值)      (统计整个Xs周期内 R17 任务积分)
       │                               │
       ├─► 超过 N_r16 上限？             ├─► 超过 N_r17 上限？
       │                               │
   (是) 🚨 触发瞬时罢工！            (是) 🚨 触发周期罢工！
   (否) ✅ 正常极速解密              (否) ✅ 正常长线解密

*(两个极端的时域判定窗完全解耦，确保快通道不被慢信令阻塞，慢通道不被快信令冲垮)*
```

### 💡 章节硬核总结

**本段协议完成了 5G 物理层调度模型中最为极端的“两极分化”时间窗组合（Two-extremes Time-window Combination）的合法性硬性兜底。将要求极低延迟（Sub-slot）的 URLLC 盲检与要求长积分周期（Multi-slot）的 FR2-2/RedCap 盲检聚合在同一个 UE 实体上，是基带硬件架构设计的噩梦。3GPP 协议通过明确双轨 `does not expect` 原语，在法理上豁免了 UE 去处理调度器配置越界的责任。它强制网络侧（gNB Scheduler）必须在下发 DCI 之前，在其内部通过两个相互正交的累加器（分别针对 Span 和 Group of slots）进行前向算力预算验证（Forward Budget Verification）。这种将硬件防灾逻辑（Hardware Disaster-prevention）前置到通信系统高层软件（Scheduler）的协议设计，是现代复杂通信系统保障极高可用性的精髓。**

## 📖 协议原文拆解 (四十七)：终局之战！“三色全开”混搭下的三重绝对防线

> **协议原文**
> When the UE is configured for carrier aggregation operation over more than three downlink cells with at least one downlink cell with monitoringCapabilityConfig = r15monitoringcapability, at least one downlink cell with monitoringCapabilityConfig = r16monitoringcapability, and at least one downlink cell with monitoringCapabilityConfig = r17monitoringcapability, the UE does not expect to
> - monitor per slot a number of PDCCH candidates or a number of non-overlapped CCEs that is larger than the maximum number as derived from the corresponding value of $N_{cells,r15/\{r16,r17\}}^{cap-r17}$, and
> - monitor per span a number of PDCCH candidates or a number of non-overlapped CCEs that is larger than the maximum number as derived from the corresponding value of $N_{cells,r16/\{r15,r17\}}^{cap-r17}$, and
> - monitor per group of $X_s$ slots a number of PDCCH candidates or a number of non-overlapped CCEs that is larger than the maximum number as derived from the corresponding value of $N_{cells,r17/\{r15,r16\}}^{cap-r17}$

### 📚 增量术语表
* **Triple Mixed CA (R15+R16+R17)**：终极大混搭载波聚合。**一句话解释**：手机同时承载着“按天结”的常规频段、“按瞬时结”的极速频段、以及“按周结”的极高频养生频段。5G 调度的地狱难度。

### 🚪 第一关：漫长算力切分大戏的“全剧终”
恭喜您！我们终于读到了 3GPP 协议关于多能力混搭（Mixed Capability）盲检算力分配的**最后一句话**。
在第（四十二）段中，我们看到了手机如何向基站提交一份三维点菜菜单 `(CA1, CA2, CA3)`，由基站选定套餐后，算出这三个维度的算力天花板。
现在，实战打响。基站的排班指令如同洪水般涌向手机。协议在这里砸下了 5G 物理层最沉重的一道三联装防爆门。

### 🎯 第二关：三路探头，独立抓拍，越线即斩
面对基站发来的海量 PDCCH 纸条，手机底层芯片会瞬间分裂出**三个绝对隔离的审查委员会**，它们各自拿着秒表和计数器，死死盯住对应业务的吞吐量：

1. **常规审查委 (Per Slot 探头)**：按“天”查账。只查 R15 业务。一旦发现今天的常规纸条数超过了 $N_{cells,r15/...}$ 算出的配额 ──► **罢工拒收（does not expect）！**
2. **极速审查委 (Per Span 探头)**：按“微秒”查账。只查 R16 业务。在这几十微秒内，只要极速纸条数量敢超过 $N_{cells,r16/...}$ 算出的配额 ──► **瞬时罢工（does not expect）！**
3. **宏观审查委 (Per Group 探头)**：按“周”查账。只查 R17 业务。在整个 $X_s$ 周期结束时，如果累计的佛系纸条总数超过了 $N_{cells,r17/...}$ 算出的配额 ──► **全面罢工（does not expect）！**

**生活类比**：
一个超级厨房（手机基带）同时处理三类外卖：
* **堂食快餐 (R15)**：要求每天最多出 100 份（Slot 结算）。
* **救护车加急餐 (R16)**：要求每 2 分钟最多出 5 份（Span 结算）。
* **大型企业团建餐 (R17)**：要求每 7 天最多出 500 份（Group 结算）。
厨房经理（协议）立下死规矩：这三个流水线的计数器**互不借用、互不干扰**。只要任何一个流水线的点餐机器发疯，超出了刚才定好的配额，哪怕其他两条线闲着，超载的那条流水线也必须立刻拔电源（触发自保机制），拒绝再出餐！

### 🔄 终极“三色全开”防御矩阵图
```text
【5G 终极形态：三栖并发控制信道盲检】

[基站下发海量混合 DCI 调度指令] 
            │
  ┌─────────┼─────────┐ (三套账本、三套时钟并行运转)
  ▼         ▼         ▼
[探头1]   [探头2]   [探头3]
Per Slot  Per Span  Per Group
(看R15)   (看R16)   (看R17)
  │         │         │
 🚨超标?   🚨超标?   🚨超标?
  │         │         │
 拒收      拒收      拒收
```

### 💡 章节硬核总结

**本段协议是 3GPP 第 10 章 PDCCH 盲检预算管理（Overbooking Management）的集大成之作与逻辑收口。它完美证明了通信标准设计中的“正交性（Orthogonality）”美学。无论系统引入多少种特性（eMBB, URLLC, RedCap），无论这三种业务在 MAC 层的调度队列如何错综复杂地交织，L1 物理层应对这种 Combinatorial Explosion（组合爆炸）的手段始终是：在数学上切分为 N 个相互正交的子空间。通过严格执行三个独立的 `UE does not expect` 逻辑断言，协议不仅在极端恶劣的多业务并发（Multi-service Concurrency）工况下保护了终端 ASIC/DSP 硬件流水线不被击穿，也为全球所有基站厂商的 Scheduler（调度器）算法划定了一条清晰不可逾越的数学约束方程组。**

## 📖 协议原文拆解 (四十八)：双老板 + 双业务！混搭双连接的“双重守恒定律”

> **协议原文**
> When a UE is configured for NR-DC operation with a total of $N_{NR-DC}^{DL,cells}$ downlink cells on both the MCG and the SCG and the UE is provided monitoringCapabilityConfig = r15monitoringcapability for $N_{NR-DC,r15}^{DL,cells}$ downlink cells and monitoringCapabilityConfig = r16monitoringcapability for $N_{NR-DC,r16}^{DL,cells}$ downlink cells where the UE monitors PDCCH, the UE expects to be provided pdcch-BlindDetection3 and pdcch-BlindDetection2 for the MCG, and pdcch-BlindDetection3 and pdcch-BlindDetection2 for the SCG with values that satisfy
> - pdcch-BlindDetection3 for the MCG + pdcch-BlindDetection3 for the SCG <= pdcch-BlindDetectionCA1, if the UE reports pdcch-BlindDetectionCA1, or
> - pdcch-BlindDetection3 for the MCG + pdcch-BlindDetection3 for the SCG <= $N_{NR-DC,r15}^{DL,cells}$, if the UE does not report pdcch-BlindDetectionCA1
>
> and
> - pdcch-BlindDetection2 for the MCG + pdcch-BlindDetection2 for the SCG <= pdcch-BlindDetectionCA2, if the UE reports pdcch-BlindDetectionCA2, or
> - pdcch-BlindDetection2 for the MCG + pdcch-BlindDetection2 for the SCG <= $N_{NR-DC,r16}^{DL,cells}$, if the UE does not report pdcch-BlindDetectionCA2

### 📚 增量术语表
* **$N_{NR-DC,r15}^{DL,cells}$ / $N_{NR-DC,r16}^{DL,cells}$**：分配给常规/极速业务的物理频段数。**一句话解释**：你手机虽然连了两个基站，总共 5 个频段，但其中 3 个频段在跑 R15 常规业务，2 个频段在跑 R16 极速业务。这俩数字就是物理上真实的分布情况。
* **pdcch-BlindDetection3 (3代账本)**：专给混搭模式下 R15 业务跨基站分配算力的账本。
* **pdcch-BlindDetection2 (2代账本)**：专给极速 R16 业务跨基站分配算力的账本。
* **CA1 / CA2**：手机上交的《混搭能力体检表》里的两项总天花板（CA1管常规，CA2管极速）。

### 🚪 第一关：两套账本，各自核对“总资产”
我们在第（四十三）段讲过，面对“主辅双基站 (NR-DC) + 常规极速混搭 (R15+R16)”这种地狱级复杂度的场景，手机内部被划成了四个物理隔离的象限，拿到了四个数字。
那基站在下发这四个数字时，能不能胡乱填？
协议在这里祭出了**“双重能量守恒定律”**！并用 `UE expects`（基站敢违规，手机就拒收）来强制担保。

既然业务是解耦的，那么查账也必须是**解耦双线查账**。

### 🎯 第二关：常规业务 (R15) 的守恒检查线
只看 3代账本（R15专用）：
主基站分到的常规算力 + 辅基站分到的常规算力，**加起来绝对不能超标！**
* **上限在哪？** 
  * 如果手机交了混搭体检表，天花板就是体检表里的 **CA1**（常规业务总上限）。
  * 如果手机没交，天花板就是你物理上真正挂了几个 **R15的频段（$N_{r15}$）**。

### 🧩 第三关：极速业务 (R16) 的守恒检查线
只看 2代账本（R16专用）：
主基站分到的极速算力 + 辅基站分到的极速算力，**加起来也绝对不能超标！**
* **上限在哪？**
  * 如果手机交了混搭体检表，天花板就是体检表里的 **CA2**（极速业务总上限）。
  * 如果手机没交，天花板就是你物理上真正挂了几个 **R16的频段（$N_{r16}$）**。

**生活类比**：
你在为两家公司（主、辅基站）做代工。
你们厂有两条流水线：【普通产品线 (R15)】 和 【加急产品线 (R16)】。
你向两家公司声明了总产能：“普通产品我两家加起来最多做 100个（CA1）；加急产品我两家加起来最多做 20个（CA2）。”
每天早上，两家公司会分别给你派单（下发 3号和2号账本配额）。
你拿到派单后立刻核对：
1. 公司A的普通单 + 公司B的普通单 <= 100？ (过关！)
2. 公司A的加急单 + 公司B的加急单 <= 20？ (过关！)
哪怕总数没超，只要其中任意一条产品线的派单和超出了该产线的专项上限，你都会直接把订单打回重做！

### 🔄 混搭双连接的“双重守恒方程组”图解
```text
【基带底层合规性检验方程组】

[主基站]                    [辅基站]                    [手机全局物理防线]
(BlindDet 3)       +      (BlindDet 3)       <=       [ pdcch-BlindDetectionCA1 ] 
 (MCG-R15)                  (SCG-R15)                    (常规算力总水池)

                                 AND (同时必须满足)

[主基站]                    [辅基站]                    [手机全局物理防线]
(BlindDet 2)       +      (BlindDet 2)       <=       [ pdcch-BlindDetectionCA2 ] 
 (MCG-R16)                  (SCG-R16)                    (极速算力总水池)

*(这两条方程式是硬编码在 L1 层的，任何 RRC 层的错误配置都会在此被精确拦截)*
```

### 💡 章节硬核总结

**本段协议在数学上完美闭环了 5G 极其复杂的 NR-DC 叠加 Mixed Numerology (R15+R16) 场景下的 PDCCH Overbooking 管理体系。在多维异构的网络拓扑中，算力的分配极易因为缺乏跨节点的实时协调而发生溢出。3GPP 巧妙地采用了“降维打击”的设计思路：通过强制性的 RRC 参数命名空间隔离（`pdcch-BlindDetection3` 锁定 eMBB，`pdcch-BlindDetection2` 锁定 URLLC），将一个复杂的二维矩阵分配问题，拆解成了两个完全正交的一维线性不等式（Linear Inequalities）。终端只需并行执行两套独立的 `UE expects` 校验和即可验证整个系统的调度安全性。这种设计既保障了物理层硬件水池的绝对安全，又最大化地减少了底层状态机的耦合分支逻辑。**

## 📖 协议原文拆解 (四十九)：混搭双基站的“局部偏科大起底” (四维局部上限申报)

> **协议原文**
> When a UE is configured for NR-DC operation and is provided monitoringCapabilityConfig = r15monitoringcapability for at least one downlink cell and monitoringCapabilityConfig = r16monitoringcapability for at least one downlink cell where the UE monitors PDCCH, the UE may indicate, through pdcch-BlindDetectionMCG-UE1 and pdcch-BlindDetectionSCG-UE1, respective maximum values for pdcch-BlindDetection3 for the MCG and pdcch-BlindDetection3 for the SCG, and through pdcch-BlindDetectionMCG-UE2 and pdcch-BlindDetectionSCG-UE2 respective maximum values for pdcch-BlindDetection2 for the MCG and pdcch-BlindDetection2 for the SCG.

### 📚 增量术语表
* **pdcch-BlindDetectionMCG-UE1 / SCG-UE1**：常规业务的局部极限。**一句话解释**：手机向基站摊牌：“我的主基站(MCG)模块最多能处理多少常规(R15)任务；辅基站(SCG)模块最多能处理多少常规任务。”注意后缀是 `UE1`，专门用来锁死第 3 代账本（常规配额）。
* **pdcch-BlindDetectionMCG-UE2 / SCG-UE2**：极速业务的局部极限。**一句话解释**：手机继续摊牌：“我的主基站模块最多能处理多少极速(R16)任务；辅基站模块最多能处理多少极速任务。”后缀是 `UE2`，专门用来锁死第 2 代账本（极速配额）。

### 🚪 第一关：从“双规”到“四防线”的局部底牌
前几段我们已经非常熟悉手机向基站上报“局部短板/偏科上限”的套路了。
因为底层硬件可能左右不对称（主基站芯片强，辅基站芯片弱）。
现在网络环境是**NR-DC双基站 + R15/R16混搭**。这意味着手机不仅要在左右（主辅）上分家，还要在上下（常规极速）上分家。
因此，手机如果想限制基站瞎派单，它必须一口气向上级提交**四张局部底牌**！

### 🎯 第二关：萝卜拔了坑还在，一一对应不可乱！
这段协议虽然长，但其核心就是建立一个**严丝合缝的映射字典**：
* 手机报上去的 `MCG-UE1` 和 `SCG-UE1` ──► 专门用来卡死基站下发的 **`BlindDetection3`**（R15 常规账本）！
* 手机报上去的 `MCG-UE2` 和 `SCG-UE2` ──► 专门用来卡死基站下发的 **`BlindDetection2`**（R16 极速账本）！

**生活类比**：
你是外包工厂负责人，手下有主厂区（MCG）和分厂区（SCG），且都能做普件（R15）和急件（R16）。
你向总局（网络）提交了一份《各厂区各产线产能封顶声明》：
1. 主厂区普件线（MCG-UE1）：最多 50 单。
2. 分厂区普件线（SCG-UE1）：最多 30 单。
3. 主厂区急件线（MCG-UE2）：最多 10 单。
4. 分厂区急件线（SCG-UE2）：最多 5 单。
以后总局给你下发指标的时候，这 4 个红线指标，每一个都必须被单独遵守，任何一条产线都不能被强行塞爆。

### 🔄 四维局部能力限制上报全景图
```text
【手机底层的 4 块硬件资源池，向外暴露 4 个硬件瓶颈参数】

               [ 常规业务 R15 产线 ]         [ 极速业务 R16 产线 ]
            ┌───────────────────────┐   ┌───────────────────────┐
[主基站 MCG] │ 申报名: MCG-UE1       │   │ 申报名: MCG-UE2       │
            │ 管控目标: BlindDet-3   │   │ 管控目标: BlindDet-2   │
            ├───────────────────────┤   ├───────────────────────┤
[辅基站 SCG] │ 申报名: SCG-UE1       │   │ 申报名: SCG-UE2       │
            │ 管控目标: BlindDet-3   │   │ 管控目标: BlindDet-2   │
            └───────────────────────┘   └───────────────────────┘

*(基站在下发最终配置前，必须通过这 4 个不等式的严格校验)*
```

### 💡 章节硬核总结

**本段协议在 RRC 信令空间内，完成了 5G 异构多连接场景下基带瓶颈特征（Hardware Bottleneck Profiling）的高精度三维刻画。在 NR-DC 叠加 Mixed CA 的四象限模型中，如果缺少这四个方向的 Sub-capacity 声明，网络侧将不得不在配额下发时采用最保守的对半均分策略，极大地浪费了终端能力。3GPP 再次通过精密的后缀命名法（Suffix Nomenclature），将 UE 侧上报的瓶颈指示灯（`-UE1` 与 `-UE2`）与网络侧的配置容器（`BlindDetection3` 与 `BlindDetection2`）进行了绝对无歧义的点对点硬连线（Hard-wiring）。这种全参数矩阵暴露的设计，赋予了基站调度算法在主辅节点、快慢信令之间进行极致挤压和动态重平衡的权限，同时确保了 L1 硅片的物理安全底线。**

## 📖 协议原文拆解 (五十)：混搭模式下，R15 常规产线居然允许“拔电源”？(0 值的突破)

> **协议原文**
> If the UE reports pdcch-BlindDetectionCA1,
> - the value range of pdcch-BlindDetectionMCG-UE1 or of pdcch-BlindDetectionSCG-UE1 is [0, 1, …, pdcch-BlindDetectionCA1], and
> - pdcch-BlindDetectionMCG-UE1 + pdcch-BlindDetectionSCG-UE1 >= pdcch-BlindDetectionCA1.
>
> Otherwise, if $N_{cells}^{DL,NR-DC,max,r15}$ is a maximum total number of downlink cells for which the UE is provided monitoringCapabilityConfig = r15monitoringcapability and the UE is configured on both the MCG and the SCG for NR-DC as indicated in UE-NR-Capability
> - the value range of pdcch-BlindDetectionMCG-UE1 or of pdcch-BlindDetectionSCG-UE1 is [0, 1, 2],
> - pdcch-BlindDetectionMCG-UE1 + pdcch-BlindDetectionSCG-UE1 >= $N_{cells}^{DL,NR-DC,max,r15}$.

### 📚 增量术语表
* **[0, 1, ..., pdcch-BlindDetectionCA1]**：局部上限的值域范围。**一句话解释**：这是协议读到现在，第一次在能力上报的范围里出现了 **`0`** 和 **`满值 (CA1)`**！以前我们看到的都是 `[1, ..., 总能力-1]`。

### 🚪 第一关：熟悉的“溢出公式”，但不寻常的值域
先看不变的部分：
不论手机有没有老老实实填总能力表（`CA1`），局部上限的加法依然必须满足 **`主基站 + 辅基站 >= 总算力`**。这个赋予基站调度弹性的“算力溢出守恒定律”是永远不变的。

**最惊人的变化在于：允许填 `0` 和 `满值`！**
在前面单独 R15 或是单独 R16 的双连接中，协议规定每个基站最少得留 1 个名额（范围是 `1 到 N-1`），意思是两边都必须得干活。
但是，在现在的**混搭双连接（R15+R16）的常规业务（-UE1）上报里**，手机居然可以告诉基站：
* “我的 MCG 常规业务上限是 **满值**！”
* “我的 SCG 常规业务上限是 **0**！”

### 🎯 第二关：为什么允许“拔掉一边的常规电源”？（物理意义）
在极其高压的“冰火两重天（R15+R16并存）”环境里，手机的底层硬件压力巨大。
有些极端的通信场景或硬件设计要求：
**“好钢必须用在刀刃上！”**
可能辅基站（SCG）那边全都是极其要命的极速 URLLC 业务（R16），手机为了保住 SCG 那边的极速业务不爆，决定在 SCG 侧**彻底关闭常规业务处理能力**（上报 SCG-UE1 = 0）。
也就是说，手机允许这样一种极端分工：
* 主基站（MCG）负责全包所有的普通慢速业务（R15）。
* 辅基站（SCG）专心致志、心无旁骛地只干极速业务（R16）。

这就叫底层的**“非对称极端专职化分配”**。为了支持这种硬件架构，协议在值域范围里解除了限制，加入了 `0`。

**生活类比**：
你在跑两家平台（主副业），且都有普通单和加急单。
以前你跑外卖，平台强制要求你两边都得接普通单，哪怕只有 1 单。
现在因为加急单实在太急了。你向平台申请了特批：“为了保证副业的加急单绝对不超时，我向副业申请**普通单接单上限为 0**（SCG-UE1=0），副业的普通单我不伺候了！所有普通单的指标，全部压在主业头上（MCG-UE1=满值）！”

### 🔄 混搭模式下常规产线 (R15) 极端能力偏科图
```text
【手机的 R15 常规产线向基站申报能力上限】

 总常规算力 (CA1) = 6
        │
        ├─► 过去规矩：MCG 和 SCG 只能在 [1 到 5] 之间分。
        │
        └─► 现在规矩 (混搭模式专属特权)：
               │
               ├─ 极端方案 A: MCG 报 6 (满值)；SCG 报 0 (拔电源)！
               │    (满足 6 + 0 >= 6) ✅ 合法！
               │
               └─ 极端方案 B: MCG 报 0 (拔电源)；SCG 报 6 (满值)！
                    (满足 0 + 6 >= 6) ✅ 合法！

*(这种机制使得终端能够在复杂的双连接中，将某一侧的基带流水线彻底清空，完全留给高优的 URLLC 业务)*
```

### 💡 章节硬核总结

**本段协议在微观参数的值域（Value Range）设定上，展现了 3GPP 对极端异构调度拓扑的深度包容。在处理 Mixed CA + NR-DC 的 eMBB（即 UE1 后缀的 R15 盲检）分支时，协议敏锐地打破了以往 `[1, ..., Capacity-1]` 的均摊强制性，将边界拓展至 `[0, ..., Capacity]`。这一看似微小的数字改动，在物理层架构上撕开了一道关键的口子：它允许终端执行极端的“业务级物理隔离（Service-level Physical Isolation）”。即 UE 可以在某一特定的 Cell Group（例如 SCG）上彻底宣称其 eMBB PDCCH 译码能力为零，从而促使网络侧将该节点的所有基带算力预算无损让渡给同节点的 URLLC（R16）流水线。这是在极限算力盘剥下，保障超低延迟业务 QCL（Quality of Service）的最有效防线。**

## 📖 协议原文拆解 (五十一)：极速产线也允许“断舍离”，但没报底牌就得承受“双重削减” (R16 数值约束)

> **协议原文**
> If the UE reports pdcch-BlindDetectionCA2
> - the value range of pdcch-BlindDetectionMCG-UE2 or of pdcch-BlindDetectionSCG-UE2 is [0, 1, …, pdcch-BlindDetectionCA2], and
> - pdcch-BlindDetectionMCG-UE2 + pdcch-BlindDetectionSCG-UE2 >= pdcch-BlindDetectionCA2.
>
> Otherwise, if $N_{cells}^{DL,NR-DC,max,r16}$ is a maximum total number of downlink cells for which the UE is provided monitoringCapabilityConfig = r16monitoringcapability and the UE is configured on both the MCG and the SCG for NR-DC as indicated in UE-NR-Capability
> - the value range of pdcch-BlindDetectionMCG-UE2 or of pdcch-BlindDetectionSCG-UE2 is [0, 1],
> - pdcch-BlindDetectionMCG-UE2 + pdcch-BlindDetectionSCG-UE2 >= $N_{cells}^{DL,NR-DC,max,r16}$.

### 📚 增量术语表
* **pdcch-BlindDetectionCA2**：混搭模式下的极速 (R16) 总能力上限。**一句话解释**：手机上报的在混搭状态下，全网能分给极速业务的总算力配额。
* **pdcch-BlindDetectionMCG-UE2 / SCG-UE2**：混搭模式下的极速局部能力上限。**一句话解释**：分别分给主基站和辅基站的极速处理能力天花板（带 `-UE2` 后缀）。

### 🚪 第一关：同样允许极速业务单边“清零”
上一段（五十段）协议规定，在混搭模式下，普通 R15 业务的局部上限可以填 `0`。
这段协议紧随其后宣布：**极速 R16 业务的局部上限，同样允许填 `0` 或满值 `[0, ..., CA2]`。**

**物理意义**：
这种对称的设计赋予了手机极端的架构自由度。
手机完全可以向基站这样报备：
* MCG-UE1 (主站常规) = 满值； MCG-UE2 (主站极速) = 0
* SCG-UE1 (辅站常规) = 0； SCG-UE2 (辅站极速) = 满值
这等于告诉基站：**“我的主芯片只干慢活，我的协处理器只干快活！”** 从而在物理节点上彻底解耦了 eMBB 和 URLLC 的处理流水线。

### 🎯 第二关：如果不报能力，极速产线的“残酷压制”又来了！
请重点看后半段（Otherwise 分支）。
如果手机在简历里**没有**专门写上极速专项能力表（CA2），而是让基站按实际连接的频段数（$N_{r16}$）硬算。
这时候，局部上限的值域变成了：**`[0, 1]`**！

**相比上一段，这个削减非常刺眼**：
* 上一段（R15 常规产线没报能力）：缺省值域是 `[0, 1, 2]`。
* 本段（R16 极速产线没报能力）：缺省值域直接被砍到了 `[0, 1]`！

**为什么极速产线如此受“排挤”？**
这再次印证了我们在第（三十二）段提到的核心原则：**极速 R16 (per span) 对芯片的压榨太狠了！**
既然你连自己的极限底牌都不敢报，协议就启动最残酷的硬件保护模式。无论你怎么调配，任何一边的基站（哪怕你把另一边清零），最多也只能给你分配 **1个** 当量小区的极速并发任务！绝对不准报 2，从而物理切断极速业务瞬时并发导致基带热熔断的任何可能性。

### 🔄 混搭模式下极速产线 (R16) 的双向钳制图
```text
【手机的 R16 极速产线向基站申报能力上限】

        ├─► 有上报总的极速能力 CA2 吗？
        │
        ├──► 【有报】
        │       └─► 范围自由：可以在 [0 到 CA2满值] 之间任意挑。
        │           (主辅加起来 >= CA2 即可，允许一边满额一边拔电源)
        │
        └──► 【没报】
                └─► 触发双重紧箍咒！
                    限制 1：可以拔电源填 0，但最高只能填 1！(值域 [0, 1])
                    限制 2：加起来必须 >= 当前极速物理频段总数 N_r16！
                    (推论：这意味着在这种缺省状态下，如果 N_r16 为 2，你两边只能各填 1；
                     如果 N_r16 大于 2，手机根本无法合法上报，基站配置必然报错失败！)
```

### 💡 章节硬核总结

**本段协议是前述 `r15monitoringcapability` 边界条件在 `r16monitoringcapability` (URLLC per-span) 维度的严格推演。在维持了允许 `0` 值存在（即支持业务级基站物理隔离）的架构弹性的同时，协议在其隐式回退（Fallback）路径中，再次亮出了属于极低延迟业务的“杀手锏”——将局部容量上限强制截断至单调的 `[0, 1]` 空间。这在数学上产生了一个严酷的副产品：如果处于缺省盲区的 UE 被网络侧强行配置了超过 2 个的 URLLC CCs，因为 $0+1 \not\ge 3$ 且 $1+1 \not\ge 3$，UE 将在逻辑上彻底无法寻找到合法的能力上报参数对，从而导致 RRC 重配置的合法性阻断（Rejection）。这是 3GPP 利用参数值域（Value Range）天然属性来实现防故障架构（Failsafe Architecture）的神来之笔。**

## 📖 协议原文拆解 (五十二)：双老板 + 养生局！混搭双连接的“第二套方程式” (R15+R17 混配)

> **协议原文**
> When a UE is configured for NR-DC operation with a total of $N_{NR-DC}^{DL,cells}$ downlink cells on both the MCG and the SCG and the UE is provided monitoringCapabilityConfig = r15monitoringcapability for $N_{NR-DC,r15}^{DL,cells}$ downlink cells and monitoringCapabilityConfig = r17monitoringcapability for $N_{NR-DC,r17}^{DL,cells}$ downlink cells where the UE monitors PDCCH, the UE expects to be provided pdcch-BlindDetection3 and pdcch-BlindDetection4 for the MCG, and pdcch-BlindDetection3 and pdcch-BlindDetection4 for the SCG with values that satisfy
> - pdcch-BlindDetection3 for the MCG + pdcch-BlindDetection3 for the SCG <= pdcch-BlindDetectionCA1, if the UE reports pdcch-BlindDetectionCA1 in pdcch-BlindDetectionMixedList1, or
> - pdcch-BlindDetection3 for the MCG + pdcch-BlindDetection3 for the SCG <= $N_{NR-DC,r15}^{DL,cells}$, if the UE does not report pdcch-BlindDetectionCA1 in pdcch-BlindDetectionMixedList1
> 
> and
> - pdcch-BlindDetection4 for the MCG + pdcch-BlindDetection4 for the SCG <= pdcch-BlindDetectionCA2, if the UE reports pdcch-BlindDetectionCA2 in pdcch-BlindDetectionMixedList1, or
> - pdcch-BlindDetection4 for the MCG + pdcch-BlindDetection4 for the SCG <= $N_{NR-DC,r17}^{DL,cells}$, if the UE does not report pdcch-BlindDetectionCA2 in pdcch-BlindDetectionMixedList1

### 📚 增量术语表
* **NR-DC + Mixed CA (R15 + R17)**：双连接叠加常规与极高频/养生模式混搭。**一句话解释**：这是协议在穷举各种组合。之前拆的是 R15+R16，这里拆的是 R15+R17。
* **pdcch-BlindDetection3**：混搭模式下常规 R15 业务的 3 号切分账本。
* **pdcch-BlindDetection4**：混搭模式下佛系 R17 业务的 4 号切分账本。
* **pdcch-BlindDetectionMixedList1**：专门为 R15+R17 混搭准备的套餐列表（里面包含配对的 CA1 和 CA2）。

### 🚪 第一关：换汤不换药的“算力守恒”
这又是一段堪称“克隆代码”的协议条文。
由于网络环境变成了 **“主辅双基站” + “常规业务与按周结长线业务（R15+R17）混搭”**。
基站在派发配额时，必须动用与之严格对应的两本分离的账册：
* 用 **3 号账本** (`pdcch-BlindDetection3`) 独立分发和约束 R15 业务。
* 用 **4 号账本** (`pdcch-BlindDetection4`) 独立分发和约束 R17 业务。

手机依然是高举 `UE expects` 大旗，左右开弓，对两本账册进行双线合规性审计。

### 🎯 第二关：双线审计的具体红线
审计逻辑和之前毫无二致：

**常规产线 (R15, 3号账本)**：
主辅基站的 3 号配额之和，**绝不允许**超过手机提交的混合套餐单（`MixedList1`）里的 **CA1 预算**。如果没交菜单，绝不允许超过当前实际接入的 R15 频段总数（$N_{r15}$）。

**养生/极高频产线 (R17, 4号账本)**：
主辅基站的 4 号配额之和，**绝不允许**超过手机提交的混合套餐单里的 **CA2 预算**。如果没交菜单，绝不允许超过当前实际接入的 R17 频段总数（$N_{r17}$）。

*(注：此处 3GPP 行文的 `CA2` 依然属于下标命名惯性，其实质指向的是 R17 业务在 `MixedList1` 组合中的天花板值，在上一段解读中曾出现过称之为 `CA3` 的笔误/变体，但此处理顺了逻辑闭环。)*

### 🔄 R15 + R17 双连接混搭过载防御公式图解
```text
【基站下发 R15 + R17 双业务、双节点混合配额】

[检查站 1：常规 R15 业务]
 (主 MCG-3) + (辅 SCG-3) <= 阈值 Limit_R15
  │
  └─ Limit_R15 的取值：
     有套餐时 = 套餐里的 CA1 (R15预算)
     没套餐时 = 实际 R15 频段个数

[检查站 2：养生 R17 业务]
 (主 MCG-4) + (辅 SCG-4) <= 阈值 Limit_R17
  │
  └─ Limit_R17 的取值：
     有套餐时 = 套餐里的 CA2 (R17预算)
     没套餐时 = 实际 R17 频段个数

*(两道算力防火墙平行运转，任何一道拦截失败都会导致底层拒收相关配置)*
```

### 💡 章节硬核总结

**本段协议在数学骨架上完美重用了 R15+R16 的分配逻辑（第四十八段），但将其定界参数精准替换为面向 R15+R17 组合态的专属 IE 集合。这是 3GPP "正交扩展（Orthogonal Extension）" 设计范式的典型应用：在拓扑逻辑完全解耦的前提下，只要更换参数字典中的后缀名（如 `BlindDetection4` 取代 `BlindDetection2`，`MixedList1` 取代单独的能力上报），即可支撑全新的业务物理模型。此处的守恒方程确保了在引入 `per group of Xs slots` 的拉伸时域盲检架构时，NR-DC 节点间的资源切割能够得到强制的 RRC 级数学约束，将基带死锁防范逻辑以最小的协议开销前置到了基站配置合法性检验阶段。**

## 📖 协议原文拆解 (五十三)：双基站+佛系混搭的“四维局部底牌” (嵌套式上报机制)

> **协议原文**
> When a UE is configured for NR-DC operation and is provided monitoringCapabilityConfig = r15monitoringcapability for at least one downlink cell and monitoringCapabilityConfig = r17monitoringcapability for at least one downlink cell where the UE monitors PDCCH, the UE may indicate, through pdcch-BlindDetectionCG-UE1 in pdcch-BlindDetectionMCG-UE-Mixed and pdcch-BlindDetectionCG-UE1 in pdcch-BlindDetectionSCG-UE-Mixed, respective maximum values for pdcch-BlindDetection3 for the MCG and pdcch-BlindDetection3 for the SCG, and through pdcch-BlindDetectionCG-UE2 in pdcch-BlindDetectionMCG-UE-Mixed and pdcch-BlindDetectionCG-UE2 in pdcch-BlindDetectionSCG-UE-Mixed, respective maximum values for pdcch-BlindDetection4 for the MCG and pdcch-BlindDetection4 for the SCG.

### 📚 增量术语表
* **pdcch-BlindDetectionMCG-UE-Mixed / SCG-UE-Mixed**：混搭模式专属的局部能力“公文包”。**一句话解释**：因为混搭模式下要上报的参数太多了，协议干脆给主基站和辅基站各发了一个“公文包（结构体）”，把各种能力指标统统打包塞在里面。
* **pdcch-BlindDetectionCG-UE1**：公文包里的 1 号文件（管常规）。**一句话解释**：专门负责锁定第 3 代账本（R15 常规配额）的局部上限。
* **pdcch-BlindDetectionCG-UE2**：公文包里的 2 号文件（管养生）。**一句话解释**：专门负责锁定第 4 代账本（R17 大周期配额）的局部上限。

### 🚪 第一关：熟悉的“摊牌”环节，升级的“文件夹”结构
如果您还记得第（四十九）段中 R15+R16 的混搭上报，这段 R15+R17 的混搭逻辑在物理含义上是完全一致的：**允许手机向基站亮出自己在各个局部硬件模块上的“偏科短板”**。

但请注意这段协议在“参数命名”上的微妙变化！
随着协议写到 R17，3GPP 的专家们发现名字越来越长、越来越容易乱。于是他们使用了编程中常见的**“结构体嵌套（Nested Structure）”**思路：
* 专门创建了一个叫 **`Mixed` (混合)** 的大文件夹，分发给 MCG 和 SCG。
* 在文件夹里面，塞入 `UE1`（对应第一项能力）和 `UE2`（对应第二项能力）。

### 🎯 第二关：建立 4 个维度的绝对控制映射
手机上交的这两个“公文包”，将死死锁住基站未来下发的四个物理配额，形成一一对应的映射：

**关于主基站 (MCG) 的公文包：**
* 包里的 `UE1` ──► 锁死主基站 R15 的配额 (`BlindDetection3 for MCG`)
* 包里的 `UE2` ──► 锁死主基站 R17 的配额 (`BlindDetection4 for MCG`)

**关于辅基站 (SCG) 的公文包：**
* 包里的 `UE1` ──► 锁死辅基站 R15 的配额 (`BlindDetection3 for SCG`)
* 包里的 `UE2` ──► 锁死辅基站 R17 的配额 (`BlindDetection4 for SCG`)

**生活类比**：
你是区域总代理（终端），手下有主干店（MCG）和加盟店（SCG）。
两家店都在接“外卖日结单（R15）”和“团购周结单（R17）”。
你给总部发了两个加密压缩包：
* 《主干店极限接单能力.zip》：解压后，表格 1 写着日结单最多 50 单；表格 2 写着周结单最多 10 单。
* 《加盟店极限接单能力.zip》：解压后，表格 1 写着日结单最多 30 单；表格 2 写着周结单最多 5 单。
总部在派单时，必须同时查验这两个压缩包里的 4 个数字，任何一个分店的任何一个业务超标，系统都会报警阻断。

### 🔄 嵌套式 (Nested) 局部能力上报架构图
```text
【R15 + R17 混搭双连接的 RRC 嵌套上报结构】

                   [ 终端 UE 发送的体检报告 ]
                               │
       ┌───────────────────────┴───────────────────────┐
       ▼                                               ▼
[主基站公文包 MCG-UE-Mixed]                     [辅基站公文包 SCG-UE-Mixed]
 │                                               │
 ├─ 📂 CG-UE1: R15常规上限                         ├─ 📂 CG-UE1: R15常规上限
 │    └─ 钳制目标 ──► (BlindDetection3 for MCG)  │    └─ 钳制目标 ──► (BlindDetection3 for SCG)
 │                                               │
 └─ 📂 CG-UE2: R17大周期上限                       └─ 📂 CG-UE2: R17大周期上限
      └─ 钳制目标 ──► (BlindDetection4 for MCG)       └─ 钳制目标 ──► (BlindDetection4 for SCG)

*(通过这种树状结构的 ASN.1 编码，实现了复杂组合下参数的完美收敛与防混淆)*
```

### 💡 章节硬核总结

**本段协议在 RRC 信令编码层面展示了 3GPP 对极高维参数空间的结构化管理（Structured Parameter Management）能力。在 NR-DC 叠加 R15/R17 Mixed CA 的场景中，若依然采用平铺直叙的参数命名法（Flat Naming），将导致 L1/L2 解析器的代码极其冗长且易错。协议创新性地引入了带有 `-Mixed` 后缀的容器型 IE（Information Element Container），在其内部采用泛型化的 `CG-UE1` 和 `CG-UE2` 子节点来分别表征当前混合组合中的第一能力域（eMBB per-slot）与第二能力域（RedCap/FR2 per-group）。这种嵌套式的 ASN.1 结构不仅使得 UE 上报四个局部瓶颈标量（Sub-capacity Scarlars）的逻辑极其清晰，也为基站侧 RRC 配置合法性校验提供了一张天然的二维映射网格，确保调度配置中的 `BlindDetection3` 和 `BlindDetection4` 受到精准的硬隔离约束。**

## 📖 协议原文拆解 (五十四)：混搭公文包里的“大账与小账”之 R15 常规产线篇

> **协议原文**
> If the UE reports pdcch-BlindDetectionCA1 in pdcch-BlindDetectionMixedList1,
> - the value range of pdcch-BlindDetectionCG-UE1 for the MCG or of pdcch-BlindDetectionCG-UE1 for the SCG is [0, 1, …, pdcch-BlindDetectionCA1], and
> - pdcch-BlindDetectionCG-UE1 for the MCG + pdcch-BlindDetectionCG-UE1 for the SCG >= pdcch-BlindDetectionCA1.
>
> Otherwise, if $N_{cells}^{DL,NR-DC,max,r15}$ is a maximum total number of downlink cells for which the UE is provided monitoringCapabilityConfig = r15monitoringcapability and the UE is configured on both the MCG and the SCG for NR-DC as indicated in UE-NR-Capability
> - the value range of pdcch-BlindDetectionCG-UE1 for the MCG or of pdcch-BlindDetectionCG-UE1 for the SCG is [0, 1, 2],
> - pdcch-BlindDetectionCG-UE1 for the MCG + pdcch-BlindDetectionSCG-UE1 for the SCG >= $N_{cells}^{DL,NR-DC,max,r15}$.

*(注：协议最后一句原文 `pdcch-BlindDetectionSCG-UE1 for the SCG` 存在命名笔误，应与其前文新采用的公文包统一结构 `pdcch-BlindDetectionCG-UE1 for the SCG` 保持一致，不过 ASN.1 字典解析中实质指向是明确的。)*

### 📚 增量术语表
* **pdcch-BlindDetectionCA1 in pdcch-BlindDetectionMixedList1**：混搭菜单 1 里的 R15 总算力天花板。**一句话解释**：手机提交的“外卖+快递”综合体检表里，专门声明的“送普通外卖（R15）”的总能力值。
* **pdcch-BlindDetectionCG-UE1**：公文包里的 R15 局部上限（前一段刚介绍过，分别用于 MCG 和 SCG）。

### 🚪 第一关：换汤不换药的“溢出与断电”
这段协议处理的是：**在 R15 + R17 混搭的大背景下，专门针对 R15（常规产线）这半壁江山的具体数值约束。**
逻辑上，它和我们在第（五十）段见到的 R15+R16 混搭中的常规产线规则，是**连标点符号都一模一样**的（仅仅是外层套了一个 `MixedList1` 的公文包路径）。

* **第一大特点（允许单边断电）**：值域包含 `0`。这意味着，为了保住 R17 极高频业务的稳定，手机完全有权在主辅基站的某一边，彻底关闭 R15 常规业务的接收（填 0），实现彻底的业务物理隔离。
* **第二大特点（溢出汇报法）**：主辅两边填的极限值加起来，必须 `>=` 总能力（CA1）。这个 $\ge$ 继续赋予了基站调度时极其宝贵的动态倾斜（软资源池共享）能力。

### 🎯 第二关：如果不交体检表？(宽容的缺省惩罚)
再看 Otherwise（缺省）分支。如果手机在建立双连接时，没交这份混搭专项体检表。
基站怎么算？
* **天花板**：直接按物理上配置了几个 R15 小区来算（$N_{cells}^{max,r15}$）。
* **局部值域**：限制在 `[0, 1, 2]`。

请回想一下，这和我们在（五十一）段看到的 R16 极速缺省惩罚（死死锁在 `[0,1]`，最高只能为 1）形成了鲜明对比！
由于现在这条产线是 **R15 (按天结，per slot)**，对底层芯片的瞬时处理压力并不致命，所以协议依然大度地给了 `[0, 1, 2]` 的宽容范围。即使你偷懒没交底细，单边基站最多也能给你塞 2 个小区的并发常规任务，而不必像极速任务那样严苛阻断。

### 🔄 混搭模式下 R15 产线的数值框定流
```text
【在 R15+R17 双连接混搭中，手机填报 R15 局部上限 (CG-UE1)】

        ├─► 有在混搭菜单中提交 R15 专属总能力 CA1 吗？
        │
        ├──► 【有交 (假设 CA1=5)】
        │       └─► 填表自由：主辅皆可在 [0, 1, 2, 3, 4, 5] 任意填！
        │           核验公式：[主 CG-UE1] + [辅 CG-UE1] >= 5 即可通过！
        │
        └──► 【没交】
                └─► 启动宽容缺省模式：
                    填表限制：主辅最高只能填到 2 (即范围 [0, 1, 2])
                    核验公式：加起来必须 >= 当前 R15 物理小区总数
```

### 💡 章节硬核总结

**本段协议是 3GPP “同构逻辑，多域复用（Isomorphic Logic over Multi-domain Re-use）”法则的经典演绎。在将 L1 监控配置上下文（Context）切换到 `MixedList1` 容器内部后，针对其 `UE1`（即 eMBB per-slot 域）的能力界定，协议直接平移了原有的放宽型不等式约束。允许 `0` 值的能力上报（Capability Zero-reporting）维持了基带跨频段切片（Band Slicing）的硬件隔离自由度；而 `[0, 1, 2]` 的回退范围（Fallback Range）则映射了 eMBB 时隙级盲检对 DSP Core 低时效性的容忍度。这证明了在无论多么复杂的 RRC 外层嵌套结构下，3GPP 对底层的物理承载压力的判断标准始终具有绝对的一致性。**

## 📖 协议原文拆解 (五十五)：混搭公文包里的“大账与小账”之 R17 佛系产线篇

> **协议原文**
> If the UE reports pdcch-BlindDetectionCA2 in pdcch-BlindDetectionMixedList1
> - the value range of pdcch-BlindDetectionCG-UE2 for the MCG or of pdcch-BlindDetectionCG-UE2 for the SCG is [0, 1, …, pdcch-BlindDetectionCA3], and
> - pdcch-BlindDetectionCG-UE2 for the MCG + pdcch-BlindDetectionCG-UE2 for the SCG >= pdcch-BlindDetectionCA2.
>
> Otherwise, if $N_{cells}^{DL,NR-DC,max,r17}$ is a maximum total number of downlink cells for which the UE is provided monitoringCapabilityConfig = r17monitoringcapability and the UE is configured on both the MCG and the SCG for NR-DC as indicated in UE-NR-Capability
> - the value range of pdcch-BlindDetectionCG-UE2 for the MCG or of pdcch-BlindDetectionCG-UE2 for the SCG is [0, 1, 2],
> - pdcch-BlindDetectionCG-UE2 for the MCG + pdcch-BlindDetectionCG-UE2 for the SCG >= $N_{cells}^{DL,NR-DC,max,r17}$.

*(注：此处 3GPP 行文再次出现了一处经典的笔误！第一个 bullet 点中的 `[..., pdcch-BlindDetectionCA3]` 在上下文逻辑中绝对应该是 `CA2`。因为我们在处理 `MixedList1` 且正在约束 `CA2`。3GPP 定稿编辑在跨代合并时没把下标统一过来，但底层 ASN.1 解析逻辑不受影响。)*

### 📚 增量术语表
* **pdcch-BlindDetectionCG-UE2**：公文包里的 R17 局部上限。**一句话解释**：专门用来卡死第 4 代账本（R17 佛系按周结模式）的局部配额上限。

### 🚪 第一关：镜像克隆！R17 产线同样享有“断电权”和“动态溢出”
既然上一段把混搭模型里 R15 产线的账算清了，这段自然是来算另一半江山——**R17 (极高频/养生模式) 产线**的账。
一模一样的底层框架逻辑：
* **包含 0 (允许单边罢工)**：手机为了保住另一侧的 R15 业务不出错，完全有权利在某一个基站上，彻底关闭 R17 的处理线程（报 0），实现极端的专职化。
* **溢出上报 ($\ge$ 守恒)**：主基站和辅基站各自报的 R17 局部极限，加在一起必须 `大于等于` 手机整体承接 R17 的总算力（CA2）。用这种溢出来告知基站自己内部存在“算力资源池的软共享”，任凭基站灵活分配。

### 🎯 第二关：宽容的缺省惩罚再现！(对比 R16)
我们再把目光聚焦到 `Otherwise`（如果手机没交体检表）的缺省分支上。
它这里规定的值域范围是：**`[0, 1, 2]`**。

这又是一个极其生动的证明！
大家还记得在第（五十一）段拆解 **R16 极速产线** 时，协议因为怕烧芯片，在缺省状态下把上限残酷地锁死在了 `[0, 1]` 吗？
现在这是 **R17 佛系产线 (per group of Xs slots)**。因为它是“按周结”，时间被大大拉长，哪怕频段再高，瞬时的算力密度也被稀释了。所以协议认为它和 R15 一样是“相对安全”的。
哪怕你没交总能力底牌，协议也敢对你宽大处理，放宽到 `[0, 1, 2]`，允许基站单边最多给你派 2 个当量的 R17 盲检任务。

### 🔄 混搭模式三大产线的缺省惩罚力度对比
```text
【当终端未上报全局能力时，单边基站局部能力上限的缺省值域】

 ├─ 常规业务 (R15, 按天结) ──────► 放宽：[0, 1, 2]  (瞬时压力适中)
 │
 ├─ 极速业务 (R16, 按微秒结) ────► 锁死：[0, 1]     (瞬时压力爆表，必须严防死守！)
 │
 └─ 养生业务 (R17, 按周大周期结) ─► 放宽：[0, 1, 2]  (瞬时压力被长周期平摊，恢复宽容)
```

### 💡 章节硬核总结

**本段协议完成了 NR-DC + MixedList1 (R15+R17) 拓扑下第二能力域（RedCap/FR2-2）边界条件收敛。通过在此处维持了与 R15 相同的 `[0, 1, 2]` 缺省回退范围（Fallback Value Range），3GPP 在数学约束层面上实质性地认定了：R17 的 `per group of slots` 盲检架构在对基带译码器的瞬时突发吞吐量（Burst Throughput Requirement）压榨上，已经成功降维到了与 eMBB (`per slot`) 相似的量级，从而彻底摆脱了 URLLC (`per span`) 所面临的极度算力危机。协议对各种特征维度的深刻物理理解，就如此巧妙地隐藏在这区区几个阿拉伯数字的取值范围差异之中。**

## 📖 协议原文拆解 (五十六)：冰火极端混搭的“守恒定律” (R16+R17 配额下发限制)

> **协议原文**
> When a UE is configured for NR-DC operation with a total of $N_{cells}^{DL,NR-DC}$ downlink cells on both the MCG and the SCG and the UE is provided monitoringCapabilityConfig = r16monitoringcapability for $N_{NR-DC,r16}^{DL,cells}$ downlink cells and monitoringCapabilityConfig = r17monitoringcapability for $N_{NR-DC,r17}^{DL,cells}$ downlink cells where the UE monitors PDCCH, the UE expects to be provided pdcch-BlindDetection2 and pdcch-BlindDetection4 for the MCG, and pdcch-BlindDetection2 and pdcch-BlindDetection4 for the SCG with values that satisfy
> - pdcch-BlindDetection2 for the MCG + pdcch-BlindDetection2 for the SCG <= pdcch-BlindDetectionCA1, if the UE reports pdcch-BlindDetectionCA1 in pdcch-BlindDetectionMixedList2, or
> - pdcch-BlindDetection2 for the MCG + pdcch-BlindDetection2 for the SCG <= $N_{NR-DC,r16}^{DL,cells}$, if the UE does not report pdcch-BlindDetectionCA1 in pdcch-BlindDetectionMixedList2
> 
> and
> - pdcch-BlindDetection4 for the MCG + pdcch-BlindDetection4 for the SCG <= pdcch-BlindDetectionCA2, if the UE reports pdcch-BlindDetectionCA2 in pdcch-BlindDetectionMixedList2, or
> - pdcch-BlindDetection4 for the MCG + pdcch-BlindDetection4 for the SCG <= $N_{NR-DC,r17}^{DL,cells}$, if the UE does not report pdcch-BlindDetectionCA2 in pdcch-BlindDetectionMixedList2

### 📚 增量术语表
* **pdcch-BlindDetectionMixedList2**：混搭菜单 2。**一句话解释**：我们在第（四十一）段提到过，这是专门给最极端的“极速(R16) + 养生(R17)”混搭组合准备的能力菜单。
* **pdcch-BlindDetection2 (2代账本)**：控制极速 R16 业务的配额参数。
* **pdcch-BlindDetection4 (4代账本)**：控制大周期 R17 业务的配额参数。

### 🚪 第一关：地狱难度的派单检查，依然是相同的“配方”
现在网络来到了最变态的并发状态：手机不但开启了双基站（NR-DC），还要同时处理**极快（R16, 按微秒结）**和**极慢（R17, 按几周结）**的两种业务。
基站这时候该怎么给主、辅基站分配算力任务呢？

协议依然坚守最底层的物理隔离防线：
* 基站必须老老实实用 **2代账本** 专门发极速配额。
* 基站必须老老实实用 **4代账本** 专门发大周期配额。
然后，手机的底层拦截机制（`UE expects`）再次举起计算器，进行严格的双线合规验算。

### 🎯 第二关：双线核对防爆机制 (<= 红线)
验证逻辑和之前的任何一种双连接混搭如出一辙，核心就是**“两边之和绝不能大于总预算”**。

1. **查 2代极速账本 (R16)**：
   主基站的 2代配额 + 辅基站的 2代配额。
   * 如果交了 `MixedList2` 菜单，那就和菜单上的 **CA1**（在这里代指第一项 R16 预算）比对，绝不能超过它。
   * 如果没交，绝不能超过实际配置的极速频段数（$N_{r16}$）。

2. **查 4代大周期账本 (R17)**：
   主基站的 4代配额 + 辅基站的 4代配额。
   * 如果交了 `MixedList2` 菜单，那就和菜单上的 **CA2**（在这里代指第二项 R17 预算）比对，绝不能超过它。
   * 如果没交，绝不能超过实际配置的大周期频段数（$N_{r17}$）。

*(注：大家发现了没有？这里的 CA1 代表了 R16，CA2 代表了 R17。而在前面 R15+R17 混搭时，CA1 是 R15，CA2 是 R17。在 3GPP 中，这种 Tuple (元组) 的参数名是相对占位符，它代表的是当前 `MixedList` 容器里的第一项和第二项，而不带绝对业务属性，全看外部语境如何定义。)*

### 🔄 冰火两重天的双线隔离配置图
```text
【R16 (极速) + R17 (跨周期) 混合双连接配置校验】

   (2 代账本：负责高爆发)            (4 代账本：负责深缓存)
   [ MCG-2 ]  [ SCG-2 ]             [ MCG-4 ]  [ SCG-4 ]
       │          │                     │          │
       └────+─────┘                     └────+─────┘
            ▼                                ▼
       【必须 <= CA1】                   【必须 <= CA2】
   (MixedList2的极速总限额)          (MixedList2的跨周期总限额)

(只有两套账本双双算对，这套极端的调度配置才会通过手机底层驱动的审查而生效)
```

### 💡 章节硬核总结

**本段协议完成了 NR-DC 在 `MixedList2` (R16+R17) 拓扑下的 RRC 算力配额下发规则约束。通过在数学形式上的完美对称复制，3GPP 展示了其处理组合爆炸（Combinatorial Explosion）问题的工程智慧：即通过极其有限的正交参数集（`BlindDetection1/2/3/4`），覆盖了单 Numerology、双 Numerology 混搭以及未来潜在的 N-Numerology 混搭在多基站架构下的所有配置分发校验。本段中的 `UE expects` 保证了即使在最极端、最容易引发缓存溢出的 URLLC 与 RedCap/FR2 业务混跑场景下，网络侧的物理算力切片依然能在各自孤立的数学维度内达成严格收敛。**

## 📖 协议原文拆解 (五十七)：冰火两重天的“四维局部底牌” (嵌套公文包的应用)

> **协议原文**
> When a UE is configured for NR-DC operation and is provided monitoringCapabilityConfig = r16monitoringcapability for at least one downlink cell and monitoringCapabilityConfig = r17monitoringcapability for at least one downlink cell where the UE monitors PDCCH, the UE may indicate, through pdcch-BlindDetectionCG-UE1 in pdcch-BlindDetectionMCG-UE-Mixed and pdcch-BlindDetectionCG-UE1 in pdcch-BlindDetectionSCG-UE-Mixed, respective maximum values for pdcch-BlindDetection2 for the MCG and pdcch-BlindDetection2 for the SCG, and through pdcch-BlindDetectionCG-UE2 in pdcch-BlindDetectionMCG-UE-Mixed and pdcch-BlindDetectionCG-UE2 in pdcch-BlindDetectionSCG-UE-Mixed, respective maximum values for pdcch-BlindDetection4 for the MCG and pdcch-BlindDetection4 for the SCG.

### 📚 增量术语表
* **pdcch-BlindDetectionMCG-UE-Mixed / SCG-UE-Mixed**：混搭模式专属的局部能力“公文包”。我们在第（五十三）段已经见过它，它是一个外层容器，用来同时向基站提交多个业务维度的局部上限。

### 🚪 第一关：旧公文包，新装填物 (灵活的容器复用)
在第（五十三）段中，协议为了解决 R15+R17 混搭的四维上报，发明了这个带 `-Mixed` 后缀的嵌套结构公文包。
现在场景换成了最极端的 **R16(极速) + R17(极高频大周期)** 混搭。
3GPP 协议展现出了高级编程语言中**“多态/泛型（Polymorphism/Generics）”**的特征：
协议并没有去发明一个全新的“公文包名字”，而是**继续复用了这个 `-Mixed` 容器！**
只是容器内部的“指针指向（目标映射）”发生了改变。

### 🎯 第二关：重新绑定底层映射规则
由于业务成分变了，这次手机上交公文包后，基站解析这些文件的映射关系变成了这样：

**关于主基站 (MCG) 的公文包：**
* 包里的 `UE1` ──► 这次它锁死的不再是常规 R15，而是**极速 R16 的配额 (`BlindDetection2 for MCG`)**！
* 包里的 `UE2` ──► 它依然锁死大周期 R17 的配额 (`BlindDetection4 for MCG`)。

**关于辅基站 (SCG) 的公文包：**
* 包里的 `UE1` ──► 锁死辅基站的极速 R16 配额 (`BlindDetection2 for SCG`)。
* 包里的 `UE2` ──► 锁死辅基站的大周期 R17 配额 (`BlindDetection4 for SCG`)。

**生活类比**：
你在跟总部汇报。总部给了你两个统一规格的“汇报表单（UE-Mixed）”，一个写东区，一个写西区。
表单里固定只有两栏：【业务1上限】和【业务2上限】。
* 昨天你接的是“普通+大件”活，你默认栏目 1 填的是普通活（R15）上限，栏目 2 填的是大件（R17）上限。
* 今天你接的是“加急+大件”活，表单名字没变，但你和总部都心照不宣：栏目 1 现在代表的是**加急活（R16）**的上限，栏目 2 依然代表大件活（R17）的上限。
这种结构复用，极大节省了协议在定义数据结构（ASN.1 IE）时的复杂度和冗余度。

### 🔄 冰火混搭 (R16+R17) 的嵌套映射架构
```text
【手机上报局部瓶颈，限制基站的瞎排班】

[主基站公文包 MCG-UE-Mixed]                     [辅基站公文包 SCG-UE-Mixed]
 │                                               │
 ├─ 📂 CG-UE1: (当前语境下指代 R16 极速)            ├─ 📂 CG-UE1: (当前语境下指代 R16 极速)
 │    └─ 死死卡住 ──► (BlindDetection2 for MCG)  │    └─ 死死卡住 ──► (BlindDetection2 for SCG)
 │                                               │
 └─ 📂 CG-UE2: (当前语境下指代 R17 养生)            └─ 📂 CG-UE2: (当前语境下指代 R17 养生)
      └─ 死死卡住 ──► (BlindDetection4 for MCG)       └─ 死死卡住 ──► (BlindDetection4 for SCG)
```

### 💡 章节硬核总结

**本段协议在 RRC 信令交互层面上，展示了 ASN.1 数据结构（Data Structure）的上下文强相关性（Context Dependency）。通过复用 `pdcch-BlindDetectionMCG/SCG-UE-Mixed` 容器结构，协议避免了为每一种可能的 Mixed CA 组合去硬编码无休止的新 IE 变量。在明确了当前环境为 R16+R17 组合后，容器内的泛型变量 `CG-UE1` 自动发生语义重载（Semantic Overloading），其约束对象从 R15 的 `BlindDetection3` 平滑过渡到了 R16 的 `BlindDetection2`。这既满足了 UE 暴露其极速与极慢双重硬件瓶颈的需求，又彰显了通信标准在定义底层信令接口时对信令开销（Signaling Overhead）与可维护性的极致追求。**

## 📖 协议原文拆解 (五十八)：混搭公文包里的极速审判！(R16 产线的严苛值域)

> **协议原文**
> If the UE reports pdcch-BlindDetectionCA1 in pdcch-BlindDetectionMixedList2,
> - the value range of pdcch-BlindDetectionCG-UE1 for the MCG or of pdcch-BlindDetectionCG-UE1 for the SCG is [0, 1, …, pdcch-BlindDetectionCA1], and
> - pdcch-BlindDetectionCG-UE1 for the MCG + pdcch-BlindDetectionCG-UE1 for the SCG >= pdcch-BlindDetectionCA1.
>
> Otherwise, if $N_{cells}^{DL,NR-DC,max,r16}$ is a maximum total number of downlink cells for which the UE is provided monitoringCapabilityConfig = r16monitoringcapability and the UE is configured on both the MCG and the SCG for NR-DC as indicated in UE-NR-Capability
> - the value range of pdcch-BlindDetectionCG-UE1 for the MCG or of pdcch-BlindDetectionCG-UE1 for the SCG is [0, 1],
> - pdcch-BlindDetectionCG-UE1 for the MCG + pdcch-BlindDetectionCG-UE1 for the SCG >= $N_{cells}^{DL,NR-DC,max,r16}$.

### 📚 增量术语表
* **pdcch-BlindDetectionCA1 in pdcch-BlindDetectionMixedList2**：冰火混搭菜单（List2）里的第 1 项总算力。由于这是 R16+R17 混搭，所以这个 `CA1` 在这里的实际身份是**极速 (R16) 算力天花板**。
* **pdcch-BlindDetectionCG-UE1**：公文包里的第 1 栏目。在此语境下代表**极速 (R16) 局部算力上限**。

### 🚪 第一关：换上马甲，依然允许“极速产线拔电源”
这段协议专门负责界定在 R16+R17 混搭下，公文包里那个代表极速业务（R16）的字段，该怎么填数值。
* **老规矩（溢出汇报法）**：主辅两边的极速局部能力上限，加起来必须 `>=` 极速总能力（CA1）。让基站自由倾斜分配。
* **特权（包含 0 值）**：范围是 `[0, 1, ..., CA1]`。在双连接下，手机完全可以宣称自己某一个基带模块不接任何极速业务，把 R16 全部压给另一个基站模块处理。

### 🎯 第二关：如果不交底牌，死刑伺候！(锁死在 [0,1])
看到这里，不得不佩服 3GPP 协议制定者在底层逻辑上近乎偏执的“一致性强迫症”。
再来看看 `Otherwise`（缺省，未交体检报告）分支！

由于当前填写的这个栏目（UE1），在当前上下文（MixedList2）里代表的是**极其吃性能、容易烧芯片的极速 R16 业务**。
所以协议在缺省值域上，再次祭出了毫不容情的极刑大棒：
**`the value range is [0, 1]`。**
绝不允许报 2 或 3！因为你不报总盘子，基站就不敢对你有任何高估。为了防止微秒级的 Span 把你冲爆，哪怕你两边加起来符合条件，基站给你分配的极限任务也被物理锁死在每侧最多 1 个载波当量！

*(注：请与第五十四段对比。当时 UE1 代表的是 R15 常规业务，所以哪怕没交体检报告，缺省值域也是宽容的 `[0, 1, 2]`。这就是协议对物理特性的极度敏锐！)*

### 🔄 混搭模式下极速产线 (R16) 的值域约束判决
```text
【在 R16+R17 双连接混搭中，手机填报极速 R16 局部上限 (CG-UE1)】

        ├─► 有在混搭菜单(List2)中提交极速专属总能力 CA1 吗？
        │
        ├──► 【有交】
        │       └─► 填表自由：主辅皆可在 [0, 1, ..., 满值] 任意填！
        │           核验公式：[主 CG-UE1] + [辅 CG-UE1] >= 满值！
        │
        └──► 【没交】
                └─► 触发底线极速惩罚！
                    填表限制：主辅最高只能填到 1 (即范围死死锁在 [0, 1])！
                    核验公式：加起来必须 >= 当前 R16 物理小区总数 N_r16！
                    (这同样意味着在这种缺省下，N_r16的物理上限只能是 2)
```

### 💡 章节硬核总结

**本段协议是 5G L1 对 `r16monitoringcapability` (URLLC per-span) 极度严苛的算力保护政策在多级嵌套 RRC 容器中的递归体现（Recursive Embodiment）。当解析至 `pdcch-BlindDetectionMixedList2` 容器内部的 `CG-UE1` 字段时，协议引擎感知到了该字段在当前 Context 下映射的物理实体为 R16 Span 译码管线。因此，在其对应的隐式回退（Fallback）逻辑中，立刻将值域空间（Value Range）执行了断崖式截断，强行钳制为 `[0, 1]`。这种“见 R16 即启动顶格保护”的设计理念贯穿了整个第 10 章的算力核算体系，确保了极低延迟信道不会成为基带发生 Buffer Overflow 的阿喀琉斯之踵。**

## 📖 协议原文拆解 (五十九)：混搭公文包的最后收口！R17 佛系产线的宽大处理

> **协议原文**
> If the UE reports pdcch-BlindDetectionCA2 in pdcch-BlindDetectionMixedList2
> - the value range of pdcch-BlindDetectionCG-UE2 for the MCG or of pdcch-BlindDetectionCG-UE2 for the SCG is [0, 1, …, pdcch-BlindDetectionCA2], and
> - pdcch-BlindDetectionCG-UE2 for the MCG + pdcch-BlindDetectionCG-UE2 for the SCG >= pdcch-BlindDetectionCA2.
>
> Otherwise, if $N_{cells}^{DL,NR-DC,max,r17}$ is a maximum total number of downlink cells for which the UE is provided monitoringCapabilityConfig = r17monitoringcapability and the UE is configured on both the MCG and the SCG for NR-DC as indicated in UE-NR-Capability
> - the value range of pdcch-BlindDetectionCG-UE2 for the MCG or of pdcch-BlindDetectionCG-UE2 for the SCG is [0, 1, 2],
> - pdcch-BlindDetectionCG-UE2 for the MCG + pdcch-BlindDetectionCG-UE2 for the SCG >= $N_{cells}^{DL,NR-DC,max,r17}$.

### 📚 增量术语表
* **pdcch-BlindDetectionCA2 in pdcch-BlindDetectionMixedList2**：冰火混搭菜单（List2）里的第 2 项总算力。由于这是 R16+R17 混搭，所以这个 `CA2` 在这里的实际身份是**佛系大周期 (R17) 算力天花板**。
* **pdcch-BlindDetectionCG-UE2**：公文包里的第 2 栏目。在此语境下代表**佛系大周期 (R17) 局部算力上限**。

### 🚪 第一关：完美的对称美学 (R17 的局部上限与溢出)
这段协议负责给整个 R16+R17 双连接混搭的参数申报画上最终句号。
对于代表 R17 (按周结) 业务的 `CG-UE2` 字段：
* 依然允许填 `0`，意味着手机可以向基站宣告：“我的辅基站不接长线任务了，全交给主基站去囤缓存！”
* 依然必须满足溢出定理：两边上报的极限加起来必须 `>=` 总极限（CA2）。为基站调度长线任务提供充足的左右逢源（Load Balancing）空间。

### 🎯 第二关：长周期的福利 (惩罚豁免，重回宽容)
最令人极度舒适的一点，体现在 `Otherwise` (缺省) 分支的值域限制上。
请看这里：**`the value range is [0, 1, 2]`**。

* 在上一段（五十八段），因为处理的是微秒级要命的 **R16 极速业务**，缺省值域被残暴地锁死在 `[0, 1]`。
* 在这一段，因为处理的是跨越多天的 **R17 长周期业务**，时间被极大拉长拉扁了，芯片有充足的喘息期。所以，就算你偷懒没交底线报告，协议对你的缺省惩罚也轻得多，直接放宽回到了 `[0, 1, 2]`。
这再次在最底层的参数设计上，印证了 3GPP 对不同业务带来的真实硅片热耗散（Thermal Dissipation）和瞬时内存压力（Memory Pressure）的极致把控。

### 🔄 MixedList2 混合公文包 (R16+R17) 缺省值域对比总结
```text
【当手机偷懒没交底细时，基站如何利用缺省值域进行排班防爆？】

面对手机发来的公文包 (CG-UE 文件夹)：
        │
        ├─► 拆开第一层：CG-UE1 (此时代表 R16 极速)
        │      └─► 🚨 危险！极速业务极其吃算力！
        │          基站配置锁死：最多只能在这个格子填 1！(范围 [0, 1])
        │
        └─► 拆开第二层：CG-UE2 (此时代表 R17 大周期)
               └─► ☕ 安全！大周期业务有足够的时间缓冲！
                   基站配置放宽：允许在这个格子填到 2！(范围 [0, 1, 2])
```

### 💡 章节硬核总结

**本段协议为 5G Rel-17 的多节点混合调度体系献上了完美的终局验证（Terminal Verification）。至此，关于 `pdcch-BlindDetectionMixedList2` 嵌套结构的解析全部完成。我们清晰地看到了 3GPP 协议设计中的“物理法则投射（Projection of Physical Laws）”：即使在极其抽象、嵌套数层的 ASN.1 RRC 变量树中，只要变量的指针（Pointer）最终指向了具备长时序积分特征（per group of slots）的 R17 盲检实体，其对应的 Fallback 保护域（Value Range）就会自动弛豫（Relaxation）为相对宽容的 `[0, 1, 2]`。这种通过控制信令值域上限来硬编码底层基带物理安全边界的手法，是移动通信工业标准有别于普通软件协议的独特魅力。**

## 📖 协议原文拆解 (六十)：三界会师！“常规+极速+养生”双连接的六维算力分割

> **协议原文**
> When a UE is configured for NR-DC operation with a total of $N_{NR-DC}^{DL,cells}$ downlink cells on both the MCG and the SCG and the UE is provided monitoringCapabilityConfig = r15monitoringcapability for $N_{NR-DC,r15}^{DL,cells}$ downlink cells, monitoringCapabilityConfig = r16monitoringcapability for $N_{NR-DC,r16}^{DL,cells}$ downlink cells, and monitoringCapabilityConfig = r17monitoringcapability for $N_{NR-DC,r17}^{DL,cells}$ downlink cells where the UE monitors PDCCH, the UE expects to be provided pdcch-BlindDetection3, pdcch-BlindDetection2, and pdcch-BlindDetection4 for the MCG, and pdcch-BlindDetection3, pdcch-BlindDetection2, and pdcch-BlindDetection4 for the SCG with values that satisfy
> - pdcch-BlindDetection3 for the MCG + pdcch-BlindDetection3 for the SCG <= pdcch-BlindDetectionCA1, if the UE reports pdcch-BlindDetectionCA1 in pdcch-BlindDetectionMixedList3, or
> - pdcch-BlindDetection3 for the MCG + pdcch-BlindDetection3 for the SCG <= $N_{NR-DC,r15}^{DL,cells}$, if the UE does not report pdcch-BlindDetectionCA1 in pdcch-BlindDetectionMixedList3
> 
> and
> - pdcch-BlindDetection2 for the MCG + pdcch-BlindDetection2 for the SCG <= pdcch-BlindDetectionCA2, if the UE reports pdcch-BlindDetectionCA2 in pdcch-BlindDetectionMixedList3, or
> - pdcch-BlindDetection2 for the MCG + pdcch-BlindDetection2 for the SCG <= $N_{NR-DC,r16}^{DL,cells}$, if the UE does not report pdcch-BlindDetectionCA2 in pdcch-BlindDetectionMixedList3
> 
> and
> - pdcch-BlindDetection4 for the MCG + pdcch-BlindDetection4 for the SCG <= pdcch-BlindDetectionCA3, if the UE reports pdcch-BlindDetectionCA3 in pdcch-BlindDetectionMixedList3, or
> - pdcch-BlindDetection4 for the MCG + pdcch-BlindDetection4 for the SCG <= $N_{NR-DC,r17}^{DL,cells}$, if the UE does not report pdcch-BlindDetectionCA3 in pdcch-BlindDetectionMixedList3

### 📚 增量术语表
* **Triple Mixed CA (R15+R16+R17)**：终极大混搭载波聚合。**一句话解释**：手机同时承载着“按天结”的常规频段、“按微秒结”的极速频段、以及“按周结”的极高频养生频段。5G 调度的地狱难度。
* **pdcch-BlindDetectionMixedList3**：混合算力套餐列表第 3 版 (终极版)。**一句话解释**：给三色全开配置定制的终极体检报告。里面同时装着 `CA1, CA2, CA3` 三大总配额。

### 🚪 第一关：手机里的“六大平行宇宙”
恭喜各位，我们终于爬到了 5G PDCCH 盲检能力配置（Capability Negotiation）这座极其繁杂的高山的最高点！
这是协议里能够出现的**最庞大、最复杂的物理拓扑组合**：
手机挂着双基站（MCG 和 SCG）。在这两个基站下面，各自都同时在跑三种完全不同结算周期的业务：
* R15 常规（按天结）
* R16 极速（按小时/微秒结）
* R17 养生（按周/大周期结）

为了在物理底层彻底隔离这三种完全不同脾气的业务，防止它们互相抢夺内存和译码器资源，基带芯片必须在内部划出 **6 个绝对隔离的处理沙盒（Sandboxes）**。

### 🎯 第二关：六本账册的“守恒对账单”
基站想要给这 6 个沙盒发任务，必须同时下发 **6 本账册**。
而且，这 6 本账册必须严格遵守各自领域的“能量守恒定律”，手机（通过 `UE expects`）会充当最严厉的财务审计员：

**【账单 1：R15 常规流水线】(看 3 号账本)**
* [主站分到的 3号 配额] + [辅站分到的 3号 配额]
* 必须 `<=` 终极菜单里的 **CA1**（常规总预算）或 $N_{NR-DC,r15}^{DL,cells}$ 物理频段数。

**【账单 2：R16 极速流水线】(看 2 号账本)**
* [主站分到的 2号 配额] + [辅站分到的 2号 配额]
* 必须 `<=` 终极菜单里的 **CA2**（极速总预算）或 $N_{NR-DC,r16}^{DL,cells}$ 物理频段数。

**【账单 3：R17 大周期流水线】(看 4 号账本)**
* [主站分到的 4号 配额] + [辅站分到的 4号 配额]
* 必须 `<=` 终极菜单里的 **CA3**（养生总预算）或 $N_{NR-DC,r17}^{DL,cells}$ 物理频段数。

**任何一本账对不上，整个这套极其宏大的 RRC 调度配置指令，都会被手机底层驱动当场驳回、视为非法！**

### 🔄 终极“三色全开”双基站算力分割阵列
```text
【基站向双连接终端下发 5G 全形态并发配置时的算力审计大屏】

             [ 主基站 MCG 派单量 ]        [ 辅基站 SCG 派单量 ]        [ 终端物理承载力上限 ]
──────────────────────────────────────────────────────────────────────────────────────────
R15 业务：  (BlindDetection3_MCG)   +   (BlindDetection3_SCG)  <=  [ MixedList3 -> CA1 ]
──────────────────────────────────────────────────────────────────────────────────────────
R16 业务：  (BlindDetection2_MCG)   +   (BlindDetection2_SCG)  <=  [ MixedList3 -> CA2 ]
──────────────────────────────────────────────────────────────────────────────────────────
R17 业务：  (BlindDetection4_MCG)   +   (BlindDetection4_SCG)  <=  [ MixedList3 -> CA3 ]
──────────────────────────────────────────────────────────────────────────────────────────

*(这三条等式同时成立，是 5G 芯片在极限压榨下还能保持不崩溃的根本数学保障)*
```

### 💡 章节硬核总结

**本段协议是 3GPP 第 10 章在 UE PDCCH Capability 维度的巅峰之作（Magnum Opus）。它将 eMBB (R15 per-slot)、URLLC (R16 per-span) 和 RedCap/FR2-2 (R17 per-group) 三大底层特性，与 NR-DC (Multi-Radio Dual Connectivity) 的拓扑维度进行了最终极的组合。在信令架构上，协议利用 `MixedList3` 和三组完全独立的 `pdcch-BlindDetectionX` 容器，构建了一个 $3 \times 2$ 的超维算力映射矩阵。更为深邃的是，通过三组并行的 `UE expects` 不等式，协议将这六个维度的调度配额强制收敛回了一个三维的物理基带极限向量 `(CA1, CA2, CA3)`。这种在极度膨胀的功能需求下，依然能保持数学模型的绝对正交（Orthogonality）与守恒（Conservation）的协议编写能力，堪称现代通信标准架构设计的范本。**

## 📖 协议原文拆解 (六十一)：终极六维底牌！三色混搭双连接的“超级公文包”

> **协议原文**
> When a UE is configured for NR-DC operation and is provided monitoringCapabilityConfig = r15monitoringcapability for at least one downlink cell, monitoringCapabilityConfig = r16monitoringcapability for at least one downlink cell, and monitoringCapabilityConfig = r17monitoringcapability for at least one downlink cell where the UE monitors PDCCH, the UE may indicate, through pdcch-BlindDetectionCG-UE1 in pdcch-BlindDetectionMCG-UE-Mixed1 and pdcch-BlindDetectionCG-UE1 in pdcch-BlindDetectionSCG-UE-Mixed1 respective maximum values for pdcch-BlindDetection3 for the MCG and pdcch-BlindDetection3 for the SCG, through pdcch-BlindDetectionCG-UE2 in pdcch-BlindDetectionMCG-UE-Mixed1 and pdcch-BlindDetectionCG-UE2 in pdcch-BlindDetectionSCG-UE-Mixed1 respective maximum values for pdcch-BlindDetection2 for the MCG and pdcch-BlindDetection2 for the SCG, and through pdcch-BlindDetectionCG-UE3 in pdcch-BlindDetectionMCG-UE-Mixed1 and pdcch-BlindDetectionCG-UE3 in pdcch-BlindDetectionSCG-UE-Mixed1 respective maximum values for pdcch-BlindDetection4 for the MCG and pdcch-BlindDetection4 for the SCG.

### 📚 增量术语表
* **pdcch-BlindDetectionMCG-UE-Mixed1 / SCG-UE-Mixed1**：升级版超级公文包（带 `-Mixed1` 后缀）。**一句话解释**：因为现在有 R15+R16+R17 三种业务了，以前只能装两份文件的老公文包不够用了，协议专门为这种终极混搭发明了能装三份文件的新型公文包。
* **pdcch-BlindDetectionCG-UE3**：公文包里的 3 号文件。**一句话解释**：专门负责锁定第 4 代账本（R17 养生模式）局部上限的新字段。

### 🚪 第一关：旧包换新包，装下“六大硬件瓶颈”
在上一段（六十段），我们看到了基站发下来的“六本账册”。
这段协议讲的是：**手机在接单之前，怎么向基站亮出自己这 6 个硬件模块各自的“短板底线”？**

因为场景升级到了终极的**“三色全开”（R15+R16+R17）**，手机必须上报 6 个局部极限值。
为了在信令上不显得乱七八糟，3GPP 再次对结构体进行了嵌套升级：启用了带有 **`-Mixed1`** 后缀的全新大文件夹。
在这个超级大文件夹里，塞进了三个格子：`UE1`、`UE2`、`UE3`。

### 🎯 第二关：萝卜拔了坑还在！六维硬连线映射
手机上交这两个超级公文包（一个代表主基站 MCG，一个代表辅基站 SCG）后，里面文件的映射关系被协议写得死死的，绝不允许乱套：

**主/辅基站公文包里的文件分工：**
* 📁 **1 号文件 (CG-UE1)** ──► 死死锁住 **BlindDetection3**（R15 常规配额）。
* 📁 **2 号文件 (CG-UE2)** ──► 死死锁住 **BlindDetection2**（R16 极速配额）。
* 📁 **3 号文件 (CG-UE3)** ──► 死死锁住 **BlindDetection4**（R17 养生配额）。

**生活类比**：
你是一家巨型外卖代运营公司的区域总管（终端）。
你手下有【东区配送站(MCG)】和【西区配送站(SCG)】。每个站都有三条独立的派送线：【普通单(R15)】、【闪送单(R16)】、【大件团购单(R17)】。
为了防止总部（基站）瞎派单把你某个站点撑爆，你给总部发了两个名为 **`Mixed1`** 的加密压缩包：
* 拆开《东区极限运力.Mixed1》：
  - UE1 报表：东区普通单最多接 50 单。
  - UE2 报表：东区闪送单最多接 10 单。
  - UE3 报表：东区大件单最多接 20 单。
西区的压缩包也是同样的格式。总部未来的排班系统，必须通过这 6 份报表的终极审核！

### 🔄 终极三色混搭嵌套公文包结构图
```text
【R15 + R16 + R17 三维双连接局部能力上报架构】

                    [ 手机终端 终极能力上报 ]
                                │
        ┌───────────────────────┴───────────────────────┐
        ▼                                               ▼
[主站公文包 MCG-UE-Mixed1]                     [辅站公文包 SCG-UE-Mixed1]
 │                                              │
 ├─ 📂 UE1: 锁定主站 R15 (BlindDet 3)             ├─ 📂 UE1: 锁定辅站 R15 (BlindDet 3)
 ├─ 📂 UE2: 锁定主站 R16 (BlindDet 2)             ├─ 📂 UE2: 锁定辅站 R16 (BlindDet 2)
 └─ 📂 UE3: 锁定主站 R17 (BlindDet 4)             └─ 📂 UE3: 锁定辅站 R17 (BlindDet 4)

*(通过极为整齐对称的 ASN.1 树状信令结构，完美消解了物理层六维矩阵分配的混乱)*
```

### 💡 章节硬核总结

**本段协议在 RRC 信令编码层面上为 5G PDCCH 监控机制的终极形态（Ultimate Concurrency）画上了句号。面对 NR-DC 叠加 Triple-Numerology 的六维状态空间，若继续沿用扁平化的参数定义，基带与网络侧的状态机将彻底陷入难以维护的代码泥潭。3GPP 在此展示了优雅的通信协议设计美学：通过引入全新的容器类型 `pdcch-BlindDetectionMCG/SCG-UE-Mixed1`，将原本的二维元组扩展为三维元组（`UE1`, `UE2`, `UE3`）。在语义绑定（Semantic Binding）上，协议强制执行了严格的正交配对：`UE1` 锚定 R15 的 `BlindDetection3`，`UE2` 锚定 R16 的 `BlindDetection2`，`UE3` 锚定 R17 的 `BlindDetection4`。这种底层硬件流水线能力的精准、结构化暴露，不仅赋予了调度器极限压榨多模基带潜能的权限，更构筑了防止异构信令风暴冲垮 L1 硅片的终极软件护城河。**

## 📖 协议原文拆解 (六十二)：终极公文包解密之“常规业务” (R15 产线的守恒与宽容)

> **协议原文**
> If the UE reports pdcch-BlindDetectionCA1 in pdcch-BlindDetectionMixedList3,
> - the value range of pdcch-BlindDetectionCG-UE1 for the MCG or of pdcch-BlindDetectionCG-UE1 for the SCG is [0, 1, …, pdcch-BlindDetectionCA1], and
> - pdcch-BlindDetectionCG-UE1 for the MCG + pdcch-BlindDetectionCG-UE1 for the SCG >= pdcch-BlindDetectionCA1.
>
> Otherwise, if $N_{cells}^{DL,NR-DC,max,r15}$ is a maximum total number of downlink cells for which the UE is provided monitoringCapabilityConfig = r15monitoringcapability and the UE is configured on both the MCG and the SCG for NR-DC as indicated in UE-NR-Capability
> - the value range of pdcch-BlindDetectionCG-UE1 for the MCG or of pdcch-BlindDetectionCG-UE1 for the SCG is [0, 1, 2],
> - pdcch-BlindDetectionCG-UE1 for the MCG + pdcch-BlindDetectionCG-UE1 for the SCG >= $N_{cells}^{DL,NR-DC,max,r15}$.

### 📚 增量术语表
* **pdcch-BlindDetectionCA1 in pdcch-BlindDetectionMixedList3**：终极三色菜单（List3）里的第 1 项总算力。由于这是 R15+R16+R17 的大锅烩，所以这个 `CA1` 在此语境下**死死锚定 R15 (常规业务) 的总算力天花板**。
* **pdcch-BlindDetectionCG-UE1**：终极公文包里的 1 号文件。代表**常规业务 (R15) 局部算力上限**。

### 🚪 第一关：换汤不换药的“溢出定理”
在上一段，手机上交了包含三个栏目的终极公文包。
这段协议开始逐一审查这三个栏目里填写的数字到底合不合法。首先被拉出来审查的是：**负责常规 R15 业务的 1 号文件 (`CG-UE1`)**。

审查逻辑完美复刻了前面的规律：
* **允许单边拔电源**：填写的数值可以是 `0`。这意味着，为了保住极速或者极高频的命脉，手机依然有权在主辅基站的某一边，彻底砍掉常规业务的处理能力。
* **溢出上报**：主基站填的 1 号值 + 辅基站填的 1 号值，必须 `>=` 总的常规算力（CA1）。赋予基站自由倾斜分配常规任务的灵活性。

### 🎯 第二关：日结业务的“宽大处理”
再看 `Otherwise`（缺省，未交体检报告）分支。
因为 1 号文件处理的是 **R15 常规业务（按天结，per slot）**，对芯片的瞬时压榨并不恐怖。
所以，即便手机偷懒没交底细，协议依然仁慈地给出了 **`[0, 1, 2]`** 的宽容值域。
这说明，在缺省状态下，基站最多敢给单边基站分配 2 个当量的 R15 常规盲检任务，而不用担心把基带烧掉。

### 🔄 终极混搭下 R15 产线的数值框定流
```text
【在三色全开(R15+16+17)双连接中，审查 R15 局部上限 (CG-UE1)】

        ├─► 有在终极菜单(List3)中提交 R15 专属总能力 CA1 吗？
        │
        ├──► 【有交】
        │       └─► 填表自由：主辅皆可在 [0 到 满值] 任意填！
        │           核验公式：[主 CG-UE1] + [辅 CG-UE1] >= 满值！
        │
        └──► 【没交】
                └─► 启动宽容缺省模式：
                    填表限制：主辅最高只能填到 2 (即范围 [0, 1, 2])
                    核验公式：加起来必须 >= 当前 R15 物理小区总数
```

### 💡 章节硬核总结

**本段协议在微观数值域上验证了 3GPP “Context-aware Capability Validation（基于上下文的能力校验）”的设计哲学。当解析引擎进入 `MixedList3` 这个包含三种时域颗粒度（per-slot, per-span, per-group）的终极 RRC 容器时，它首先针对代表 eMBB（per-slot）的 `CG-UE1` 字段执行校验。协议在这里毫无阻力地平移了前面所有混搭模式下对于 R15 产线的宽容法则：允许 `0` 值的极化配置（Polarized configuration），并在 Fallback 路径中保留 `[0, 1, 2]` 的安全缓冲范围。这确保了在极度复杂的调度状态机中，属于低优先级基础承载的 R15 盲检任务，其配置边界既不会越权占用其他两栖业务的资源池，也不会被过分严苛地一刀切死。**

## 📖 协议原文拆解 (六十三)：终极公文包解密之“极速业务” (R16 产线的严防死守)

> **协议原文**
> If the UE reports pdcch-BlindDetectionCA2 in pdcch-BlindDetectionMixedList3,
> - the value range of pdcch-BlindDetectionCG-UE2 for the MCG or of pdcch-BlindDetectionCG-UE2 for the SCG is [0, 1, …, pdcch-BlindDetectionCA2], and
> - pdcch-BlindDetectionCG-UE2 for the MCG + pdcch-BlindDetectionCG-UE2 for the SCG >= pdcch-BlindDetectionCA2.
> 
> Otherwise, if $N_{cells}^{DL,NR-DC,max,r16}$ is a maximum total number of downlink cells for which the UE is provided monitoringCapabilityConfig = r16monitoringcapability and the UE is configured on both the MCG and the SCG for NR-DC as indicated in UE-NR-Capability
> - the value range of pdcch-BlindDetectionCG-UE2 for the MCG or of pdcch-BlindDetectionCG-UE2 for the SCG is [0, 1],
> - pdcch-BlindDetectionCG-UE2 for the MCG + pdcch-BlindDetectionCG-UE2 for the SCG >= $N_{cells}^{DL,NR-DC,max,r16}$.

### 📚 增量术语表
* **pdcch-BlindDetectionCA2 in pdcch-BlindDetectionMixedList3**：终极三色菜单（List3）里的第 2 项总算力。在此语境下，它**死死锚定 R16 (极速 URLLC 业务)** 的总算力天花板。
* **pdcch-BlindDetectionCG-UE2**：终极公文包里的 2 号文件。代表**极速业务 (R16) 局部算力上限**。

### 🚪 第一关：保持队形！极速产线的“弹性溢出”与“单边断电”
接下来轮到审查终极公文包里的第 2 个文件了（负责极速 R16 业务的 `CG-UE2`）。
对于好学生（交了 `CA2` 专项体检报告的）：
* **允许断电 (含 0)**：面对要命的微秒级爆发任务，手机依然有权决定把某一侧基站的极速接收管线彻底关闭（填 0），全力保另外一侧。
* **动态溢出 ($\ge$)**：主辅上报的上限之和必须溢出总算力，给基站预留“削峰填谷”的灵活调度空间。

### 🎯 第二关：再现极刑！缺省状态下的“锁死惩罚”
当我们再次把目光移向 `Otherwise`（缺省）分支时，熟悉的冷酷配方又回来了！

由于 2 号文件审查的是 **R16 极速业务（按微秒/Span结）**，对芯片瞬时吞吐量的压榨达到了顶点。
只要手机敢不交底细，协议立刻启动最严苛的物理防爆模式：
**`the value range is [0, 1]`。**

* 不管基站怎么想给你加塞任务，在缺省状态下，单边基站的极速并发任务**绝对不能超过 1 个小区当量**！
* `[0, 1]` 的死锁，直接把极速调度的自由度降到了冰点，宁可降低网速和调度容量，也绝不能让基带芯片发生 Buffer 击穿导致整个手机死机。

### 🔄 终极混搭下 R16 产线的数值框定流
```text
【在三色全开(R15+16+17)双连接中，审查 R16 局部上限 (CG-UE2)】

        ├─► 有在终极菜单(List3)中提交 R16 专属总能力 CA2 吗？
        │
        ├──► 【有交】
        │       └─► 填表自由：主辅皆可在 [0 到 满值] 任意填！
        │           核验公式：[主 CG-UE2] + [辅 CG-UE2] >= 满值！
        │
        └──► 【没交】
                └─► 触发最高级别防爆惩罚！
                    填表限制：主辅最高只能填到 1 (即范围死死锁在 [0, 1])！
                    核验公式：加起来必须 >= 当前 R16 物理小区总数 N_r16！
                    (此条件直接反向卡死了 N_r16 在这种状态下最多只能是 2)
```

### 💡 章节硬核总结

**本段协议是对 5G L1 层“木桶效应（Canvas Effect）”管理的一次精准重演。当解析器步入 `MixedList3` 容器的第二维度（即表征 URLLC per-span 的 `CG-UE2` 节点）时，协议的容忍度骤然收紧。与上一段 R15 宽容的 `[0, 1, 2]` Fallback 值域形成极度强烈的反差，本段将 R16 的隐式回退边界粗暴地一刀切断至 `[0, 1]`。这种将极低延迟（Low-latency）与极高瞬时算力（High-peak-compute）强绑定的物理特质，通过冰冷的 ASN.1 枚举范围（Enumeration Range）映射进了信令规范。它在终极并发架构中，再次牢牢守住了 URLLC 调度的硬件绝对安全底线。**

## 📖 协议原文拆解 (六十四)：终极公文包解密之“养生业务” (R17 产线的重归宽容)

> **协议原文**
> If the UE reports pdcch-BlindDetectionCA3 in pdcch-BlindDetectionMixedList3
> - the value range of pdcch-BlindDetectionCG-UE3 for the MCG or of pdcch-BlindDetectionCG-UE3 for the SCG is [0, 1, …, pdcch-BlindDetectionCA3], and
> - pdcch-BlindDetectionCG-UE3 for the MCG + pdcch-BlindDetectionCG-UE3 for the SCG >= pdcch-BlindDetectionCA3.
> 
> Otherwise, if $N_{cells}^{DL,NR-DC,max,r17}$ is a maximum total number of downlink cells for which the UE is provided monitoringCapabilityConfig = r17monitoringcapability and the UE is configured on both the MCG and the SCG for NR-DC as indicated in UE-NR-Capability
> - the value range of pdcch-BlindDetectionCG-UE3 for the MCG or of pdcch-BlindDetectionCG-UE3 for the SCG is [0, 1, 2],
> - pdcch-BlindDetectionCG-UE3 for the MCG + pdcch-BlindDetectionCG-UE3 for the SCG >= $N_{cells}^{DL,NR-DC,max,r17}$.

### 📚 增量术语表
* **pdcch-BlindDetectionCA3 in pdcch-BlindDetectionMixedList3**：终极三色菜单（List3）里的第 3 项总算力。在此语境下，它**死死锚定 R17 (佛系大周期业务)** 的总算力天花板。
* **pdcch-BlindDetectionCG-UE3**：终极公文包里的 3 号文件。代表**佛系养生业务 (R17) 局部算力上限**。

### 🚪 第一关：收官之战！三色矩阵最后一块拼图
这是终极公文包里的最后一份文件，也是 3GPP 协议关于多载波、多版本混合盲检算力配置的**最后一段底层约束**。
审查 3 号文件（`CG-UE3`）的逻辑依然遵循着雷打不动的物理层契约：
* **包含 0 的值域**：支持彻底关闭单侧基站的 R17 处理线程，实现极致的跨节点业务专职化。
* **$\ge$ 的溢出效应**：支持两侧局部预算之和溢出全网总预算，确保基站在长周期调度时依然能左右逢源、动态挪腾。

### 🎯 第二关：长周期=低压力！缺省模式的“免刑令牌”
让我们再一次欣赏 3GPP 协议在数值域设计上的极致工整：
在 `Otherwise` (未上报能力底牌) 的缺省分支里，由于 3 号文件掌管的是 **R17 跨时隙组 (per group of Xs slots)** 业务。
因为时域尺度被极大拉伸，瞬时的极化译码（Polar Decoding）压力被宽裕的时间窗彻底稀释。
所以，协议立刻收起了上一段（六十三段）对待 R16 极速业务那套“死锁在 [0,1]”的酷刑。
**R17 的缺省值域，毫无悬念地恢复到了 `[0, 1, 2]` 的宽容状态！**

**生活类比回顾**：
至此，我们已经看遍了三条流水线在外包不交体检报告时的默认惩罚：
* **常规线 (R15, 日结)**：允许单边接 2 单。 (难度中等)
* **加急线 (R16, 秒结)**：只准单边接 1 单！(会死人的，严防死守)
* **大件线 (R17, 周结)**：允许单边接 2 单。 (慢慢做总能做完)

### 🔄 终极混搭下 R17 产线的数值框定流
```text
【在三色全开(R15+16+17)双连接中，审查 R17 局部上限 (CG-UE3)】

        ├─► 有在终极菜单(List3)中提交 R17 专属总能力 CA3 吗？
        │
        ├──► 【有交】
        │       └─► 填表自由：主辅皆可在 [0 到 满值] 任意填！
        │           核验公式：[主 CG-UE3] + [辅 CG-UE3] >= 满值！
        │
        └──► 【没交】
                └─► 恢复宽容缺省模式：
                    填表限制：主辅最高可以填到 2 (即范围 [0, 1, 2])
                    核验公式：加起来必须 >= 当前 R17 物理小区总数 N_r17！
```

### 💡 章节硬核总结

**本段协议为 5G Rel-17 的 PDCCH 盲检预算大系（Overbooking Budget System）敲下了终场定音的法槌。当解析器步入 `MixedList3` 的第三维度（表征 RedCap/FR2 per-group 的 `CG-UE3` 节点）时，协议在 Fallback 逻辑中完美复刻了 `[0, 1, 2]` 的弛豫值域（Relaxed Value Range）。至此，R15 的宽容、R16 的极刑、R17 的回归，在长达数十段的数学公式编织中，构成了一套蔚为大观且逻辑绝对自洽的 L1 约束框架。它向整个通信工业界证明：无论未来上层网络如何把不同时效（Numerology）、不同节点（DC）、不同载波（CA）的业务像俄罗斯俄罗斯方块一样疯狂堆叠，底层终端的基带硅片都能依靠这套极其工整、正交隔离、带有动态弹性和缺省防灾的 ASN.1 代数方程组，牢牢守住不崩溃的最后底线。**

---
*(长舒一口气)*
**第 10 章最艰涩、最绕脑的“盲检算力矩阵配置”山峰，已经被我们彻底翻越了！**

