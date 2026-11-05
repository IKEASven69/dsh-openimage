# 竞品调研：tigerowo/infinite-canvas（无限画布·后端增强版）

> 2026-11-05。数据来源：GitHub API 直连（repo 元数据/README/docs/features.md/canvas types.ts/agent-tools.ts/CHANGELOG/release 资产）、npm registry、raw.githubusercontent 源码与 README 截图（15 张已逐张解读，存 `D:\coding\.tmp-tigerowo\shots\`）。骨架沿用 `research-basketikun-infinite-canvas-2026-11-04.md`。

## 基本盘

| 项 | 值 |
|---|---|
| 星标 | 1,537★（fork 326），创建于 2026-06-30，最近推送 2026-11-04 |
| 首发节奏 | CHANGELOG 继承至 v0.0.3（2026-05-19，早于建仓日，即版本史从底座带过来）；本仓库 GitHub release 21 个（2026-06-30 → 2026-11-03 v0.8.4），**约每 1-2 周一个** |
| 协议 | **AGPL-3.0**（basketikun 是 MIT）——代码不可抄，只可借鉴思想 |
| 定位 | 同名开源 AI 创作工作台：**在 basketikun 纯前端底座上加 Go 后端的多用户/云同步/视频增强版**。README 原话（L78）：「本项目基于 basketikun(纯前端) 为底，合并 HuFakai 生图增强版基础上，针对视频和视频生成逻辑配置更加完善，完善后端云同步机制，不再依赖纯前端」 |
| 变现 | 3 个赞助位（TokenDance/88API/秘塔 MiniMax H3，含 aff 链接）+「项目定制二次开发」邮箱 + 广告位招商；无商业版话术 |
| 分发 | Docker compose / Render demo / **Windows EXE+MSI 安装包**（release 资产 `Infinite.Canvas_0.8.4_x64-setup.exe` + 双语 MSI，Tauri 风格命名）/ New API 一键配置跳转 / 微信群 + L 站 |
| npm | `@tigerowo/canvas-agent@0.1.3` 在架，**月下载 314**（api.npmjs.org 2026-10-06~11-04） |

**与 basketikun 的关系是本次调研最重要的结论：不是巧合同名，是公开承认的衍生分叉**（前端的组件名/目录结构/infinite-canvas.tsx/canvas-mini-map 等可直接对上；CHANGELOG 早期版本号也延续自底座）。它继承手写画布引擎，真正的增量全部在「后端化 + 视频/导演台 + 多用户治理」三个方向。

## 技术架构

1. **画布引擎：继承 basketikun 的手写 DOM-transform，依旧零画布库**。web/package.json 无 react-flow/konva/fabric/tldraw/pixi（已核对全文依赖）。组件清单与 basketikun 同构（infinite-canvas.tsx / canvas-connections.tsx / canvas-mini-map.tsx / 蒙版·放大·裁切·分割·角度五 dialog 全套保留）。新增性能项：v0.8.4「画布连线按需渲染」、WebP 缩略图缓存（后台并发 2 张）。
2. **前端换壳 Next.js 16.2 + React 19 + antd 6 + pro-components + TanStack Query + zustand**（basketikun 是 Vite 7 + Router 7，无 antd pro）。新增依赖泄露功能方向：`@photo-sphere-viewer`（全景球查看器）、`mediabunny`（视频处理）、`@uiw/react-codemirror`（渠道传参转译/工作流 JSON 编辑器）。
3. **后端是全新 Go 单体**（Gin+GORM，SQLite/MySQL/PG 三选）：JWT 登录、admin 控制台（用户/渠道/AI 日志/提示词/素材库/系统 Skill 管理）、`handler/` 按 provider 分文件（apimart/ark_seedance/autodl/kie/mimo/minimax/runninghub/tokendance/comfy_bridge）、任务表（canvas_image/video/audio_task）+ 后端全局轮询、`{code,data,msg}` 统一响应。存储：S3/R2/腾讯 COS/阿里 OSS/WebDAV + 匿名公网临时上传（7 天有效，v0.8.4）。
4. **节点模型：8 类节点**（canvas/types.ts:12-21：Image/**Panorama**/Text/Config/Video/Audio/**Director**/Group；basketikun 5 类+Group）。metadata 关键增量（types.ts:35-116）：`cameraControl`（相机/镜头/焦距/光圈，写入提示词）；批量扇出 lineage（isBatchRoot/batchRootId/batchChildIds/primaryImageId/imageBatchExpanded，>1 张时主图节点+子图节点叠卡）；`firstFrameNodeId/lastFrameNodeId`（视频首尾帧绑图片节点）；`multiShot/shotType/klingMultiPrompt[]/klingElementList[]`（Kling 分镜时间线与元素库）；5 家 TTS 参数（grok/glm/mimo/gemini/miniMax）+ 声音克隆（mimoVoiceCloneAudioNodeId）；panorama 三字段（2048×1024 固定输出 equirectangular）。**生成节点 metadata 重试+引用丢失警告（继承自底座）仍在**（docs/canvas/canvas-node-manual.md:44-45）。
5. **生成治理：后端化**——图片/视频/音频任务持久化（刷新/换机/关页恢复，后端轮询），单条停止+停止失败重试（v0.8.1），终态记录按保留期后台清理；AI 调用日志（请求参数/响应/耗时/模型/渠道）；渠道「自定传参转译」（JSON/multipart/URL 表单 + 异步任务 ID/状态/结果提取表达式，浏览器 Worker 直连模式与后端模式双轨）；管理员可关注册/关自定义渠道/关云端渠道/全局开关视频音频生成。**积分体系露出**：admin credit-logs 页 + 前台 ⚡ 显示。
6. **视觉语言**：与 basketikun 同范式（底部居中浮岛工具栏约 16 图标、DOM 节点、SVG 连线、小地图、点阵/网格/纯色三背景、浅深主题），外加：节点拖拽**对齐吸附+辅助线**（v0.8.3）、生成配置节点四模式 tab（生图/文本/视频/音频）+参考图计数芯片+组装提示词 @引用弹层、摄像机弹窗（RED V-Raptor 8K / Zeiss Supreme Prime / 35mm 小广角 / f/1.2 大光圈档位卡片）。顶栏五入口：我的画布/生图工作台/视频创作台/提示词库/我的素材。

## 导演台与全景图（对 basketikun 的最大差异，独立 3D 产品线）

- **导演台（Director 节点）**：每个节点内嵌独立 3D 场景（关闭可恢复、随项目云同步）。导演/机位双视角；8 素体人物 × 20 姿势、群众阵列（行×列+间距）、基础几何体、本地模型导入（缺失标记+重导恢复）；场景树（可见性/锁定/位旋缩属性）；**机位关键帧时间轴**（位置/旋转/FOV 关键帧、30FPS 吸附、60FPS 预览、速度曲线控制柄、群众组轨道、空格播放、位置轨迹显示）；机位截图 → 自动建图片节点并连线；活动机位导出 **1280×720/30FPS H.264 MP4 → 画布视频节点**。宣传语「3D导演台&不再抽卡：角色站位，镜头角度，角色姿势」（截图解读）。
- **全景图（Panorama 节点）**：文生/图生全景（固定 2048×1024）+ 本地严格 2:1 导入时二选一弹窗（「作为全景图导入/作为普通图片导入」，截图实证）；photo-sphere 球形查看；直连导演台作等距柱状环境背景。
- **画布级摄像机控制**：图片/视频/配置节点可挂相机+镜头+焦距+光圈，自动拼进提示词，失败重试重组装一次，随节点复制与刷新恢复（features.md「摄像机」节）。

## Agent 链路（对比 basketikun 34 工具）

- **双运行时**：①「创作 Agent」**纯浏览器内跑 API**（Chat/Responses 可切、流式、jsonrepair 修复、**单次 JSON >12 个画布操作整批拒绝**、非法类型不静默转换）——这是它比 basketikun 多的形态，basketikun 只有本地桥；②Codex 模式：本地桥 `npx -y @tigerowo/canvas-agent@latest`（127.0.0.1:3210，Token 自动生成+Origin 绑定，架构同 basketikun），**外加 Codex 插件市场**（`codex plugin marketplace add` 本仓库，.agents/plugins/marketplace.json + 自动打开画布 Skill + chrome/edge/firefox/brave 复用标签）。
- **工具清单 25 个**（web/src/.../agent-tools.ts 全数）：arrange_nodes / create_connection / create_group / **create_primary_script_node** / create_text_node / delete_connection / delete_node / **edit_image** / generate_audio / generate_image / generate_video / **get_canvas_summary** / get_connected_nodes / **get_downstream_nodes** / get_generation_config / get_generation_task / **get_media_task_status** / get_node / get_selected_nodes / **get_upstream_nodes** / query_canvas_nodes / read_skill_file / set_agent_state / update_node / update_text_node。外部 MCP 剔除 set_agent_state/read_skill_file = **23 个**。粒度哲学与 basketikun 相反：粗粒度 + 图遍历三件套（上游/下游/连通）+ 任务状态查询，少了 workbench_*/prompts_search/assets_* 等外围工具。
- **10 阶段创作流水线状态机**（types.ts:201-211）：intake → concept → **script → breakdown → references → storyboard** → video → audio → review → complete——面向「一句话成片」的导演流程内置协议。
- **Skill 系统**（basketikun 还是 TODO，它已上线）：输入框最多选 5 个 Skill；系统 Skill 管理员维护（目录+附属文件，`read_skill_file` 按相对路径读，20000 字符上限），用户 Skill 登录入库/未登录 localforage；内置 11 个创作 Skill 文件（image-character-sheet/image-storyboard/script/organize/video-editing/video-extension/video-multi-shot/video-single-shot/workflow/audio/core）。
- **类审批门设计**：Agent 设置有「自动提交媒体」开关——**默认关闭时只创建并配置节点与连线、不提交生成任务**（features.md「Agent」节）；画布内 Codex 默认放行画布 MCP 工具但节点删除保留确认卡。这是三个「无限画布」里最接近 human-in-loop 的现成实现，但仍无 dsh ROADMAP 那种逐任务审批队列。

## 功能对照

**它有、basketikun 没有**：Go 后端+多用户+admin+JWT+积分/日志；账号云同步（画布/素材/生成记录/渠道配置，按时间戳合并）；S3 系/对象存储+匿名临时上传；Seedance 火山方舟双协议（标准+Agent Plan）+首尾帧+Kling 元素/多镜头时间线+视频创作台元素列表与负面分镜；导演台 3D+关键帧运镜+MP4 出片；全景图节点+球形查看+导入导演台；画布摄像机控制；创作工作流模板市场（公开/个人模板、{{变量}}表单、AI 创建工作流、单图/多图系列）；系统/用户双轨 Agent Skill；创作 Agent 浏览器 API 形态；渠道自定传参转译；RunningHub/ComfyUI 工作流渠道 + Go 版 comfy-bridge；Windows 安装包；对齐吸附辅助线；任务停止/重试持久化。

**它有、dsh-openimage ROADMAP 没有/较弱**（更新 basketikun 报告的对照结论）：导演台+全景图整条 3D 线；摄像机参数→提示词；首尾帧连线生成视频；积分/多用户/admin 治理面（自托管团队版定位）；创作工作流模板市场；系统 Skill 后台管理。

**dsh 有、它没有（差异化卖点维持）**：dsh 生态原生 agent（无需 npx 起桥）；**逐任务生成审批门**（它的「自动提交」开关是全局一刀切，没有按任务审批队列）；队列治理（并发闸/插队/等待预估）；tiled 2× 放大（它只有普通 upscale dialog，继承底座）；SenseNova 免费额度直通。注意：它的 admin+积分+AI 日志是「服务端治理」，与 dsh 的「生成审批门」不完全同层——自托管团队场景它是更完整的答案。

## 行动建议

1. **法务红线先说：AGPL-3.0，一行代码都不能搬**。可借鉴的只有产品思想；实现时以 basketikun（MIT）为唯一代码参考源，tigerowo 只看交互与数据模型设计。
2. **它验证了 dsh 的两个既定路线**：手写画布引擎天花板足够（两条 1.5k-9.6k★ 产品同引擎）；审批门方向正确——它的「自动提交默认关」证明纯自动 agent 烧配额是真实痛点，dsh 的逐任务审批门仍是三者中最强的治理答案，宣传口径继续打这点。
3. **值得抄的四个小件（均按思想重实现）**：①首尾帧节点连线面板（firstFrame/lastFrame 绑图片节点，视频生成的自然延伸，成本低感知强）；②摄像机参数档位→提示词（相机/镜头/焦距/光圈四卡片+自动拼接，生图质感控制的便宜大招）；③Agent 自动提交开关（默认只建节点不提交——审批门的降级形态，可先上这个再演进队列）；④批量扇出主图/子图叠卡（batchRoot+primaryImageId+展开设置主图，比 basketikun 的批量更完整）。
4. **战略预警：它抢的是「自托管团队/工作室」生态位**——多用户+admin+积分+云同步+Windows 安装包，是三个同名产品里唯一能卖给小团队的形态。dsh 若只做个人浏览器插件，可与之错位共存；若未来做团队版，它已是先行者且迭代速度（4 个月 21 个 release）不慢。导演台/全景图 3D 线投入重（three.js 级工程），dsh 短期不建议跟进，但「不再抽卡」这个叙事值得记录——当用户开始用 3D 摆位替代抽卡时，生图工具的差异化会往「可控性」迁移，与 dsh 的审批门/队列治理同属「可控」叙事，可合并成一个卖点。
5. **观察项**：它的创作 Agent（浏览器 API 形态+10 阶段流水线+Skill 系统）迭代最猛，若 dsh agent 后续做分镜/成片流程，其 intake→script→storyboard→review 状态机是最贴近生图场景的公开参考实现。

## 查证边界（如实声明）

- npm 主包名不存在（`infinite-canvas` 的 72 次月下载是同名老包，非该项目）；其 agent 包 `@tigerowo/canvas-agent` 月下载 314。
- HuFakai/infinite-canvas（被合并的「生图增强版」）本次未单独调研，其许可证不明——AGPL 仅为 tigerowo 仓库现状。
- Windows 安装包「Tauri 风格」仅为资产命名推断，仓库内未见 src-tauri 目录，实际打包框架未证实。
- 导演台 3D 渲染库未定位（tree 中无 three.js 显式依赖，可能在 app 内联或依赖树深处），不妄下结论。
- docs/progress/todo.md 为空壳（仅标题），无路线图可引。
