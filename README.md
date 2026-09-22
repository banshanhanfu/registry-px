# PuXian Registry 生态库规划（registry-px）

> PuXian（普贤）生态库的规划与实现仓库。
> 定位：参考 Go / Rust / Python 生态，为 PuXian 规划并落地一批 registry 生态库（纯 `.px`
> 实现，随 `registry/<name>/<version>/<name>.px` 版本化分发，`pxpkg add` 可拉取）。

## 为什么参考 Go / Rust / Python

- **基础层已经比 Go 厚**：PuXian 现有 13 个 stdlib + 368 个 runtime native，已覆盖
  Go 标准库的 net/http（甚至超出：HTTP/1.1/2/3+QUIC、WS、SSE、路由/中间件/会话）、
  SQLite、json/xml/base64/hex、regexp、strings、time、crypto（AES/RSA/Ed25519/
  MD5/SHA256/HMAC/PBKDF2）、zip/gzip、os 与 os/exec 等。
- **缺口在第二层**：Go 靠第三方生态补齐的领域库——UUID、JWT、decimal、CSV、
  CLI、日志、测试断言、任务池、配置合并、模板、校验、重试…… 这些正是 PuXian
  生态最应该先补的地基。
- **T3 增量来自 Rust / Python**：case 转换、集合组合子、网络地址（IP/CIDR）、
  编码（base58/checksum）、解析类（.env/glob/ini/diff）、展示类（table/ansi），
  以及精确有理数（fractions）、二分（bisect）等 Python 标准库面。

## 路线图

| 梯队 | 主题 | 状态 |
|---|---|---|
| T0 | 门槛库：uuid / jwt / decimal / csv / cli / log / testkit / workerpool | ✅ 8/8 完成 |
| T1 | 重要库：config / template / validator / retry / toml / datetime / passhash / secure_random / datastruct / concurrent_map / mailparse / stats | ✅ 12/12 完成 |
| T2 | 差异化：metrics / tar / fsnotify / big / idcard / cnnum | ✅ 6/8 完成（qrcode、pg·mysql 驱动待专项） |
| T3 | 参考 Rust/Python：strcase / fractions / ipaddr / dotenv / glob / checksum / itertools / base58 …（T3b 续：table / jsonpath / diff / ini / textwrap / bisect / ulid / ansi） | ✅ T3a 8/8 + T3b 8/8 |

详细清单与论证：
- Go 面：[`docs/library-plan-go.md`](docs/library-plan-go.md)
- Rust 面：[`docs/library-plan-rust.md`](docs/library-plan-rust.md)
- Python 面：[`docs/library-plan-python.md`](docs/library-plan-python.md)

## 进度

### T0 门槛库（8/8 ✅）

| 库 | 版本 | 参考 | 验证 |
|---|---|---|---|
| uuid | 0.1.0 | google/uuid | 双模式 PASS · golden |
| jwt | 0.1.0 | golang-jwt/jwt | 双模式 PASS · Python OpenSSL 交叉验证 |
| decimal | 0.1.0 | shopspring/decimal | 双模式 PASS · Python Decimal 对拍 |
| csv | 0.1.0 | encoding/csv | 双模式 PASS · Python csv 对拍 |
| cli | 0.1.0 | flag + cobra 子集 | 双模式 PASS |
| log | 0.1.0 | log + log/slog | 双模式 PASS |
| testkit | 0.1.0 | testify + testing | 双模式 PASS |
| workerpool | 0.1.0 | errgroup + workerpool | 编译模式 PASS（并发需 px build） |

> 每个库遵循官方写库规范：纯函数优先、Result 错误、<500 行、`px fmt`/`px lint` 0 错、
> 编译（px build）与解释（px run）双模式一致（workerpool 含并发仅编译模式）。

### T1 重要库（12/12 ✅、双模式 PASS）

config ✓ template ✓ validator ✓ retry ✓ toml ✓ datetime ✓ passhash ✓ secure_random ✓ datastruct ✓ concurrent_map ✓ mailparse ✓ stats ✓

