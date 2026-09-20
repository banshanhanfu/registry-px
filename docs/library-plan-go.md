# PuXian 生态库规划：参考 Go 标准库与第三方生态

> 目标：为 PuXian（普贤）规划一组 registry 生态库（纯 `.px` 实现），按优先级落地，
> 让语言从"作者自证"走向"生态可用"。
> 依据：PuXian 0.2.0-m155 aarch64 实测 + 官方仓库（spec.md / native_index.json /
> ECOSYSTEM.md / ECOSYSTEM_GAPS.md / ROADMAP.md）。

## 1. 现状盘点：PuXian 已有 ≈ Go 标准库的哪些部分

368 个 runtime native + 13 个 stdlib（collections / semver / webroute / yaml / pxml /
lunar / gfx / png / edge / html / cookiejar / multipart / smtp），对照 Go 标准库：

| Go 标准库 | PuXian 现状 |
|---|---|
| net/http + 路由/中间件/限流/会话/优雅关闭 | ✅ 超出（HTTP/1.1/2/3+QUIC、WS、SSE、vhost、SNI） |
| database/sql（SQLite） | ✅ native `sqlite_*` |
| encoding/json / xml / base64 / hex | ✅ native |
| regexp / strings / sort / math / math/rand | ✅ native + std.collections / std.strings |
| time（格式化/解析/时区）、timer/cron | ✅ native `time_format` / `time_parse` / `cron` |
| crypto/*（AES/RSA/Ed25519/MD5/SHA256/HMAC/PBKDF2） | ✅ native |
| archive/zip、compress/gzip、net/smtp、net/url | ✅ std.smtp / std.url + native |
| os、os/exec、os/signal、path | ✅ 大把 `os_*` native |

**结论**：PuXian 的基础层比 Go 标准库还厚。缺口不在"地基"，在**第二层**——Go 靠
第三方生态补齐的领域库。

## 2. 缺口地图（参考 Go 生态该补的）

### T0 门槛库 —— 没有它们，写任何正经应用都不顺手，建议第一批做

| 库 | 对应 Go | 为什么 | 实现要点（纯 .px 可写） |
|---|---|---|---|
| **uuid** | google/uuid | 请求 ID/消息 ID/文件名，几乎所有服务都要 | v4 = `os_random_hex` 拼格式；无需新 native |
| **jwt** | golang-jwt/jwt | 认证事实标准；有 HMAC-SHA256/RSA/Ed25519 native 底座 | base64url + `hmac_sha256`/`rsa_sign` 组合 |
| **decimal** | shopspring/decimal | 金额/精度计算，Float 不能用于钱 | 字符串+整数运算实现，纯算法 |
| **csv** | encoding/csv | 数据交换最常用格式，目前完全没有 | RFC 4180 解析器；<300 行 |
| **cli** | flag / cobra（子集） | 命令行程序参数解析；只有裸 `args` native | 支持 `--flag value`、子命令 |
| **log** | log/slog | 现在 `log` native 无级别；需要级别/结构化/时间戳 | 包一层 env 控制级别 + `time_format` |
| **testkit** | testify | 现在只有 `assert` 和顶层 `test_xxx`；缺丰富断言/临时目录 | assert 增强 + mkdir 临时区 |
| **workerpool** | errgroup + ant | Go 并发模式精华；chan/spawn 已有但缺任务池封装 | spawn+chan 实现，标注需编译模式 |

### T1 重要库 —— 服务端/配置/数据结构，第二批

| 库 | 对应 Go | 理由 |
|---|---|---|
| **config** | viper | `yaml_parse` + `env` 都有，缺"层层合并"的封装层 |
| **template** | text/template | 邮件（smtp 有了）、报表、代码生成都靠它；核心是纯字符串替换+循环 |
| **validator** | go-playground/validator | Web 服务请求校验，和 route/middleware 天然衔接 |
| **retry** | hashicorp/go-retryablehttp | 网络/DB 重试+退避，服务韧性 |
| **toml** | BurntSushi/toml | 现代配置格式三件套（yaml/toml/pxml）补齐 |
| **datetime** | time 扩展 | 工作日/日期差/ISO8601/相对时间（`time_format` 已有基础，缺运算） |
| **passhash** | bcrypt / argon2 | 密码存储；先做 PBKDF2-SHA256+salt（native 有），预留换算法 |
| **secure_random** | crypto/rand | UUID v4/令牌/盐都需要密码学安全随机（`os_random_hex` 接近，做成库） |
| **datastruct** | container/list+heap+ring | deque/优先队列/LRU/set——写缓存、调度器都要 |
| **concurrent_map** | sync.Map | 并发安全容器（有 mutex，包一层） |
| **mailparse** | net/mail | smtp 能发不能解析；收件/归档场景需要 RFC 5322 解析 |
| **stats** | gonum/stat 子集 | 均值/方差/分位数/直方图——监控数据聚合适配单二进制定位 |

### T2 差异化 —— 契合 PuXian 定位（单二进制/边缘/AI）的杀手锏

| 库 | 理由 |
|---|---|
| **metrics** | Prometheus 客户端；`/metrics` 端点 + 单二进制，监控场景完美 |
| **tar** | archive/zip 有了，tar 补齐镜像/备份场景 |
| **fsnotify** | 文件监听（`fd_wait`/poll 可实现） |
| **big** | math/big 大整数；合约/哈希实验 |
| **pgclient / mysqlclient** | 大工程；注意官方 M150 已加 PG 的 MD5/SCRAM 认证原语——先做薄的或等官方 |
| **qrcode** | 纯语言 QR 生成，边缘/线下场景 |
| **中文生态** | 身份证/手机号校验、农历（已有 std.lunar）、cnnum——本地化差异化 |

## 3. 战略建议

1. **先做门槛库，别碰大件。** uuid/jwt/decimal/csv/cli/log/testkit/workerpool 每个都
   能在 <500 行内完成（官方 AI 友好规范），先让"写任何应用都要用"的库落地，语言才
   变得可用——这是生态的正反馈起点。
2. **照着 Go 的 API 心智抄。** 用户熟 Go 就用 Go 的命名与函数形态（`csv_parse`、
   `jwt_encode/decode`、`uuid_v4`……），迁移成本最低。PuXian 自己的 native 也已经
   这么命名了（`time_format` ≈ `time.Format`、`http_get` ≈ `http.Get`）。
3. **遵守写库规范十八条**（官方 ECOSYSTEM_GAPS §1）：纯函数优先、顶层不写状态、
   Result 错误、文件 <500 行、`px fmt` + `px lint` 0 错、双模式输出一致。registry
   流程已就绪（semver/pxpkg/px.pkg.lock），写完按
   `registry/<name>/<version>/<name>.px` 发布即可被 `pxpkg add` 拉取。
4. **每个库都是一次 dogfood。** 写库会暴露语言缺口（官方历史：M58 暴露
   MINI_SUBSET §十三 8 项，M66-M70 反向推动语言修复）。写库 = 同时帮语言成熟。

## 4. 语言面对写库的约束（实测/spec 确认）

- 并发语法（spawn/chan/select/mutex）需**编译模式**（`px build`）；解释器 `px run`
  是 Mini 子集，无并发 → 含并发的库标注"需编译模式"
- `let` 不可变，`var` 可变；`{}` 字面量是空块=null，空 dict 要另建
- 多行 list/dict/调用参数在括号内可用（M70/M119 已修）
- 模块顶层 `let/var` 自 M70 起可导出为全局状态槽（import 合并初始化），但库作者
  应保持初始化为纯值或惰性 init 函数
- 逻辑：Result 是唯一错误通道，无 try/throw（官方明确不做）