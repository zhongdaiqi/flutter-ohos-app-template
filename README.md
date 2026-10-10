# flutter-ohos-app-template

> Minimal Flutter app for OpenHarmony, used to test [flutter-ohos-builder](https://github.com/zhongdaiqi/flutter-ohos-builder) end to end.

Use this template to start a new Flutter + HarmonyOS project with CI/CD pre-configured.

## Usage

1. Click **Use this template** to create a new repo
2. The included workflow will automatically build your app on every push
3. Built `.app` artifacts are available in the Actions tab

## Required repository secrets

This repository now supports only the following GitHub Actions secret names. Please add them exactly as written in:

Settings → Secrets and variables → Actions

Required Secrets:

- `OHOS_SIGN_ALG`
- `OHOS_SIGN_CERT_BASE64`        # .cer file, single-line base64
- `OHOS_SIGN_KEY_ALIAS`
- `OHOS_SIGN_KEY_PASSWORD`
- `OHOS_SIGN_PROFILE_BASE64`     # .p7b file, single-line base64
- `OHOS_SIGN_STORE_FILE_BASE64`  # .p12 keystore, single-line base64
- `OHOS_SIGN_STORE_PASSWORD`
- `OHOS_SIGN_MATERIAL_BASE64`    # zip of the DevEco `material/` directory, single-line base64

Important: the older `SIGN_*` naming scheme is no longer supported by this repo. Use only the `OHOS_SIGN_*` names above.

## Why OHOS_SIGN_MATERIAL_BASE64 is required

DevEco/hvigor generates a companion `material/` directory next to the `.p12` keystore. This directory is required to decrypt the encrypted key/store passwords during signing. The CI workflow expects the `material/` directory to be packaged as a zip and supplied through `OHOS_SIGN_MATERIAL_BASE64`.

Without it, the build fails with errors such as:

- `ERROR: sign material is missing`
- `ENOENT ... /material`

## Generating secret values

### 1) Single-line base64 for a file

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

### 2) material/ directory zip + base64

From the directory that contains `material/`:

```bash
zip -r material.zip material/
base64 -w0 material.zip > material.zip.base64
```

On macOS:

```bash
zip -r material.zip material/
base64 material.zip | tr -d '\n' > material.zip.base64
```

Then validate the zip before putting it in the secret:

```bash
base64 -d material.zip.base64 > tmp_material.zip
unzip -l tmp_material.zip
```

You should see entries beginning with `material/`, for example:

- `material/`
- `material/fd/0`
- `material/ac`
- `material/ce`

If the zip does not contain a top-level `material/` directory, the CI build will fail.

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
