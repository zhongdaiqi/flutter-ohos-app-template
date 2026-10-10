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

为什么需要 `OHOS_SIGN_MATERIAL_BASE64`

DevEco/hvigor 在生成并加密 keystore (.p12) 时，会在 .p12 同级生成一个 `material/` 目录，里面包含若干二进制 blob，hvigor 在解密 key/store 密码时需要读取该目录。CI 需要你把该目录打包成 zip 并做 single-line base64，然后放入 `OHOS_SIGN_MATERIAL_BASE64`。否则签名步骤会失败并报 "sign material is missing"。

## 生成并验证 base64 的示例命令

（本节已在 README 中保留，示例步骤用于生成 single-line base64 和验证。你可以参考已有内容。）

## Secrets 的来源说明（华为后台 / 本地生成）

以下配置来自两个典型来源：

1) 华为开发者后台 / AppGallery Connect / 发布签名侧
2) 本地 DevEco Studio / 生成 keystore 的机器

下面按每个 Secret 说明其典型来源与获取位置。

### 1) 来自华为开发者后台 / AppGallery Connect / 发布管理
#### OHOS_SIGN_CERT_BASE64
- 典型来源：华为开发者后台 → 应用 / 项目 → 证书管理 / 签名证书 / 发布证书
- 常见获取方式：在应用的发布或签名配置页面中，下载 `.cer` 证书文件

#### OHOS_SIGN_PROFILE_BASE64
- 典型来源：华为开发者后台 → 应用 / 发布配置 / 配置文件 / Profile / 签名配置
- 常见获取方式：从发布签名配置页面导出或下载 `.p7b` / Profile 配置文件

#### OHOS_SIGN_ALG
- 典型来源：证书或签名配置页面中声明的签名算法
- 常见值举例（示例）：`SHA256withECDSA`（若证书为 ECDSA）或 `SHA256withRSA`（若为 RSA）

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
- 获取方式：必须在生成 `.p12` 的本地环境打包 `material/` 并以受控方式提供（推荐生成���直接在仓库 Secrets 中添加该值或使用内部安全存储与短期凭证）

### 谁提供这些值
- 证书管理员 / 发布负责人：负责提供 `OHOS_SIGN_CERT_BASE64`、`OHOS_SIGN_PROFILE_BASE64`、`OHOS_SIGN_ALG`
- Keystore 生成者 / 签名工程师：负责提供 `.p12`、alias、密码、material
- CI / DevOps：负责将这些值写入仓库 Actions Secrets，并验证 CI 构建

### 安全建议
- `.p12`、`material/`、密码等为高敏感内容，禁止通过公共聊天/邮件明文传输。
- 建议由生成者直接在仓库 Secrets 中添加或使用公司受控存储与短期凭证供 CI 下载。
- 不在日志中打印密码或 keystore 内容。

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
