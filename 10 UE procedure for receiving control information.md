---
# 🔖 章节：10 UE procedure for receiving control information 
*(终端接收控制信息的流程)*
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

