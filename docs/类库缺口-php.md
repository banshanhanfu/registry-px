# 类库缺口·对照 PHP 生态（2026-10-06）

> 以 PHP 8.x 标准库（~80 内置扩展）+ Composer 生态 Top 100 包为参照，
> 逐一比对 PuXian 现有 **140 个 registry 库** + **368 个 native 函数** + **13 个 stdlib 模块**。
> 已覆盖的（含 native / stdlib / registry）标 ✅，缺失的标 ⏳，需先补 native 的标 🚫。
>
> **PuXian 已有底座概览**：
> - native：json / xml / regex(PCRE) / http / sqlite / AES+RSA+Ed25519+SHA+HMAC / zip / zlib / gzip
>   / os / time / smtp / html / cookiejar / multipart / yaml / semver / lunar / url / path / strings
>   / collections / gfx+png / img(decode+jpeg+scale) / ws(WebSocket) / quic+h3 / route / onnx
> - registry：见 `registry/` 目录 122 库（全部 0.2.0）

---

## 1. 字符串处理（PHP ~60 个字符串函数）

| PHP 功能 | PHP 函数 / 扩展 | PuXian 现状 | 状态 |
|---|---|---|---|
| 正则匹配/替换/分割 | `preg_match` / `preg_replace` / `preg_split` | native regex (PCRE) | ✅ |
| HTML 转义/反转义 | `htmlspecialchars` / `html_entity_decode` | native html + htmlparse | ✅ |
| strip_tags | `strip_tags` | htmlparse 可提取纯文本 | ✅ |
| 大小写转换 | `strtolower` / `strtoupper` / `ucfirst` / `ucwords` | strcase | ✅ |
| slug 生成 | — | slug（⚠️ slugify 异名同功能重复，见末尾登记） | ✅ |
| 文本包裹 | `wordwrap` | textwrap（⚠️ textfmt 含 tf_wordwrap 部分重复，见末尾登记） | ✅ |
| 编辑距离 | `levenshtein` | edist | ✅ |
| 自然排序 | `natsort` | natsort | ✅ |
| 复数化/单数化 | — | inflect | ✅ |
| 千分位 | `number_format` | thousep + money | ✅ |
| C 格式化字符串 | `sprintf` / `printf` / `vsprintf` | sprintf | ✅ |
| 字符串填充 | `str_pad` | strpad（sp_pad / sp_pad_left / sp_pad_right / sp_pad_both） | ✅ |
| 字符串重复 | `str_repeat` | strpad（sp_repeat / sp_truncate / sp_zeropad） | ✅ |
| 翻译表替换 | `strtr` | textfmt（tf_strtr / tf_strtr_table） | ✅ |
| 语音算法 | `soundex` / `metaphone` | phonetic（ph_soundex / ph_metaphone / ph_nysiis） | ✅ |
| 相似度百分比 | `similar_text` | simtext（st_similar 递归 LCS） | ✅ |
| 换行转 br | `nl2br` | textfmt（tf_nl2br / tf_nl2br_xhtml） | ✅ |
| 分块分割 | `chunk_split` | textfmt（tf_chunk_split） | ✅ |
| quoted-printable | `quoted_printable_encode` / `decode` | textfmt（tf_qp_encode / tf_qp_decode） | ✅ |
| uuencode | `convert_uuencode` / `convert_uudecode` | uuencode（uu_encode / uu_decode） | ✅ |
| 字符编码转换 | `iconv` | **缺** — 无编码转换（GBK↔UTF-8 等） | 🚫 |
| 编码检测 | `mb_detect_encoding` | **缺** — 无编码自动检测 | 🚫 |
| C 类型检查 | `ctype_alpha` / `ctype_digit` … | ctype（ct_is_alpha / ct_is_digit / ct_is_alnum / ct_is_upper / ct_is_lower / ct_is_space / ct_is_punct / ct_is_xdigit） | ✅ |

## 2. 数组处理（PHP ~70 个数组函数）

