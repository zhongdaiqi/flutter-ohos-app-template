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
- `OHOS_SIGN_CERT_BASE64`        # .cer 文件，single-line base64
- `OHOS_SIGN_KEY_ALIAS`
- `OHOS_SIGN_KEY_PASSWORD`
- `OHOS_SIGN_PROFILE_BASE64`     # .p7b 文件，single-line base64
- `OHOS_SIGN_STORE_FILE_BASE64`  # .p12 keystore，single-line base64
- `OHOS_SIGN_STORE_PASSWORD`
- `OHOS_SIGN_MATERIAL_BASE64`    # DevEco 生成的 material/ 目录�� zip，single-line base64

重要提示：仓库不再支持旧的 `SIGN_*` 命名，请统一使用 `OHOS_SIGN_*`。

为什么需要 `OHOS_SIGN_MATERIAL_BASE64`

DevEco/hvigor 在生成并加密 keystore (.p12) 时，会在 .p12 同级生成一个 `material/` 目录，里面包含若干二进制 blob，hvigor 在解密 key/store 密码时需要读取该目录。CI 需要你把该目录打包成 zip 并做 single-line base64，然后放入 `OHOS_SIGN_MATERIAL_BASE64`。否则签名步骤会失败并报 "sign material is missing"。

## 生成并验证 base64 的示例命令

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

注意：将产生的 `.base64` 文件内容（单行）复制到 GitHub secret 的值中。

### 2) material/ 目录打包并生成 base64

在包含 `material/` 的父目录运行：

```bash
zip -r material.zip material/
base64 -w0 material.zip > material.zip.base64  # Linux
# macOS:
# base64 material.zip | tr -d '\n' > material.zip.base64
```

验证：

```bash
base64 -d material.zip.base64 > tmp_material.zip
unzip -l tmp_material.zip
```

输出必须包含以 `material/` 为顶层目录的条目（例如 `material/fd/0`、`material/ac` 等），否则 CI 会报错。

## 常见故障与排查要点（中文）

- 错误：`ERROR: sign material is missing`
  - 排查：确认 `OHOS_SIGN_MATERIAL_BASE64` 是否存在于 Actions Secrets，且为 single-line base64；本地用 `unzip -l` 校验 zip 内容是否包含 `material/` 顶层目录。

- 错误：`Init keystore failed` 或 `toDerInputStream rejects tag type 80`
  - 排查：说明 `OHOS_SIGN_STORE_FILE_BASE64` 可能不是正确的 `.p12` 文件，或对应密码 `OHOS_SIGN_STORE_PASSWORD` 错误。请确认上传的是原始 `.p12` 的 base64 编码。

- Secret 放置位置错误：
  - 请确保 Secrets 放在仓库级别的 Actions Secrets（Settings → Secrets and variables → Actions），不要放到 Variables 或未审批的 Environment（未审批的 Environment 下，workflow 无法读取 secrets）。

- secret 值有换行或被截断：
  - 复制到 GitHub secret 时请确保粘贴为单行且没有被自动换行或截断。

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
