# 竞品调研：basketikun/infinite-canvas（无限画布）

> 2026-11-04。数据来源：GitHub API 直连（repo 元数据/README/docs/源码 tree/CHANGELOG），无搜索页抓取。

## 基本盘

| 项 | 值 |
|---|---|
| 星标 | 9,664★（Trendshift 热榜徽章） |
| 首发节奏 | 2026-05-19 v0.0.1 → 2026-10-10 v0.20.0，**5 个月 40 个 release** |
| 协议 | MIT（允许闭源商用） |
| 定位 | 浏览器本地优先的 AI 图片/视频创作无限画布工作台 |
| 变现 | README 8 个赞助位（全是 API 中转商 aff 链接）+ 作者商业版坤肉AI（企业源码出售/私有化部署） |
| 社区 | linux.do（L 站）主阵地 + QQ 群；README 自带「开发期不保证历史数据兼容」CAUTION |

定位与 dsh-openimage **几乎完全重合**（生图+参考图编辑+对话+提示词库+素材+画布编排），是最直接的竞品。

## 技术架构

1. **画布引擎纯手写，零画布库**。web/package.json 里没有 react-flow/konva/fabric/tldraw/pixi 任何一个。`infinite-canvas.tsx` 仅 10.8KB：容器 div + CSS `transform: translate(x,y) scale(k)` + pointer capture + rAF 节流；节点是 DOM 元素，连线独立 SVG（canvas-connections.tsx），小地图、撤销栈、几何工具各自独立模块（lib/canvas/*）。**结论：AI 生图工作台这个量级，手写 DOM-transform 完全够，不需要引画布库**——dsh-openimage 现有手写路线被验证。
2. **零后端**：web 是纯静态站（Vite 7 + React Router 7 + React 19 + antd 6 + zustand + tailwind 4 + localforage/IndexedDB），Docker/Render 只托管静态文件。API Key 存浏览器、前端直连 OpenAI 兼容端点，调用方式可自定义脚本适配中转站。跨设备同步走可选 WebDAV。
3. **节点模型**：image/text/video/audio/config 五种 + Group 节点（框选 Ctrl/Cmd+G 成组，连线到组=引组内全部媒体）。生成节点 metadata 记 prompt/model/size/quality/count/references，**重试时引用丢失会明确警告中止**。textarea 支持 `@` 引用连线资源（图自动缩略图+编号）。
4. **图像编辑链**：蒙版重绘 dialog、upscale dialog、裁切、分割、角度 dialog——与 dsh-openimage 的蒙版重绘/tiled 放大正面对撞。

## Agent 链路（dsh 生态最值得关注）

- 形态：本地 npm 包 `@basketikun/canvas-agent`，`npx -y @basketikun/canvas-agent@latest` 一行启动，监听 127.0.0.1:17371，网页首次带 token 连接后**绑定 Origin**（防其他站点复用本地 agent）。网页↔agent 走 HTTP/SSE，agent↔Codex/Claude Code 走 MCP；另有 Codex App 插件自动注册 MCP。
- **34 个工具**：`canvas_list_projects / get_state / get_selection / export_snapshot / apply_ops / create_node / create_attachment_nodes / create_text_node(s) / create_config_node / create_image_prompt_flow / create_generation_flow / generate_text|image|video|audio / update_node / update_node_text / move_nodes / resize_node / delete_nodes / connect_nodes / select_nodes / set_viewport / run_generation / generation_get_status / workbench_image_get_config|generate / workbench_video_get_config|generate / prompts_search / assets_list|add` + `site_navigate`。
- `canvas_apply_ops` 批量原语（zod discriminatedUnion 全量校验）：add_node/update_node/delete_node/delete_connections/connect_nodes/set_viewport/select_nodes/run_generation。
- 关键安全设计（README 原话）：**"without giving a web server direct access to browser state"**——agent 读状态走本地桥，浏览器是唯一状态权威；多标签页按 clientId 隔离活动画布/回合；debug 日志自动脱敏 token 与图片 Data URL。
- TODO 方向：Claude Code CLI 适配器 → Claude Agent SDK；Skill 网页安装+本地记忆受控持久化。

## 功能对照（vs dsh-openimage ROADMAP）

**它有、我们 ROADMAP 没有/较弱**：远程 URL 节点插件系统+TS SDK；Group 节点；`@` 资源引用；小地图；WebDAV 同步；提示词源注册表（7 内置源+自定义 JSON，刷新失败保留上次缓存，= 我们 ROADMAP P2-6 同款）；视频/音频节点；undo/redo 覆盖 viewport/背景/助手会话。

**我们有、它没有（差异化卖点）**：
- **dsh 生态原生**：agent 工具直接是 dsh 三轴工具，无需 npx 起本地桥+手填 token；
- **生成审批门（human-in-loop）**——它纯前端无服务端治理，agent 循环烧配额只能靠用户自觉；审批门是我们 ROADMAP 认定的杀手锏，也是它架构上做不了的；
- 队列治理（并发闸/插队/等待预估）；tiled 2× 放大（它只有普通 upscale）；画板画廊（批量变体/zip/PNG 元数据）；SenseNova U1.5 Token Plan 免费额度直通。

## 行动建议

1. 画布引擎维持手写，不引库（它 9.6k★ 验证了该路线的上限足够）。
2. 值得抄的四个小件：Group 节点、`@` 引用语法、小地图、生成节点 metadata 重试+引用丢失警告。
3. agent 工具粒度对标它的 34 工具清单：`select_nodes`/`set_viewport`/`apply_ops` 批量原语是它比我们（三工具）细的地方；但我们有审批门这张它没有的牌。
4. 它 5 个月 9.6k★ 证明品类需求真实且获客路径=L 站+GH trending；dsh-openimage 宣传口径继续打「审批门+队列治理」的治理面，避开与它拼功能清单。
