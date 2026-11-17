# 技术报告审计 · 别人交来的报告怎么核

> 云厂商 AI 助手、外包、同事交来一份安全事件处置报告／运维故障复盘／升级记录，你要判断它可信不可信。
> 核心判断只有一条：**报告通常「事实可信、结论放大」**——操作日志是真的，但每个动作都被升格成结论。

## 一、先切两层，别整体信也别整体扔

| 层 | 内容 | 怎么核 |
|:--|:--|:--|
| **事实项** | 版本号、CVE 编号、端口、进程、文件字节、日志条数、时间戳 | 打官方源 / 亲自实测 |
| **判断项** | 「最新」「已消除」「攻击未成功」「高频扫描」「全球危机」「已加固」 | 只能内部对账 |

典型画像：确实升级了、确实封了 IP（事实可信），但「升到某个版本」被写成「升到最新、已消除」（判断放大）。前半段的危机叙事（某国首相呼吁、某国关闭政府网站、某厂商公布的攻击次数）通常零来源，整段当叙事看，当不了决策依据。

## 二、六步顺序

1. **事实项逐个打官方源**：CVE 编号 → CVE 记录页＋厂商聚合公告；「最新版本」→ 官方 news/releases 看发布日；版本还中不中招 → 官方 advisories 的 `Vulnerable:` 区间逐条比。
2. **判断项做算术对账**：明细累加 vs 汇总、计数器 vs 物理上限、频率换算、自称大小 vs 实际字节。这一步最省力也最致命，报告自己给的数字常常互不兼容，不用外部证据就能掀翻。
3. **扫内部矛盾**：同一动作两个时间点、同一软件两个起始版本、警告项措辞反了（把「安全组未放通 SSH 外网」列成警告，不放通才是对的）、自检表写「CPU 正常」而正文写「CPU 被大量消耗」、明细时间乱序像手工拼的。
4. **能实测就实测，先建基线**：外侧能查的（端口通不通、`Server:` 头、TLS 版本、证书 SAN）直接查。铁律是**任何「路径返回 200 ＝ 文件泄露」的判断，必须先请求一个必然不存在的路径做基线**——SPA 会把未知路径兜底成 index.html，200 和固定长度都是假的成功。
5. **缺口分「流程缺口」和「结果缺口」**：没评估端口暴露面是流程缺口，实测外网确实不通是结果没问题。混着写，读者会以为天塌了。
6. **按固定骨架交付**：会误导决策的硬错误 → 内部矛盾（附原文两处对照）→ 没覆盖的缺口 → 小问题一次列完 → **站得住的部分（必须写）** → 只读核查清单。

## 三、几条硬判据

- **不按「听起来像 AI 写的」判假，也不按「官方口吻」判真**：只按能查到的源。真实 CVE 编号可能完全属实，按语感判假会翻车。
- **「审核状态：已完成」＝自审自签**，不作可信依据。
- **每条结论落到一条命令或一个 URL**；落不到就写「需人工确认」，不写「应该是」。
- **交出去的核查脚本只读**：不改配置、不动 iptables、不重启服务；`bash -n` 过一遍，关键统计段先拿合成样本验口径再交付。没验过的脚本等于又一份不可信报告。
- **实测推翻了前一轮说法，就放在最前面认账并撤回**，别藏在后面。
- 涉及改边（升级、封禁、重启）属高风险动作，报告里的改动只审计不代做。

## 怎么用

把本仓放进你的 skills 目录（目录名用 `technical-report-audit`）：

```bash
git clone https://github.com/feverZHONG/liya-technical-report-audit.git <你的数据根>/skills/technical-report-audit
```

纯文档型，无脚本依赖。命令与 URL 按自己环境替换。

## 姊妹仓库

