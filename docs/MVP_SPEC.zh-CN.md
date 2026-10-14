# CopyLoop MVP Spec

版本：0.1-draft · 日期：2026-10-14 · 状态：规格已定义，运行时尚未实现。

## 1. 产品定义

用户提供一个可访问的网站、需要复现的操作流程和输出约束。CopyLoop 观察参考网站，生成一个可编辑、可独立运行的本地版本，并给出功能、视觉和动效的逐项对比结果。

第一版面向能运行本地开发环境的开发者和设计工程师。首要价值是缩短“观察参考 → 实现 → 找差异 → 修正”的循环，同时让交付边界可以核验。

本次仓库交付包含规格、案例和验收契约。CLI、skill、参考环境、生成器及自动验收器属于后续实现，不在 README 中提前宣称可用。

## 2. MVP 范围

| 维度 | 第一版承诺 |
| --- | --- |
| 输入 | HTTP(S) URL、有限操作流程、视口、参考环境配置及输出约束 |
| 平台 | 本机开发；Chromium 桌面视口与移动尺寸视口，不等于真实手机浏览器认证 |
| 观察 | DOM、可访问性树、可见文本、布局、截图、有限操作轨迹、支持的 CSS/Web Animations 动效 |
| 生成 | 独立 React/Vite 项目；单用户数据存储使用 IndexedDB |
| 交付 | 源码、锁定依赖、启动说明、构建产物、静态对比报告、结构化结果 |
| 模型 | 使用用户已有 coding agent；第一条支持路径为宿主执行 skill，模型不硬编码 |
| 修改 | 按差异报告进行有限轮修正，记录人工介入与全部失败尝试 |
| 数据 | 合成测试内容、独立浏览器 profile、独立参考与候选存储 |

本版不包含：原生 App、移动原生端、任意后端还原、支付、团队权限、多端同步、Canvas/WebGL 复现、任意 JS 动画恢复、网站全量爬取和自动上线。外部服务可以明确排除或替换为自有接口，不能以隐藏 mock 冒充已复现。

对于任意新站点，先产生支持范围与未知项；不能因 URL 能打开就承诺可完成。第一案例通过只代表该案例的 alpha 能力。

## 3. 用户动线

1. **定义任务**：填写 reference URL、三个至十个目标流程、输出目录及预算。界面默认展示本次会访问的页面和会执行的写操作。
2. **观察与冻结范围**：运行参考环境，采集状态和动作；形成 observation manifest、流程图、布局基线及未知项。用户已有明确范围时直接执行，遇到真正影响范围的缺口才请求补充。
3. **生成候选**：coding agent 从证据包实现候选项目。先连通一条可保存数据的完整流程，再扩大范围。
4. **自动对比**：相同输入、环境和初始状态分别作用于参考和候选；生成带定位信息的功能、截图和时序差异。
5. **修正**：agent 根据失败证据修改候选。每次修改产生新 candidate revision，旧结果不可搬到新版本。
6. **交付**：报告列出通过、失败、未知、未运行和范围外项；给出启动方式与完整源码。失败时仍能交付检查点，但不能显示“复现完成”。

以下仅为拟定 CLI 契约，当前不可执行：

```text
copyloop init <case-file> --out <workspace>
copyloop observe <workspace>
copyloop brief <workspace>
# 在现有 coding agent 中运行随项目发布的 skill，写入 candidate/
copyloop verify <workspace>
copyloop report <workspace>
```

`verify` 的进程退出码计划为：0=全部必需门槛通过，1=候选不通过，2=环境或证据不足。退出 2 不能转换为通过。`report` 只是渲染已有结果，不能改变结论。

## 4. 技术结构与职责

技术基线：TypeScript、Node.js 24 LTS、Playwright + Chromium。依赖精确版本、浏览器 revision 和容器 digest 在 M1 实际运行时锁定；此处不伪造尚未测量的版本和镜像 hash。

| 模块 | 输入 → 输出 | 责任 |
| --- | --- | --- |
| Case loader | case.json → validated case | 校验范围、流程 ID、视口、重复次数和预算 |
| Reference runner | pinned reference + seed → isolated URL | 启动、重置、校验参考环境；不向生成器暴露参考服务器源码 |
| Observer | URL + actions → evidence bundle | 保存每步截图、DOM、AX、布局和时序证据；失败不能丢弃 |
| Brief builder | evidence → implementation brief | 提取组件、状态、存储语义和未知项；标明证据来源 |
| Agent skill | brief + workspace → candidate revision | 使用宿主 coding agent 编写与修正候选；不负责自己判定通过 |
| Verifier | candidate + frozen contract → results.json | 独立检查接收结果、持久化、视觉、动效和参考依赖 |
| Reporter | results + artifacts → report.html | 分项展示差异与证据，保留所有尝试和版本关系 |

