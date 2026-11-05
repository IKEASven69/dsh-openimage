# 无限画布竞品调研计划（wave 2）

> 2026-11-05 立项。前置：basketikun/infinite-canvas 已完成深度调研（design/research-basketikun-infinite-canvas-2026-11-04.md），其流水线与报告结构作为本计划模板。范围仅限 dsh-openimage 无限画布线，不混其他仓库事项。

## 1. 目标与产出物

- **单品报告**：每家 A 级竞品一份 `design/research-<name>-<date>.md`（basketikun 报告即模板：基本盘/架构/Agent 面/功能对照/行动建议）
- **总对照矩阵**：`design/canvas-competitor-matrix.md`，全部竞品 × 统一评分卡一页可读
- **UI 图库**：每家一个 `.tmp-<name>-ui/` 目录（README 图床直下 + 必要时实拍），供界面对照
- **ROADMAP 增补建议**：矩阵定稿后提炼「值得抄/必须防/差异卖点」三条清单送用户审，**不直接改 ROADMAP**

## 2. 竞品分级清单（仓名/数据 2026-11-05 已验证）

### A 级——深度对标（全模板流水线 + 界面细读）

| 目标 | 身份 | 体量 | 为什么 |
|---|---|---|---|
| ~~basketikun/infinite-canvas~~ | ✅ 已完成 | 9.6k★ | 定位最重合 |
| dsh-image-gen | shanliuling/dsh-image-gen | 1097★ / npm 月下载 **57k** / 11-04 仍日更 | dsh 生态头部，同插件位直接竞争；已有画布草稿，正扩画布=正面撞车 |
| tigerowo/infinite-canvas | tigerowo/infinite-canvas | 1.5k★ | 同名同定位第二家（导演台/全景图），差异点在哪要弄清 |
| hero8152/Infinite-Canvas | hero8152/Infinite-Canvas | 3.2k★ | ComfyUI/API/ModelScope 多后端画布，后端编排面值得对标 |

### B 级——专项快扫（只取画布相关，每家 5 个调用内）

| 目标 | 身份 | 取什么 |
|---|---|---|
| dsh-agnes-media | zzsszz1122/dsh-agnes-media（3★/月下载 2.4k/11-03 活跃） | dsh 生态内媒体工作流的画布形态；旧 ROADMAP 已对照过，只增量扫 changelog |
| dsh-ark9canvas | Aik358/dsh-ark9canvas（1★/09-02 后停滞） | 已停滞，确认无新威胁即可；审批门来源已完成历史使命 |
| Toonflow-app | HBAI-Ltd/Toonflow-app（20.4k★） | 体量最大 but 全能型（短剧/漫剧）；只取它的无限画布交互章 |
| stablediffusion-infinity | lkwq007/stablediffusion-infinity（3.9k★） | outpainting 无限画布鼻祖，只取扩图画布的坐标/分块思路 |
| opencove | DeadWaveWave/opencove（1.6k★） | agent-workspace 画布：agent 任务/知识节点化交互参考 |

### C 级——引擎/信号通道（非竞品，贯穿全程按需查）

- **引擎参考**（只取技术决策，不做全扫）：tldraw（相机/空间索引性能文）、jsoncanvas（开放画布文件格式——导入导出格式对标它的 spec）、xiaoiver/infinite-canvas-tutorial（实现教程=查漏清单）、leaferjs/leafer-ui（国产引擎，事件模型）
- **商业品快照**（官网只看定位与定价页）：TapNow、LibTV、坤肉AI（=basketikun 商业版，看开源版给商业版导流什么）
- **需求信号**：linux.do 搜 basketikun/dsh-image-gen 发布帖**评论区用户原话**（免费用户调研）；npm 下载量 `api.npmjs.org/downloads/point/last-month/<pkg>`；GH trending 周榜

## 3. 统一评分卡（矩阵的列，每家都过一遍）

