# flutter-ohos-app-template

> Minimal Flutter app for OpenHarmony, used to test [flutter-ohos-builder](https://github.com/zhongdaiqi/flutter-ohos-builder) end to end.

Use this template to start a new Flutter + HarmonyOS project with CI/CD pre-configured.

---

# 中文说明 / Chinese Documentation

本仓库为 OpenHarmony 的 Flutter 模板，并内置 CI（使用 `zhongdaiqi/flutter-ohos-builder`）。
以下内容为中文说明，包含如何配置仓库 Secrets 以通过 CI 进行签名构建。

## 必填仓库 Secrets（仅支持 OHOS_SIGN_* 命名）
请在仓库 Settings → Secrets and variables → Actions 中按精确名称添加下列 Secrets：

- `OHOS_SIGN_ALG`
  - 示例值（推荐）：`SHA256withECDSA`
- `OHOS_SIGN_CERT_BASE64`        # .cer 文件，single-line base64
- `OHOS_SIGN_KEY_ALIAS`
- `OHOS_SIGN_KEY_PASSWORD`
- `OHOS_SIGN_PROFILE_BASE64`     # .p7b 文件，single-line base64
- `OHOS_SIGN_STORE_FILE_BASE64`  # .p12 keystore，single-line base64
- `OHOS_SIGN_STORE_PASSWORD`
- `OHOS_SIGN_MATERIAL_BASE64`    # DevEco 生成的 material/ 目录的 zip，single-line base64

重要提示：仓库不再支持旧的 `SIGN_*` 命名，请统一使用 `OHOS_SIGN_*`。

## Secrets 与 build-profile.json5 的实际对应关系

这些 GitHub Actions Secrets 最终会被注入到 `ohos/build-profile.json5` 中的 `signingConfigs` 节点，用于告诉 hvigor / DevEco 运行签名时如何使用证书、keystore、profile 和 material。

对应结构大致如下：

```json5
{
  "signingConfigs": [
    {
      "name": "default",
      "type": "HarmonyOS",
      "material": {
        "certpath": "/tmp/ohos-sign/app.cer",
        "keyAlias": "YOUR_KEY_ALIAS",
        "keyPassword": "YOUR_KEY_PASSWORD",
        "profile": "/tmp/ohos-sign/app.p7b",
        "signAlg": "SHA256withECDSA",
        "storeFile": "/tmp/ohos-sign/app.p12",
        "storePassword": "YOUR_STORE_PASSWORD"
      }
    }
  ]
}
```

也就是说，每个 Secret 实际会映射到以下字段：

- `OHOS_SIGN_ALG`
  - 对应：`signingConfigs[].material.signAlg`
  - 例子：`SHA256withECDSA`

- `OHOS_SIGN_CERT_BASE64`
  - 对应：`signingConfigs[].material.certpath`
  - 具体作用：脚本会把它解码为 `/tmp/ohos-sign/app.cer`，然后写入 `certpath`

- `OHOS_SIGN_KEY_ALIAS`
  - 对应：`signingConfigs[].material.keyAlias`

- `OHOS_SIGN_KEY_PASSWORD`
  - 对应：`signingConfigs[].material.keyPassword`

- `OHOS_SIGN_PROFILE_BASE64`
  - 对应：`signingConfigs[].material.profile`
  - 具体作用：脚本会把它解码为 `/tmp/ohos-sign/app.p7b`，然后写入 `profile`

- `OHOS_SIGN_STORE_FILE_BASE64`
  - 对应：`signingConfigs[].material.storeFile`
  - 具体作用：脚本会把它解码为 `/tmp/ohos-sign/app.p12`，然后写入 `storeFile`

- `OHOS_SIGN_STORE_PASSWORD`
  - 对应：`signingConfigs[].material.storePassword`

- `OHOS_SIGN_MATERIAL_BASE64`
  - 不是直接写入 `build-profile.json5` 的一个字段
  - 而是：被脚本解码为 `material.zip`，随后 `unzip` 到 `material/` 目录，供 hvigor 在签名阶段读取
  - 它是配套签名材料的一部分，帮助解密存储在 keystore 中的加密密码和签名依赖文件

补充说明（最重要）：
- 这几个值不是“随便写字符串”，它们最终都要对应到 `material` 结构中的具体字段。
- `material` 目录的作用不是展示在 JSON 里某个字段，而是“提供签名执行时所需的解密材料/目录”。
- 如果 `OHOS_SIGN_MATERIAL_BASE64` 缺失，CI 会在签名阶段提示 `sign material is missing`，这是因为 hvigor 运行时找不到它所依赖的 `material/` 目录。

## 为什么需要 `OHOS_SIGN_MATERIAL_BASE64`

DevEco/hvigor 在生成并加密 keystore (.p12) 时，会在 .p12 同级生成一个 `material/` 目录，里面包含若干二进制 blob，hvigor 在解密 key/store 密码时需要读取该目录。CI 需要你把该目录打包成 zip 并做 single-line base64，然后放入 `OHOS_SIGN_MATERIAL_BASE64`。否则签名步骤会失败并报 "sign material is missing"。

## Secrets 的来源说明（华为后台 / 本地生成）

以下配置来自两个典型来源：

1) 华为开发者后台 / AppGallery Connect / 发布签名侧
2) 本地 DevEco Studio / 生成 keystore 的机器