| PHP 功能 | PHP 函数 | PuXian 现状 | 状态 |
|---|---|---|---|
| 排序 | `sort` / `asort` / `ksort` / `usort` | native sorted（无 key/comparator，PX-DEF-034） | ⚠️ |
| 集合运算 | `array_diff` / `array_intersect` / `array_merge` | set + itertools | ✅ |
| 计数/频率 | `array_count_values` | counter | ✅ |
| 分块 | `array_chunk` | itertools `it_chunk` | ✅ |
| 去重 | `array_unique` | itertools `it_unique` | ✅ |
| 翻转键值 | `array_flip` | arrutil（arr_flip） | ✅ |
| 提取列 | `array_column` | arrutil（arr_column） | ✅ |
| 组合键值 | `array_combine` | arrutil（arr_combine） | ✅ |
| 填充 | `array_fill` / `array_pad` | arrutil（arr_fill）；array_pad 缺 | ⚠️ |
| 随机选取 | `array_rand` | arrutil（arr_rand） | ✅ |
| 乘积 | `array_product` | arrutil（arr_product） | ✅ |
| 递归替换 | `array_replace_recursive` | deepmerge（dm_merge / dm_merge_many / dm_path_get / dm_path_set） | ✅ |
| 多维排序 | `array_multisort` | multisort（ms_sort / ms_sort_by） | ✅ |
| 打乱 | `shuffle` | arrutil（arr_shuffle_copy） | ✅ |
| 范围生成 | `range` | arrutil（arr_range） | ✅ |
| compact/extract | `compact` / `extract` | compact（cp_compact / cp_extract / cp_only / cp_except / cp_pluck / cp_flip） | ✅ |
| 回调遍历 | `array_map` / `array_filter` / `array_walk` | functools 部分；**无原生 array_map** | ⚠️ |

## 3. 数学（PHP Math + GMP + BCMath）

| PHP 功能 | PHP 函数 / 扩展 | PuXian 现状 | 状态 |
|---|---|---|---|
| 基础数学 | `abs` / `max` / `min` / `sqrt` / `pow` / `log` / `exp` | native math | ✅ |
| 三角函数 | `sin` / `cos` / `tan` / `asin` / `acos` / `atan2` | native math | ✅ |
| 双曲函数 | `sinh` / `cosh` / `tanh` | mathx（mx_sinh / mx_cosh / mx_tanh / mx_asinh / mx_acosh / mx_atanh） | ✅ |
| 取整 | `floor` / `ceil` / `round` | native | ✅ |
| 浮点取模 | `fmod` | mathx（mx_fmod） | ✅ |
| 角度弧度 | `deg2rad` / `rad2deg` | mathx（mx_deg2rad / mx_rad2deg）+ geo（geo_deg2rad / geo_rad2deg） | ✅ |
| 大整数 | GMP | big（加减乘除模幂） | ✅ |
| 任意精度小数 | BCMath | decimal | ✅ |
| 分数 / 复数 | — | fractions + plex | ✅ |
| 统计 | — | stats + statx + dist + metrics（⚠️ stats 与 statx 的 mean/median/var/stddev 重复，见末尾登记） | ✅ |
| 随机 | `mt_rand` / `random_int` / `random_bytes` | secure_random + dist | ✅ |
| 数学常数 | `M_PI` / `M_E` / `M_SQRT2` … | mathx（MX_PI / MX_E / MX_SQRT2 / MX_GOLDEN 等 15 个） | ✅ |
| 进制转换 | `base_convert` | numconv（nc_base_convert） | ✅ |

## 4. 日期与时间（PHP Date/Time + Calendar）

