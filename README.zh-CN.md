<h1 align="center">Toco AI</h1>

<p align="center">
  <b><a href="https://github.com/toco-ai/toco-ai/blob/main/README.md">English</a> | <a href="https://github.com/toco-ai/toco-ai/blob/main/README.ja-JP.md">日本語</a> | <a href="https://github.com/toco-ai/toco-ai/blob/main/README.zh-CN.md">中文 (Simplified)</a></b>
</p>

<p align="center">
  <a href="https://tocoai.dev/en/docs/installation"><img src="https://img.shields.io/badge/IntelliJ_IDEA-2024.03+-blue?style=for-the-badge" alt="IntelliJ IDEA" /></a>
  <a href="https://tocoai.dev/en/docs/installation-vscode"><img src="https://img.shields.io/badge/VS_Code-1.82.0+-blue?style=for-the-badge" alt="VS Code" /></a>
  <a href="https://discord.gg/NubsdbF3MK"><img src="https://img.shields.io/badge/Discord-Join_Community-5865F2?style=for-the-badge" alt="Discord" /></a>
  <a href="https://x.com/TocoAI"><img src="https://img.shields.io/badge/X-@TocoAI-000000?style=for-the-badge" alt="X" /></a>
</p>

> **TocoAI: Stop Vibe Coding. 建模驱动的 AI Coding 工具。**
> 先建模，后写代码。告别“开盲盒”式的 AI 编程。

