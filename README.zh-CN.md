<div align="center">

[English](./README.md) · [日本語](./README.ja-JP.md) · **简体中文**

<img height="100" alt="TocoAI" src="assets/logo.png" />

<h1>TocoAI</h1>

<h3>基于 DSL-Spec 的服务端 Harness Engineering</h3>

[![][docs-shield]][docs-link]
[![][doc-license-shield]][doc-license-link]
[![][stack-shield]][stack-link]
[![][engine-shield]][engine-link]

<br/>

[![][github-stars-shield]][github-stars-link]
[![][github-issues-shield]][github-issues-link]
[![][github-contributors-shield]][github-contributors-link]


</div>

---

TocoAI 是服务端研发的 **Harness Engineering** 方案。Spec 不应是一次性生成代码的文本，而是**控制系统的结构层**。

- 通过 **DSL-Spec** 约束大模型生成，并保持和代码持续一致。

- 通过 **建模引擎** 稳态渲染结构性代码，使代码始终稳定不漂移

让 AI 在持续迭代维护的服务端项目中始终在人的设计意图下工作。

<video src="https://github.com/user-attachments/assets/9c3dab56-f927-42d6-83c0-77a25ff65ecc" controls width="100%"></video>

> [!TIP]
> 写一个能跑的系统，用 Cursor。建一个能长期维护的系统，用 TocoAI。

<details>
<summary><kbd>目录</kbd></summary>