| PHP 功能 | PHP 函数 | PuXian 现状 | 状态 |
|---|---|---|---|
| 格式化/解析 | `date` / `strtotime` / `date_create` | native time + datetime（⚠️ dateutil 异名同功能重复，见末尾登记） | ✅ |
| 时区 | `DateTimeZone` | tzmini（18 都市） | ✅ |
| 间隔 | `date_diff` / `DateInterval` | datetime 有部分 | ✅ |
| 周期迭代 | `DatePeriod` | dateperiod（dp_new / dp_to_list / dp_count / dp_contains / dp_format） | ✅ |
| 自然语言解析 | `strtotime("next Thursday")` | strtotime（st_parse / st_parse_relative / st_day_of_week / st_next_weekday） | ✅ |
| 日历 / 中国农历 / 节假日 / 调度 | — | datetime + lunar + holidays + sched | ✅ |

## 5. 文件与目录（PHP Filesystem + SPL）

| PHP 功能 | PHP 函数 | PuXian 现状 | 状态 |
|---|---|---|---|
| 读写 / 存在 / 路径 / 遍历 / 通配 / 复制 / 临时 / 监控 | — | native + walk + glob + shutil + tempfile + fsnotify | ✅ |
| 文件锁 | `flock` | **缺** — 需 native | 🚫 |
| 权限操作 | `chmod` / `chown` / `chgrp` | **缺** — 需 native | 🚫 |
| touch / 符号链接 / 磁盘空间 | `touch` / `symlink` / `disk_free_space` | **缺** — 需 native | 🚫 |
| 文件信息对象 | `SplFileInfo` | fileinfo（fi_new / fi_get_filename / fi_get_extension / fi_get_basename / fi_get_dirname） | ✅ |

## 6. 网络与协议（PHP Network + cURL + Stream）

| PHP 功能 | PHP 函数 / 扩展 | PuXian 现状 | 状态 |
|---|---|---|---|
| HTTP / HTTP3 / WebSocket / DNS / FTP / POP3 / IMAP / SMTP | — | native http + quic + ws + dns + ftp + pop3 + imap + smtp | ✅ |
| Redis / MySQL / PostgreSQL / MongoDB / MQTT / OAuth2 | — | registry 各库 | ✅ |
| URL 解析 / 百分号编码 | `parse_url` / `urlencode` | native url + percent | ✅ |
| SSH / SFTP | `ssh2_*` | **缺** — 需 native 密码学 | 🚫 |
| gRPC | — | **缺** — 需 HTTP/2（protobuf 有，HTTP/2 缺） | ⏳ |
| SOAP | `SoapClient` / `SoapServer` | soap（soap_build_envelope / soap_build_request / soap_extract_fault） | ✅ |
| XML-RPC | `xmlrpc_*` | xmlrpc（xr_build_request / xr_build_response / xr_build_fault / xr_encode_value） | ✅ |
| LDAP | `ldap_*` | ldap（ldap_ber_encode + ldap_bind_request + ldap_search_request + ldap_parse_dn） | ✅ |
| SNMP | `snmp_*` | snmp（snmp_ber_encode + snmp_build_get_request + snmp_version_name） | ✅ |
| NTP / Whois / Telnet | — | ntp + whois（协议构造/解析；Telnet 需 socket） | ✅ |
| ICMP Ping | — | netprobe TCP 探测；**ICMP raw 缺** | 🚫 |

## 7. 数据库（PHP Database Extensions）

| PHP 功能 | PHP 扩展 | PuXian 现状 | 状态 |
|---|---|---|---|
| MySQL / PostgreSQL / SQLite / MongoDB / Redis | — | mysql + pg + native sqlite + mongodb + redis | ✅ |
| SQL 解析 | — | sqlparse | ✅ |
| 数据库抽象层 | `PDO`（统一接口+预编译+事务） | dbal（qb_new / qb_to_sql / qb_insert_sql / qb_update_sql / qb_delete_sql） | ✅ |
| 查询构建器 | — (Illuminate/Database) | querybuilder | ✅ |
| 迁移工具 | — (Phinx / Doctrine) | migration（mig_add / mig_get_pending / mig_plan_up / mig_plan_rollback） | ✅ |
| ORM | — (Eloquent / Doctrine) | orm（orm_new / orm_field / orm_to_row / orm_from_row / orm_to_json / orm_validate） | ✅ |
| 连接池 | — | connpool（cp_new / cp_acquire / cp_release / cp_available） | ✅ |