### T2 差异化（6/8 ✅ 已完成，2 项待专项）

| 库 | 状态 | 说明 |
|---|---|---|
| big | ✅ | 大整数（Python int 对拍） |
| metrics | ✅ | Prometheus 文本格式 |
| tar | ✅ | uStar；Python tarfile 互操作 |
| fsnotify | ✅ | 轮询快照 diff |
| idcard | ✅ | 身份证校验 GB 11643-1999 |
| cnnum | ✅ | 中文数字转换 |
| qrcode / pg·mysql 驱动 | ⏳ 待专项 | 超出<500行生态库单库规模（RS 纠错+扫码验证 / 完整 wire protocol），建议独立里程碑 |

### T3 参考 Rust/Python 生态的增量库（T3a 8/8 ✅ 首批 · T3b 8/8 ✅ 第二批）

| 库 | 状态 | 参考 | 说明 |
|---|---|---|---|
| strcase | ✅ 0.1.0 | heck / inflection | 命名风格互转（双模式 PASS） |
| fractions | ✅ 0.1.0 | fractions.Fraction | 精确有理数（双模式 PASS） |
| ipaddr | ✅ 0.1.0 | ipaddress / std::net | IPv4/IPv6/CIDR 校验与运算（双模式 PASS） |
| dotenv | ✅ 0.1.0 | python-dotenv / dotenvy | .env 解析（双模式 PASS） |
| glob | ✅ 0.1.0 | glob / fnmatch | 通配匹配与列举（双模式 PASS·登记 PX-DEF-014） |
| checksum | ✅ 0.1.0 | crc32fast / zlib | CRC32 / Adler32（双模式 PASS·Python zlib 对拍·登记 PX-DEF-015） |
| itertools | ✅ 0.1.0 | itertools | 集合组合子（双模式 PASS） |
| base58 | ✅ 0.1.0 | bs58 | Base58 / Base58Check（双模式 PASS·Python 对拍·登记 PX-DEF-016/017） |

### T3b 第二批（8/8 ✅）

| 库 | 版本 | 参考 | 验证 |
|---|---|---|---|
| table | ✅ 0.1.0 | comfy-table / tabulate | 对齐表格渲染 ASCII/UTF-8（双模式 PASS） |
| jsonpath | ✅ 0.1.0 | jsonpath-ng / jmespath | JSON 路径取值（键/下标/通配，双模式 PASS） |
| diff | ✅ 0.1.0 | difflib | LCS 行级 diff + 相似度（双模式 PASS） |
| ini | ✅ 0.1.0 | configparser | INI 配置解析（双模式 PASS） |
| textwrap | ✅ 0.1.0 | textwrap | 换行/缩进/去缩进排版（双模式 PASS） |
| bisect | ✅ 0.1.0 | bisect | 有序序列二分查找/插入（双模式 PASS） |
| ulid | ✅ 0.1.0 | ulid crate | 可排序唯一 ID（Crockford Base32，双模式 PASS） |
| ansi | ✅ 0.1.0 | colored / anstream | ANSI 颜色/样式/去转义（双模式 PASS） |

> 专项（待条件）：bytes_pack / rate / parser / xlsx / pdf / functools / faker / shutil 子集。
> ⚠️ 写库中暴露的语言缺陷持续登记：见 [`docs/语言缺陷.md`](docs/语言缺陷.md)（PX-DEF 系列，实测基线 0.2.0-m182；001/004/011 已由官方修复移除，当前 13 条有效）。

## 写库规范（引用 PuXian 官方）

- 纯函数优先，顶层只导出 `def/struct/enum/trait/impl/const`，不写顶层 `let/var` 状态
- 错误走 Result（`Ok(x)`/`Err(e)` + `?`/`!`），致命才 `panic`
- 每文件 <500 行（AI 友好），`px fmt` + `px lint` 0 错，双模式输出一致
- 需要版本化分发 → 发布到 registry：`registry/<name>/<version>/<name>.px`

## License

Apache-2.0