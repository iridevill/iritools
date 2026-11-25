# iritool

**A Windows toolkit for understanding and tuning your PC, with safety, reversibility, and clear limits built in.**

iritool combines live hardware telemetry, a configurable desktop overlay, system inventory, startup management, and reversible gaming profiles in one WPF desktop app. It is an independent community project and is not affiliated with Microsoft or hardware vendors.

## What it can do

- **Dashboard and hardware monitor:** live CPU, GPU, memory and storage readings where the hardware provider exposes them; history charts and an explicit capability report when a sensor is missing.
- **Desktop overlay:** configurable metrics, compact or vertical layout, opacity, scale, four anchors, drag-edit mode, click-through mode, and optional Gaming Mode behavior. Missing sensor data is shown as `—`. It does not inject into games, draw over exclusive-fullscreen applications, or provide an FPS counter.
- **System inventory:** inspect Windows, devices, drivers, storage and installed software through read-only probes.
- **Startup manager:** inspect startup entries and manage supported entries with previews and reversible, journaled operations.
- **Gaming Mode:** apply supported, reversible power and per-game preferences, with applicability checks and recovery after an interrupted session.
- **Benchmark and stability checks:** cancellable CPU render workload (10 s) or stability run (30 s) with battery acknowledgment and thermal telemetry safety gates. Each run reports score, sample consistency, start→peak temperature and first-third/last-third throttle, and the last 20 runs are stored in `%LOCALAPPDATA%\Iritool\benchmarks.json`. Scores measure this workload only and are not directly comparable with third-party benchmark suites.
- **Settings:** persistent appearance, monitoring and overlay preferences with normalization for older or invalid settings.

Sensor availability varies by machine, vendor, permissions and driver. The app reports unsupported or stale readings instead of inventing values. System-changing actions require a preview or confirmation and record enough information to support restoration; do not apply changes you do not understand.

## Requirements

- Windows 10 version 1809 (build 17763) or newer, or Windows 11; x64 only.
- No separate .NET Desktop Runtime: the released build is a self-contained single file that bundles the .NET 10 runtime and the WPF assemblies. Only building from source needs the .NET 10 SDK.
- Some hardware sensors and system operations may need administrator access. iritool does not silently elevate itself.

## Install

1. Download `iritool-Setup-1.0-x64.exe` from the releases page and run it.
2. Setup is per user and never asks for administrator rights. It installs into `%LOCALAPPDATA%\Programs\iritool`, adds a Start Menu shortcut, an optional desktop shortcut and an Add/Remove Programs entry, and upgrades an existing 0.5.x install in place.
3. Prefer no installation? `iritool-0.5.4-win-x64.zip` contains the same build as a portable single file; run `iritool.exe` from any folder.

Uninstall from Settings → Apps, or run `uninstall.exe --quiet` in the install folder.

The builds are not code signed, so Windows SmartScreen may show "Windows protected your PC" the first time; choose **More info → Run anyway**. `SHA256SUMS.txt` in the release lists the checksum of every download; verify one with `Get-FileHash <file> -Algorithm SHA256`.

## Troubleshooting a failed start

A crash log is written to `%LOCALAPPDATA%\iritool\logs\crash.log`. WPF reports a markup failure only as the outermost message (for example `Provide value on 'System.Windows.Baml2006.TypeConverterMarkupExtension' threw an exception`); the log and the in-app error dialog show the nested exception that names the missing resource, file or type.

## Build and verify

Install the .NET 10 SDK, then from the solution directory:

```powershell
dotnet build .\iritool.slnx -c Release
dotnet test .\iritool.slnx -c Release
```

The app is under `iritool/`; automated tests are under `iritool.Tests/`. Tests use fakes for system-changing integrations; they do not validate every real device or Windows configuration.

## Contributing

Contributions are welcome, including bug reports, accessibility improvements, translations, hardware-provider fixes, documentation, tests, and carefully scoped features. This is a volunteer project; contributions are unpaid and made under the repository license.

1. Open an issue describing the problem, Windows version, hardware/provider, reproduction steps, and expected behavior. Remove serial numbers, usernames, paths, and other personal data from logs/screenshots.
2. For a code change, fork the repository and create a focused branch. Keep system-changing behavior opt-in, previewable, reversible, and covered by fake-backed tests.
3. Build and run the test suite. Include the exact commands and results in your pull request; mention any manual checks that could not be performed.
4. Do not submit bundled drivers, proprietary SDKs, secrets, telemetry dumps, or unrelated formatting changes. Avoid adding dependencies without explaining their maintenance and licensing impact.
5. Describe accessibility impact (keyboard, screen reader, contrast, scaling) and hardware limitations in the pull request.

To coordinate or ask about contributing, contact **[@iridevill on Telegram](https://t.me/iridevill)**.

## Project status

This project is under active development. Check `NOTES.md` and the issue tracker for known limitations and hardware-specific validation gaps. The UI and services should not be treated as a guarantee that every Windows tweak improves performance; benefits depend on the system and workload.

## Acknowledgments
iritool was built with the help of two AI coding agents: **Freebuff** and **Claude Code**.

## فارسی

**iritool** یک ابزار ویندوزی برای مشاهده وضعیت سخت‌افزار، تنظیمات قابل بازگشت و پروفایل بازی است. امکانات آن شامل مانیتور سخت‌افزار، اورلی قابل تنظیم، گزارش اطلاعات سیستم، مدیریت برنامه‌های شروع به کار، Gaming Mode و آزمون CPU است. اطلاعات سنسوری که در دسترس نباشد با `—` نمایش داده می‌شود؛ اورلی در بازی تزریق نمی‌کند و FPS نشان نمی‌دهد.

برای مشارکت داوطلبانه می‌توانید باگ، پیشنهاد دسترس‌پذیری، ترجمه، اصلاح پشتیبانی سخت‌افزار، مستندات، تست یا قابلیت محدود و قابل بازگشت ارسال کنید. ابتدا مسئله را با نسخه ویندوز، سخت‌افزار و مراحل بازتولید توضیح دهید؛ سپس تغییر کوچک و مشخصی بسازید، تست‌ها را اجرا کنید و نتیجه را در Pull Request بنویسید. این مشارکت‌ها داوطلبانه و بدون پرداخت هستند. اطلاعات شخصی را از لاگ‌ها و تصویرها حذف کنید.

**نیازمندی‌ها:** ویندوز ۱۰ نسخه ۱۸۰۹ (build 17763) یا جدیدتر، یا ویندوز ۱۱، فقط ۶۴ بیتی. نسخه‌ای که با ستاپ نصب می‌شود self-contained است و به نصب جداگانه .NET 10 نیاز ندارد؛ فقط ساختن از سورس به .NET 10 SDK نیاز دارد. اگر برنامه هنگام اجرا با خطای XAML متوقف شد، جزئیات کامل (شامل استثنای درونی) در `%LOCALAPPDATA%\iritool\logs\crash.log` نوشته می‌شود.

برای هماهنگی یا پرسش درباره مشارکت، در تلگرام با **[@iridevill](https://t.me/iridevill)** در تماس باشید.