- [🚀 快速开始](#-快速开始)
- [🏗️ Harness Engineering 思路](#️-harness-engineering-思路)
- [⚙️ 核心组件](#️-核心组件)
  - [📐 DSL-Spec](#-dsl-spec)
  - [🔧 建模引擎](#-建模引擎)
  - [🧭 Human-in-the-Loop](#-human-in-loop)
- [⚖️ 与其他工具的对比](#️-与其他工具的对比)
- [🎬 演示案例](#-演示案例)
- [📋 真实案例](#-真实案例)
- [🗺️ 适用场景和局限性思考](#️-适用场景和局限性思考)
- [🤝 社区与参与](#-社区与参与)

</details>

---

## 🚀 快速开始

- 安装和配置
  - [IntelliJ Plugin →][intellij-link]
  - [VS Code Plugin →][vscode-link]
- [DSL-Spec 语法参考 →][dsl-docs-link]

---

## 🏗️ Harness Engineering 思路

服务端系统需要长时间维护，更像是建造一座大厦，而不是 3D 打印。我们用**建筑思维**来构建：

- 我们需要精准的设计图纸——于是定义了一套类 DDD、符合 CQRS 的 **DSL-Spec**
- 图纸有了，我们更需要一致且可维护的代码结构——于是开发了**建模引擎**，稳态还原结构性代码，这部分在复杂项目中占到约 **80%**
- 最后在结构稳固、接口齐全的大楼里，用 AI 完成室内装修——那些难以用 DSL-Spec 描述的 if/else 业务逻辑

<div align="center">

<img src="assets/tocoai-arch.png" alt="TocoAI 架构图" width="100%"/>

</div>

---

## ⚙️ 核心组件

### 📐 DSL-Spec

> *"形式化符号是排除各种胡说八道的极其有效的工具。自然语言的'自然性'，不过是我们能轻松地用它说出那些荒谬性并不显而易见的话。"*
>
> — Edsger Dijkstra, EWD667, 1978

需求和架构设计统一表达为结构化的 DSL-Spec，作为整个系统的 **Single Source of Truth**。DSL-Spec 人类可读，机器可解析。它不是配置文件——**它是可执行的架构意图**。

DSL-Spec 覆盖服务端系统的完整设计层级：

| 层级 | 元素 |
|------|------|
| **领域模型** | Entity / Relation / Enum / EO (Value Object) |
| **聚合对象** | BO / 领域事件 |
| **数据传输层** | DTO / VO |
| **查询方案** | ReadPlan |
| **写入方案** | WritePlan (Based on BO) |
| **流程方案** | FuncFlow（任务编排）/ 消息管理 |
| **服务接口** | API / RPC / Service |

<br/>

一个 DTO 的 DSL-Spec 定义，自动生成：

<table>
<tr>
<th width="22%">含糊需求</th>
<th width="50%">DSL-Spec</th>
<th width="28%">自动生成文件</th>
</tr>
<tr>
<td>

*"房型信息，包含对应的房间列表——另外还有可用房间数量。"*

| 歧义点 | DSL 决策 |
|---|---|
| 根实体是哪个？ | `fromEntity: room_type` |
| 房间如何关联？ | `room.room_type_id` 外键 → **反向注入** |
| `available_count` 从哪来？ | 实体中无此字段 → `customField` |

</td>
<td>

```json
{
  "dto": {
    "name": "room_type_with_rooms_dto",
    "fromEntity": "room_type",
    "reverseExpandList": [{
      "foreignKeyInOtherEntity": "room_type_id",
      "dtoFieldName": "room_list",
      "dto": { "name": "room_base_dto", "fromEntity": "room" }
    }],
    "customFieldList": [{
      "name": "available_count",
      "type": "Integer"
    }]
  }
}
```

</td>
<td>

```
RoomTypeWithRoomsDto.java                (~60 Lines)
RoomTypeWithRoomsDtoManager.java         (~25 Lines)
RoomTypeWithRoomsDtoManagerImpl.java
RoomTypeWithRoomsDtoConverter.java       (~80 Lines)
RoomTypeWithRoomsDtoService.java         (~70 Lines)
RoomTypeWithRoomsDtoDataAssembler.java
RoomTypeWithRoomsDtoBaseDataAssembler.java
```

</td>
</tr>
</table>

[查看完整 DSL-Spec 语法文档 →][dsl-docs-link]

<br/>

#### DSL-Spec：系统研发的 Ontology

传统开发中，测试、CI/CD、变更管理各自依赖不同的数据来源——测试依赖开发者对代码的理解，CI 只能验证编译和用例，变更管理靠人工追踪影响面。这些流程并非真正"读懂"了系统，而是在代码事实发生后被动跟进。DSL-Spec 的核心价值在于成为整个研发链路的唯一 Ontology（本体）：系统的结构、规则、关系在 Spec 中一次定义，所有下游流程从同一个真相派生，而不是各自维护一份对现实的近似。

1. **测试**：Spec 中显式声明的业务规则直接成为测试断言的来源，测试边界从 Spec 派生而非从代码逆向推导；Function Flow 每个节点通过 Context 对象定义清晰的输入输出契约，单元测试可独立针对节点编写，无需 mock 完整调用链。引擎生成的 80% 结构性代码其正确性由引擎保证，测试资源得以集中在真正有业务风险的 20%。

2. **CI/CD**：流水线可内置 Spec 合规检查——验证代码库中的生成代码是否与当前 Spec 完全一致，一旦有人绕过引擎手改，CI 立即拦截，从机制上杜绝技术债的产生；模型变更时引擎同步输出 DDL migration 脚本，CI 将 schema 变更与代码部署强制绑定按序执行，消除数据库与代码不同步的窗口期；API 签名变化由引擎自动 diff，CI 可判断是否为 breaking change，决定是否触发接口版本升级流程。

3. **变更管理**：修改一个聚合字段，引擎自动计算并列出所有受影响的 Service、DTO、API 与数据库表，影响面从 Spec 直接推导，不再依赖开发者经验；变更评审从看代码 diff 变成看 Spec diff，非技术干系人也能直接参与；每次 Spec 提交自动形成结构化变更记录，完整追溯"谁改了什么、波及了哪些模块"，可直接作为合规审计的溯源依据。

<br/>

|  | 自然语言 Spec | 程序式 IaC | **DSL-Spec** |
|--|:------------:|:---------:|:----------:|
| **含义明确、可读性强** | 人人能读，但歧义不报错，只在生产事故里现身 | 精确，但需理解执行流程才能看懂意图 | ✅ 像自然语言一样可读，像程序一样精确，模糊即编译错误 |
| **细节清晰（What + How）** | 只有模糊的 What，How 全靠 AI 自由发挥 | 只有 How，架构意图藏在执行逻辑里无法单独审查 | ✅ What（数据意图）与 How（结构约束）同时显式表达 |
| **可校验、易维护（变更可查）** | 机器无法校验，改一个字段靠人肉追踪影响范围 | 可执行但无独立意图层，变更影响面需依赖分析工具 | ✅ 机器可解析可校验，改 DSL-Spec，引擎自动级联同步所有受影响结构 |
| **始终和代码一致** | 启动时准确，三个月漂移，六个月成历史文献 | 代码即实现，但架构意图随迭代逐渐丢失 | ✅ Spec 与代码的关系永远是 `=`，不是 `≈` |
| **对研发全链路的本体价值** | 无法作为本体，测试/CI/变更管理各自维护一份对系统的近似理解 | 可作为执行层本体，但意图层缺失，下游流程难以校验设计意图 | ✅ 统一 Ontology，测试断言、DDL 变更、影响面分析均从同一真相派生 |

<br/>

### 🔧 建模引擎

引擎覆盖全部结构性代码的生成：

- 分层骨架（`persist` / `manager` / `service` / `entrance`）
- 接口契约与数据模型
- CQRS 命令查询拆分
- 跨层转换器（`DtoConverter` / `VoConverter`）
- 数据拼装器（`DataAssembler`）
- 聚合写链路（`BoService`）

**开发者只需关注剩余 20% 的业务逻辑**，Review 成本骤降一个数量级。

> [!NOTE]
> 建模引擎的本质，是将软件工程的质量控制**左移到"设计时"**，将 DSL-Spec 定义的代码框架确定性地渲染生成。结构性代码由引擎生成，而非由大模型逐次生成——这从根本上消除了 AI 的随机错误和架构漂移。无论项目迭代多少轮、经手多少人，基础结构始终和设计保持一致，不会随时间腐化。

> [!IMPORTANT]
> 引擎计划于 **2026 年下半年**开源。

<br/>

### 🧭 Human-in-the-Loop

AI 从来不是神。对于对系统有长远影响的领域划分、数据结构定义、接口描述、读写方案（事务约束、操作性能），我们提供 AI 修改与手动修改相结合的标准手段。

**让 AI 始终是人的助理，而不是主人。**

---

## ⚖️ 与其他工具的对比

Cursor、Claude Code 是优秀的通用编程助手，事实上 TocoAI 内部也在使用它们实现业务逻辑。我们解决的是不同层面的问题。

> [!NOTE]
> TocoAI 的设计场景是：**关系型数据库驱动的服务端系统，需要长期迭代、多人协作**。如果你在做原型验证、脚本工具、前端项目，Cursor 就够了。

|  | Cursor / Claude Code | TocoAI |
|--|:--------------------:|:------:|
| **定位** | 通用对话式编程助手 | 服务端工程 Harness，特定场景的结构化解决方案 |
| **代码来源** | LLM 按 prompt 生成 | DSL-Spec → 引擎确定性生成（80%）+ LLM 实现业务逻辑 |
| **架构一致性** | 依赖 prompt 和 Review 维持 | 由 DSL-Spec + 引擎保证，不依赖人 |
| **适合阶段** | 快速验证、原型、碎片化需求 | 需要长期维护的复杂业务系统 |
| **团队规模** | 个人或小团队效果最好 | 团队越大、迭代越久，优势越明显 |
| **学习成本** | 几乎没有 | 需要理解 DSL-Spec 和建模方式 |

---

## 🎬 演示案例

### BnB 民宿预订系统

一个完整的民宿预订平台，全链路覆盖房型管理、购物车、订单结算、库存校验、会员积分消费记录——用来演示 TocoAI 从需求到交付的完整工作流。

**业务覆盖：** 房型库存 &nbsp;·&nbsp; 购物车 &nbsp;·&nbsp; 订单 / 子订单 &nbsp;·&nbsp; 支付 &nbsp;·&nbsp; 积分扣减 &nbsp;·&nbsp; 消费明细

[过程示例 →][bnb-demo-link] · [项目代码 →][bnb-code-link]

---

## 📋 真实案例

### 大型医院 HIS 系统

**背景：** 新一代全院管理系统，120+ 核心模块，200+ 业务流程，多团队并行开发。

**难点：** 医疗业务零容错，多人跨时间协作极易产生逻辑漏洞和技术债。

**结果：** 架构规范统一执行，生成代码准确率显著提升，整体项目 AI 代码采纳率近 **97%**，交接时新成员可快速上手项目全局。

[查看案例详情 →][case-his-link]

<br/>

### 金融保证金支付系统重构

**背景：** 重构遗留系统，原技术栈为 SQLServer 2008 + 大量存储过程 + 多个三方中间件。

**难点：** 业务流程散落，数据和业务关联关系不明，性能瓶颈明显。

**结果：** 通过 DSL-Spec 重新建模，业务关联关系显式化，端到端流程可追溯，系统可在 TocoAI 框架下持续迭代。

---

## 🗺️ 适用场景和局限性思考

所有 DSL 都是 **domain-specific** 的。它只在特定领域内有效，不存在能覆盖一切的通用 DSL——强行通用化的尽头是造出另一门编程语言。TocoAI 的 DSL-Spec 只描述关系型数据库驱动的服务端系统，这是有意为之的边界，不是能力限制。

即使在单一领域内，DSL-Spec 也不是用来取代编程代码的。我们的边界是**把适合结构化描述的信息 DSL-Spec 化**，其余的业务逻辑交给编程语言和 AI。

**老项目兼容：** 可以在老项目中开发新模块，存量遗留代码的整体接管不在覆盖范围内。正确的路径是用新模块承接新需求，逐步把老逻辑迁移进 DSL-Spec 管辖范围——**重构是 AI 时代的日常工作**。

Harness Engineering 的思路本身是通用的，期待其他团队在各自领域延伸——嵌入式、前端组件、基础设施配置，都可以考虑属于自己的 DSL-Spec。

---

## 🤝 社区与参与

- 访问 [tocoai.cn][docs-link] 获取完整文档
- 加入 [Discord 社区][discord-link] 提问、讨论功能和分享实践
- 在 [GitHub Issues][github-issues-link] 提交 Bug 报告或功能建议
- 建模引擎计划于 **2026 年下半年**开源，欢迎 Star 关注

没有完美的技术，只有适应的场景。毕竟 AI 改变研发才一年，但软件开发本身已经有七八十年的历史。

[贡献指南 →](CONTRIBUTING.md)

---

<div align="center">

Copyright © 2025 TocoAI. 文档基于 [CC BY 4.0][doc-license-link] 协议发布。

</div>

<!-- LINK GROUP -->
[docs-shield]: https://img.shields.io/badge/文档-tocoai.cn-brightgreen?style=flat-square&color=73DC8C&labelColor=black
[docs-link]: https://tocoai.cn/docs

[license-shield]: https://img.shields.io/badge/License-Apache_2.0-blue?style=flat-square&color=4B78E6&labelColor=black
[license-link]: LICENSE

[doc-license-shield]: https://img.shields.io/badge/文档协议-CC%20BY%204.0-lightgrey?style=flat-square&color=a78bfa&labelColor=black
[doc-license-link]: https://creativecommons.org/licenses/by/4.0/

[stack-shield]: https://img.shields.io/badge/Stack-Java_|_Spring_Boot-orange?style=flat-square&color=ffcb47&labelColor=black
[stack-link]: https://tocoai.cn

[engine-shield]: https://img.shields.io/badge/引擎开源-2026_下半年-pink?style=flat-square&color=FA9BFA&labelColor=black
[engine-link]: https://tocoai.cn/docs/engine

[github-stars-shield]: https://img.shields.io/github/stars/toco-ai/toco-ai?style=flat-square&color=ffcb47&labelColor=black&logo=github
[github-stars-link]: https://github.com/toco-ai/toco-ai/stargazers

[github-issues-shield]: https://img.shields.io/github/issues/toco-ai/toco-ai?style=flat-square&color=ff80eb&labelColor=black&logo=github
[github-issues-link]: https://github.com/toco-ai/toco-ai/issues

[github-contributors-shield]: https://img.shields.io/github/contributors/toco-ai/toco-ai?style=flat-square&color=c4f042&labelColor=black&logo=github
[github-contributors-link]: https://github.com/toco-ai/toco-ai/graphs/contributors

[intellij-link]: https://tocoai.cn/docs/installation
[vscode-link]: https://tocoai.cn/docs/installation-vscode
[dsl-docs-link]: ./assets/dsl.md
[engine-docs-link]: https://tocoai.cn/docs/engine
[bnb-demo-link]: https://tocoai.cn/docs/your-first-toco-project
[bnb-code-link]: https://github.com/toco-ai/homestay
[case-his-link]: ./case-study/TocoAI-HIS-DrugInventory-Case-Study.zh-CN.md
[case-finance-link]: https://tocoai.cn/cases/finance
[discord-link]: https://discord.gg/NubsdbF3MK
