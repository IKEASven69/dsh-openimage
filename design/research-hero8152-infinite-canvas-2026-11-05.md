# 竞品调研：hero8152/Infinite-Canvas（无限画布）

> 2026-11-05。数据来源：GitHub API 直连（`gh repo view` / `gh api` contents+tree+commits）、raw.githubusercontent.com 源码下载（main.py 19,038 行、canvas.js 786KB、smart-canvas.js 963KB、api-settings.js 205KB）、api.npmjs.org、README 8 张截图逐张解读。无搜索页抓取。
>
> **一句话前提：本项目已于 2026-08-28 宣布停更**（README 及 update-notes.json 原文「此项目已停更，全新版本请前往：www.DX-OS.com 下载」），作者转向商业平台 DX-OS。GitHub 版定格在 VERSION=2026.08.28，之后不再维护——调研价值在于它的**后端编排面遗产**和 **DX-OS 化的演化方向**，而非直接对位竞争。

## 基本盘

| 项 | 值 |
|---|---|
| 星标 / fork | 3,181★ / 628 fork（`gh repo view` 实测） |
| 活跃区间 | 2026-05-11 创建 → 2026-08-28 停更，344 commits（5月9 / 6月44 / 7月37 / 8月10） |
| Release | **无 GitHub Releases**；版本走仓库根 VERSION 文件（2026.08.28），前端 `/api/check-update` 拉 GitHub 比对提示升级 |
| 协议 | **自定义 "Other"**：已申请著作权，**禁止商用**；个人/公司内部可用；二开必须保持开源+署名（README 原文） |
| 定位 | 本地 Python 服务 + 浏览器画布的**多后端生图工作台**：ComfyUI / OpenAI 协议 API / RunningHub / 火山方舟 / ModelScope / 即梦 CLI / Midjourney / Gemini·Codex CLI |
| 变现 | API 中转商 aff 链接（apib.ai / fhl.mom）+ 赞赏码 + 继任商业平台 DX-OS（账号体系/APP 商店/开发者模式）；RunningHub 系统工作流内置带官方缩略图（疑似官方合作） |
| 分发 | B站（space.bilibili.com/78652351）主阵地 + YouTube 教程 + Chrome 应用商店采集插件；仓库直接内置 **Python 3.10 运行时 + 全套 wheels 离线包**（packages/*.whl、python/*.pyd），Windows .bat / mac .command 一键装 |
| npm | 无 npm 包。`api.npmjs.org/downloads/point/last-month/infinite-canvas` 返回 72 次/月，但那是同名无关包（本仓库根无 package.json；`@hero8152/infinite-canvas` 404） |
| 社区 | open issues 141 个（停更后不再处理）；B站评论/私信做支持 |

## 技术架构

1. **本地 FastAPI 单体后端 + 框架化前端，双端都无画布库**。`main.py` 19,038 行（~100 个 REST 端点 + 1 个 `/ws/stats` WebSocket）；前端是 static/ 下原生 HTML+JS（**无 React/Vue/构建链**，仅 tailwind-cdn + lucide + three.js vendor），画布引擎与 basketikun 同款：`world.style.transform = translate(x,y) scale(k)`（canvas.js:1197）+ 小地图（canvas.js:1273）+ undo 栈（14761，nodes+connections 全量 JSON 快照）。**两块画布**：canvas.js 786KB（节点工作流画布）与 smart-canvas.js 963KB（composer 驱动智能画布，见 Agent 面）。
2. **离线优先的平民化安装**：仓库直接提交 Python 3.10 解释器与 17 个 whl（fastapi/pydantic/httpx/uvicorn/pillow），`安装依赖.bat` 不联网。这是它吃下「不会配环境的 C 端用户」的关键一招，也是 3.2k★ 的主要来源（B站教程引流）。
3. **持久化在服务端文件**：画布 JSON（软删除/restore/purge 回收站三件套，main.py:17312-17337）、history.json 全局历史、data/asset_library.json 素材库、`/api/storage-settings` 可改存储目录；浏览器 localStorage 存画布列表 UI 态。无云同步（公网分享走 `/api/temp-sh/upload` 传 temp.sh 图床 + shared-folders 本地共享目录）。

## 后端编排面（本次调研重点：多渠道接入 / 自定义调用）

**渠道模型：`protocol` 字段驱动的统一 Provider 注册表**（main.py:785 `default_api_providers()` + 1263 `normalize_provider()`）：

- 内置强制保留 3 平台：ModelScope（免费 LLM+生图+LoRA 列表，带 `ms_defaults_version` 种子升级机制）、RunningHub、火山方舟；其余全可增删。用户自定义平台 id 形如 `custom-api-N`（api-settings.js:3513 `addProvider()`）。
- 每个 provider 携带：`base_url` + **按 provider 分文件存 .env 的 key**（`provider_key_env()`，UI 回显走 `mask_secret` 打码）、`protocol`、`image_request_mode`（openai 同步/异步协议）、**`image_generation_endpoint` / `image_edit_endpoint` 端点覆盖**、分模态模型表（image/chat/video 三列）。
- 协议族实测有 8 种：`openai` / `runninghub` / `volcengine` / `jimeng`（即梦 CLI，吃会员积分）/ `codex` / `gemini-cli`（CLI 即协议）/ `apimart` / modelscope；另有 Midjourney（中转站代理）与 ComfyUI（实例直连）独立通道。**连 OpenAI Codex CLI 和 Gemini CLI 都被包装成一种「渠道协议」**——CLI 装好即作为生图后端出现（CLI/ 目录带三平台安装脚本，还 vendor 了 gpt-image-2-skill-0.7.3.tgz）。
- **渠道治理 API 面很全**（main.py:13294-14043）：`/api/providers/test-connection`（连通性）、`/api/providers/probe-async`（探测异步协议）、`/api/providers/{id}/fetch-models`（**拉上游模型清单 + 按生图/LLM/视频自动分栏勾选**，截图4实证：67 个模型、搜索框、分模态 tab、未选计数、应用到模型列表）、`验证地址` 按钮。dsh 的渠道设置页应对齐这套「测试/探测/拉模型」三件套。
- **灰色设计**：`is_tudou_base_url()` + `locked_recommended_provider_rule()`（main.py:1198-1228）——对特定中转站域名（tudou，即 aff 推广的那些）**锁定推荐模型规则**，用户不可改。aff 商业化的代码级痕迹。
- **任务模型**：两段式提交/轮询——`/api/canvas-image-tasks`、`/api/canvas-comfy-tasks`、`/api/image-task-query`（submit→task_id→poll，main.py:14332-14571）；即梦/MJ/RunningHub 的排队信息（queue_idx/queue_length）原样透传到前端展示。全局队列只有**排队位置显示**（`/api/queue_status` 返回 total+position，main.py:17744）——**全仓无 Semaphore/并发闸代码**，无审批门、无配额护栏。
- **ComfyUI 多实例负载均衡是它独有的硬货**（main.py:18155 `/api/generate` + 3279 `reserve_best_backend`）：多实例注册表（`/api/comfyui/instances` GET/PUT）→ 按各实例 `/queue` 长度选最闲的 → **若目标实例缺输入图，自动从其他实例拉取并转传补齐**（跨实例输入同步）→ 提交后轮询 history 拉取产物；产物收集时**后端层过滤 PreviewImage/对比图冗余输出**（有主输出就丢弃预览节点图）。
- **自定义调用（映射工作流）是它最完整的自定义面**：上传任意 ComfyUI API 格式工作流 JSON（`/api/workflows` CRUD + config + run）→ 在映射编辑器里**可视化预览全图（68 节点级）→ 点选节点、勾选要暴露的参数、自定义名称与控件类型**（图片上传/文本/滑杆）→ 右侧实时预览画布节点表单 → 「运行测试」就地验证（截图7实证）。RunningHub 同款映射编辑器（api-settings.js 的 rh-editor，按 nodeId::fieldName 分组字段、节点 popover、app/工作流双模式，且**系统工作流与用户工作流双层 merge、隐藏覆盖保留**，main.py:1062-1160）。映射结果直接生成画布上的 COMFYUI 节点表单（截图8：文生图/细节增强/图片编辑/自定义 tab + 参数滑杆 + 上游 Group 自动填「IMAGES·需要2」）。

## Agent 面

- **没有外部 Agent 桥**（无 MCP、无本地 agent 进程、无工具清单）——与 basketikun 的 34 工具 MCP 路线完全相反。它的「Agent」是三件内置物：
  1. **服务端路由 agent**（main.py:17476 `/api/chat/agent`）：LLM 先决策 action（chat / generate_image / edit_image），edit 无参考图时**自动继承上一条消息的图并继承其尺寸**，generate 时按语义**拆分并行 prompt**（`chat_split_parallel_prompts`）批量出图，自动选 provider/model，回复里带 `agent_reply` 决策说明。
  2. **智能画布 smart-canvas**（smart-canvas.js 963KB）：composer 输入框（非节点图优先）+ `@` mention picker（引连线资源+素材库角色/场景）+ 引擎下拉（每引擎动态参数）+ **级联运行**（cascadeRun，loop 尾节点自动串联下游，smart-canvas.js:15108 `cascadeTailForLoop`）+ 自动布局 + 工作流导入导出 + MiniMax 多参视频工作台（9图/3视频/3音频参考上限对齐 ComfyUI schema）。
  3. **画布内 LLM 节点**：每节点独立 system prompt 开关、节点/聊天双模式、视觉模型识别连线图（截图5：ModelScope Qwen3-VL-8B-Instruct「已连接1张图片（视觉模型可识别）」）。
- **周边自动化**：Photoshop 直连插件（tools/photoshop-asset-connector，socket 驱动 PS 调画布全部功能）+ Chrome 采集插件（网页图/视频/文字抓进素材库）。方向是「把画布能力塞进别的工具」，不是「让外部 agent 操控画布」。

## 功能对照（11 维评分卡 vs dsh-openimage）

| 维度 | hero8152/Infinite-Canvas | 对 dsh 的含义 |
|---|---|---|
| 画布引擎 | 手写 DOM transform，无库，双画布共 1.75MB 原生 JS | 同路线再验证；但单文件近 800KB 已现维护代价（它选择停更重写） |
| 节点模型 | 卡片：image/prompt/loop/group/promptGroup/output/llm；**生成器按渠道分节点**：API/MJ/ModelScope/Comfy/RH/MiniMax/LTX/video 九种，端口菜单加输入输出 | 渠道=专属节点的范式 vs dsh 统一生成节点：它换来了每渠道专属 UI（如 RH 工作流选择器），代价是学习成本 |
| 衍生链 | loop 节点 1-100 次、循环/并发双模式、prompt 按编号/换行自动拆分、环检测（loopPromptVisiting）；级联运行；「一键运行 N 节点 × M 轮」 | **loop+级联+全图批跑**三件套比 dsh 现有的批量扇出完整 |
| 编辑链 | 笔刷蒙版（maskBrushSize/undo/redo）、裁切、扩展图片、upscale 工作流、360 全景生成+预览（Qwen-Image-Edit-2511）、视频帧抽取 | 360 全景和视频帧抽取是 dsh 没有的差异化小件 |
| 生成治理 | 仅排队位置显示 + ComfyUI 侧排队透传；**无并发闸/审批门/配额** | dsh 的审批门+队列治理仍是空白地带，无威胁 |
| Agent 面 | 服务端路由 agent（决策/继承参考图/拆分并行）+ 智能画布 composer + LLM 节点；无 MCP | 它的「action 路由 + 参考图继承」交互值得 dsh 对话链路借鉴；外部 agent 桥它没做 |
| 提示词库 | 仓库自带 301 行预设模板（九宫格机位/剧情四格/360全景，带 MJ/即梦/Flux 各平台参数建议）+ 画布可编辑模板分组 | 素材沉淀弱于 basketikun 的源注册表；模板文件直接随仓库分发是个低成本好做法 |
| 持久化 | 服务端 JSON 文件 + 回收站（软删/恢复/彻底删）+ 可改存储目录 + 素材库 AI 分类/自动打标 + temp.sh 公网图床 | 回收站三件套和「拖图进素材夹」值得抄；无云同步 |
| 视觉语言 | 暗色默认+亮色主题切换、点阵网格底、圆角浮卡、贝塞尔连线圆端口、右侧滑入资产面板、composer 底栏、小地图 | 与 dsh 现行方向一致，无新范式 |
| 变现分发 | aff 中转链接 + 赞赏 + B站引流 + Chrome 商店插件 + **停更转商业平台 DX-OS（APP 商店/账号权限/开发者模式）** | 证明「开源画布→商业平台」转化路径存在；它的 3.2k★ 全靠 B站教程+离线安装包降低门槛 |
| 活跃度 | **已死**：2026-08-28 停更，141 open issues 无人处理，无 release | 无持续威胁；DX-OS 才是它的续作 |

## 行动建议

1. **渠道接入三件套直接对标**：`test-connection` / `probe-async`（异步协议探测）/ `fetch-models`（拉模型+分模态勾选导入）是中转站用户的高频痛点三连，dsh 渠道设置页应补齐（尤其 fetch-models 的「全部/生图/LLM/视频」分栏勾选 UI，截图4 已验证体验成立）。
2. **ComfyUI 多实例负载均衡 + 跨实例输入同步**（reserve_best_backend）是本地 ComfyUI 用户的真痛点，dsh 若接 ComfyUI 通道可直接按 main.py:3279 的思路实现；产物侧「过滤预览节点冗余输出」也一并抄。
3. **映射工作流编辑器**（上传 API 格式 JSON → 图上点选暴露参数 → 自定义控件 → 就地试跑 → 生成画布表单节点）是「自定义调用」的完整形态，比暴露 JSON 文本框高一个档次；ComfyUI 与 RunningHub/其他工作流平台可共用同一套映射 schema。
4. **防备点**：它已停更，威胁不在它本身，而在 **DX-OS**——画布+Agent+APP 商店+多人协同+免费图床的平台化续作。建议下一步对 DX-OS 做一次独立调研（需浏览器访问 www.DX-OS.com，超出本次 gh/curl 通道范围，未查）。
5. **规避点**：渠道=专属生成节点的九种节点模型不宜照抄——dsh 统一生成节点+渠道 profile 的模型更省学习成本；它的 aff 锁定推荐规则（locked_recommended_provider_rule）是商业化反噬用户体验的样本，dsh 不应做。
