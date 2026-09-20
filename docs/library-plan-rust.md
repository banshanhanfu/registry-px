# PuXian 生态库规划：参考 Rust 标准库与生态

> 目标：继 `library-plan-go.md`（T0/T1/T2）之后，以 **Rust 生态**为第二参照系，
> 规划 T3 一批纯 `.px` 生态库，补齐 Go 规划未覆盖的字符串/网络/编码/集合工具面。
> 依据：PuXian 0.2.0-m155 aarch64 实测 + 现有 26 个 registry 库的落库经验。

## 1. 现状盘点：PuXian 已有 ≈ Rust 生态的哪些部分

对照 Rust std + 常用 crates：

| Rust 生态 | PuXian 现状 |
|---|---|
| std::collections（HashMap/BTreeMap/VecDeque/BinaryHeap） | ✅ `dict`/`list` + registry `datastruct`（deque/heap/LRU/栈/队列） |
| std::time / chrono | ✅ native `time_format`/`time_parse` + registry `datetime` |
| std::net（IpAddr/CIDR 子集） | ❌ 缺：只有字符串可存，无校验/运算 → 待补 `ipaddr` |
| std::fs / glob crate | ⚠️ `list_dir`/`file_stat` 有；通配匹配与递归列举缺 → 待补 `glob` |
| std::env + dotenvy | ⚠️ native `env` 有；`.env` 文件解析缺 → 待补 `dotenv` |
| std::process / std::io | ✅ native `os_*` / 文件 IO |
| std::fmt（格式化） | ⚠️ 字符串拼接有；对齐/表格/ANSI 缺 → 待补 `table`/`ansi` |
| std::str（case 转换/切片增强） | ⚠️ 基础切片有；snake/camel/kebab 互转缺 → 待补 `strcase` |
| itertools crate | ❌ 缺 → 待补 `itertools`（product/permutations/chunk/window） |
| rand / uuid / getrandom | ✅ registry `secure_random` + `uuid` |
| serde / serde_json / toml / csv | ✅ native json + registry `toml`/`csv` |
| clap / anyhow / tracing / regex | ✅ registry `cli`/`log` + native `regex_match`（Result 即 anyhow 心智） |
| num-bigint / num-rational | ✅ registry `big`；❌ 有理数缺 → 待补 `fractions` |
| crc32fast / adler2 | ❌ 缺 → 待补 `checksum` |
| base64 / hex / bs58 | ✅ native base64/hex；❌ base58 缺 → 待补 `base58` |
| ulid crate | ❌ 缺 → 待补 `ulid` |
| tar / flate2 / zip / notify / fsnotify | ✅ registry `tar`/`fsnotify` + native zip/gzip |
| byteorder / zerocopy | ❌ 二进制 pack/unpack 缺 → 待补 `bytes_pack`（与 Python `struct` 同源） |
| nom / pest（解析器组合子） | ⚠️ regex 有；通用组合子解析器工程量大 → 专项 |
| rayon / crossbeam / dashmap | ⚠️ 并发仅编译模式（PX-DEF-006）；`concurrent_map` 已覆盖容器层 |

**结论**：Rust 生态对 PuXian 的启示仍然集中在"第二层工具面"。T0-T2 已吃掉大部分，
T3 的增量来自：**case 转换、集合组合子、网络地址、编码（base58/checksum）、
解析类（.env/glob）、展示类（table/ansi）**。

## 2. T3 候选清单（参考 Rust 生态）

按价值 × 可行性排序（每库预期 <500 行，纯函数可写）：

