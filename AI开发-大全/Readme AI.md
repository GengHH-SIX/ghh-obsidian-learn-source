
## 本地环境（已配置的工具）

1. 已经安装的IDE
	 - ==Vscode + github copilt== **（微软）**
	 - ==Qoder CN IDE== **(阿里)**
	 - TraeCode **(字节)**
	 - Cursor **（SpaceX \[Anysphere\]）**
	 
2. 已安装的CLI （命令行 AI 编程助手）
	- ==Claude Code== **(Anthropic)**  --通过CC-Switch 配置和使用国内的大模型
	-  Quder  CLI CN **(阿里)** --手动安装
	- 百炼 CLI **(阿里)** --根据官网教程将`安装`和`鉴权`通过`Qoder CN` App（就是Agent）来执行
	
	
	✅ 一句话总结
	- ‌**百炼 CLI**‌ = 百炼平台的终端控制台，管的是"大模型和应用" （*偏向Agent，感觉更加底层；通过指令 bl .. 直接操作bl Agent 操作 大模型*）
	- ‌**Qoder CLI CN**‌ = 终端版 AI 编程助手，管的是"写代码" （*偏向Codeing, 通过输入自然语言，使得Agent判断需要调用的工具，在终端中执行响应的指令来完成编码）
	
3. 已安装的工具App
	  - CC-Switch **（开源社区）**
	  - Warp (终端)
	  - ==Qoder CN== (**阿里** 相当于是‘Qoder CLI 助手’)  ==**好像没有登录能领部分免费额度**==
	  
    ✅ 一句话总结
	- ‌**Qoder CN**‌ = 桌面版 AI 编程助手，管的是"写代码" ，类似一个==程序员==，你通过==自然语言下达要求==！！！更适合非编程人员。（*偏向Codeing, 通过输入自然语言，使得Agent判断需要调用的工具，在终端中执行响应的指令来完成编码；同时也能通过指令执行`百炼 CLI` 的指令，来控制“大模型和应用”，感觉更高层一些*）
	
  4. 使用的大模型厂商
  - ==阿里百炼== (https://bailian.console.aliyun.com/cn-beijing/costing-balance/free-quota?model=qwen-image-2.0)  --免费额度
  - ==火山方舟（火山引擎）==(https://console.volcengine.com/ark/region:cn-beijing/subscription/coding-plan)  --免费额度 + 购买的Codeing Plan
  - 智普 (https://bigmodel.cn/console/overview)
  - cebence (https://cubence.com/dashboard)


## 国外生态

| 所属公司/组织                                                                              | 自研主要大模型                                                                                                              | 工具名称                | 产品定位                             |
| ------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------- | ------------------- | -------------------------------- |
| Anthropic（🇺🇸美国）                                                                    | ‌Claude Opus 5‌（最新旗舰）、‌Claude Fable 5‌（旗舰通用）、‌Claude Mythos 5‌（高阶有限开放）；还有 Claude Sonnet、Claude Haiku 等轻量版            | ‌==Claude‌ Code==   | 大语言模型 + 官方编程工具 Claude Code       |
| OpenAI（🇺🇸美国）                                                                       | ‌GPT-5.6 Sol‌（旗舰）、‌GPT-5.6 Terra‌（均衡）、‌GPT-5.6 Luna‌（轻量）；前代：GPT-4o、o3/o4-mini、GPT-5.x 系列                             | ‌==Codex‌==         | 通用 AI 智能体 / 编程助手                 |
| Anomaly（开源社区）                                                                        | ❌ 不自研模型，支持接入 75+ 种第三方大模型（Claude、GPT、Gemini、DeepSeek、Ollama 等）                                                        | ‌OpenCode‌          | 开源 AI 编程代理（Coding Agent）         |
| **Anysphere**‌（🇺🇸 美国）2026 年 6 月被 ‌**SpaceX（马斯克旗下）以 600 亿美元全股票收购**‌，现为 SpaceX 全资子公司 | Claude Opus 5、Grok 4.6（xAI/马斯克自研）、Gemini 等                                                                           | ==Cursor==          | AI 原生代码编辑器（基于 VS Code 深度改造，不是插件） |
| Google 谷歌（🇺🇸美国）                                                                    | ‌Gemini 3.8‌ 系列（最新）、‌Gemini 3.8 Live‌（实时语音）、‌Gemini Ultra‌（旗舰）、‌Gemini Pro‌（主力）、‌Gemini Nano‌（端侧）、‌Gemini Flash‌（轻量） | ‌Gemini Code Assist | 大语言模型系列                          |
| Nous Research（🇺🇸美国，开源）                                                             | ‌Hermes 系列‌（Hermes 3 等，基于 Llama 微调，侧重工具调用与长程规划）、‌Nomos 系列‌；Hermes Agent 是其智能体框架产品                                    | ‌Hermes‌            | 开源 AI 智能体框架（OpenClaw 进化版）        |
| Amantus Machina（奥地利，开源）                                                              | ❌ 不自研模型，框架本身支持接入多种大模型                                                                                                | ‌OpenClaw‌          | 开源 AI 智能体框架（"龙虾"）                |
| Inflection AI（🇺🇸美国）                                                                | ‌Inflection-2.5‌（最新，性能接近 GPT-4 但训练算力仅 40%）、‌Inflection-2‌、‌Inflection-1‌                                             | ‌Pi‌                | 个人陪伴型 AI 聊天机器人                   |


- 按照工具细分

| 工具名称          | 所属公司/组织                    | 工具类型分类                                        | 产品定位                                                          | 自研主要大模型                                                                                                  |
| ------------- | -------------------------- | --------------------------------------------- | ------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------- |
| ‌Claude‌ Code | Anthropic（🇺🇸美国）          | 🧠 大语言模型 + 🔧 ==Coding== 工具 + 🤖 **Agent** 能力 | 大语言模型系列 + 官方编程工具 Claude Code（CLI 终端编程 Agent）                  | Claude Opus 5（最新旗舰）、Claude Fable 5（旗舰通用）、Claude Mythos 5（高阶有限开放）；还有 Claude Sonnet、Claude Haiku 等轻量版      |
| ‌Codex‌       | OpenAI（🇺🇸美国）             | 🔧 ==Coding== 工具 + 🤖 **Agent** 工具            | 通用 AI 智能体 / 编程助手，支持"计算机使用"能力操控桌面软件                            | GPT-5.6 Sol（旗舰）、GPT-5.6 Terra（均衡）、GPT-5.6 Luna（轻量）；前代：GPT-4o、o3/o4-mini、GPT-5.x 系列                       |
| ‌OpenCode‌    | Anomaly（开源社区）              | 🔧 ==Coding==工具                               | 开源 AI 编程代理（Coding Agent），多模型中立的终端编程助手                         | ❌ 不自研模型，支持接入 75+ 种第三方大模型（Claude、GPT、Gemini、DeepSeek、Ollama 等）                                            |
| ‌Cursor‌      | Anysphere → SpaceX（🇺🇸美国） | 🔧 ==Coding==工具                               | AI 原生代码编辑器（基于 VS Code 深度改造，不是插件），主打 Tab 补全、代码生成、重构、Agent 模式   | ❌ 不自研模型，默认接入 Grok 4.6（xAI）、Claude Opus 5、Gemini 等第三方模型                                                   |
| ‌Gemini‌      | Google 谷歌（🇺🇸美国）          | 🧠 大语言模型                                      | 大语言模型系列（其衍生产品 Gemini Code Assist 才是 Coding 工具，但「Gemini」本身指模型） | Gemini 3.8 系列（最新）、Gemini 3.8 Live（实时语音）、Gemini Ultra（旗舰）、Gemini Pro（主力）、Gemini Nano（端侧）、Gemini Flash（轻量） |
| ‌OpenClaw‌    | Amantus Machina（奥地利，开源）    | 🤖 **Agent** 开发工具                             | 开源 AI 智能体框架（俗称"龙虾"），用于构建和运行自主智能体                              | ❌ 不自研模型，框架本身支持接入多种大模型                                                                                    |
| ‌Hermes‌      | Nous Research（🇺🇸美国，开源）   | 🤖 **Agent** 开发工具 + 🧠 开源模型                   | 开源 AI 智能体框架（OpenClaw 进化版），同时也出品开源微调模型                         | Hermes 系列（Hermes 3 等，基于 Llama 微调，侧重工具调用与长程规划）、Nomos 系列；Hermes Agent 是其智能体框架产品                            |
| ‌Pi‌          | Inflection AI（🇺🇸美国）      | 💬 对话聊天机器人（其他）                                | 个人陪伴型 AI 聊天机器人，主打高情商、倾听式对话                                    | Inflection-2.5（最新，性能接近 GPT-4 但训练算力仅 40%）、Inflection-2、Inflection-1                                       |
### 补充说明

- ‌**闭源模型厂商**‌：OpenAI、Anthropic、Google、Inflection AI —— 既有模型也有面向用户的产品
- ‌**开源框架厂商**‌：Anomaly（OpenCode）、Amantus Machina（OpenClaw）、Nous Research（Hermes）—— 主要做智能体/编程框架，模型需用户自行配置接入（可以接国产模型、本地模型等）
- ‌**Nous Research**‌ 比较特殊：既做开源模型（Hermes 系列微调模型），也做智能体框架（Hermes Agent），属于"训模人做 Agent"的路线
- 
## 国内生态

| 厂商/公司                 | 自研主要大模型                                                                                                 | Coding 编程工具                                                                        | Agent / 智能体开发工具                                                                 |
| --------------------- | ------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------- | ------------------------------------------------------------------------------- |
| ‌**字节跳动‌（火山引擎）**      | ==豆包==大模型 2.1 Pro / 2.2；Doubao-Seed-Code（豆包编程模型）；**Seedance 视频模型**                                      | ‌==Trae‌==（AI 原生 IDE，SOLO 模式可自主开发）；‌豆包 MarsCode‌（VS Code/JetBrains 插件 + Cloud IDE） | 火山引擎方舟（MaaS 平台）；==扣子（Coze）==Agent 平台；豆包工作（企业办公 Agent）                           |
| ‌**阿里巴巴‌（千问/Qwen）**   | ‌==Qwen==3.8‌（2.4T 参数 MoE，旗舰）；Qwen3.8-Max / 27B；Qwen-Coder（代码专用）                                        | ‌==Qoder‌==（原通义灵码，独立 IDE + 插件 + CLI）；QoderWork（企业级编程 Agent）                        | ‌千问办公 QwenWork‌（桌面/云端/协同三栖 Agent，整合悟空 + MuleRun + QoderWork）；==百炼平台==（Agent 开发） |
| ‌**DeepSeek‌（深度求索）**  | ‌==DeepSeek-V4‌==（Pro / Flash，Agentic Coding 开源 SOTA）；DeepSeek-V3.2；DeepSeek-R1（推理模型）；DeepSeek-Coder 系列 | ‌==DeepSeek Harness== (dsh)‌（Agent 运行时 / 终端编程 Agent，PTC 模式 + 创造模式）                 | DeepSeek Harness 框架（自进化 Agent 运行时骨架）；开源 Function Calling 能力业界领先                 |
| ‌**MiniMax‌**         | ‌==MiniMax M2.7‌==（Agentic 编程强，性价比高）；MiniMax M1；海螺大模型                                                   | ‌==MiniMax Agent‌==（端到端编程 Agent，桌面端 + 网页端，专家 Agent 功能）                             | MiniMax Agent 平台（专家 Agents、端到端应用构建）；海螺 AI（C 端）                                  |
| ‌**月之暗面‌（Kimi）**      | ‌==Kimi K3‌==（2.8T MoE，100 万 Token 上下文）；Kimi K2.6（长程编码）；Kimi K2                                         | ‌==Kimi Code‌==（CLI 终端 Agent + IDE 插件 + Web 端，支持 Plan / goal / Sub-agents / Swarm） | ‌==Kimi Researcher‌==（深度研究 Agent，模型即 Agent）；Kimi 助手（内置联网、代码等工具调用）               |
| ‌百度‌（文心/ERNIE）        | 文心大模型 4.0 / 5.0 系列；ERNIE Speed / Lite / Tiny；文心代码大模型                                                    | ‌文心快码 Comate‌（AI 编程助手，已并入百度搭子）                                                     | ‌百度搭子 DuMate‌（通用智能体平台）；AgentBuilder（智能体开发工具）；AppBuilder（AI 原生应用开发）              |
| ‌腾讯‌（混元）              | ‌混元 Hy4‌（最新，OpenRouter 全球第二调用量）；混元 Hy3（295B MoE）；混元 Hy2                                                 | ‌CodeBuddy‌（编程助手，IDE + CLI）；QClaw（编程 Agent，曾独立后并入云产品部）                             | ‌WorkBuddy‌（办公智能体）；‌腾讯元宝‌（C 端 AI 助手 + Agent）；腾讯云智能体开发平台（B 端）；元器（C 端 Agent 平台）   |
| ‌智谱 AI‌（GLM）          | ‌GLM-5.3-Flash‌（代号 Ox-Alpha，6 天 62 万亿 Token）；‌GLM-5.2‌（长程任务旗舰）；GLM-5（744B MoE）；GLM-4 系列                 | ‌CodeGeeX‌（开源代码模型 + 编程助手）；Coding Plan 订阅                                           | ‌AutoGLM 沉思‌（深度研究 Agent，边想边干）；智谱清言（内置 Agent 能力）；MCP 工具生态                        |
| ‌商汤科技‌（日日新/SenseNova） | ‌SenseNova U1 Pro‌（交付级原生多模态 Agent 基座）；SenseNova 6.5 / V6；秒画文生图模型                                        | 代码小浣熊（AI 智慧编程）                                                                     | ‌如影营销智能体‌（电商多 Agent 矩阵）；‌Seko 2.0‌（短剧创作智能体）；‌FinAgent‌（金融智脑）；日日新 SenseNova 开放平台 |
| ‌华为‌（盘古/云）            | 盘古大模型 5.0 系列；‌PanGu-Coder‌（盘古研发大模型）；盘古气象/医学等垂直模型                                                        | ‌CodeArts Snap‌（盘古助手，IDE 插件 + 多模型路由）；码道代码智能体                                       | 华为云 AI Agent 开发平台；盘古政务/金融等行业 Agent；昇腾生态适配                                       |
| ‌零一万物‌（Yi）            | ‌Yi-Large‌（千亿参数闭源旗舰）；Yi-Lightning；Yi-34B / 9B / 6B（开源）；Yi-Coder                                         | 万知 AI 工作平台（含编程辅助）                                                                  | ‌万仔‌（企业级 Agent，私有化部署）；‌Open AgentKit (OAK)‌（开源 Agent 一站式开发平台，LangCrew 框架）       |
### 补充说明

- ‌**Coding Agent 第一梯队**‌（按国际评测表现）：智谱 GLM-5.x、月之暗面 Kimi K3、DeepSeek V4 Pro —— 三家国产模型均跻身全球 Coding Agent 前十
- ‌**开源路线代表**‌：DeepSeek（模型+Agent 框架全开源）、智谱 GLM（核心模型开源）、零一万物 Yi（中小尺寸开源）
- ‌**企业级市场侧重**‌：百度（文心 + 百度搭子）、华为（盘古 + 昇腾私有化）、腾讯（混元 + 企业微信/钉钉生态）、商汤（行业垂直解决方案）
- ‌**性价比王者**‌：DeepSeek（API 成本约为 GLM 的 1/6）、腾讯混元 Hy 系列（高性价比定位）


## 国内的Agent平台

以下是国内主流的、与 ‌**Coze（扣子）**‌ 类似的 AI Agent 开发平台/工具完整汇总：

| 平台名称                      | 所属公司/组织    | 产品定位                      | 核心特点                                                                                                   | 适用人群/场景                            |
| ------------------------- | ---------- | ------------------------- | ------------------------------------------------------------------------------------------------------ | ---------------------------------- |
| ‌==扣子 Coze‌==             | 字节跳动       | 零代码/低代码智能体开发平台（标杆）        | 可视化节点编排、丰富插件市场、多渠道发布（抖音/飞书/微信/网页）；3.0 升级为「AI团队协作平台」，支持多 Agent 组队干活                                     | 运营、自媒体、开发者、中小企业，快速搭建 Bot 与对外服务型智能体 |
| ‌==阿里云百炼‌==               | 阿里巴巴       | MaaS+Agent 一体化企业级平台       | 深度集成通义千问全系列，兼容第三方开源模型；可视化多角色编排、并行子任务、流程回流；原生打通钉钉、ECS、OSS、电商系统；支持混合云私有化部署                               | 中大型企业、电商商家、需要内网离线运行的团队，阿里云存量客户     |
| ‌腾讯元器‌                    | 腾讯         | C 端 Bot 创作 + 微信生态分发平台     | 轻量化工作流，支持 @多智能体协同、共享素材库；天然对接微信/QQ/企业微信，智能体可直接发群、做公众号问答；配套 QClaw 本地客户端执行                                | 私域运营、微信生态从业者、轻量化自动化需求              |
| ‌腾讯云 ADP‌                 | 腾讯         | B 端企业级智能体开发平台             | 覆盖构建、评测、分发、管控、观测全环节；支持 LLM+RAG、工作流、Multi-Agent 多种范式；原生打通企微、腾讯会议，权限/组织架构自然衔接                            | 传媒、医疗、金融等知识协作密集的中大型企业              |
| ‌百度文心智能体平台‌（原灵境矩阵）        | 百度         | 零代码公开 Bot 发布平台            | 文心大模型底座，知识增强路线；一键分发百度全域流量（搜索、小度、百家号）；行业知识库成熟（法律、教育、职场）                                                 | 自媒体、知识博主、政务/教育企业，内容创作类智能体          |
| ‌百度千帆 AppBuilder‌         | 百度         | 企业内部 Agent 编排平台           | 以 Agent 为核心的一站式大模型服务平台；支持自主规划 Agent、工作流 Agent、多智能体协作；「文件→知识→可审服务」管线成熟                                  | 政务、国企、大型集团的知识密集型升级场景               |
| ‌==Dify‌==                | 语灵人工智能（开源） | 开源低代码 LLMOps + Agent 开发平台 | Apache 2.0 开源，可完全私有化部署；可视化 Workflow/Chatflow 编排；支持 100+ 大模型接入；原生兼容 MCP 协议；LLMOps 完善（版本管理、A/B 测试、全链路审计） | 技术团队、重视数据主权的企业、需要深度自定义流水线的场景       |
| ‌火山引擎 HiAgent‌            | 字节跳动       | 企业级一站式智能体工作站              | 覆盖开发、评测、运行、观测、优化、治理全生命周期；支持低/高/无代码混合开发；强调企业级扩展与私有化安全集成；字节系工程化底座                                        | 要把 Agent 纳入「数字员工」治理框架的大型企业         |
| ‌飞书 Aily‌（飞书智能体）          | 字节跳动（飞书）   | 飞书生态内的企业级 Agent 平台        | 原生集成飞书消息、文档、审批流；支持多 Agent 组队分工；能主动工作、持续关注项目变化；员工零安装                                                    | 飞书重度使用的企业和团队，内部协作场景                |
| ‌实在 Agent‌                | 实在智能       | 执行型数字员工（RPA+大模型融合）        | 自研 ISSUT 屏幕语义理解技术，‌不依赖 API‌ 直接操作软件界面；「TARS大模型(大脑)+ISSUT(眼睛)+RPA(双手)」三位一体；全栈信创适配                        | 制造、能源、金融等有大量老旧系统/无API软件的企业，央国企信创场景 |
| ‌蚂蚁 Agentar‌              | 蚂蚁数科       | 金融级全栈智能体开发平台              | 「可信智能体」底座，内置金融 MCP 服务广场（投研、合规、风控）；推理可解释、知识可追溯、交互可监控、评测可归因；信通院可信AI最高评级5级                                | 银行、证券、保险等强监管金融行业                   |
| ‌讯飞星辰 Agent 平台‌           | 科大讯飞       | 多模态智能体开发平台                | 基于星火大模型；支持多模态交互、智能编程助手 iFlyCode；智能体广场覆盖生活、工作、学习、创作等场景；支持 MCP 全链路                                       | 教育、医疗、政企等多模态交互需求场景                 |
| ‌360 智语‌                  | 360        | 大型政企复杂业务智能体平台             | 全球首创 L4 企业级智能体蜂群工厂（SEAF）；囊括「L2工作流+L3推理智能体+L4多智能体蜂群」三代编辑器；三态分离（开发/管理运营/用户）；信创全栈适配                       | 党政军、央国企、关基行业等安全与合规前置要求的单位          |
| ‌华为盘古智能体平台‌               | 华为         | 国产化信创工业级智能体平台             | 盘古大模型底座，国产信创全栈适配；工业、政务专用多智能体编排；边缘端+云端协同；数据本地不出域                                                        | 制造工厂、政务单位、信创项目、工业数字孪生              |
| ‌零一万物 Open AgentKit（OAK）‌ | 零一万物       | 开源 Agent 一站式开发平台          | 基于 LangCrew 框架；支持多模型接入；企业级「万仔」Agent 可私有化部署                                                             | 需要开源框架深度定制的技术团队，企业私有化场景            |


### 选型速查

| 你的需求                   | 首选平台                   |
| ---------------------- | ---------------------- |
| ‌零代码快速搭 Bot，要发布到抖音/微信‌ | ==扣子 Coze==                |
| ‌企业私有化部署，数据绝对不出域‌      | ==Dify（开源自托管）、阿里云百炼（混合云）== |
| ‌深度用飞书/钉钉，要内部协作 Agent‌ | 飞书 Aily（飞书）/ 阿里云百炼（钉钉） |
| ‌打通老旧系统、没有 API 的软件自动化‌ | 实在 Agent               |
| ‌金融/政务强合规、强审计场景‌       | 蚂蚁 Agentar、360 智语、华为盘古 |
| ‌技术团队要深度自定义、多模型接入‌     | Dify、LangFlow          |
| ‌微信/企微生态私域运营‌          | 腾讯元器、腾讯云 ADP           |
