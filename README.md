# flutter-ohos-app-template

> Minimal Flutter app for OpenHarmony, used to test [flutter-ohos-builder](https://github.com/zhongdaiqi/flutter-ohos-builder) end to end.

Use this template to start a new Flutter + HarmonyOS project with CI/CD pre-configured.

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
