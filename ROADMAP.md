# ROADMAP — dsh-openimage

> 2026-10-31 立项。来源：三竞品对照深挖（dsh-agnes-media / dsh-image-gen / dsh-ark9canvas）+ 历轮 deferred 清账 + v18 真机复核发现。
> **2026-11-05 全场竞品扫描修订**：11 目标工作流调研（basketikun/tigerowo/hero8152/dsh-image-gen/agnes/ark9/toonflow/sd-infinity/opencove/jsoncanvas/商业三家+linux.do），总矩阵见 `design/canvas-competitor-matrix.md`，16 条抄清单已获批准并入本图。
> **定位不变式**：SenseNova 生图的「管理与编辑」工作台。接入层不做（Token Plan 登录/额度可视化/多 key 池交给 dsh-connect-sensenova-token-plan 与 dsh-sensenova-pool，兼容其注册的 provider）；非 SenseNova 模型不绑（无多 provider/订阅免 Key/本地 ComfyUI）。
> **宣传主轴（措辞收窄版）**：「三合一治理的生图画布——审批门+队列治理+画布生成入口，活跃竞品中独此一家」；获客话术用 dsh-image-gen #99 痛点（切页即丢、已计费无结果）。**时间窗**：image-gen 画布内 AI 指令条方案就绪未上线（issue #39），随时可放——我方画布入口体验优先级因此前置。**代码红线**：tigerowo 为 AGPL-3.0，一行不搬，只借交互与数据模型思想。

## P0 — 已完成（2026-10-31 工作流实施 + 2026-11-01 真机验证）

> 全部三项落地并真机验证：①孤儿清扫——根因证伪（AST+动态复现台证明原实现本可达，真凶是 SSE 回放窗口冲回），改为共用 sweepOrphanRunning+队列视图触发点，活体验证服务端真翻转；②RPS 节流——snxGenPost 唯一收口处发起间隔 2000ms（可配），真机实测相邻 POST 间隔 2003ms（tiled 放大改走同收口，评审发现的绕过口已堵）；③审批门——server 审批文件 CAS 状态机+双路由+三工具挂起（fail-closed），客户端队列页审批区（倒计时/批准/拒绝/全部拒绝），插件级 harness 49/49（拒绝与超时上游 0 次调用），真机拒绝路径文件真翻转。独立评审 8 条（1 medium 已修=放大绕过节流；7 low 记录在案）。

| # | 项 | 内容 | 来源 |
|---|---|---|---|
| 1 | **孤儿清扫运行时修复** | v18.3 的清扫源码正确但活体未触发（疑似 applyDoc 所在闭包无 autosave 引用，ReferenceError 被加载链 `.catch(()=>{})` 静默吞掉）。先定位作用域，改用可靠持久化路径，再做活体验证（开画布→服务端状态翻转）。 | v18.3 真机复核遗留 |
| 2 | **RPS 预限流节流** | 相邻两次生成 POST 之间强制最小间隔（可配 `snxGenMinIntervalMs`，默认 2000ms，队列调度层执行而非失败后重试），从源头避开上游 RPS 429（实测免费档并发 2 即 `rps exhausted`）。 | dsh-sensenova-pool 预限流思路 + 真机 429 实证 |
| 3 | **agent 生成审批门（human-in-loop）** | 对话轴三工具（generate/edit/enhance）执行前挂起：请求进工作台「审批」区，人工批准/拒绝/超时（默认 120s）自动拒绝；拒绝不产生任何上游调用与计费；总开关默认关，面板可开。 | dsh-ark9canvas 杀手锏；防 agent 循环烧免费额度 |

## P1 — 后续轮次

| # | 项 | 内容 | 来源 |
|---|---|---|---|
| 4 | 任务宿主进程化 | 生成提交后由 server 侧任务表驱动，关页签/关浏览器不中断（客户端只读展示进度）——架构级根治孤儿任务；改动大，需先出设计稿。**验收标准=后台任务五条底线**（提交即返回/关页不断/宿主重启自动恢复/每个界面都有取消入口/重试次数可见，抄 agnes-media）。 | dsh-agnes-media 架构 |
| 5 | 参考图库 | 角色/场景/风格本地库：灯箱「存入图库」+ 表单快捷引用（替代每次重新上传）。 | dsh-image-studio 模式 |
| 6 | 灵感库外部源 | 自定义 JSON prompt 源（host 代理拉取防 CORS/CSP），**失败保留上次成功缓存（刷新失败不清空）**，手动+可选定期刷新；标准格式 `[{title?,prompt,negative?,tags?[]}]`，读端容错跳过坏条目。 | dsh-ark9canvas 源机制 + dsh-image-gen 快照/缓存细节 |
| 7 | 极小缩放端口热区合拢吞边 | SPEC-17 起 deferred 的空间分配张力（热区半径已按相机反比，再压伤正常缩放可点性）；需新的交互方案（如热区分级）。 | 历轮 deferred |
| 8 | 透明背景 | SenseNova images/generations 是否支持 background 参数需查官方文档，支持才做。 | dsh-ark9canvas |

