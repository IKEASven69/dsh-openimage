# 竞品调研：shanliuling/dsh-image-gen（dsh 生态头部生图插件）

> 2026-11-05。数据来源：`gh repo view` / `gh api`（README、git tree 223 文件、package.json、releases×39、issues×~90、src/canvas-tools.ts、src/canvas-state.ts、src/shared.ts、src/index.ts、src/client/tl/tl-canvas-bridge.ts、src/client/tl/studio-tl-canvas.tsx）+ `curl api.npmjs.org` + raw.githubusercontent 下载 README 截图 9 张（存 `D:\coding\.tmp-dsh-image-gen\`，动图 WebP 用 ffmpeg/PIL 抽首帧逐张解读）。无搜索页抓取。

## 基本盘

| 项 | 值 |
|---|---|
| 星标 | **1,121★** / 67 forks（调研当天比委托时的 1097 又涨 24，日增约 10+） |
| 月下载 | **59,326**（npm `dsh-image-gen`，2026-10-06 → 11-04，api.npmjs.org 实测） |
| 节奏 | 2026-08-17 v0.1.0 → 2026-11-05 v0.10.3，**80 天 39 个 release（≈2 天一版）**，最新一版当天发布 |
| 协议 | Apache-2.0（README 营销海报写 MIT，以 LICENSE 为准，属海报笔误） |
| 定位 | DeepSeek Harness 原生图像创作插件：对话批量生图 + **「看得懂草图的 AI 画布」** + Studio + 多模型对比 + 500+ Prompt 灵感库 + 图库 + 本地 ComfyUI |
| 变现 | **零变现**：无赞助位、无商业版、无付费项；「ChatGPT/Grok/Google 订阅免 Key」是获客钩子而非收入 |
| 分发 | npm + `dsh plugin add` 宿主面板 + AtomGit 国内镜像 + linux.do 友链 + dsh-insights.com 健康徽章 + skills/install-dsh-image-gen（让对话 Agent 自己装自己） |

**与我们正面对撞的点**：它的画布（v0.6.0「AI-aware infinite canvas」）随头部插件预装进每个 dsh 用户的侧边导航（对话/AI 创作画布/工作台/多模型对比/灵感库/图库/ComfyUI/设置），且对话生成的新图**实时自动落画布**——画布正在变成宿主的默认输出面。

## 技术架构

1. **画布引擎 = tldraw 5.3.0 库路线**（`package.json` devDependencies `"tldraw": "5.3.0"`，打包进 client bundle、发布时移入 devDeps 减安装体积——某 release 原话）。定制层很薄：`src/client/tl/` 共 11 个文件，`studio-tl-canvas.tsx` 359 行封装落地/选区取消/资产去重（资产 ID `asset:ig-{galleryId}`），其余是品牌主题 token、CSS、选区同步、画廊↔画布拖传。文件头注释（studio-tl-canvas.tsx:25）：**"tldraw owns subsequent edits, deletion, undo and persistence"**——撤销/删除/持久化全用 tldraw 原生。截图证实是原版 tldraw 皮肤：Page 1 页签、左侧竖排工具栏（选择/手/画笔/橡皮/箭头/文字/便签/图片）、右上样式面板（色板/描边粗细/虚实线）、点阵网格，仅套品牌色。**与 basketikun 的手写 DOM-transform 路线相反；两个 9k★/1.1k★ 级样本各验证了一条路线**：手写上限够用（basketikun），tldraw 迭代极快（它 80 天 39 版）。
2. **画布持久化两层分治**：画布文档 = tldraw 自带 IndexedDB 持久化（v0.6.7「无限画布持久化」：手绘草稿/生成图/布局跨浏览器刷新、跨 DSH 重启保留，来源元数据随行）；宿主侧 agent 视图 = 纯内存镜像（canvas-state.ts:9 注释 "Nothing here is persisted: the mirror dies with the host process"），浏览器实例经 canvas-state-route 推紧凑快照，agent 只读镜像。无跨设备同步（无 WebDAV）。
3. **节点模型 = tldraw shape + 出生证 meta**：图/手绘笔迹/文字/便签四类；`CanvasNodeSummary`（shared.ts:117-140）带 `galleryId / attachmentId / 完整宿主引用 / name / width / height / text / prompt / provider / model`——**每张落画布的图都带 prompt/provider/model 出生证**（tl-canvas-bridge.ts:18-24），agent 可以 "regenerate this / vary this" 直接复用原参数而非猜像素。选区摘要带 **selectionSignature**（count+kinds+item 身份的 JSON 签名，canvas-state.ts:53）——两张不同的「1 张选中图」永不共享签名，陈旧截图不会跨选区错配。无 Group 节点语义、无 `@` 引用语法（这两点是 basketikun 有而它没有）。
4. **落地总线三原则**（tl-canvas-bridge.ts，v0.6.7/v0.9.2 演进而来）：①**one-shot 投递**——投递即从队列移除，删除过的图永不在对账时复活；②**实时落地仅画布打开期间**——基于宿主会话事件流订阅，切历史会话/关画布不补发；③**显式历史导入去重**——「发送到画布」按钮 + 拖拽，同记录按 galleryId 去重（#80/#81，两天内闭环）。防重叠布局复用。
5. **生成/编辑链**：对话批量（单次上限 10 张、逐张顺序、单张失败不阻断、失败的**自动单独重跑**、「成功 X/共 Y」汇报）；edit_image 多图参考（最多 5 张垫图）、角色一致性、圈哪改哪（画笔圈选→`edit_image source=canvas_selection`，v0.6.10 起选区含涂鸦时自动携带截图兜底）、同卡片版本切换；Studio 1-4 候选满意才入库、参数面板、一键多模型并发对比（截图：右侧 4 家 Provider 勾选清单各带费用标注，「开始对比(4)」）；**无 upscale、无蒙版 inpainting UI、无 tiled 放大**——营销海报画布面板画的「局部重绘/扩展画面/添加元素」按钮在实际 UI/代码中**均不存在**（超前宣传）。ComfyUI：多命名工作流、预设占位符、Agent 按名称选择，但未接 Studio/对比。
6. **提示词库**：内置三源 JSON（awesome-gpt-image-2 + baoyu-compose + handraw-style 手绘风格手册，带 `version.txt` 快照与 `build-handraw-snapshot.mjs` 构建脚本），500+ 条带分类（截图：角色人物 81/图表信息图 55/文创出版 11…）、风格/排版/主题色可组合；Agent 检索（`find_inspiration`）+ 「用风格 #123」点名引用；灵感页浏览/筛选/收藏/一键带入工作台；案例图片本地缓存，浏览不耗额度。工作台收藏栏（常用 Prompt/参考图收藏复用，v0.10.x）。
7. **Provider 矩阵**：Gemini / OpenAI / OpenAI-Compatible 多命名中转渠道（#83）/ Seedream / DashScope / xAI Grok / 智谱 GLM-Image / 本地 ComfyUI / **三个订阅免 Key 通道（chatgpt-sub/grok-sub/google-sub，OAuth 登录，令牌存宿主 Credentials、浏览器不接触）**。订阅通道覆盖批量/编辑/对比全功能。工程质量面：65 个测试文件、662 项测试、每 release 双语说明 + 验证清单（含 Windows Chrome 真实鼠标拖拽测试）、CI 徽章、SECURITY.md。

## Agent 面（dsh 生态内的形态差）

- **形态**：原生 dsh 插件，工具直接注入宿主 `ctx.tools.register`（index.ts:472/493/566/680），**无本地桥、无 token 配对**——对比 basketikun 需要 `npx` 起 127.0.0.1:17371 本地 agent + Origin 绑定，它的分发摩擦为零。这是「生态原生」的天花板形态。
- **工具仅 6 个**：`generate_image / generate_images / edit_image / find_inspiration / canvas_state / view_canvas`（对比 basketikun 34 个）。粒度粗但描述写得极重：每个工具 description 内嵌使用时机/反例（"Do not call read, glob... to locate or verify the image"；provider 参数 "OMIT unless the user explicitly requests... never guess"）——把 Agent 行为约束放在工具描述而非框架里。
- **画布感知链**：浏览器 tldraw 推快照 → 宿主内存镜像 → `canvas_state` 文本摘要（节点清单截断 DIGEST_NODE_LIMIT=24、选区细列截断 12、"+N more" 折叠、「updated N ago」时间戳）→ `view_canvas` 取选区截图进对话（截图按需持久化，用完即弃，canvas-state.ts:20-26 注释解释了为何不 eager 落盘：内容寻址库无删除，eager 会永久泄漏磁盘）。选区同步状态栏（准备中/就绪/失败可重试）让用户看到 agent 何时能看画布。
- **画布内无 agent 入口**（见下节 #39）：用画布使唤 AI 必须切回对话侧栏。

## 必答①：画布下一步动向

1. **画布内 AI 指令条（最明确、最高优先级的「下一步」信号）**——Issue #39（2026-09-12 建，09-30 关闭，state_reason=completed）方案写得非常透：参考 tldraw AI demo 形态；画布角落唤起式 AI 按钮→输入条（回车发/Esc 收）；**临时会话关画布即焚**；**无文字回复 UI，只显三态（进行中/成功/失败）+脱敏失败原因+重试，生成图直接落画布，「内容即回答」**；自动附选区上下文；复用落地总线防重叠+出生证。作者自评「一到两天工作量，git 历史有现成代码可照抄」。**但截至 v0.10.3 未上线**：223 文件树中无 canvas-chat 路由/组件，65 个测试文件无对应测试，39 个 release 正文 grep「指令条/命令条/command bar/canvas-chat/临时会话」零命中。结论：**方案已定、代码有存货、随时可以两天内放出——这是它画布的最短下一步**。
2. **图表 DSL / 反向画布命令通道（L1 流程图脑图）**——#39 附言原话列为「规划中的后续相关项……与本项目独立，不互相依赖」。即作者已在规划让 agent 反向操作画布（命令通道）与流程图/脑图 DSL。方向 = 从「agent 只读画布（state/view）」走向「agent 写画布」，与我们 agent 直驱画布的路线对撞。无对应 issue，时间表未知。
3. **画布成为默认输出面已完成半程**：对话生成实时落画布（#81 修复后订阅宿主会话事件流）、历史图拖入/发送到画布（#80）、画布主题随宿主明暗切换、落地不回放历史。下一步自然是把交互入口（指令条）搬进画布。
4. **宣传面已占位**：营销海报的画布面板已画出「局部重绘 / 扩展画面 / 添加元素」按钮——实际未实现。说明画布内 inpaint/outpaint 在叙事上已准备，实现随时可跟上（对它的 tldraw 路线而言成本低）。
5. 周边动向（间接）：#99（open:reopened）生成中断+生成历史记录建议——若采纳会先落在画廊而非画布；ComfyUI 上画布/画廊（#97 open、#15 reopened）；Seedream 5.0 图层拆分（#106 open）。

## 必答②：审批门 / 队列治理——**确认没有**（预估验证成立）

- **无审批门**：4 个生成类工具 + 2 个画布工具全部 `ctx.tools.register(defineTool(...))` 直接注册，无 approval/confirm/consent 参数；在 index.ts（838 行）/canvas-tools.ts/canvas-state.ts 三个核心文件 grep `approval|confirm|consent|quota|budget` 唯一命中是 canvas-tools.ts:66 描述里的 "canvas_state confirms a selection exists"（语义无关）。Agent 拿到工具即无门直烧配额。
- **无持久生成队列**：#99（open，用户附根因分析，作者未推翻）实证——生成状态是 `StudioView` 组件内 `useState`，组件卸载即 `requestRef.current?.abort()`；服务端 `serveStudio` 在浏览器连接关闭时 `controller.abort()` 取消上游生成。**生成任务不脱离 UI 生命周期存在**，切个页面就人财两空（上游已计费）。期望中的「已中断记录+一键重试」尚无。
- **无并发闸/插队/等待预估**：批量=逐张顺序，无并发上限概念；无队列深度/等待时间暴露。
- **已有的是稳定护栏而非治理**：150s 可配置超时（#89）+ 慢通道 keep-alive 保活流；`maxImageBytes` 附件上限；ZIP 导出按实际输出字节限流；webAdmission（web 路由准入，安全向）；批量单次 10 张上限（超限提示 Agent 拆多次——**靠提示词约束，非硬闸**）。
- **反例证据**：#44 Google 渠道 429 配额耗尽只能报错；#87 Studio 已计费结果因 tab 切换丢失（已修，但暴露无事务性治理）。**「生成审批门 + 队列治理」在它当前架构里没有存在，且用户侧痛点（#99）正被用「修组件生命周期」而非「建队列」的方式回应**——我们的差异化主张依然成立。

## 功能对照（11 维 vs dsh-openimage 无限画布）

| 维 | 它 | 相对我们 |
|---|---|---|
| 画布引擎 | tldraw 5.3.0 + 359 行薄封装 | 我们手写 DOM-transform（basketikun 验证可行；它验证库路线迭代快） |
| 节点模型 | shape+出生证 meta（prompt/provider/model/galleryId/attachmentId）；四类节点；无 Group/无@引用 | 我们有 @引用/Group 规划则占优；出生证字段设计值得对齐 |
| 衍生链 | 结果全落画布可续迭代；多模型对比=并发扇出；无显式 lineage 树 | 我们 lineage/批量扇出若做显式树则占优 |
| 编辑链 | 圈哪改哪+草图成图+角色一致性+版本切换；**无 upscale/无 inpainting UI/无 tiled** | 我们 tiled 放大/蒙版重绘是实打实的差异点 |
| 生成治理 | **无审批门、无持久队列、无并发闸**（#99 实证） | **我们杀手锏成立，正面撞不输** |
| Agent 面 | 6 工具+超重描述；生态原生零桥接；只读画布 | 我们工具更细+审批门；它桥接体验已是天花板（无桥） |
| 提示词库 | 500+ 三源、版本快照、Agent 检索+#引用、本地缓存 | 同款思路（我们 P2-6 提示词源注册表），它已上线且被 5.9 万月下载验证 |
| 持久化与同步 | 画布 tldraw IndexedDB+宿主镜像内存；图库 IndexedDB+服务端内容寻址；无跨设备同步 | 若我们做跨设备同步（WebDAV 等）则独有 |
| 视觉语言 | 原版 tldraw 皮肤+品牌 token；浮岛工具条（预览/裁切/下载/ALT）；画廊 overlay 四 tab | 我们自定义布局范式差异化空间大；「原生 tldraw 脸」辨识度低但零学习成本 |
| 变现与分发 | 零变现；分发 = npm+宿主面板+AtomGit+订阅免Key钩子+对话自装 skill | 我们可学「订阅免 Key」获客钩子与 dsh-insights 徽章 |
| 活跃度 | 80 天 39 版（2 天/版）、当天仍在发版、issue 多为 24-48h 闭环、662 测试 | 我们需以治理面+画布深度打，不拼 release 频率 |

## 行动建议

1. **把它 #39 的指令条方案当成预警表**：若它两周内上画布内 AI 指令条，dsh 用户「画布即入口」心智将被它锁死。我们的画布内生成入口（审批门形态）应尽快可见地落地，宣传口径继续打「审批门+队列治理」——它 #99 的痛点（切页即丢、已计费无结果）是我们最好的获客话术素材，且其维护者正在用小修而非队列的方式回应。
2. **值得抄的四件小件**：①出生证 meta 字段集（prompt/provider/model/galleryId/attachmentId 随 shape 走，regenerate/vary 复用原参数）；②落地总线三原则（one-shot 防复活、仅画布打开实时落地、显式导入去重）+防重叠布局；③canvas_state 摘要的 token 预算设计（节点/选区双截断+selectionSignature 防陈旧截图错配）；④失败单张自动重跑+「成功 X/共 Y」汇报格式。
3. **画布引擎结论修正为「双路线均验证」**：basketikun 证明手写上限够，dsh-image-gen 证明 tldraw 路线 2 天一版的速度。我们维持手写，但需把省下的引擎时间花在它结构上做不了的事：审批门、持久生成队列、lineage 树、tiled 放大——这四样它在「薄封装 tldraw+组件内生成状态」的架构上补课成本高。
4. **订阅免 Key 是它最锋利的获客钩子**（有 ChatGPT/Grok/Google 订阅即零门槛），我们的 SenseNova Token Plan 免费额度直通应对标其「订阅行 OAuth 登录→输入框旁随手切换渠道」的体验，并考虑把「渠道可用性告知模型」（它 #98 还 open）做成我们有而它没有。
5. 监控点：v0.11+ 若出现 canvas-chat 路由/指令条 release，或出现任何 approval/queue 字样，立即复调研。