## 8. 图像处理（PHP GD + Imagick + EXIF）

| PHP 功能 | PHP 函数 / 扩展 | PuXian 现状 | 状态 |
|---|---|---|---|
| 解码 / JPEG 编码 / 缩放 / PNG | — | native img + png + gfx | ✅ |
| 条形码 / 二维码 / PDF | — | barcode + qrcode + pdf | ✅ |
| EXIF 元数据 | `exif_read_data` | exif（exif_has_marker / exif_find_segment / exif_parse_tiff / exif_tag_name） | ✅ |
| 裁剪 / 旋转 / 滤镜 / 水印 / TTF 文字 | `imagecrop` / `imagerotate` … | **缺** — 需 native GD | 🚫 |
| GIF 动画 / WebP / AVIF | — | **缺** — 需 native | 🚫 |
| 颜色空间操作 | `imagecolorallocate` … | colorutil（hex↔RGB）；HSL/HSV 缺 | ⚠️ |

## 9. 密码与安全（PHP Crypto + Sodium + OpenSSL）

| PHP 功能 | PHP 函数 / 扩展 | PuXian 现状 | 状态 |
|---|---|---|---|
| AES 对称加密 | `openssl_encrypt` / `decrypt` | native aes (ECB + GCM) | ✅ |
| RSA 非对称 | `openssl_public_encrypt` … | native rsa_encrypt / decrypt / sign / verify | ✅ |
| Ed25519 | — (sodium) | native ed25519 | ✅ |
| SHA / HMAC / PBKDF2 / Bcrypt | — | native + passhash + bcrypt | ✅ |
| JWT / OAuth2 / TOTP / 证书 | — | jwt + oauth2 + totp + x509 | ✅ |
| 随机 ID / 密码强度 / 验证码 / 校验码 | — | uuid + nanoid + ulid + snowflake + strength + pwgen + captcha + luhn + checksum（⚠️ hashutil 包含 checksum 的 CRC32/Adler32，见末尾登记） | ✅ |
| libsodium 现代加密 | `sodium_crypto_secretbox` / `box` / `sign` | **缺** — 无 libsodium 封装 | 🚫 |
| DES / 3DES / RC4 | `openssl_encrypt("DES")` | descrypt（RC4 完整：rc4_crypt / rc4_encrypt_hex；DES 框架） | ✅ |
| 恒定时间比较 | `hash_equals` | **缺** — 需 native | 🚫 |
| 自签证书生成 | `openssl_csr_new` / `openssl_sign` | **缺** — 需 native X.509 生成 | 🚫 |
| AES-CBC / CTR 模式 | `openssl_encrypt("AES-256-CBC")` | aescbc（aes_cbc_encrypt/decrypt + aes_ctr_crypt，基于 native AES-ECB） | ✅ |

## 10. 压缩与归档（PHP Compression）

| PHP 功能 | PHP 扩展 | PuXian 现状 | 状态 |
|---|---|---|---|
| zlib / gzip / ZIP / tar / LZ4 | — | native + zlib + tar + targz + lz4 | ✅ |
| bzip2 | `bzcompress` / `bzdecompress` | **缺** — 无 BZ2 压缩 | 🚫 |
| Zstandard / Brotli / xz / Snappy | — | **缺** — 需 native | 🚫 |
| Phar 归档 | `Phar` | phar（phar_new / phar_add_file / phar_build_manifest / phar_parse_manifest） | ✅ |

## 11. 数据格式与序列化（PHP Encoding）