## P2 — 抄清单落地（2026-11-05 批准，来源=对照矩阵②）

| # | 项 | 内容 | 来源 |
|---|---|---|---|
| 9 | 画布导入交互 | ~~基本已存在~~（核对 2026-11-05：drop→ingestDroppedImageSrc→素材节点+PNG tEXt 参数读回+多文件 idx 偏移落点，v11.2 起在位）；剩余增量=视频文件（不做，agnes 领地）。**关闭**。 | dsh-agnes-media |
| 10 | 节点出生证 meta | 核对：prompt/params{size,model,negative,seed,reference,refs[role,strength],upscale}+上游 seed 回填已在位（v18.5 起）；**本批补 batch{index,count}**；剩余增量=引用丢失前置警告（现由服务端落盘失败 errMsg 兜底，暂不加）。**基本关闭**。 | dsh-image-gen 出生证 + basketikun 重试警告 |
| 11 | agent×画布工具 | canvas_state 摘要 token 预算（节点清单 24/选区细列 12/+N 折叠+selectionSignature 防陈旧）+落地总线三原则（one-shot 防删除复活/仅画布打开实时落地/历史显式导入去重）+失败单张自动重跑+「成功 X/共 Y」汇报+读画布现状提议下一步生成。 | dsh-image-gen + toonflow ACT |
| 12 | 画布组织 | 打组/解组+组内连线语义（连线到组=引组内全部媒体）+语义命名+tidy 一键整理+标签颜色+节点左上角状态徽标（生成中/完成/失败）+节点内一行提示词摘要。 | toonflow + opencove + basketikun Group |
| 13 | ⌘K 全局搜索 | 搜索范围纳入提示词文本与图片节点（不只文件名），画廊+画布通吃。 | opencove |
| 14 | .canvas 双向互通 | jsoncanvas 导入导出：导出降级 4 标准类型（图→file 相对路径/说明→text Markdown），私有生成参数嵌 text 节点 Markdown；导入只消费标准字段、未知 type 跳过不报错。 | jsoncanvas spec |
| 15 | 类型化连线校验 | 拖线即校验「参考图→生成节点」兼容性、拒绝自连。 | toonflow |
| 16 | 尺寸 snap 公式核对 | 选区→生成尺寸取整策略对照 quality-budget+16px 对齐公式核对一次，不一致则改齐。 | sd-infinity + ark9canvas |
| 17 | 回收站三件套 | 软删/恢复/彻底删，画廊+画布共用。 | hero8152 |
| 18 | 自托管兼容 | 非 HTTPS/内网 IP 无安全上下文的降级方案（剪贴板等 API 探测+提示）+镜像绑定 0.0.0.0 文档。 | linux.do 三楼层实证 |
| 19 | llms.txt / sitemap | 仓库与文档补 llms.txt+llms-full.txt+sitemap，让 AI 助手可直接引用我方定位（agentic-SEO）。 | 坤肉AI |
| 20 | 提示词源注册表 | 接多上游源的提示词中心（分类 chips+标签筛选+复制/加入素材），P1 #6 之上的目录层。 | basketikun 提示词中心（761 条形态） |

## 不做（定位外，明示防止范围漂移）

- 文生视频 / 首帧参考体系 / 短剧分镜（agnes-media 领地）
- 多 provider / 订阅免 Key / 本地 ComfyUI（image-gen 领地，与 SenseNova 绑定定位冲突）
- 接入层全部（登录/额度可视化/多 key 轮换池——两个成熟竞品已覆盖，我们兼容其 provider）

## 已完成（不入图）

并发闸+队列重排插队、对话轴等待预估服务端兜底、minimap、tiled 2× 放大、增强、蒙版重绘、批量变体/zip、真实链路验证+真测常规化、孤儿清扫+客户端看门狗（v18/v18.3）。