[免费体验申请 (首月赠 30,000 Credits)](https://tocoai.dev/) | [官方文档](https://tocoai.dev/) | [Discord 社区](https://discord.gg/NubsdbF3MK) | [关注 X](https://x.com/TocoAI)

---

### ⏱️ 10秒看懂 Toco AI
![TocoAI: 需求 -> 模型 -> 80% 确定性代码 极速演练|348](https://tocoai-asset.oss-cn-hangzhou.aliyuncs.com/assets/toco-hero-demo.gif)
*<p align="center">👆 <b>更可靠的AI架构师</b> ➔ <b>更可控的后端代码</b> ➔ <b>更轻松的后端变更</b></p>*

> 💡 **来自 TocoAI 团队的心里话**
> 感谢关注！做 TocoAI 的初衷很简单：我们受够了 AI 无视架构乱写代码，也厌倦了给 AI 逐行排雷。希望这款工具能让你告别到处打补丁的日常，把精力真正还给业务设计。欢迎和我们一起，建立更可靠的 AI 研发规范！🚀

---

### 🎯 我们解决什么问题？
现在用 AI 写代码（比如 Cursor 或 Claude Code），大家最怕遇到什么？
1. **代码越改越乱**：只靠自然语言对话，AI 大模型无法维持对整个系统架构的长期记忆。改个小需求，大模型很容易破坏底层数据结构或漏改关联业务，导致代码逻辑和原有设计脱节。这就是为什么 TocoAI 必须引入可视化的**领域驱动设计（DDD）**，让可视化设计模型成为管住 AI 乱写代码的“标准图纸”与“操作底线”。
2. **Review 到眼瞎**：不敢完全相信 AI 大模型，生成的每一行代码都要仔细盯，生怕埋了隐蔽的 Bug，心智负担极大。

**Toco AI 的思路不一样：**
我们不让 AI 大模型直接“猜”代码。而是让 AI 先把业务领域模型（包含实体、聚合、值对象等）和读写服务设计好，你确认这些可视化设计模型没问题后，系统再根据设计模型**确定性地**生成代码。

---

### ⚙️ 它是怎么工作的？
为了消灭大模型生成代码的幻觉，Toco AI 把开发流拆成了两步：

```text
      [ Natural Language Description / Reference Documents ]
                         │
                         ▼
┌─────────────────────────────────────────────────┐
│                 Toco AI Architect               │
│ Output: Visual Models (Domain / R&W Services)   │
└─────────────────────────────────────────────────┘
                         │ (Models verified by you)
                         ▼
┌───────────────────────┐ ➕ ┌───────────────────────┐
│ Internal Code Engine  │    │     AI Programmer     │
│(Gen 80% Skeleton Code)│    │ (Fills 20% Core Logic)│
│  Zero-hallucination,  │    │ e.g., complex checks/ │
│    100% accurate      │    │    math calculations  │
└───────────────────────┘    └───────────────────────┘
            │                            │
            ▼                            ▼
   [ Clean, Standardized, and Maintainable Backend Code ]
```

1. **AI 帮你建模**：把业务文档喂给工具，AI 自动生成可视化的设计模型，并作为项目的永久记忆保存下来。
2. **引擎生成 80% 骨架结构代码**：内置引擎把设计模型 1:1 翻译成符合 DDD（领域驱动设计）规范的骨架结构代码。**这部分代码是由引擎确定性生成的，0 幻觉，不用 Review**。
3. **AI 填 20% 核心业务逻辑**：剩下的特定业务细节（如积分怎么扣、权限怎么卡），由 AI 程序员在骨架内填充。**你只需要盯着这 20% 的核心业务逻辑代码看就行**。
4. **一处修改，全局同步**：需求变了？直接去修改可视化设计模型。系统会自动触发级联更新，**相关的领域模型、读写服务以及 80% 的骨架结构代码都会实现自动的级联同步与全量对齐**。随后，AI 程序员将自动在新的骨架内为您调整变更那 20% 的核心业务逻辑，极大降低漏改隐患。

---

### ⚔️ 和主流 AI 编辑器比一比

| 破局维度 | Toco AI (建模驱动) | 传统 AI 助手 (如 Cursor / Claude Code) |
| --- | --- | --- |
| **代码怎么来的** | **引擎 + 设计模型**：80% 的骨架结构代码由引擎确定性生成，架构清晰、符合规范且零幻觉。 | **全靠大模型猜**：完全靠提示词进行概率生成，容易跑偏。 |
| **你的日常工作** | **做设计**：专注核心业务建模（领域模型/读写服务）、接口契约与顶层架构流转。 | **当审核员**：逐行查代码语法，到处排雷。 |
| **Review 压力** | **做填空题**：审查成本大幅下降，只需看 20% 的核心业务逻辑。 | **改小作文**：每一行都得看，负担极重。 |
| **改需求时** | **修改设计模型**：设计模型一变，代码与设计自动全量对齐同步。 | **到处打补丁**：纯靠文本让 AI 大模型找哪里该改，极易漏改。 |

---

### ⚡ 免费体验申请
为了控制大模型的高昂算力成本，同时保证**稳定且高质量的生成体验**，Toco AI 现已全面开放免费体验申请。申请通过后，首月免费赠送 **30,000 Credits** 额度。

👉 **[前往官网申请免费体验](https://tocoai.dev/)**

⭐ **强烈建议您先 [Follow toco-ai 组织](https://github.com/toco-ai)，或加入 [Discord 社区](https://discord.gg/NubsdbF3MK)、关注 [官方 X 账号](https://x.com/TocoAI)。当产品发布重大更新或发放专属福利时，您将第一时间收到通知！**

**获批后的极速启动步骤：**
1. **安装插件**：支持主流 IDE，插件下载与详细配置请参考 [IntelliJ 官方安装指南](https://tocoai.dev/en/docs/installation) 或 [VS Code 官方安装指南](https://tocoai.dev/en/docs/installation-vscode)。
   * **IntelliJ IDEA (2024.03+)**：进入 `Settings ➔ Plugins ➔ ⚙️ ➔ Install Plugin from Disk...` 选择下载的 `.zip` 安装包并重启。
   * **VS Code (1.82.0+)**：在扩展面板选择 `Install from VSIX…` 安装下载的 `.vsix` 包并重启。
2. **登录体验**：在 IDE 侧边栏打开 Toco AI 面板，登录账号。点击「**示例需求**」，即可体验从“需求 -> 建模 -> 出代码”的确定性极速闭环。

---

### 🔌 生态配合与全栈支持 (Ecosystem & Full-Stack)
现代研发讲究工具链协同，Toco AI 致力于融入并赋能您的全栈开发流：

| 生态维度 / 场景 | 支持状态 | 核心工作模式与价值 |
| ------ | ------ | ------ |
| **后端工程 (MDD 模式)** | ✅ 灵活的全栈打法 | 新建 Toco Project 开启独创的模型驱动模式，专注处理基于数据库驱动的后端工程。在应对企业级后端工程与频繁需求变更时，其提效与防止代码越改越乱的优势将成倍放大。 |
| **前端工程 (对话模式)** | ✅ 灵活的全栈打法 | 在非 Toco 项目中，Toco AI 无缝降级为类似 Claude Code 的增强型对话助手。其内置的后端 API MCP（模型上下文协议）能力，让前端开发也能精准感知后端接口契约。 |
| **多语言支持** | 🚧 高速开发中 | MDD 引擎目前已深度支持 Java Spring 体系，基于 Go / Python / Node.js 的架构生成引擎正在高速开发中。 |
| **MCP 协议广度生态** | 📅 规划中 | 未来，借助 MCP 生态协议，Toco AI 将支持更丰富的自定义能力扩展，让您可以无缝接入企业的私有开发工具链、内部知识库以及更多的外部功能套件，打通全栈协同中枢。 |
| **Agent Skills (智能体技能集市)** | 📅 规划中 | 开放自定义技能，你可以把团队常用的开发 SOP 写成 Agent 技能，在社区共享“怎么让 AI 干活”的经验。 |

---

### 🔒 隐私与安全
你的代码只属于你，我们为核心业务资产提供最高规格的保护：
* **数据主权承诺**：默认情况下，你的本地代码绝不会被擅自上传，且业务需求、生成的可视化设计模型和代码，绝不会被我们拿去训练公共 AI 大模型。除非你主动在偏好设置中选择开启数据共享，且即便开启，所有数据也会经过严格的去标识化与匿名化处理。
* **支持私有化**：支持企业级私有化部署，数据全落盘在本地或你信任的私有云环境。
* **云端基建安全**：平台数据落盘于业界顶级的云数据中心（AWS / Google Cloud 等），通过严格的网络隔离设计，确保工程文件的高可用与隐私的强隔离保护。

---

### 🤝 欢迎来社区玩
我们希望和开发者一起，建立更好的 AI 研发规范：

* **贡献 `.toco` 规则**：欢迎在 [toco-rules-library](https://github.com/toco-ai/toco-rules-library) 仓库提交和优化 **TocoAI 各个 Agent（如领域架构师、规划者、开发者等）的规则提示词模板**。
* **抄作业与最佳实践**：去 [toco-domain-models](https://github.com/toco-ai/toco-domain-models) 仓库获取电商、医疗等真实场景的参考设计模型；在官方聚合仓库 [awesome-tocoai](https://github.com/toco-ai/awesome-tocoai) 探索由社区贡献的完整项目源码与架构模板。

---

**Stop Vibe Coding. Start Deterministic Engineering.**

👇 **[点击这里申请免费体验](https://tocoai.dev/)，或加入 [Discord 社区](https://discord.gg/NubsdbF3MK) 和大家聊聊！**
[关注 @TocoAI 最新动态获取第一手资讯](https://x.com/TocoAI)