REA 作为未来可选分析适配器。MVP 不依赖其安装或 native binary 工具；源码辅助模式与浏览器观察模式单独计分。

## 5. 运行与证据契约

每次 run 至少记录：

- case/spec/seed 的 hash，参考 commit、运行镜像 digest、候选 commit 或完整文件 manifest。
- OS、Node、Playwright、Chromium 精确版本，字体清单、视口、DPR、语言、时区、主题、reduced-motion。
- 每个 scenario 的初始状态、动作、接收端断言、实际结果、截图/轨迹 artifact hash。
- `pass | fail | unknown | not_run | out_of_scope` 状态；所有必需项必须为 pass。
- 生成与修正次数、工具调用、人工修改、总耗时；模型实际返回的 input/output/cache token 及计价来源。没有计数时写 unknown，不能写 0。

规划输出目录：

```text
workspace/
  task.json
  observations/       # 原站证据，不直接作为候选运行资源
  brief/
  candidate/          # 可单独安装、构建、运行的源码
  runs/<run-id>/
    manifest.json
    results.json
    reference/
    candidate/
    diffs/
    report.html
```

参考与候选使用不同 origin、profile 和数据库。允许复用经明确记录的字体及测试资源；不得嵌入原站 iframe、转发原站 API 或把原站源码换壳交付。关闭参考服务后，候选仍须通过核心流程。

## 6. 观察规则

MVP 支持有界页面和显式操作，不使用“遍历整个网站”作为默认行为。原站登录若有需要，使用专用本地测试账号；账号创建属于参考搭建，不属于首案例的复现范围。

生成器可读取浏览器渲染结果、DOM/AX、computed style、公开行为和批准资源。首案例的评测禁止读取上游仓库、服务端源码、JS bundle/source map 或参考数据库。参考搭建者和评测器可使用上游源码与数据接口，但相关目录和凭据不挂载给生成器。若宿主 agent 无法隔离这些文件，必须标记 source-assisted，不能宣传为黑盒复现。

参考观察冻结前，逐条走通合同流程。规格里的假设与实际行为冲突时，记录差异、修订并重新冻结；不在候选失败后追溯放宽标准。

## 7. 动效范围

分别记录稳定画面与运动过程。静态视觉比较允许等待动画完成；运动采集保持动画启用，保存触发时刻、终止条件、关键属性和帧序列。

只对能可靠观察的 CSS transitions/animations 与 Web Animations 属性承诺度量。`transform`、`opacity`、元素位置/尺寸是第一批支持属性。其他动效输出 unknown，不冒充没有动效。

linkding 并非重动效产品。第一案例只验证实际存在的展开/收起、焦点与状态切换；原站没有动画时要求即时切换，不能凭空加“更好看”的动画。另建自有 200 ms 动效校准样本，验证工具能识别错误时长与未执行动画。校准通过不代表复杂商业网站动效复现通过。

## 8. 预算与失败处理

默认每次生成实验最多 1 次初始生成 + 3 次自动修正，最多 120 分钟 agent 活跃时间。默认不存在自动付费模型调用；使用用户选择的宿主或显式配置的模型预算。token/费用不可观测时必须在报告中保留该限制。

超限保存检查点并返回 partial；断网、浏览器崩溃、缺少字体、参考变化、登录过期分别归类，不能循环重试直到挑出一次成功。每次运行保存 run ID、attempt 和 failure reason。

普通 run 可继续定位和修复；发布验收固定同一候选 revision 连续运行。中途改代码、改 seed、改阈值后重新开始发布验收。

## 9. 里程碑

| 阶段 | 交付与出口 |
| --- | --- |
| M0：本次 | MVP spec、案例、验收标准、公开仓库；运行能力仍为 not implemented |
| M1：可重复参考 | 固定版本的 linkding、本地测试页面、seed/reset、环境锁；原站流程和自比较稳定性验证 |
| M2：证据与验收器 | 浏览器采集、流程适配器、结果判定、视觉/动效校准、故意错误候选的拒绝测试 |
| M3：单次生成 | 第一版 skill、独立候选项目；观察证据到运行版本的完整执行记录 |
| M4：闭环修复 | 失败定位、最多三轮修正、独立 holdout、同候选重复验证 |
| M5：case-001 alpha | 全部必需门槛通过、另一位开发者从干净环境复跑；发布注明单案例范围 |

M5 以后再选不同类型的网站考察泛化能力。当前不把三个案例、跨模型对照或原生 App 扩展塞进 MVP。

## 10. 首案例与验收入口

首案例：[linkding 书签工作台](CASE_001_LINKDING.zh-CN.md)。所有必须满足的门槛见 [ACCEPTANCE.zh-CN.md](ACCEPTANCE.zh-CN.md)。机器可读范围在 [case.json](../cases/linkding-v1/case.json)。

本规格定义的是拟实现的目标与判定方式。首次运行前，参考响应、动效时长、成功率、token 成本和生成耗时均未测量。