### 1) 来自华为开发者后台 / AppGallery Connect / 发布管理
#### OHOS_SIGN_CERT_BASE64
- 典型来源：华为开发者后台 → 应用 / 项目 → 证书管理 / 签名证书 / 发布证书
- 常见获取方式：在应用的发布或签名配置页面中，下载 `.cer` 证书文件

#### OHOS_SIGN_PROFILE_BASE64
- 典型来源：华为开发者后台 → 应用 / 发布配置 / 配置文件 / Profile / 签名配置
- 常见获取方式：从发布签名配置页面导出或下载 `.p7b` / Profile 配置文件

#### OHOS_SIGN_ALG
- 典型来源：证书或签名配置页面中声明的签名算法
- 常见值举例：`SHA256withECDSA`（若证书为 ECDSA）或 `SHA256withRSA`（若为 RSA）

### 2) 来自本地 DevEco Studio / 生成 keystore 的机器
#### OHOS_SIGN_STORE_FILE_BASE64
- 典型来源：本地生成的 `.p12` / `.pfx` keystore 文件，由 DevEco Studio 或证书管理员导出

#### OHOS_SIGN_KEY_ALIAS
- 典型来源：生成 `.p12` 时设置的 alias，记录于生成者处或 DevEco 的签名配置里

#### OHOS_SIGN_KEY_PASSWORD
- 典型来源：alias 的密码，由生成 keystore 的人设定并安全传递

#### OHOS_SIGN_STORE_PASSWORD
- 典型来源：`.p12` keystore 的密码，由生成 keystore 的人设定并安全传递

#### OHOS_SIGN_MATERIAL_BASE64
- 典型来源：生成 `.p12` 的机器上与 `.p12` 同目录下的 `material/` 目录（由 DevEco/hvigor 生成）
- 获取方式：必须在生成 `.p12` 的本地环境打包 `material/` 并以受控方式提供

### 谁提供这些值
- 证书管理员 / 发布负责人：负责提供 `OHOS_SIGN_CERT_BASE64`、`OHOS_SIGN_PROFILE_BASE64`、`OHOS_SIGN_ALG`
- Keystore 生成者 / 签名工程师：负责提供 `.p12`、alias、密码、material
- CI / DevOps：负责将这些值写入仓库 Actions Secrets，并验证 CI 构建

### 安全建议
- `.p12`、`material/`、密码等为高敏感内容，禁止通过公共聊天/邮件明文传输。
- 建议由生成者直接在仓库 Secrets 中添加或通过公司受控存储与短期凭证供 CI 下载。
- 不在日志中打印密码或 keystore 内容。

## 生成并验证 base64 的示例命令

下面提供 Linux / macOS / Windows 三个平台的示例命令（仅用于本地生成 single-line base64 与验证）。

### 1) 单个文件生成 single-line base64

Linux:

```bash
base64 -w0 app.cer > app.cer.base64
base64 -w0 app.p7b > app.p7b.base64
base64 -w0 app.p12 > app.p12.base64
```