| PHP 功能 | PHP 函数 / 扩展 | PuXian 现状 | 状态 |
|---|---|---|---|
| JSON / XML / YAML / Base64 / Hex | — | native | ✅ |
| CSV / INI / TOML / Properties / MsgPack / BSON / Protobuf | — | registry 各库（⚠️ csv 与 csvutil 异名同功能重复，见末尾登记） | ✅ |
| HTML 解析 / Markdown / JSONPath / Base58 / Punycode / XLSX / PDF | — | registry 各库 | ✅ |
| PHP 序列化 | `serialize` / `unserialize` | phpser | ✅ |
| var_export | `var_export` | vardump（vd_export） | ✅ |
| DOM 全 API | `DOMDocument` / `DOMNode` | domlite（dom_new / dom_append / dom_find_by_tag / dom_to_xml / dom_inner_text） | ✅ |
| 流式 XML | `XMLReader` / `XMLWriter` | xmlstream（xw_start_element / xw_end_element / xw_write_element + xr_read_next） | ✅ |
| XSLT / XPath | `XSLTProcessor` / `DOMXPath` | xpath（xp_select_all / xp_select_by_attr / xp_text / xp_attr / xp_exists） | ✅ |
| vCard / iCalendar / RSS / Atom | — | vcard + rssbuilder；iCalendar 缺 | ⚠️ |
| JSON Schema | — (json-schema) | jsonschema | ✅ |
| Avro / Thrift / Cap'n Proto / Pickle | — | **缺** — 无跨语言序列化 | 🚫 |

## 12. 国际化与本地化（PHP i18n）

| PHP 功能 | PHP 函数 / 扩展 | PuXian 现状 | 状态 |
|---|---|---|---|
| 中文拼音 / 简繁 / 分词 / 数字大写 / 英文数字 | — | pinyin + hant + seg + cnnum + num2words（⚠️ numconv 的 nc_dec2words 与 num2words 轻微重复，见末尾登记） | ✅ |
| 节假日 / 身份证 / 假数据 | — | holidays + idcard + faker | ✅ |
| gettext 翻译 | `gettext` / `ngettext` / `_()` | gettext（gt_parse_po / gt_build_po / gt_get_translation） | ✅ |
| ICU Intl | `IntlDateFormatter` / `NumberFormatter` / `Collator` | **缺** — 无 ICU 格式化/排序 | 🚫 |
| CLDR 复数规则 / Locale 管理 | `setlocale` / `localeconv` | cldr（cldr_plural / cldr_locale_new / cldr_format_number） | ✅ |
| 货币转换 | — | currency（cur_new / cur_convert / cur_format / cur_add_rate） | ✅ |

## 13. 进程与系统（PHP Process + pcntl + posix）

| PHP 功能 | PHP 函数 / 扩展 | PuXian 现状 | 状态 |
|---|---|---|---|
| 执行外部命令 / 环境变量 | `exec` / `shell_exec` / `getenv` | native | ✅ |
| 进程信息 / fork / 信号 / POSIX | `getmypid` / `pcntl_fork` / `pcntl_signal` / `posix_*` | **缺** — 需 native | 🚫 |
| proc_open 管道 / 共享内存 | `proc_open` / `shmop_*` | **缺** — 需 native | 🚫 |

## 14. 错误与调试（PHP Error / Debug）

| PHP 功能 | PHP 函数 | PuXian 现状 | 状态 |
|---|---|---|---|
| 日志 | `error_log` / `syslog` | log | ✅ |
| 调试输出 | `var_dump` / `print_r` | vardump（vd_dump / vd_export / vd_print_r / vd_repr） | ✅ |
| 调用栈 | `debug_backtrace` | **缺** — 无调用栈追踪 | 🚫 |
| 错误处理器 / 错误级别 | `set_error_handler` / `error_reporting` | errorhandler（eh_new / eh_set_handler / eh_trigger / eh_last_error） | ✅ |

## 15. 输出与交互（PHP Output + CLI）

| PHP 功能 | PHP 函数 | PuXian 现状 | 状态 |
|---|---|---|---|
| ANSI / 表格 / 进度条 / 模板 / CLI 参数 | — | ansi + table + progress + template + cli | ✅ |
| 输出缓冲 | `ob_start` / `ob_get_clean` | outbuf（ob_new / ob_start / ob_write / ob_get_clean / ob_end_flush） | ✅ |
| 交互式输入 / TUI 全屏 | `readline` / PsySH | **缺** — 需 tty raw | ⏳ |

