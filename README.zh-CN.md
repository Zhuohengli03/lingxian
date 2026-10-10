<div align="center">

<img src="assets/logo.png" width="96" alt="灵弦 logo" />

# 灵弦 · Lingxian

**为古琴而做的打谱编辑器：敲减字，五线谱与简谱同时落笔。**

<a href="README.md"><img src="https://img.shields.io/badge/English-efe7d8?style=for-the-badge&labelColor=efe7d8&color=efe7d8" alt="English" /></a>
<a href="README.zh-CN.md"><img src="https://img.shields.io/badge/%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87-2b2118?style=for-the-badge" alt="简体中文" /></a>

[**www.lingxian.app**](https://www.lingxian.app) &nbsp;·&nbsp; [宣传片](assets/video/lingxian-promo.mp4) &nbsp;·&nbsp; [架构](#架构) &nbsp;·&nbsp; [工程亮点](#工程亮点)

</div>

> [!NOTE]
> 这是灵弦的**公开展示仓库**。产品源码不公开，这里放的是项目介绍、架构和部分视觉材料。

<p align="center">
  <a href="assets/video/lingxian-promo.mp4"><img src="assets/video/promo-teaser.gif" width="720" alt="灵弦宣传片片段，点击观看完整版" /></a>
  <br/><sub>▶ 点击观看完整宣传片（68 秒）</sub>
</p>

## 为什么做

一千多年来，古琴谱都靠**减字谱**记写：一个字里同时写着弦、徽位、左手用指、右手技法和装饰，但不记节奏，琴人至今多半手抄、手批。灵弦想让减字谱也能像普通乐谱一样，敲得进去、听得出来、比得了版本、发得出去、印得出来。

## 能做什么

| | |
|---|---|
| **减字谱编辑** | 敲「散勾三」，减字与五线谱同时落笔；支持 37 种演奏技法、复合技法、装饰音、段落与时值，**11 种定弦 × 5 种调式** |
| **一份谱，四种视图** | 减字 + 五线谱 · 减字 + 简谱 · 纯减字横版 · 纯减字**竖版**（传统排法），屏幕与 PDF 用同一套排版规则 |
| **简谱转减字** | 粘贴简谱或导入 MusicXML，按全谱上下文一次规划弦、徽、指法；每个音都可以从排好序的候选里改选 |
| **协作与教学** | 实时协作、邀请、课堂、逐字批注、版本历史与对比 |
| **导出** | PDF（标题、段落、定弦、减字 + 五线谱、批注序号与文末注释）；MusicXML 可导给 MuseScore / Sibelius |
| **调音器** | 麦克风实时识音，七根弦可点可听 |
| **指法大全** | 149 条指法词条，附古谱原文与各家说法，中英双语 |
| **琴灵** | 编辑器里的助手，从整理好的知识库回答「怎么做」类问题 |
| **琴学札记** | 琴史典故、技法、乐理与中华文化文章，中英双语 |

### 一份谱，四种视图

<table>
  <tr>
    <td width="50%"><img src="assets/screens/view-staff.jpg" alt="减字 + 五线谱" /><br/><sub>减字 + 五线谱</sub></td>
    <td width="50%"><img src="assets/screens/view-jianpu.jpg" alt="减字 + 简谱" /><br/><sub>减字 + 简谱</sub></td>
  </tr>
  <tr>
    <td><img src="assets/screens/view-horizontal.jpg" alt="纯减字横版" /><br/><sub>纯减字 · 横版</sub></td>
    <td><img src="assets/screens/view-vertical.jpg" alt="纯减字竖版" /><br/><sub>纯减字 · 竖版</sub></td>
  </tr>
</table>

### 编辑器

<img src="assets/screens/editor.jpg" alt="灵弦编辑器" />

### 简谱转减字

<img src="assets/screens/transcribe.jpg" alt="简谱转减字编辑器与候选减字" />

## 架构

<img src="assets/architecture.svg" alt="架构：浏览器端、七弦引擎、Next.js 服务端、PostgreSQL" />

项目是 pnpm + Turborepo 的 monorepo。音乐逻辑集中在共享的**七弦引擎**里：不可变的 Token 数据模型、JSON 数据驱动的技法库、音高引擎、时值引擎（每四分音符 16,384 ticks）和版本 diff。Web、API 和后台进程都建立在这个引擎之上。

| 层 | 技术 |
|---|---|
| 前端 | Next.js 14（App Router）、React 18、TypeScript、Tailwind CSS、Zustand |
| 记谱与音频 | VexFlow 5、自研减字字体渲染、jsPDF、Tone.js / Web Audio、AudioWorklet |
| 协作 | Yjs（Y.Doc），独立的 WebSocket 进程 |
| 数据 | PostgreSQL、Prisma 5 |
| 账号与支付 | Auth.js（邮箱 / 短信 / 第三方登录）、Stripe |
| 质量 | Vitest、Playwright（含 WebKit） |
| 运维 | Docker、GHCR、Nginx、PostHog（用户同意后才启用） |

## 工程亮点

### 转译是演奏路径规划，不是查表

<img src="assets/transcription.svg" alt="转译管线：候选图、Beam Search、信达雅三层" />

古琴上同一个音高，常常可以在不同的弦、不同的徽位、用不同的手指弹出来。规划器先枚举每个音的所有合法候选（散 / 按 / 泛 × 弦 × 徽分），再对**整段做 beam search**；带状态的推理器按前文手位加上下文代价，比如弦距、徽距和左手用指连续性。训练好的**排序模型**只在合法候选上重排，不自己生成减字。识别出的旋律型会折叠成装饰（2-3-2 → 进复，3-2-3 → 退复）；源谱没标滑音的地方不挂绰注，不替演奏者加花。

### 音高来自弦长比：十三徽纯律

<img src="assets/hui-just-intonation.svg" alt="十三徽按弦长比定位" />

徽位就是精确的振动弦长比（1/8 … 1/2 … 7/8）。散音、按音、泛音的音高都由比例算出，不套用十二平均律。

### 让一千年前的字形在网页上正确显示

- 技法、装饰、用指全部用 **JSON 数据驱动**：新增一个技法只需要改数据，编辑器、指法大全和 PDF 导出读的是同一份配置。
- 减字靠 OpenType 连字拼成。WebKit 不画总进距为 0 的文字段，所以 iPad 上一些减字会整格空白。现在先用 canvas 量出每段文字的进距，只给进距为 0 的那段补一个发丝空格，几何位置不变。
- 竖版逐字计算格高；编辑器和 PDF 导出共用同一套纯函数排版。

### 调音器的实时 DSP

处理流程是：AudioWorklet 预处理（去直流、高通滤波）→ YIN 基频检测 → 置信度融合 → 含泛音模式的八度候选评估 → Viterbi 平滑。

### 对用户的曲谱负责

- 自动保存在同一个事务里写回主记录；重新打开时如果自动保存更新，就静默恢复，并用一句话说明改了什么。
- 曲谱内容进入转译模型的训练池之前，统一经过去标识化处理。
- 产品统计只在用户同意后启动，只记动作和计数，不记曲谱内容。

## 一些数字

- **2026 年 2 月**开工，至今 **1,200+ 次提交**
- 约 **2,100 个 TypeScript 文件**，其中测试文件 **570+**
- 指法词条 **149** 条 · 演奏技法 **37** 种 · 定弦 **11** 种

---

<div align="center">
<sub>© 灵弦 Lingxian。保留所有权利。本仓库中的截图、视频与示意图未经许可不得转载使用。</sub><br/>
<a href="https://www.lingxian.app">www.lingxian.app</a>
</div>