macOS:

```bash
base64 app.cer | tr -d '\n' > app.cer.base64
base64 app.p7b | tr -d '\n' > app.p7b.base64
base64 app.p12 | tr -d '\n' > app.p12.base64
```

Windows (PowerShell)：

```powershell
# 生成 single-line base64 文件（示例：app.p12 -> app.p12.base64）
[Convert]::ToBase64String([IO.File]::ReadAllBytes('app.p12')) | Out-File -Encoding ASCII app.p12.base64

# 另一种：使用 certutil 来解码/编码
certutil -encodehex app.p12 app.p12.base64 0x40000000  # 0x40000000 表示不分页输出（single-line）
# 注意：certutil 的行为在不同 Windows 版本略有差异，推荐使用 PowerShell 的 Convert 方法来保证 single-line
```

### 2) material/ 目录打包并生成 base64

Linux / macOS:

```bash
zip -r material.zip material/
base64 -w0 material.zip > material.zip.base64  # Linux
# macOS:
# base64 material.zip | tr -d '\n' > material.zip.base64
```

Windows (PowerShell)：

```powershell
# 在 material 的父目录运行，确保 zip 中包含顶层 material/ 目录
Compress-Archive -Path .\material -DestinationPath material.zip
# 生成 single-line base64
[Convert]::ToBase64String([IO.File]::ReadAllBytes('material.zip')) | Out-File -Encoding ASCII material.zip.base64
```

### 3) 本地解码与验证（查看文件类型 / 列表）

Linux / macOS:

```bash
# 解码
base64 -d app.p12.base64 > tmp_app.p12
# 查看文件类型
file tmp_app.p12
# 如果安装了 openssl，可以查看证书信息（不会导出私钥）
openssl pkcs12 -in tmp_app.p12 -nokeys -clcerts -passin pass:YOUR_STORE_PASSWORD -nodes -info

# material.zip 验证
base64 -d material.zip.base64 > tmp_material.zip
unzip -l tmp_material.zip
```

Windows (PowerShell / certutil):

```powershell
# 使用 PowerShell 解码 base64 到文件
$base64 = Get-Content -Raw app.p12.base64
[IO.File]::WriteAllBytes('tmp_app.p12', [Convert]::FromBase64String($base64))
Get-Item tmp_app.p12 | Select-Object Name,Length

# 或使用 certutil 解码
certutil -decode app.p12.base64 tmp_app.p12

# 若安装了 OpenSSL（例如通过 Git for Windows / WSL），可以查看证书信息：
# openssl pkcs12 -in tmp_app.p12 -nokeys -clcerts -passin pass:YOUR_STORE_PASSWORD -nodes -info

# material.zip 验证
$base64 = Get-Content -Raw material.zip.base64
[IO.File]::WriteAllBytes('tmp_material.zip', [Convert]::FromBase64String($base64))
Expand-Archive -LiteralPath tmp_material.zip -DestinationPath tmp_material_dir -Force
Get-ChildItem -Recurse tmp_material_dir | Select-Object FullName,Length
```

输出校验要点：
- `tmp_app.p12` 的 `file` / `Get-Item` 类型应该显示为 PKCS#12 / binary 文件，而不是 Zip。若显示为 Zip，说明你可能把 material.zip 的 base64 放错位置到 p12 的 secret。
- `tmp_material.zip` 解压后应包含顶层 `material/` 目录，并且目录下有若干文件（例如 `material/fd/0`、`material/ac` 等）。

---

## Usage

1. Click **Use this template** to create a new repo
2. The included workflow will automatically build your app on every push
3. Built `.app` artifacts are available in the Actions tab

## Structure

```
lib/main.dart          # Counter app entry point
ohos/                  # OpenHarmony platform config
.github/workflows/     # CI/CD using flutter-ohos-builder
```

## Local Development

Requires [DevEco Studio](https://developer.huawei.com/consumer/cn/deveco-studio/) and the
[Flutter OHOS fork](https://gitcode.com/openharmony-sig/flutter_flutter).

```bash
flutter create --platforms ohos .
flutter run -d <device-id>
```

## License

MIT
