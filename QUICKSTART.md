# PuXian 安装与环境 QuickStart

> 本文档记录在 **openEuler 22.03 LTS（aarch64）** 上安装 PuXian 的实测过程：
> 遇到的问题、解决路径、经验教训与语言语法注意事项。
> 版本基线：**v0.2.0-m206**（2026-09-24 · 编译器已自举 · 三轨语义一致）。

## 1. 快速安装

### 1.1 x86_64 / RHEL 系（有 rpm 仓库）

```bash
curl -fsSL -o install-rpm.sh https://soft.xiusoft.cn/puxian/install-rpm.sh
sudo bash install-rpm.sh        # 写入仓库 + 导入 GPG 公钥（双验签）
sudo dnf install puxian         # RHEL9 / openEuler22.03 x86_64
sudo yum install puxian         # RHEL7 / CentOS7
```

### 1.2 aarch64（无 rpm 仓库 → 官方引导包）

```bash
# 1. 从 GitHub Releases 下载 aarch64 原生引导包（全静态、零 glibc 依赖）
TAG=v0.2.0-m206
curl -fL -O "https://github.com/NanzhanGroup/PuXian/releases/download/${TAG}/puxian-bootstrap-aarch64-${TAG}.tar.gz"

# 2. 校验 sha256（官方 sha256sums.txt 或镜像 version.json 均可对）
curl -fL -O "https://github.com/NanzhanGroup/PuXian/releases/download/${TAG}/puxian-bootstrap-aarch64-${TAG}.tar.gz.sha256"
sha256sum -c puxian-bootstrap-aarch64-${TAG}.tar.gz.sha256

# 3. 安装（布局与官方 install.sh 一致，便于以后升级）
sudo mkdir -p /usr/local/share/puxian/${TAG}
sudo tar -C /usr/local/share/puxian/${TAG} -xzf puxian-bootstrap-aarch64-${TAG}.tar.gz --strip-components=1
sudo ln -sf /usr/local/share/puxian/${TAG}/tools/px  /usr/local/bin/px
sudo ln -sf /usr/local/share/puxian/${TAG}/tools/pxc /usr/local/bin/pxc

# 4. 验证
px --version
```

> ⚠️ 官方 `tools/install.sh` 目前只支持 x86_64（aarch64 直接报错退出），
> aarch64 部署按包内 `README-aarch64.md` 手动安装即可（即上文第 3 步）。

## 2. 本次安装踩到的坑（问题 → 根因 → 解法）

| # | 现象 | 根因 | 解法 |
|---|------|------|------|
| 1 | `dnf install puxian` 装不上 | 本机是 aarch64，rpm 仓库（含 openEuler）目前**只有 x86_64** | aarch64 走官方引导包 `puxian-bootstrap-aarch64-<tag>.tar.gz` |
| 2 | 国内镜像 `soft.xiusoft.cn` 的 `/releases/` 目录 404 | 镜像只同步了源码 tarball，**没有同步 aarch64 引导包** | 从 GitHub Releases 下载引导包（网络通）；源码 tarball 仍可用镜像加速 |
| 3 | `px --version` 正常但编译报 VM 件不可执行 | aarch64 引导包**不含 VM 轨件**（`pxc_vm`/`pxi_vm` 由 x86_64 侧产出）；属官方设计 | 自动落 C 轨（提示信息即说明），产物语义与 VM 轨等价，无需处理；`--print-plan` 可核对编译计划 |
| 4 | 官方 `tools/install.sh` aarch64 直接 exit 1 | 脚本架构检测只放行 x86_64 | 手动安装：参照包内 `README-aarch64.md`，布局对齐官方（`/usr/local/share/puxian/<tag>` + bin 软链） |
| 5 | 引导包是"静态零依赖" | 旧版包（≤m167）的 pxc 是动态件，需 GLIBC_2.38；新版（≥M168）全静态 | 选 M168 之后的包；装完用 `file` / `ldd` 验证：`statically linked`、无 interpreter |