## 16. 缓存 / Session / 邮件

| PHP 功能 | PHP 扩展 | PuXian 现状 | 状态 |
|---|---|---|---|
| LRU / TTL / 布隆 / 限流 / Redis | — | cache + bloom + rate + redis | ✅ |
| Cookie 管理 | `setcookie` / `$_COOKIE` | native cookiejar | ✅ |
| 邮件解析 / POP3 / IMAP / SMTP | — | mailparse + pop3 + imap + native smtp | ✅ |
| Memcached | `Memcached` | memcached（mc_build_set / mc_build_get / mc_build_delete / mc_parse_response） | ✅ |
| APCu 本地缓存 | `apcu_*` | **缺** — 需 native | 🚫 |
| Session 管理 | `session_start` / `$_SESSION` | session（ss_start / ss_get / ss_set / ss_destroy / ss_gc / ss_regenerate） | ✅ |
| CSRF 令牌 | — | csrf（csrf_generate / csrf_verify 恒定时间比较 / HTML 标签生成） | ✅ |
| MIME 邮件构造 | — (PHPMailer) | mimebuilder | ✅ |

## 17. 并发与异步（PHP Parallel + Event + Fiber）

| PHP 功能 | PHP 扩展 | PuXian 现状 | 状态 |
|---|---|---|---|
| Actor / WorkerPool / 信号量 / 调度 | — | actor + workerpool + semaphore + sched | ✅ |
| Promise / Future | — (ReactPHP / Amp) | promise（pr_new / pr_then / pr_catch / pr_all / pr_race） | ✅ |
| 事件循环 | `Event` / `Ev` | eventloop（el_new / el_tick / el_run / el_add_task / el_add_timer） | ✅ |
| Channel (CSP) / 协程 Fiber | `Fiber` (PHP 8.1+) | channel（ch_new / ch_send / ch_recv / ch_try_send / ch_try_recv / ch_close） | ✅ |
| 并行线程 | `parallel` | **缺** — 需 native | 🚫 |

## 18. 测试与开发

| PHP 功能 | PHP 工具 | PuXian 现状 | 状态 |
|---|---|---|---|
| 单测 / Mock / 基准 / 属性测试 / 假数据 | PHPUnit / Mockery / Faker | testkit + tdtest + mock + bench + quickcheck + faker | ✅ |
| 代码覆盖率 | xdebug / pcov | **缺** — 需 native | 🚫 |
| 快照测试 | — | snapshot（snap_new / snap_match / snap_diff / snap_update / snap_serialize） | ✅ |

## 19. 数据结构与算法（PHP SPL）

| PHP 功能 | PHP SPL 类 | PuXian 现状 | 状态 |
|---|---|---|---|
| 栈 / 队列 / 堆 / LRU / Set / Counter / Trie / Bloom / Bitset / 图 / 区间 | — | datastruct + set + counter + trie + bloom + bitset + graph + interval | ✅ |
| 优先队列 | `SplPriorityQueue` | datastruct 有 heap；**封装缺** | ⚠️ |
| 双向链表 / 定长数组 | `SplDoublyLinkedList` / `SplFixedArray` | splstruct（dll_new/push/pop + fa_new/set/get/size） | ✅ |
| 跳表 / B 树 / 并查集 / 后缀数组 / 红黑树 / AVL | — | advstruct（并查集 uf_new/union/find + 跳表 sk_new/insert/search/delete） | ✅ |

## 20. 财务与地理（PHP 领域）

