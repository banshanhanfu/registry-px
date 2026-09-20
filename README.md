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
| T0 | 门槛库：uuid / jwt / decimal / csv / cli / log / testkit / workerpool | 📋 规划 |
| T1 | 重要库：config / template / validator / retry / toml / datetime / passhash / secure_random / datastruct / concurrent_map / mailparse / stats | 📋 规划 |
| T2 | 差异化：metrics / tar / fsnotify / big / pg·mysql 驱动 / qrcode / 中文生态 | 📋 规划 |

详细清单与论证见 [`docs/library-plan-go.md`](docs/library-plan-go.md)。

## 进度

- [ ] T0 门槛库（8 个）
- [ ] T1 重要库（12 个）
- [ ] T2 差异化（按需）

## 写库规范（引用 PuXian 官方）

- 纯函数优先，顶层只导出 `def/struct/enum/trait/impl/const`，不写顶层 `let/var` 状态
- 错误走 Result（`Ok(x)`/`Err(e)` + `?`/`!`），致命才 `panic`
- 每文件 <500 行（AI 友好），`px fmt` + `px lint` 0 错，双模式输出一致
- 需要版本化分发 → 发布到 registry：`registry/<name>/<version>/<name>.px`

## License

Apache-2.0