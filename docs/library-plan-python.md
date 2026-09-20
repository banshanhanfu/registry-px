# PuXian 生态库规划：参考 Python 标准库与生态

> 目标：继 `library-plan-go.md`（T0/T1/T2）与 `library-plan-rust.md`（T3 Rust 面）之后，
> 以 **Python 标准库 + 常用第三方**为第三参照系，规划 T3 的解析/算法/数据面生态库。
> 依据：PuXian 0.2.0-m155 aarch64 实测 + 现有 26 个 registry 库的落库经验。

## 1. 现状盘点：PuXian 已有 ≈ Python 标准库的哪些部分

| Python 标准库 | PuXian 现状 |
|---|---|
| json / base64 / hex / hashlib / hmac / re | ✅ native（json_parse/regex_match/sha256/hmac 等） |
| random / secrets / uuid | ✅ registry `secure_random` + `uuid` |
| decimal / fractions | ✅ registry `decimal`；❌ `fractions` 缺 → 待补 |
| datetime / zoneinfo | ✅ native `time_format`/`time_parse` + registry `datetime` |
| collections（deque/Counter/OrderedDict） | ✅ registry `datastruct`（deque/heap/LRU）；dict 天然有序 |
| heapq / bisect | ✅ heap 有（datastruct）；❌ `bisect` 缺 → 待补 |
| itertools / functools | ❌ 缺 → 待补 `itertools` / 部分 `functools` |
| glob / fnmatch / shutil | ⚠️ `list_dir`/`file_stat` 有；❌ 通配匹配缺 → 待补 `glob`；shutil 依赖文件复制 native（待验证） |
| configparser / tomllib / dotenv | ✅ registry `toml`；❌ INI 与 `.env` 缺 → 待补 `ini`/`dotenv` |
| difflib | ❌ 缺 → 待补 `diff` |
| textwrap / string.Template | ⚠️ registry `template`（Go 心智）；❌ wrap/indent/dedent 缺 → 待补 `textwrap` |
| struct / array | ❌ 二进制 pack/unpack 缺 → 待补 `bytes_pack`（与 Rust `byteorder` 同源） |
| argparse / logging / unittest | ✅ registry `cli`/`log`/`testkit` |
| pathlib / os.path / subprocess | ✅ native `path_*`/`os_*` + std.path |
| sqlite3 / csv / zipfile / tarfile / gzip | ✅ native sqlite/zip/gzip + registry `csv`/`tar` |
| statistics / math / random | ✅ registry `stats` + native math/rand |
| email | ✅ registry `mailparse`（RFC 5322） |
| ipaddress | ❌ 缺 → 待补 `ipaddr`（与 Rust 文档共用） |
| pydantic / click / requests / jinja2 | ✅ validator/cli/http/template 已覆盖 |

## 2. T3 候选清单（参考 Python 生态）

按价值 × 可行性排序（每库预期 <500 行，纯函数可写）：

| # | 库 | 对应 Python | 为什么 | 实现要点 |
|---|---|---|---|---|
| 1 | **itertools** | itertools | 与 Rust 文档同列；product/排列/组合/分块应用面极广，写库自举用得上 | 纯函数返回 list：`it_product`/`it_permutations`/`it_combinations`/`it_chunk`/`it_window`/`it_cycle`/`it_take`/`it_drop`/`it_zip_longest` |
| 2 | **fractions** | fractions.Fraction | 精确有理数（费率/配比/坐标运算），`decimal` 之外的第二精确数值面 | 分子/分母约分（辗转相除 gcd）+ 四则/比较/`fraction_of`；内部用 int 表示，`Ok([num, den])` 或 dict |
| 3 | **ipaddr** | ipaddress | 接口自然、应用面广；与 Rust 文档共用 | 见 Rust 文档；对拍 Python `ipaddress` 结果 |
| 4 | **diff** | difflib | 配置对比、测试 diff 输出、日志变更检测 | LCS 行级 diff（小规模 DP 即可）：`diff_lines(a, b)` → 带 `-`/`+`/` ` 前缀行；`diff_ratio` 相似度 |
| 5 | **ini** | configparser | 老配置格式仍大量存在（.ini/.conf）；补齐三件套后的第四种 | 节 `[sec]` + `k=v` + 注释；`ini_parse(s)` → dict（节 → 键值 dict） |
| 6 | **dotenv** | python-dotenv | 与 Rust 文档共用；部署配置分层 | 见 Rust 文档 |
| 7 | **glob** | glob + fnmatch | 与 Rust 文档共用；批量文件处理 | 见 Rust 文档 |
| 8 | **bisect** | bisect | 有序序列插入/查找（区间、日程、排行榜） | `bisect_left(a, x)`/`bisect_right`/`bisect_insort`；二分 |
| 9 | **textwrap** | textwrap | 日志/告警/帮助文本排版；纯字符串 | `text_wrap(s, width)`/`text_fill`/`text_dedent`/`text_indent`；按词断行 |
| 10 | **jsonpath** | jsonpath-ng / jmespath | 从嵌套 dict/list 按路径取数，配置/响应提取通用 | `jsonpath_get(data, "a.b[0].c")` → Ok(值)；分段解析 `a`/`[i]`/`[*]` 子集；纯遍历 |
| 11 | **bytes_pack** | struct | 二进制协议/文件头；大小端、定宽整型 | `bytes_pack(fmt, ...)`/`bytes_unpack(fmt, b)`；基于 native `bytes_get/set/slice/concat/to_int`；格式符仿 `<IHQd` 子集 |
| 12 | **functools** | functools（子集） | partial/compose 让回调风格写库更顺；无状态设计 | `fn_partial(f, args…)`（返回闭包）、`fn_compose`、`fn_memoize(f)`（外部 dict 传入）；注意闭包捕获限制（PX-DEF-001） |
| 13 | **faker** | Faker | 演示数据/测试夹具生成；数据驱动 | 内置中文/英文姓名、手机号（1xx 段）、地址词表；`fake_name`/`fake_phone`/`fake_email`/`fake_uuid`；词表较大 → 控制在 500 行内 |
| 14 | **shutil（子集）** | shutil | 文件复制/移动/删除树；依赖 native 文件读写能力 | 需确认 native `os_copy`/读文件 API；若缺失 → 标注待官方补 native |
| 15 | **openpyxl / reportlab / pandas** | 第三方大件 | 表格文件/PDF/数据分析——纯二进制或超大规模 | 超出单库规模 → 专项（与 qrcode、pg/mysql 驱动同档） |

## 3. 批次建议

- **T3a（第一批，覆盖两文档交集）**：strcase（Rust）、fractions、ipaddr、
  dotenv、glob、checksum（Rust）、itertools、base58（Rust）。
- **T3b（第二批，按 Python 面优势排序）**：diff、jsonpath、bisect、ini、textwrap、
  table（Rust）、ulid（Rust）、ansi（Rust）。
- **专项/待条件**：bytes_pack、functools、faker、shutil 子集、parser、xlsx、pdf。

> 总表与进度见 `README.md` 路线图 T3 行；两文档重复项（itertools/ipaddr/dotenv/
> glob/table/ansi/ulid/bytes_pack）以本文档 + Rust 文档交叉引用为准，实现时合并为
> 单一 registry 库，不重复造轮子。