1. **画布引擎**：手写/库、渲染层（DOM/SVG/Canvas/WebGL）、性能上限证据
2. **节点模型**：类型集、Group、@引用、metadata 重试、插件扩展
3. **衍生链**：批量扇出视觉、lineage 可视化、参考图流向
4. **编辑链**：蒙版重绘/放大/裁切/分割/角度/透明背景
5. **生成治理**：队列/并发闸/插队/等待预估/**审批门**/配额护栏
6. **Agent 面**：工具数与粒度、审批机制、桥形态（本地/服务端）、多标签隔离
7. **提示词库+素材沉淀**：源管理/失败保缓存/搜索筛选/收藏
8. **持久化与同步**：IndexedDB/服务端/WebDAV/导入导出格式（jsoncanvas?）
9. **视觉语言**：浮岛工具栏/徽章/端口样式/主题/布局范式（画布即全部 vs 对话轴+画布）
10. **变现与分发**：赞助位/商业版/L站/QQ 群/trending 路径
11. **活跃度**：release 节奏、最近 commit、issue 响应

## 4. 执行模板（已验证流水线，每家 A 级照此走）

1. `gh repo view --json` 元数据（stars/license/release/pushed）+ npm 下载量
2. README 全文（contents API base64）→ 赞助位/功能清单/社区阵地
3. `git/trees?recursive=1` → 目录结构 + package.json deps → 引擎判断（有无画布库）
4. docs 目录：features/节点手册/快捷键/todo 四件套
5. CHANGELOG 版本节奏 + 最近 release 正文
6. UI 图库：README 图床图直下 + Read 过目解读；有在线 demo 则 opencli 后台实拍（`--window background --site-session ephemeral`）
7. 按评分卡写单品报告 → 3 条行动建议
- **dsh-image-gen 特殊项**：可选真机安装截图——专用 dsh 实例+备用端口，**绝不碰用户 profile**（按既有测试纪律：测毕 taskkill+进程体检+端口释放）

## 5. 批次安排

- **Wave 1（dsh 生态内，先定 positioning）**：dsh-image-gen → agnes-media 增量 → ark9canvas 确认停滞。1 轮会话。
- **Wave 2（独立开源工作台，定 feature bar）**：tigerowo → hero8152 → Toonflow 画布章。1 轮会话。
- **Wave 3（引擎/商业品/信号，查漏）**：jsoncanvas 格式对照 + L 站评论区原话采集 + 商业品定价页。1 轮会话，可与 Wave 2 并行。
- **收口**：汇总矩阵 → 「值得抄/必须防/差异卖点」三清单送审 → ROADMAP 增补（批准后）→ 记忆更新。

工作量：A 级每家约 10-15 个 API 调用 + 读图；全程 3 轮会话内收完。

## 6. 通道纪律（本次已踩坑固化）

- **GH 搜索只用英文关键词**；中文概念词（如"无限画布"）会捞进 1MB 乱码书库（Dujltqzv/Some-Many-Books 类）触发安全审核——已实证
- `gh search` 一律 `--jq` 截断 description；README/文档走 contents API 或 raw 直链，不走搜索结果页
- npm 下载量走 api.npmjs.org；WebSearch 只留给 GH/API 到不了的渠道（配额整月易耗尽）
- 下载的图片/存档放独立 .tmp 目录，不进仓
- linux.do 匿名走 t/<id>.json；search.json 403，退路 DDG site:+jina

## 7. 已知预设（待矩阵验证/推翻）

- dsh-image-gen 是真威胁（57k 月下载 vs 我们 0.1.0 刚上线），它的画布下一步动向是 Wave 1 必答项
- 审批门+队列治理在纯前端竞品（basketikun/tigerowo/hero8152）处架构性缺失，在 dsh 插件竞品（image-gen/agnes）处待验证——若 image-gen 也没有，就是我们宣传主轴
- 界面借鉴清单（待补全）：批量徽章+扇形衍生链、Group、@引用、小地图、提示词源失败保缓存