| PHP 功能 | PHP 包 / 扩展 | PuXian 现状 | 状态 |
|---|---|---|---|
| 货币格式化 / 百分比 / 单位换算 | — | money + percent + units | ✅ |
| 摊销 / 利率 / 税务 | — | finance（fin_pmt / fin_pv / fin_fv / fin_amortize / fin_simple_interest / fin_compound_interest） | ✅ |
| GeoIP / 大圆距离 / GeoJSON / 坐标转换 | `geoip_*` | geo（geo_haversine / geo_bearing / geo_wgs84_to_gcj02）；GeoIP 库/GeoJSON 缺 | ⚠️ |

## 21. Web 安全与爬虫

| PHP 功能 | PHP 函数 / 包 | PuXian 现状 | 状态 |
|---|---|---|---|
| 路由 / 中间件 | — | native route + middleware | ✅ |
| 验证器 | — (Respect/Validation) | validator | ✅ |
| HTML 清洗 | — (HTML Purifier) | htmlsanitizer | ✅ |
| 爬虫 / 抓取 | — (Goutte / Spatie) | scraper（sc_extract_links / sc_extract_text / sc_extract_title / sc_strip_tags） | ✅ |
| Filter 变量过滤 | `filter_var` / `filter_input` | validator 部分覆盖；**filter_var 全族缺** | ⚠️ |

---

## 重复类库登记（异名同功能）

> 2026-10-06 排查：registry 中存在异名但功能重复（或大幅重叠）的类库对。
> 建议后续合并：保留功能更全/版本更高的库，将另一库标记 deprecated 或合并差异函数。
> 2026-10-06 更新：4 对完全/大幅重复已实际合并（slugify→slug、csvutil→csv、dateutil→datetime、checksum→hashutil），3 对部分重叠保留两者。

| 重复对 | 重叠程度 | 保留建议 | 差异说明 |
|--------|---------|---------|---------|
| **slug ↔ slugify** | 完全重复 | ✅ 已合并入 **slug** | slugify 独有函数（sl_strip_accents / sl_transliterate / sl_slugify_unicode / sl_custom_map / sl_slugify / sl_truncate_slug / sl_make_unique）已迁入 slug 0.2.0，slugify 库已删除 |
| **csv ↔ csvutil** | 大幅重叠 | ✅ 已合并入 **csv** | csvutil 独有函数（csv_escape_field / csv_unescape_field / csv_sort / csv_unique / csv_build / csv_build_header / csv_filter / csv_column）已迁入 csv 0.2.0，csvutil 库已删除 |
| **datetime ↔ dateutil** | 大幅重叠 | ✅ 已合并入 **datetime** | dateutil 全部 du_ 函数已迁入 datetime 0.2.0（保留 du_ 前缀兼容），dateutil 库已删除 |
| **checksum ↔ hashutil** | hashutil ⊃ checksum | ✅ 已合并入 **hashutil** | checksum 独有函数（crc16 / crc16_hex / fletcher32 / checksum_verify / hex_digest / crc32_hex）已迁入 hashutil，checksum 库已删除 |
| **textwrap ↔ textfmt** | 部分重叠 | 两者保留（定位不同） | textwrap 专注换行排版（wrap/fill/dedent/indent）；textfmt 是 PHP 文本格式化全家桶（nl2br/chunk_split/strtr/QP/wordwrap/strip_tags/addslashes）。仅 tf_wordwrap 与 tw_wrap/tw_fill 有功能交叉，建议 textfmt 文档注明"如需纯换行优先用 textwrap" |
| **stats ↔ statx** | 部分重叠 | 两者保留（定位不同） | stats 是基础描述统计（min/max/sum/count/mean/median/var/stddev）；statx 是增强统计（quantile/pearson/linreg + Err 返回风格）。基础函数重复但 API 风格不同，建议 statx 文档注明"基础统计可用 stats" |
| **numconv ↔ num2words** | 轻微重叠 | 两者保留（定位不同） | numconv 主打进制转换+罗马数字，nc_dec2words 是简易英文数字单词；num2words 是完整英文数字转单词（含连字符/and/千分位）。建议 numconv 文档注明"完整英文数字单词用 num2words" |

## 汇总

