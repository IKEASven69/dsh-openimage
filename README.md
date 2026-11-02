<div align="center">

# dsh-openimage

**DeepSeek Harness 的图像生成工作台 —— 内置商汤 SenseNova U1.5 深度支持，兼容任意 OpenAI 兼容生图端点**

对话轴直出生图 · 无限画布 · 画板画廊 · 队列治理 · 蒙版重绘 · tiled 2× 放大

</div>

---

`dsh-openimage` 不只是"给 agent 一个生图工具"，而是补齐生成之后的**全部管理与编辑**：无限画布（节点衍生链/蒙版重绘/对齐吸附）、画板画廊（批量变体/zip 导出/PNG 元数据）、队列治理（并发闸/插队/审批门/等待预估）、tiled 2× 分块放大与增强。内置 SenseNova U1.5 深度支持（Token Plan 免费额度直通），也可指向任意 OpenAI 兼容生图端点。

## 安装

```bash
# npm
dsh plugin --profile web add dsh-openimage

# 或从 GitHub 跟进最新提交
dsh plugin add github:IKEASven69/dsh-openimage
```

安装后在「设置 → 插件」确认已启用，左侧栏出现「生图」入口。

## 配置

「设置 → 插件」页的 SenseNova 设置卡填入 API Key（或设 `SENSENOVA_API_KEY` 环境变量）。**Token Plan 免费额度**用户在 [console.sensenova.cn](https://console.sensenova.cn) 创建 Key 后直接可用，无需订阅、无需信用卡。

也可把生图端点指向任意 OpenAI 兼容服务（设置卡修改 Base URL）。

## 功能

### 对话轴
- 会话里发消息即出图，工具卡内直接显示结果
- 参数回填 / 设为参考图 / 同参数重抽 / 同种子重生成
- 等待预估（历史耗时均值）、生成审批门（可选，人工把关防 agent 循环烧额度）

### 工作台 · 生图
- 提示词 + 比例芯片 + 风格 + 负面词 + 种子 + 张数 + 画质（2K/4K）
- 多参考图（风格/角色/构图角色 + 强度滑杆）
- 生成前决策面板：按下之前就知道将生成几张、什么模型、落到哪个画板
- 命名预设 + 提示词库（收藏 + 最近记录）

### 工作台 · 画布
- 无限画布：多选拖拽、对齐参考线吸附、节点剪贴板（Ctrl+C/X/V）
- 节点衍生链：父子连线、改接、断开、版本树
- 蒙版重绘（画笔大小/不透明/硬度/羽化）、消除、抠图、扩图
- minimap 小地图、全屏模式、自动保存 + 并发写保护（rev）+ 全量撤销

### 工作台 · 画廊
- 画板体系：新建/重命名/最近使用排序/生成自动归板（跟随画廊）
- 筛选/搜索/收藏/标签
- 批量条：批量变体、批量导出（带 PNG 元数据）、批量导出 zip、批量删除

### 工作台 · 队列
- 全局并发闸（1–4 可调）+ 等待表重排/插队
- 暂停/恢复、全历史翻阅、行级详情（排队+运行两段耗时）
- 取消/取消其他/重跑失败/清除已失败
- **生成审批门**（可选）：agent 调用生图/编辑/增强先停进审批区，人工批准才执行——拒绝与超时零上游调用、零计费

### 工作台 · 编辑与对比
- 灯箱：滤镜调整（亮度/对比/饱和/色温/晕影+胶片预设）、参数清单行级回填、标签、衍生链翻页
- tiled 2× 分块放大（重叠羽化缝合）、增强（2K/4K、去水印）
- 变体（幅度滑杆 + 种子锁）、批量变体
- 对比页：滑块/并排/叠化三模式 + 3–4 图网格

## 生态定位

接入层已有成熟插件，本插件专注生成之后的一切：

- [dsh-connect-sensenova-token-plan](https://github.com/eghrhegpe/dsh-connect-sensenova-token-plan) —— Token Plan 登录 / 积分可视化 / provider 注册
- [dsh-sensenova-pool](https://github.com/Phoeky/dsh-sensenova-pool) —— 多 Key 轮换池 + RPM 预限流

本插件兼容上述插件注册的 provider 与端点配置。

## 测试

- **53 项端到端验收套件**（`scripts/e2e.spec.js`，真实浏览器执行，含审批门全路径）
- 审批门插件级运行时 harness（49 项断言：拒绝/超时/中止上游 0 次调用）
- 静态门禁：未定义样式变量 / UI emoji / 类名双向审计 / 裸 confirm 四类 0 违规

## License

MIT
