# PuXian Registry 生态库规划（registry-px）

> PuXian（普贤）生态库的规划与实现仓库。
> 定位：参考 Go 生态，为 PuXian 规划并落地一批 registry 生态库（纯 `.px` 实现，随
> `registry/<name>/<version>/<name>.px` 版本化分发，`pxpkg add` 可拉取）。

## 为什么参考 Go

- **基础层已经比 Go 厚**：PuXian 现有 13 个 stdlib + 368 个 runtime native，已覆盖
  Go 标准库的 net/http（甚至超出：HTTP/1.1/2/3+QUIC、WS、SSE、路由/中间件/会话）、
  SQLite、json/xml/base64/hex、regexp、strings、time、crypto（AES/RSA/Ed25519/
  MD5/SHA256/HMAC/PBKDF2）、zip/gzip、os 与 os/exec 等。
- **缺口在第二层**：Go 靠第三方生态补齐的领域库——UUID、JWT、decimal、CSV、
  CLI、日志、测试断言、任务池、配置合并、模板、校验、重试…… 这些正是 PuXian
  生态最应该先补的地基。

## 路线图

| 梯队 | 主题 | 状态 |
|---|---|---|
| T0 | 门槛库：uuid / jwt / decimal / csv / cli / log / testkit / workerpool | ✅ 8/8 完成 |
| T1 | 重要库：config / template / validator / retry / toml / datetime / passhash / secure_random / datastruct / concurrent_map / mailparse / stats | 📋 规划 |
| T2 | 差异化：metrics / tar / fsnotify / big / pg·mysql 驱动 / qrcode / 中文生态 | 📋 规划 |

详细清单与论证见 [`docs/library-plan-go.md`](docs/library-plan-go.md)。

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

### T1 重要库（12/12 ✅）

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

> ⚠️ 语言缺陷登记：见 [`docs/语言缺陷.md`](docs/语言缺陷.md)（PX-DEF-001~012，写库过程中持续追加）

## 写库规范（引用 PuXian 官方）

- 纯函数优先，顶层只导出 `def/struct/enum/trait/impl/const`，不写顶层 `let/var` 状态
- 错误走 Result（`Ok(x)`/`Err(e)` + `?`/`!`），致命才 `panic`
- 每文件 <500 行（AI 友好），`px fmt` + `px lint` 0 错，双模式输出一致
- 需要版本化分发 → 发布到 registry：`registry/<name>/<version>/<name>.px`

## License

Apache-2.0