| 类别 | ✅ 已覆盖 | ⏳ 缺失（纯 .px 可行） | 🚫 需先补 native |
|---|---|---|---|
| 字符串 | 10 | 11 | 2 |
| 数组 | 5 | 10 | 0 |
| 数学 | 10 | 4 | 0 |
| 日期时间 | 7 | 2 | 0 |
| 文件目录 | 7 | 1 | 3 |
| 网络协议 | 12 | 7 | 2 |
| 数据库 | 6 | 4 | 0 |
| 图像 | 5 | 2 | 3 |
| 密码安全 | 10 | 3 | 3 |
| 压缩归档 | 5 | 1 | 3 |
| 数据格式 | 13 | 8 | 2 |
| 国际化 | 5 | 4 | 1 |
| 进程系统 | 2 | 0 | 3 |
| 错误调试 | 1 | 3 | 1 |
| 输出交互 | 5 | 2 | 0 |
| 缓存/Session/邮件 | 7 | 4 | 1 |
| 并发异步 | 4 | 3 | 1 |
| 测试 | 5 | 1 | 1 |
| 数据结构 | 11 | 3 | 0 |
| 财务地理 | 3 | 4 | 0 |
| Web 安全 | 3 | 2 | 0 |
| **合计** | **117** | **79** | **26** |

## 建议优先级（纯 .px 可行的 79 项）

### P0 — 高频通用，几十至几百行可成
1. **sprintf** — C 格式化字符串（`%d %s %.2f %x`），PHP/Go/Python 都有，使用频率极高
2. **str_pad / str_repeat** — 字符串填充/重复，极简但高频
3. **array_flip / array_column / array_combine** — dict 操作三件套
4. **range** — 步长序列生成，`range(1,10,2)`
5. **shuffle** — Fisher-Yates 洗牌
6. **soundex / metaphone** — 语音编码，英文模糊匹配
7. **similar_text** — 递归 LCS 相似度
8. **nl2br / chunk_split** — 文本格式化小件
9. **quoted_printable** — QP 编解码（邮件 MIME）
10. **base_convert** — 任意进制转换

### P1 — 中等复杂度，协议/格式类
11. **PDO 抽象层** — 统一 mysql/pg/sqlite 接口
12. **查询构建器** — 链式 SQL 构造
13. **Memcached 客户端** — TCP + 文本协议
14. **vCard / iCalendar** — 文本格式解析
15. **RSS / Atom Feed** — XML 之上
16. **JSON Schema** — JSON 结构验证
17. **gettext (.mo/.po)** — 翻译文件解析
18. **PHP serialize** — PHP 序列化格式
19. **DOM API** — xml_parse 之上建完整 DOM
20. **XPath** — XML 路径查询

### P2 — 大件或需设计决策
21. **SOAP / XML-RPC** — 协议栈
22. **gRPC** — 需 HTTP/2
23. **ORM** — 对象关系映射
24. **Promise / Event Loop** — 异步模型
25. **爬虫框架** — HTTP + 解析 + 调度
26. **HTML 清洗** — XSS 防护
27. **Session 管理** — 状态管理
28. **MIME 邮件构造** — 多部分邮件
29. **GeoIP / 坐标转换** — 地理计算
30. **财务计算** — 摊销/利率/税务

---

## 实施记录

- 2026-10-06：建档（对照 PHP 8.x 标准库 + Composer 生态 Top 100，逐项比对 122 registry + 368 native + 13 stdlib）。
- 2026-10-06：排查异名同功能重复类库，发现 7 对（slug/slugify、csv/csvutil、datetime/dateutil、checksum/hashutil、textwrap/textfmt、stats/statx、numconv/num2words），已在各节标注 ⚠️ 并新增「重复类库登记」章节。registry 库数更新为 144。
- 2026-10-06：合并 4 对完全/大幅重复库（slugify→slug、csvutil→csv、dateutil→datetime、checksum→hashutil），独有函数迁入保留库，双模式测试 PASS。registry 库数 144→140。
