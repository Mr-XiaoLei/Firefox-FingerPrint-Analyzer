<div align="center">

<img src="https://raw.githubusercontent.com/LoseNine/Firefox-FingerPrint-Analyzer/main/icon.png" width="96" height="96" alt="Ruyi Trace" />

# 如意 Trace · Ruyi Trace

[**🇨🇳 简体中文**](./README.md) &nbsp;|&nbsp; [🇬🇧 English](./README.en.md)

A research-grade, instrumented Firefox build for fingerprint risk-control analysis and one-click AI-assisted JavaScript reverse-engineering.

![Platform](https://img.shields.io/badge/platform-Windows%20x64-blue)
![Firefox](https://img.shields.io/badge/firefox-DOMTrace-orange)
![License](https://img.shields.io/badge/license-MPL--2.0-green)
![Status](https://img.shields.io/badge/status-research-yellow)

</div>

---

### 这是什么

**如意 Trace** 是一个面向**网站指纹风控研究**与**辅助 JS 逆向学术分析**的桌面工具，由两部分组成：

1. **如意定制 trace 内核**（已随包发布）。
2. **Electron 客户端**（`RuyiTrace.exe`）——一键启动定制内核、采集 NDJSON 日志、管理日志文件。

> ⚠️ **仅限学术研究、安全教育与防御性分析使用。** 请勿用于绕过任何网站的服务条款或业务风控。

### 核心价值

> **ruyiPage 抓轮廓 → 如意 Trace 采集运行时日志 → 把日志交给 AI → 自动得到补环境代码与指纹风控分析报告。**

由于探针位于 **C++ 内核层而非 JS 层**，页面脚本无法通过原型检测、`toString` 嗅探、或已知 hook 特征发现监听存在 —— 因此采集的 trace 数据可作为研究指纹检测策略的高保真基线。

### 推荐工作流

1. **先用 [ruyiPage](https://github.com/LoseNine/ruyipage) 自动化框架，并使用其 Release 中配套的火狐指纹浏览器分析目标网页**，获取网站加载的全部 JS 文件并抓取完整的网络数据包，建立网站的整体轮廓。
2. **再用本工具（如意 Trace）启动定制 Firefox 进行实际访问**，生成 NDJSON 运行时调用日志。
3. **把日志直接交给 AI**，让 AI 结合第 1 步拿到的 JS 文件和数据包，对相关文件进行定位、补环境与指纹风控分析。

### 快速上手

1. **保持完整目录结构**：`RuyiTrace.exe` 必须与 `firefox/` 子目录处于同一文件夹。
2. **双击运行 `RuyiTrace.exe`**。
3. 在窗口中：
   - 填写**启动页面**（默认 `https://xjbedu.site/`）
   - 选择**日志目录**（默认 `<exe目录>/log`）
   - 确认顶部"定制内核"徽章为绿色（**定制内核已就绪**）
4. 点击 **开始采集** —— 定制 Firefox 自动启动并加载目标页面，右上角胶囊开始计时。
5. 在浏览器中正常浏览 / 触发指纹检测脚本。
6. 完成后点击 **停止采集**，日志保存为 `trace_<时间戳>_<PID>.ndjson`。
7. 点击 **打开目录** 取走 NDJSON 日志文件。

### 把日志交给 AI

NDJSON 每行一条事件，结构如下：

```json
{
  "t": "call",
  "api": "CanvasRenderingContext2D.fillText",
  "args": ["BrowserLeaks,com", 4, 17],
  "stack": [
    {"file": "https://example.test/fp.js", "line": 42, "col": 17}
  ]
}
```

**推荐 AI 分析提示词模板：**

```
你是一名前端安全研究员。我已经做了如下前置工作：
1. 先使用 ruyiPage 自动化框架（https://github.com/LoseNine/ruyipage）
   ——并使用其 release 中配套的火狐指纹浏览器——分析了目标网页，
   获取了网站加载的全部 JS 文件，并抓取了完整的网络数据包，
   建立了网站整体轮廓。
2. 然后使用如意 Trace 工具采集了一份完整的 DOM/JS API 运行时调用日志（NDJSON）。

现在我把这份运行时日志直接交给你，请你结合第 1 步拿到的 JS 文件与网络数据包：
1. 在 NDJSON 中识别所有指纹采集点（Canvas / WebGL / WebRTC / Audio /
   Navigator / Screen / Crypto），并定位到具体是哪个 JS 文件的哪个函数。
2. 还原每个指纹函数的完整调用链与入口脚本。
3. 输出一份补环境（环境替换）JavaScript 代码模板，使脚本运行结果与真实浏览器一致。
4. 列出该网站采用的指纹风控策略与对抗优先级。

[在此粘贴 NDJSON 内容]
```

把日志贴给 ChatGPT、Claude、Gemini 等任意大模型即可获得：
- 📊 **指纹风控分析报告**（哪些 API 被读取、调用顺序、关键参数）
- 🛠️ **补环境代码模板**（Canvas / WebGL / Audio / Navigator 自动还原）
- 🔍 **完整的 JS 调用图谱**（从入口函数到底层 API 的完整路径）

### 文件结构

```
RuyiTrace/                  (≈ 427 MB)
├── RuyiTrace.exe           Electron 客户端 (≈ 87 MB)
├── README.md               本说明
└── firefox/                如意定制 trace 内核 (≈ 340 MB)
    ├── firefox.exe
    ├── RUYI_DOMTRACE.txt   ← 内核标志文件，请勿删除
    └── ... DLL / 资源
```

### 高级使用

**手动指定环境变量启动 Firefox（不通过客户端）：**

```cmd
set MOZ_DOM_TRACE=1
set MOZ_DOM_TRACE_FILE=D:\logs\trace.ndjson
set MOZ_DISABLE_LAUNCHER_PROCESS=1
firefox\firefox.exe -no-remote -new-instance https://example.com
```

**环境变量速查：**

所有开关都要在启动 `firefox.exe` 前通过 `set` / `export` 设置。进程启动时只读取一次，运行中修改无效。

#### 总开关与输出

| 变量 | 可选值 | 默认 | 用途 |
|------|--------|------|------|
| `MOZ_DOM_TRACE` | `1` / `0` / 不设 | 不设 | 主开关。`1` 开核心 DOM/BOM；`0` 强制全关，优先级最高 |
| `MOZ_DOM_TRACE_FILE` | 文件路径 | 空 | 总锚点。设置后自动派生各模块文件 |
| `MOZ_DOM_TRACE_PTYPE` | `parent` / `content` / 任意串 | `parent` | 进程类型标签，写进记录和文件名 |
| `MOZ_DOM_TRACE_LIMIT` | 整数，`0` 为无限 | `0` | 核心 trace 事件上限，不含 jscall |
| `MOZ_DOM_TRACE_INTERNAL` | `1` / 不设 | 关 | 记录浏览器内部噪声，如 `chrome://`、`resource://`，用于排查内部行为 |
| `MOZ_DOM_COOKIE_TRACE_FILE` / `MOZ_DOM_STORAGE_TRACE_FILE` / `MOZ_DOM_WASM_TRACE_FILE` | 文件路径 | 由锚点派生 | 单独把某个模块导到其他文件 |
| `MOZ_DISABLE_LAUNCHER_PROCESS` | `1` / 不设 | 不设 | Windows 下避免 launcher 提前退出，便于稳定 PID |

锚点派生文件包括 `trace_process_<pid>`、`_cookie_`、`_storage_`、`_eval_`、`_event_`、`_descriptor_`、`_wasm_`、`_jscall_`。

#### JS 调用 trace（jscall）

jscall 用于抓 JS 函数调用、参数和返回值，是逆向加密函数、补环境和定位指纹入口的主力。

| 变量 | 可选值 | 默认 | 用途 |
|------|--------|------|------|
| `MOZ_DOM_JSCALL_TRACE` | `1` / 不设 | 不设 | jscall 主开关，可独立于 `MOZ_DOM_TRACE` |
| `MOZ_DOM_JSCALL_TRACE_FILE` | 文件路径 | 由锚点派生 | jscall 输出文件，设置后也会启用 jscall |
| `MOZ_DOM_JSCALL_LIMIT` | 整数，`0` 为无限 | `50000` | jscall 事件上限。逆向时常设 `0`；命中上限会写 `limit_reached` 哨兵 |
| `MOZ_DOM_JSCALL_FLUSH_INTERVAL` | `0`..`1000000` | `256` | 每多少条刷盘。担心进程被杀时可调小，如 `8`，不建议设为 `1` |
| `MOZ_DOM_JSCALL_SUMMARY_INTERVAL` | `1`..`1000000` | `256` | 每多少条输出一次汇总统计行 |
| `MOZ_DOM_JSCALL_NATIVE` | `1` / 不设 | 关 | 连原生 builtin 调用也记录 |
| `MOZ_DOM_JSCALL_SELFHOSTED` | `1` / 不设 | 关 | 连 self-hosted 调用也记录，如 `Array.map` |
| `MOZ_DOM_JSCALL_TARGET_ONLY` | `1` / 不设 | 关 | 只记录命中 detail 过滤器的目标函数及子调用，大幅降噪 |

detail 过滤器用于抓参数和返回值真值，下面四类条件是 OR 关系：

| 变量 | 可选值 | 默认 | 用途 |
|------|--------|------|------|
| `MOZ_DOM_JSCALL_DETAIL_FUNCS` | 逗号分隔函数名 | 空 | 按函数名定向，如 `encrypt,sign`。命中者强制深抓不截断 |
| `MOZ_DOM_JSCALL_DETAIL_URL_CONTAINS` | 逗号分隔 URL 子串 | 空 | 调用方或被调方 URL 含子串则抓，旧式用法，通常配函数名 |
| `MOZ_DOM_JSCALL_DETAIL_SCRIPT_URL` | 逗号分隔 URL 子串 | 空 | 按脚本来源整体抓，不需要预知函数名，适合锁单脚本逆向 |
| `MOZ_DOM_JSCALL_DETAIL_SHA256` | 逗号分隔 sha256 | 空 | 按源码 sha256 抓，用于 URL 不稳定的动态或混淆脚本 |

序列化开关控制参数和返回值抓取深度：

| 变量 | 可选值 | 默认 | 用途 |
|------|--------|------|------|
| `MOZ_DOM_JSCALL_SHALLOW` | `1` / 不设 | 关 | 浅序列化。保留标量、TypedArray、Array，普通对象塌缩；可降低热函数开销 |
| `MOZ_DOM_JSCALL_SHALLOW_DEPTH` | `0`..`12` | `0` | 浅序列化下普通对象递归层数，`2` 足够覆盖多数嵌套 |
| `MOZ_DOM_JSCALL_MAX_VALUE_BYTES` | `256`..`131072` | `65536` | 单值序列化字节上限 |
| `MOZ_DOM_JSCALL_DEEP_LONG_STR` | `512`..`131072`，`0` 为关 | `0` | 长字符串整体抓的触发阈值，让长密文、HMAC hex 不被截断 |

#### opcode 级 trace

opcode trace 下沉到逐 JS 字节码 op，适合分析自建字节码 VM / jsvmp。它覆盖 C++ 解释器和 Baseline JIT；解释器层有栈值，Baseline JIT 只有 `pc` 和 `op`，用 `tier` 字段区分。

| 变量 | 可选值 | 默认 | 用途 |
|------|--------|------|------|
| `MOZ_DOM_JSCALL_OPCODE_URL` | 逗号分隔 URL 子串 | 空 | opcode trace 总开关。务必收窄到单脚本，日志量极大 |
| `MOZ_DOM_JSCALL_OPCODE_PC_START` | 整数 | `0` | 只 trace `PC >=` 此值的 op，可配合 `_END` 锁字节码片段 |
| `MOZ_DOM_JSCALL_OPCODE_PC_END` | 整数，`0` 为不限 | `0` | 只 trace `PC <=` 此值的 op。`_END > _START` 时窗口生效 |
| `MOZ_DOM_JSCALL_OPCODE_STACK` | `1` / 不设 | 关 | 每条 op dump 栈顶值，仅解释器层有值 |
| `MOZ_DOM_JSCALL_OPCODE_STACK_SLOTS` | `1`..`32` | `3` | 抓栈顶几槽。`SetElem` 通常需要 3；`Call` 抓数组参数时可调大 |
| `MOZ_DOM_JSCALL_OPCODE_STACK_FULL` | `1` / 不设 | 关 | 栈槽深序列化且不截断。适合抓内层 cipher 的完整密钥和明文，但必须配 PC 窗口 |
| `MOZ_DOM_JSCALL_OPCODE_LIMIT` | 整数，`0` 为无限 | `0` | opcode 事件上限，独立于 jscall |
| `MOZ_DOM_JSCALL_OPCODE_OPERANDS` | `1` / 不设 | 关 | 每条 op 带 `operands`，包括 local 槽号、常量、jump 偏移、atom 名和字符串；用于识别 VM 的 pc、栈指针、key 槽 |

#### vm_step（JSVMP 指令流）

vm_step 会把派发循环里的多条宿主 op 折叠成一条 VM 指令记录，核心产出 `(vm_pc, op_byte, vm_pc_next)`。可以手动声明槽位，也可以用 autodetect 自动锁派发器。

| 变量 | 可选值 | 默认 | 用途 |
|------|--------|------|------|
| `MOZ_DOM_JSVMP_TRACE` | `1` / 不设 | 关 | vm_step 总开关 |
| `MOZ_DOM_JSVMP_SCRIPT_URL` | 逗号分隔 URL 子串 | 空 | 脚本过滤，必填；否则容易误命中其他脚本 |
| `MOZ_DOM_JSVMP_AUTODETECT` | `1` / 不设 | 关 | 自动检测派发器，免声明 `DISPATCH_PC` / `PC_SLOT`，会写 `vm_dispatch_detected` |
| `MOZ_DOM_JSVMP_MAX_DISPATCH` | `1`..`12` | `1` | autodetect 最多锁几个派发器 |
| `MOZ_DOM_JSVMP_DISPATCH_PC` | 整数（宿主 pc） | 无 | 取字节那条 `GetElem` 的 pc；autodetect 时可省 |
| `MOZ_DOM_JSVMP_PC_SLOT` | 整数（槽号） | 无 | VM pc 所在宿主槽；autodetect 时可省 |
| `MOZ_DOM_JSVMP_PC_KIND` | `arg` / `local` | `local` | `PC_SLOT` 是实参还是局部 |
| `MOZ_DOM_JSVMP_KEY_SLOT` | 整数（槽号） | 无 | 可选。rolling key 槽，只有独立 key 槽时才声明 |
| `MOZ_DOM_JSVMP_KEY_KIND` | `arg` / `local` | `local` | `KEY_SLOT` 是实参还是局部 |
| `MOZ_DOM_JSVMP_BRANCH_PC` | 整数（宿主 pc） | 无 | VM 派发比较 op 的 pc，产出 `vm_branch{lhs,rhs,cmp,taken}` |
| `MOZ_DOM_JSVMP_DUMP_BYTECODE` | `1` / 不设 | 关 | 首次 dispatch 时 dump 整段字节码数组，写 `vm_bytecode` hex，最大 1 MB |
| `MOZ_DOM_JSVMP_CONST_SLOT` | 整数（槽号） | 无 | 可选。VM 常量表槽，dump `vm_const{values}`，最多 4096 元素 |
| `MOZ_DOM_JSVMP_CONST_KIND` | `arg` / `local` | `local` | `CONST_SLOT` 是实参还是局部 |
| `MOZ_DOM_JSVMP_MIN_BYTECODE` | 整数（字节数） | `0` 为不限 | autodetect 滤噪，只喂字节码长度达到阈值的 `GetElem` |
| `MOZ_DOM_JSVMP_MIN_HITS` | 整数 > 0 | `6` | autodetect 候选最少命中数。检测只在解释器阶段，设得过高可能锁不到 |
| `MOZ_DOM_JSVMP_MIN_SPAN` | 整数（下标跨度） | `0` 为不限 | autodetect 最有效滤噪项，候选下标跨度达到阈值才锁 |
| `MOZ_DOM_JSVMP_OBSERVE_OPS` | 整数 > 0 | `2000` | `MIN_SPAN` 模式下锁定前的观察窗口 |
| `MOZ_DOM_JSVMP_LIMIT` | 整数，`0` 为无限 | `0` | vm_step 事件上限 |

#### WASM、HTTP 与 WebSocket

| 变量 | 可选值 | 默认 | 用途 |
|------|--------|------|------|
| `MOZ_DOM_WASM_DUMP` | `0` / 不设 | 开 | WASM 模块反汇编 dump 开关，`0` 关闭 |
| `MOZ_DOM_WASM_DUMP_MAX` | 字节数 | `8388608` | 单模块 dump 上限 |
| `MOZ_DOM_WASM_DUMP_TOTAL` | 字节数 | `134217728` | 所有模块累计 dump 上限 |
| `MOZ_DOM_WASM_INSN` | `1` / 不设 | 关 | WASM 指令级 trace，逐函数反汇编 |
| `MOZ_DOM_WASM_INSN_FUNC_INDEX` | 函数索引，`-1` 为全部 | `-1` | 只反汇编指定函数 |
| `MOZ_DOM_WASM_INSN_MAX_FUNCS` | 整数，`0` 为无限 | `0` | 最多反汇编几个函数 |
| `MOZ_DOM_WASM_INSN_MAX_BYTES` | 字节数 | `1048576` | 单函数反汇编上限 |
| `MOZ_DOM_HTTP_PACKET_TRACE` | `0` / 非 `0` | 开 | HTTP 报文 trace，非 `0` 即开启 |
| `MOZ_DOM_HTTP_PACKET_TRACE_DIR` | 目录路径 | 锚点目录 | HTTP 报文输出目录 |
| `MOZ_DOM_WS_TRACE` | `1` / 不设 | 关 | WebSocket 帧 trace 总开关，在 C++ 网络层抓解压 / 掩码前的明文 |
| `MOZ_DOM_WS_MAX_BYTES` | `256`..`16777216` | `65536` | 单帧 payload 字节上限 |

#### 推荐开关组合

下面组合省略了公共前缀 `MOZ_DOM_`，实际设置时请写完整变量名。

| 场景 | 推荐组合 |
|------|----------|
| 逆向某个已知加密函数 | `JSCALL_TRACE=1` + `JSCALL_TARGET_ONLY=1` + `JSCALL_DETAIL_FUNCS=encrypt,sign` + `JSCALL_SHALLOW=1` + `JSCALL_LIMIT=0` |
| 不知道函数名，或函数名被混淆 | `JSCALL_TRACE=1` + `JSCALL_DETAIL_SCRIPT_URL=<script-url-part>` + `JSCALL_SHALLOW=1` + `JSCALL_DEEP_LONG_STR=512` |
| 看穿自建字节码 VM | `JSCALL_TRACE=1` + `JSCALL_OPCODE_URL=<script-url-part>` + `JSCALL_OPCODE_STACK=1` + `JSCALL_OPCODE_OPERANDS=1` + `JSCALL_OPCODE_LIMIT=<n>` |
| 时延敏感的反爬 VM | `JSCALL_DETAIL_SCRIPT_URL=<script-url-part>` + `JSCALL_SHALLOW=1`，保留 JIT，避免深序列化 |
| 还原已知槽位的 jsvmp 指令流 | `JSVMP_TRACE=1` + `JSVMP_SCRIPT_URL=<script-url-part>` + `JSVMP_DISPATCH_PC=<pc>` + `JSVMP_PC_SLOT=<slot>`，按需加 `KEY_SLOT` / `BRANCH_PC` / `DUMP_BYTECODE` / `CONST_SLOT` |
| 自动还原 jsvmp | `JSVMP_TRACE=1` + `JSVMP_SCRIPT_URL=<script-url-part>` + `JSVMP_AUTODETECT=1`，重混淆站点可加 `JSVMP_MIN_BYTECODE=128` + `JSVMP_MIN_SPAN=200` |
| 抓 WebSocket 帧 | `TRACE_FILE=<path>` + `WS_TRACE=1` |

#### 使用纪律

1. 先收窄再开量大的开关。`JSCALL_OPCODE_*` / `JSVMP_*` 必须配 `JSCALL_OPCODE_URL` 或 `JSVMP_SCRIPT_URL` 锁单脚本，否则可能产生 GB 级日志。
2. `JSCALL_OPCODE_STACK_FULL` 必须配 `JSCALL_OPCODE_PC_START` / `JSCALL_OPCODE_PC_END` 和 `JSCALL_OPCODE_LIMIT`，并在驱动侧监控日志大小，超过阈值就停止浏览器。
3. 反爬站不要禁用 JIT，不要深序列化热路径。优先使用 `JSCALL_SHALLOW=1`。
4. vm_step 要全程不断流需要 tier-pin，仅适合离线对拍：`JIT_OPTION_baselineInterpreterWarmUpThreshold=0` + `JIT_OPTION_normalIonWarmUpThreshold=2000000000`。真实 solve 场景慎用。
5. 已知结构的 mini-VM 优先手动声明 `DISPATCH_PC` / `PC_SLOT`；每会话 pc 漂移的场景再用 autodetect。
6. 进程可能被杀时调小 `JSCALL_FLUSH_INTERVAL`，例如 `8`，降低缓冲丢失风险。
7. 多进程会分别写带 `<pid>` 的文件。跨模块分析可用 `origin_call_id` 串起“哪次调用读了哪些指纹 / 写了哪个 cookie”的数据流。

### 常见问题

**Q：为什么不能用 Puppeteer / Playwright？**
A：它们注入 JS 钩子，可被页面脚本探测（`toString`、原型链、`navigator.webdriver` 等）。如意 Trace 在 C++ 层埋点，从 JS 视角完全不可见。

**Q：日志文件特别大怎么办？**
A：单次会话可能产生数百 MB。建议：**分段读取**——把 NDJSON 按行切成多块（例如每 5–10 万行一段），分批投喂给 AI 分析；同时可以 (1) 短时间内只访问目标页 (2) 使用 `MOZ_DOM_TRACE_LIMIT` 限制单进程行数 (3) 用 `MOZ_DOM_TRACE_PTYPE` 只保留关心的进程类型。

**Q："定制内核"徽章显示红色？**
A：表示选中的 `firefox.exe` 旁边没有 `RUYI_DOMTRACE.txt` 标志文件 —— 你大概率指向了系统自带的官方 Firefox。请确保 `RuyiTrace.exe` 与 `firefox/` 子目录处在同一文件夹。

**Q：可以采集 HTTPS / Service Worker / Web Worker 内的调用吗？**
A：可以。`MOZ_DOM_TRACE_PTYPE` 自动传递到所有子进程。

### 免责声明

本项目修改了 Mozilla Firefox 源代码（MPL-2.0），二进制中**不包含 Mozilla 商标**。本项目**不隶属于 Mozilla 基金会**，不代表 Mozilla 的官方立场。"Mozilla" 与 "Firefox" 是 Mozilla 基金会的注册商标。

使用本工具产生的一切后果由使用者自行承担。请遵守你所在司法辖区的法律法规及目标网站的服务条款。

---

<div align="center">
<sub>如意 · 心想事成 — Ruyi · "as you wish"</sub><br/>
<sub>Built for researchers, by researchers.</sub>
</div>