- [liya-subtraction-skill](https://github.com/feverZHONG/liya-subtraction-skill) —— 技能库做减法：冗余检测 / 拆薄 / 合并 / 归档判断
- [liya-persona-authoring](https://github.com/feverZHONG/liya-persona-authoring) —— 给 AI agent 写身份文件（SOUL.md 类）：创作流程 / 砍装饰留行为 / 减法与漂移对照
- [liya-sillytavern-cards](https://github.com/feverZHONG/liya-sillytavern-cards) —— 酒馆角色卡写法：V2 格式 / PList+Ali:Chat / 三个 Python 工具
- [liya-sillytavern-worldbook](https://github.com/feverZHONG/liya-sillytavern-worldbook) —— 酒馆世界书（Lorebook）：触发链源码实证 + 体检 / 模拟 / 生成工具
- [liya-vision-recognition-traps](https://github.com/feverZHONG/liya-vision-recognition-traps) —— 视觉模型识图陷阱：22 条实测与对策（附真 OCR 通道、生图物理体检、两图差分）
- [liya-chat-game-referee](https://github.com/feverZHONG/liya-chat-game-referee) —— 群聊小游戏裁判：扫雷 / 五子棋 / 大话骰 / 骗子牌 / 掷骰决斗，一位裁判带六个引擎
- [liya-spy-game](https://github.com/feverZHONG/liya-spy-game) —— 谁是卧底：黑板规则 / 出题方法论 / 词库验证 / 身份分配器
- [liya-sea-turtle-soup](https://github.com/feverZHONG/liya-sea-turtle-soup) —— 海龟汤：推理方法论 + 档案流水线（turtle CLI）
- [liya-delegation-and-verification](https://github.com/feverZHONG/liya-delegation-and-verification) —— 委派与验收：任务书写法 / 并行隔离 / 把「自报」验成事实
- [liya-tavern-card-refinement](https://github.com/feverZHONG/liya-tavern-card-refinement) —— 酒馆角色卡精修：7 字段清单 / 槽位归位 / 6 类断言 / 可用性验收
- [liya-prose-quality-metrics](https://github.com/feverZHONG/liya-prose-quality-metrics) —— 稿子读起来「平」怎么办：先量再改（对话占比·句长σ·标点谱）＋ 7 个工具
- [liya-ruozhiba-wordbank](https://github.com/feverZHONG/liya-ruozhiba-wordbank) —— 弱智吧题防御手册：160 道逐题拆解 + 三连防御法（拆前提→指谬误→反杀）
- [liya-subtitle-proofreading](https://github.com/feverZHONG/liya-subtitle-proofreading) —— 字幕校对 / 重建 / 外挂 SRT（5 个纯标准库工具）
- [liya-corpus-line-mining](https://github.com/feverZHONG/liya-corpus-line-mining) —— 从语料 / 会话库挖可复用原句：候选池筛选 + 人审落库（零依赖）
- [liya-story-revision-plan](https://github.com/feverZHONG/liya-story-revision-plan) —— 小说全稿修订方案：评估 / 缺口清单 / 逐章大纲 / 信息融合 / 优先级
- [liya-dev-workflow](https://github.com/feverZHONG/liya-dev-workflow) —— 开发全流程方法论：环境侦查 / 计划 / spike / TDD / 调试 / 推送排障 / 同步验收
- [liya-news-verification](https://github.com/feverZHONG/liya-news-verification) —— 验证伞：轻量核查 / 交付前多源验证 / 链接危险识别 / 厂商官宣核实 / 链接考古
- [liya-knowledge-persistence](https://github.com/feverZHONG/liya-knowledge-persistence) —— 知识持久化：信息该放记忆层 / 文件 / 技能库的分层规范
- [liya-incident-review](https://github.com/feverZHONG/liya-incident-review) —— 社群事件复盘：素材收集 → 时间线重构 → 交叉验证 → 矛盾管理（输出理解不输出建议）
- [liya-source-code-investigation](https://github.com/feverZHONG/liya-source-code-investigation) —— 外部项目调查：源码审计 / 拆包分层 / 数据实测 / 身份链（结论导向，非取用）
- [liya-character-voice-simulation](https://github.com/feverZHONG/liya-character-voice-simulation) —— 角色声线推演：锚点表双向用——分队推演（隔离上下文）＋ 反查认说话人
- [liya-dialogue-system-builder](https://github.com/feverZHONG/liya-dialogue-system-builder) —— 台词系统脚手架：触发维度画格子 / 模板+变量兜底 / 覆盖率验证（含 17 子命令工作台与示例角色）
- [liya-md-link-maintenance](https://github.com/feverZHONG/liya-md-link-maintenance) —— Markdown 文档库搬家与断链修复：git mv 保历史 / 断链自动修 / 收尾全量校验
## 提思路 / 提修正

- 你踩过的「报告看着通顺、结论其实是编的」案例 → 开 [Issue](https://github.com/feverZHONG/liya-technical-report-audit/issues)
- 想直接改 → Fork + PR

## 许可

**双许可**——文档与代码分开：

- **代码**（`scripts/` 下的文件）：**MIT**（全文见 `LICENSE`）
- **文档**（`SKILL.md`、`references/`、本 README 正文）：**[CC BY 4.0](https://creativecommons.org/licenses/by/4.0/)**，可自由使用、改编、商用，**但要署名**（莉娅 / [@feverZHONG](https://github.com/feverZHONG)）并注明来源

本仓当前是纯文档仓，`LICENSE` 留作后续脚本的默认许可。

---

*莉娅（[@feverZHONG](https://github.com/feverZHONG)）· 宇宙美好记录官*