## 3. 经验教训

1. **先看官方 README 再动手**：页面（`soft.xiusoft.cn/puxian`）明确写了"非 x86_64 当前无 rpm 仓库……aarch64 用户请用 puxian-bootstrap-aarch64-<tag>.tar.gz"，照着文档走能少走一半弯路。
2. **下载后必验 sha256**：发布包有两条校验链——GitHub Release 的 `.sha256` 资产 + 镜像 `version.json` 的 `tarball_sha256` 字段，两条都能对上才装。
3. **rpm 仓库的 gpgkey/repomd 双验签是给维护者看的**：作为用户侧，更实际的检查是 `file` 产物确认为 `static` + `ldd` 无 `not found`。
4. **编译时提示"VM 版编译器在本机不可执行"不是错误**：是 aarch64 官方通道的设计（VM 轨件仅 x86_64 侧产出），工具自动回退 C 轨，别被提示吓到。
5. **三轨语义一致是项目红线**：解释（`px run`）/ VM 字节码（默认 `px build`）/ C 文本轨（`px build --c`）行为一致，若发现分叉按官方口径报 issue（带最小复现单文件）。
6. **验收不要只看 `--version`**：写 hello / fib / Result / 并发示例跑 `px run` + `px build` + 产物直跑，再扫一遍工具链子命令，才算环境可用（见 §5）。

## 4. 语法注意事项（写 .px 前先读）

- **`match` 是表达式**（有返回值），不是语句；分支用 `case <值>:` + 缩进块，`case _` 兜底：

  ```python
  let desc = match x:
      case 0:
          "zero"
      case _:
          "nonzero"
  ```

- **`Result` 惯用法不是模式匹配**：用 `Ok(x)` / `Err(e)` 构造 + `?` 传播 + `!` 强制解包 + `is_ok()/is_err()/unwrap()`：

  ```python
  def safe_div(a, b):
      if b == 0:
          return Err("division by zero")
      return Ok(a // b)

  def calc(x):
      y = safe_div(10, x)?      # Err 则立即返回 Err
      return Ok(y + 1)
  ```

- **管道 `|>` 优先级低于比较运算符**：`msg |> to_upper() != "HELLO"` 会解析错，比较时加括号：

  ```python
  if (msg |> to_upper()) != "HELLO":
      ...
  ```

- **没有 Python 式三元 `a if cond else b`**（会报语法错误 E2001），用 `match` 或 if/else。
- **字符串插值**用 `${expr}`；切片支持步长 `a[i:j:k]`（str 按 UTF-8 字符）。
- **解释器是 Mini 子集**：`px run` 不支持 channel（并发代码报 R1002）——并发示例要用 `px build` 编译后跑。
- 三轨统一规则（M159–M181）值得留意：求值顺序词法左→右、迭代期间修改被迭代容器报 R1003、解包严格 R1002、`d[int]` 索引 R1002（键须为字符串）、算术要数值/比较要同型等。
- 编译失败先 `px build --print-plan` 归因（引擎轨 / 引用裁剪 / CC / 目标架构），再对症处理。

## 5. 快速验收清单（环境装好后跑一遍）

```bash
px --version                                    # 版本
px run  hello.px                                # 解释轨
px build hello.px && ./build/hello              # 编译轨 + 产物直跑
file build/hello                                # statically linked
ldd build/hello || true                         # 无 not found / not a dynamic executable
px fmt hello.px --check                         # 格式化
px lint hello.px --strict                       # 静态检查
px test hello.px                                # 测试框架
px mcp                                          # MCP 服务器（stdio，8 工具）
```

## 6. 相关链接

- 官方仓库：<https://github.com/NanzhanGroup/PuXian>
- 国内镜像（rpm 仓库 + 发布 tarball）：<https://soft.xiusoft.cn/puxian>
- 生态库本仓库：<https://github.com/banshanhanfu/registry-px>

---

License: Apache-2.0（与 PuXian 一致）