| # | 库 | 对应 Rust | 为什么 | 实现要点 |
|---|---|---|---|---|
| 1 | **strcase** | heck / convert_case | API/命令/枚举名互转是写库与工具链刚需；纯字符串零依赖 | snake_case ↔ kebab-case ↔ camelCase ↔ PascalCase ↔ SCREAMING_SNAKE；词边界分词 + 连接符 |
| 2 | **itertools** | itertools | 集合组合子（笛卡尔积/排列/组合/分块/滑动窗口），函数式写库利器 | 生成式 list 结果（纯函数返回 list，避免惰性生成器语法）；`it_product`/`it_permutations`/`it_combinations`/`it_chunk`/`it_window`/`it_cycle`/`it_take`/`it_drop` |
| 3 | **ipaddr** | std::net + ipnet | 边缘/网络场景校验 IP、算 CIDR、判断包含关系；纯整数位运算 | IPv4/IPv6 解析校验、`ip_in_cidr`、`cidr_network`/`cidr_broadcast`/`cidr_mask`、`ip_cmp` |
| 4 | **dotenv** | dotenvy | 配置分层（env 优先 → .env 文件兜底）是部署刚需 | 解析 `KEY=value`（引号/注释/export 前缀）；返回 dict；`dotenv_load(path)` + `dotenv_parse(s)` |
| 5 | **glob** | glob crate + std::fs | 文件批量处理（日志/缓存清理/备份）都要通配列举 | `glob_match(pattern, name)`（`*`/`?`/`[...]` 子集）+ 组合 `list_dir` 递归 `glob_list(dir, pattern)` |
| 6 | **checksum** | crc32fast / adler2 | 传输校验、缓存键、简单完整性；比 crypto 快、可移植 | 表驱动 CRC32（IEEE，同 zlib `crc32`）+ Adler32（同 zlib `adler32`）；对拍 Python zlib |
| 7 | **base58** | bs58 | 比特币地址/Key 的展示编码；Base58 去歧义字符 | 大数进制转换（byte string ↔ int ↔ base58 字母表）；附 Base58Check（4 字节 SHA256 双哈希校验，native sha256 已有） |
| 8 | **table** | comfy-table | CLI 输出对齐表；与 registry `cli` 天然衔接 | 列宽计算 + 对齐（左/右/中）+ 边框样式（ASCII/UTF-8）；`table_render(rows, opts)` |
| 9 | **ulid** | ulid | 排序友好、可读性强的 ID（时间有序），补充 uuid | 48bit 毫秒时间戳 + 80bit 随机 → Crockford Base32（26 字符）；native `time`/`os_random_hex` |
| 10 | **ansi** | colored / anstream | CLI 颜色输出（log/cli 增强）；纯字符串包不依赖 tty 判断 | 8/256 色前景/背景、粗体/下划线；`ansi_color(s, fg, style)`；tty 探测失败时降级纯文本 |
| 11 | **bytes_pack** | byteorder | 二进制协议/文件头读写（长度前缀、大小端） | `bytes_pack(fmt, args…)` / `bytes_unpack(fmt, b)`，仿 Python `struct`/`<IHQ` 等；基于 native `bytes_get/set/slice/concat/to_int` |
| 12 | **rate** | governor | 令牌桶限流；服务韧性基础件 | 无状态 API：`rate_bucket(tokens, capacity, rate, last_ts, now) → Ok(新桶, 是否放行)`；用显式 dict 传递桶状态（守写库规范顶层无状态） |
| 13 | **parser** | nom / pest | 通用解析器组合子——价值高但工程量大 | 需 and/or/rep/seq 组合子 + 错误定位；估算 >500 行 → 专项 |
| 14 | **xlsx / pdf** | rust_xlsxwriter / printpdf | 表格/文档生成——纯二进制容器，规模大 | 超出单库规模 → 专项（与 qrcode、pg/mysql 驱动同档） |

## 3. 批次建议

- **T3a（第一批，本仓库本轮落地）**：strcase、fractions（见 Python 文档）、ipaddr、
  dotenv、glob、checksum、itertools、base58 —— 全部纯函数、<400 行、双模式可测。
- **T3b（第二批）**：table、jsonpath（见 Python 文档）、diff（见 Python 文档）、
  ansi、ulid、bisect（见 Python 文档）。
- **专项/待条件**：bytes_pack、rate、parser、xlsx、pdf。

> ⚠️ 含并发/IO 轮询的库需注意双模式限制（PX-DEF-006）：T3 清单全部设计为
> 纯函数 + 显式状态入参，确保 `px run` 解释轨与 `px build` 编译轨结果一致。