<!-- fa -->

# نقشه کامل پروژه — iritool / iritool

> این فایل یک «مپ» مو‌به‌مو از کل محتویات مخزن است: هر پوشه، هر فایل، نقش آن، ارتباطش با بقیه و
> مسیرهایی که در زمان اجرا لمس می‌کند. مبنای این نقشه، خواندن مستقیم سورس و شمارش خطوط در تاریخ
> نگارش آن است. اگر فایلی اضافه/حذف شد، همین فایل باید به‌روز شود.

---

## 1. پروژه در یک نگاه

| مورد | مقدار |
|---|---|
| نام محصول/خروجی | `iritool` (نسخه `1.0`) |
| نام پروژه/سولوشن/نیم‌اسپیس | `iritool` / `iritool` |
| نوع برنامه | اپلیکیشن دسکتاپ ویندوز (WPF) |
| فریم‌ورک | `net10.0-windows`، `PlatformTarget = x64` |
| معماری | MVVM (CommunityToolkit.Mvvm) با Composition Root دستی در `App.xaml.cs` |
| تم | فقط Dark (زغالی/مشکی + بنفش)، WPF-UI 4.3.0 |
| تعداد بخش‌های منو | ۹ بخش |
| تست‌ها | پروژه xUnit جدا (`iritool.Tests`) |
| نصب | Inno Setup، per-user، بدون نیاز به ادمین |

پکیج‌های NuGet (فقط ۴ مورد در اپ + ۳ مورد تست):

| پکیج | نسخه | کاربرد |
|---|---|---|
| `WPF-UI` | 4.3.0 | پنجره Fluent، تایتل‌بار، کنترل‌های Fluent، دیکشنری تم |
| `CommunityToolkit.Mvvm` | 8.4.2 | `ObservableObject`، `[ObservableProperty]`، `[RelayCommand]` |
| `LibreHardwareMonitorLib` | 0.9.6 | خواندن سنسورهای CPU/GPU/RAM/دیسک |
| `System.Management` | 10.0.2 | کوئری‌های WMI برای System Information |
| `xunit` / `xunit.runner.visualstudio` / `Microsoft.NET.Test.Sdk` | 2.9.3 / 2.8.2 / 18.12.0 | فقط در پروژه تست |

---

## 2. درخت کامل مخزن

```
D:\iritool\                 ← ریشه مخزن (git)
├─ README.md                                 معرفی محصول (EN + بخش فارسی)
├─ NOTES.md                                  تاریخچه فازبه‌فاز توسعه و تصمیم‌ها
├─ LICENSE                                   MIT، Copyright (c) 2026 iridevill
├─ .gitignore                                bin/obj/.vs/.zwork و …
├─ icon.ico / icon.png                       آیکون نصب‌کننده (منبع Inno Setup)
├─ docs/
│  └─ architecture/
│     └─ PROJECT_GRAPH.md                    گراف Mermaid از سیم‌کشی سرویس‌ها
├─ installer/
│  ├─ build-installer.ps1                    پایپ‌لاین ساخت فایل Setup
│  ├─ iritool-installer/                     اپ WPF نصب‌کننده (رابط کاربری جدید)
│  ├─ iritool-uninstaller/                   حذف‌کننده کوچک per-user
│  ├─ iritool.iss / scripts/iritool.iss      مسیر قدیمی Inno Setup (باقی‌مانده)
│  └─ assets/*.bmp                           آرت قبلی Inno (استفاده نمی‌شود)
├─ .freebuff/                                متادیتای ابزار (project-id)
├─ .zwork/                                   فضای کار موقت/تست/انتشار (git-ignore)
├─ ui-ux-pro-max-skill-2.15.0/               اسکیل طراحی UI/UX (مرجع، جزو اپ نیست)
└─ iritool/                  پوشه سولوشن
   ├─ iritool.slnx           سولوشن (اپ + تست)
   ├─ .vs/                                   کش ویژوال استودیو (ignore)
   ├─ .zwork/                                خروجی obj/publish اختصاصی اپ (ignore)
   ├─ iritool/               پروژه WPF  ← قلب برنامه
   └─ iritool.Tests/         پروژه تست xUnit
```

نکته مهم درباره تودرتویی: مسیر واقعی پروژه اپ
`iritool/iritool/` است و تست‌ها در
`iritool/iritool.Tests/` کنار آن قرار دارند.

---

## 3. جریان اجرای برنامه (از استارت تا صفحه)

```
iritool.exe
   │
   ▼
App.OnStartup (App.xaml.cs)
   ├─ ElevationService.WasRestartRequested / WasStartedFromAutostart  ← پارس آرگومان‌ها
   ├─ ElevationService.TryAcquireInstance(mutex)  ── ناموفق؟ → ActivateRunningInstance() → Shutdown(0)
   ├─ هندلرهای خطا: DispatcherUnhandledException / AppDomain / UnobservedTask / SessionEnding
   ├─ SettingsService.LoadAsync()  →  AppSettings (اگر خطا: پیش‌فرض‌ها)
   ├─ ThemeService.Apply(theme, accent)  →  Applied event (پالت در همان‌جا در Visual Tree اعمال می‌شود)
   ├─ LibreHardwareTelemetryProvider  →  HardwareCompatibilityService  →  HardwareTelemetryMonitor
   ├─ WindowsToolService(new CommandRunner(isElevated))
   ├─ RegistryAccessor + PowerPlanService + ProcessController
   ├─ ChangeSessionService(journal, executor).InitializeAsync()  ← بازیابی سشن‌های قبلی
   ├─ ChangePreviewService / WindowsInventoryProvider
   ├─ GamingModeService / GameDetectionService / AdministratorService / AutostartTaskService
   ├─ WindowsStartupEntryProvider(new AuthenticodeVerifier())
   ├─ MainViewModel(...)   ← همه سرویس‌ها اینجا تزریق می‌شوند
   ├─ MainWindow.Show()    ← شِل، DataContext = MainViewModel
   └─ HardwareTelemetryMonitor.Start()  ← اولین اسنپ‌شات بعد از وجود پنجره
          │
          └─ Updated → اولین اجرا؟ → HardwareCapabilityReportService.EnsureFirstRunAsync(...)

App.OnExit
   ├─ unsubscribe (theme, telemetry, SessionEnding)
   ├─ GameDetectionService.Stop()
   ├─ RestoreActiveSessions()          ← برگرداندن تغییرات OnExit با مهلت ۱۰ ثانیه
   ├─ Dispose: MainViewModel, monitor, settings, changes
   └─ ReleaseMutex + Dispose
```

نکته کلیدی: `App.xaml.cs` تنها «ریشه ترکیب» (composition root) است؛ هیچ سرویس‌ای در XAML یا
code-behind صفحه‌ها ساخته نمی‌شود. تغییر تم باعث بازسازی کل پنجره شِل می‌شود (`OnThemeApplied`)
چون WPF استایل‌های seal شده را با پالت جدید دوباره نمی‌خواند؛ ViewModel مشترک می‌ماند تا
وضعیت صفحه/انتخاب/تلمتری از دست نرود.

---

## 4. فایل‌های ریشه مخزن

| فایل | خط | نقش |
|---|---|---|
| `README.md` | 58 | معرفی محصول، فهرست قابلیت‌ها، محدودیت‌ها (بدون تزریق در بازی، بدون FPS، بدون رندر Fullscreen انحصاری)، پیش‌نیازها (ویندوز ۱۰/۱۱ x64)، دستور Build/Test، راهنمای مشارکت، لینک تلگرام `@iridevill`، و بخش فارسی. |
| `NOTES.md` | 944 | دفتر مهندسی پروژه. فاز ۱ تا فاز ۵ + هات‌فیکس‌ها. مرجع «چرا» هر تصمیم، نتایج بیلد/تست، محدودیت‌های سخت‌افزاری مشاهده‌شده و چک‌لیست‌های تأیید دستی. |
| `LICENSE` | — | MIT، دارنده: `iridevill`، سال ۲۰۲۶. |
| `.gitignore` | 20 | `/vs`، `**/bin/`، `**/obj/`، `*.user`، `*.suo`، `*.cache`، `*.log`، و `.zwork/` (فضای کار ابزار). |
| `icon.ico` / `icon.png` | — | آیکون نصب‌کننده/برند در ریشه؛ `installer/iritool.iss` با `SetupIconFile` به `icon.ico` اشاره می‌کند. |

### `docs/architecture/PROJECT_GRAPH.md`
گراف Mermaid از سیم‌کشی سرویس‌ها (App → Settings/Theme/Provider/Main/Gaming و …) به‌همراه بخش
«Overlay boundary»: اینکه اورلی به `HardwareTelemetryMonitor.Updated` سابسکرایب می‌شود، حلقه سنسور
دوم نمی‌سازد، `RegisterHotKey` فقط `Ctrl+Alt+O` را در اختیار دارد، حالت قفل از `WS_EX_TRANSPARENT`
استفاده می‌کند و رندر Fullscreen انحصاری/هوک بازی/FPS وجود ندارد.

### `installer/iritool.iss`
اسکریپت Inno Setup:
- متغیرها: `AppName=iritool`, `AppVersion=1.0`, `AppExeName=iritool.exe`,
  `PublishDir=..\iritool\iritool\.zwork\publish`
- نصب در `{localappdata}\Programs\iritool`، `PrivilegesRequired=lowest`، فقط `x64os`
- خروجی در `..\.zwork\installer` با نام `iritool-Setup-1.0-x64.exe`
- فشرده‌سازی `lzma2/ultra64`، حذف `*.pdb` از بسته، شورتکات Start Menu همیشگی و Desktop اختیاری
  (`Tasks: desktopicon`, به‌صورت پیش‌فرض unchecked)، اجرای اختیاری بعد از نصب.

### `.freebuff/`
فقط متادیتای ابزار است (`project-id` و `.gitignore` خودش). هیچ کد محصول در آن نیست.

### `.zwork/` (در ریشه)
فضای کار transient و git-ignore شده: `attachments/` (متن‌های پیست‌شده)، اسکریپت‌های PowerShell
برای تست‌های دستی UI Automation (`click.ps1`، `final-check.ps1`، `gaming-test.ps1`،
`hwnd-probe.ps1`، `dialog-probe.ps1`، `crash-test.ps1`، `consent-test.ps1`، `gaming-off.ps1`)،
`dotnet-test.log`، کپی محلی Inno Setup (`inno/` با `Default.isl` و Examples) که برای کامپایل
نصب‌کننده استفاده شده، و خروجی‌های `publish`/`installer`. برای اجرای برنامه لازم نیست.

### `iritool/iritool.slnx`
سولوشن نسخه XML جدید (`.slnx`) با دو پروژه:
```
<Project Path="iritool/iritool.csproj" />
<Project Path="iritool.Tests/iritool.Tests.csproj" />
```
یعنی `dotnet test` روی سولوشن، کل مجموعه تست را اجرا می‌کند (در فاز ۲ این‌طور نبود؛ فاز ۳ اضافه شد).

---

## 5. پروژه اپ — فایل‌های ریشه پروژه

| فایل | خط | نقش |
|---|---|---|
| `iritool.csproj` | 39 | `WinExe`، `net10.0-windows`، `UseWPF`، `x64`، `Nullable`، متادیتای محصول (`Product/AssemblyTitle=iritool`, `Version=1.0`)، `ApplicationManifest=Assets\app.manifest`، `ApplicationIcon=Assets\app.ico`، جاسازی `app.ico`+`app.png` با `<Resource>` (بدون کپی به خروجی)، ۴ PackageReference. |
| `AssemblyInfo.cs` | 10 | تنظیمات اسمبلی (تم‌های WPF، `ThemeInfo`). |
| `App.xaml` | 51 | `ShutdownMode=OnMainWindowClose`؛ merge دیکشنری‌ها: `ui:ThemesDictionary Theme="Dark"`، `ui:ControlsDictionary`، `Assets/Colors.xaml`، `Assets/Icons.xaml`، `Assets/Styles.xaml`؛ و ۹ `DataTemplate` که ViewModel → View را نگاشت می‌کنند (Dashboard، WindowsTools، GamingMode، HardwareMonitor، DesktopOverlay، Benchmark، StartupManager، SystemInformation، Settings). |
| `App.xaml.cs` | 458 | Composition root + single-instance mutex (`Local\Iritool.SingleInstance`) + فعال‌سازی نمونه در حال اجرا (شامل نام‌های legacy) + هندلر خطاهای سراسری + بارگذاری تنظیمات + بازسازی شِل هنگام تغییر تم + `RestartElevated` + `RestoreActiveSessions`. |
| `MainWindow.xaml` | 294 | شِل: `ui:FluentWindow` با `ExtendsContentIntoTitleBar`، پهنای ۱۳۴۰×۸۶۰، حداقل ۱۰۸۰×۷۲۰، `WindowCornerPreference=Round`، سه لایه Ellipse نور محیطی، `ui:TitleBar` ۴۴px، سایدبار ۲۵۲px (برند + `ItemsControl` ناوبری + فوتر نسخه/کلاپس)، هاست صفحه با انیمیشن ورود ۳۰۰ms، بنر بازیابی سشن‌ها. آیکون و لوگو با `pack://application:,,,/Assets/app.ico|app.png` از منابع جاسازی‌شده خوانده می‌شوند، نه از فایل کنار exe. |
| `MainWindow.xaml.cs` | 66 | تنها کد code-behind شِل: انیمیشن عرض سایدبار `252 ↔ 76` روی `IsSidebarExpanded` با `DoubleAnimation` ۲۲۰ms و `CubicEase`؛ سابسکرایب/آن‌سابسکرایب `PropertyChanged` در `DataContextChanged`. |
| `Assets/app.manifest` | 20 | `supportedOS` ویندوز ۱۰/۱۱، `dpiAware=true/pm` + `dpiAwareness=PerMonitorV2`، و `requestedExecutionLevel` = `asInvoker` (برنامه هرگز خودش elevation نمی‌خواهد). |
| `Assets/app.ico` / `Assets/app.png` | — | آیکون و لوگوی جاسازی‌شده در اسمبلی (`<Resource>`، خوانده‌شده با pack URI از `application`). |

---

## 6. پروژه اپ — پوشه‌ها

### 6.1 `Assets/` (تم و طراحی)

| فایل | خط | محتوا |
|---|---|---|
| `Colors.xaml` | 135 | ۶۲ کلید: پالت (backdrop `#0B0B12→#08080C`، کارت‌ها `rgba(255,255,255,0.06–0.09)`، استروک `rgba(255,255,255,0.10)`)، اکسنت `#8B5CF6` / `#6D28D9` / `#B983FF`، رنگ‌های معنایی سبز/کهربایی/قرمز، متن‌ها، فونت‌ها (`Segoe UI Variable Display/Text`، `Cascadia Mono`) و shadowها. |
| `Icons.xaml` | 50 | ۱۸ استریم‌جرومتری ۲۴×۲۴ (بدون فونت آیکون و بدون اموجی). کلید هر آیکون با `Icon.` شروع می‌شود و `IconKeyToGeometryConverter` آن را resolve می‌کند. |
| `Styles.xaml` | 482 | ۳۸ استایل/کلید: تایپوگرافی، کارت، chip، دکمه‌ها، `WT.NavButton` (حالت active با pill ۳px)، انیمیشن ورود صفحه، ردیف‌های تنظیمات و … . همه رنگ‌ها با `DynamicResource` تا تعویض پالت زنده کار کند. |
| `app.manifest` | 20 | توضیح در بخش ۵. |
| `app.ico`, `app.png` | — | منابع اجرایی آیکون/لوگو. |

### 6.2 `Models/` (مدل‌های دامنه — بدون وابستگی به WPF)

| فایل | خط | محتوا |
|---|---|---|
| `AppSection.cs` | 19 | enum نُه بخش: `Dashboard, WindowsTools, GamingMode, HardwareMonitor, DesktopOverlay, Benchmarks, StartupManager, SystemInformation, Settings`. شناسه پایدار برای ناوبری و میان‌برها. |
| `QuickTool.cs` | 11 | رکورد کاشی میان‌بر داشبورد (`Title`, `Description`, `IconKey`, `Target`). |
| `SystemFact.cs` | 8 | رکورد ساده `(Label, Value)` برای پنل «System overview». |
| `SystemSnapshot.cs` | 17 | محیط سیستم از .NET: توضیح OS، نام ماشین، معماری، نسخه runtime، کاربر، تعداد پردازنده منطقی، زمان گرفتن. |
| `HardwareMetric.cs` | 17 | بازمانده فاز ۱ (tile placeholder با `IsPlaceholder`). **در کد فعلی ارجاعی ندارد** و حذف نشده است. |

**`Models/Hardware/` — قلب مدل تلمتری**

| فایل | خط | محتوا |
|---|---|---|
| `SensorKind.cs` | 14 | enum نوع سنسور: `Load, Temperature, Clock, Power, Voltage, Memory, Throughput, Fan, …`. |
| `SensorAvailability.cs` | 30 | taxonomy: `Available, Partial, SensorUnavailable, DeviceUnavailable, AccessDenied, ReadFailed, DeviceNotDetected`. تمایز «سنسور نیست» از «صفر است» و از «نیاز به elevation دارد». |
| `SensorReading.cs` | 106 | یک خواندن: مقدار + واحد + availability + `Detail`. اگر available نباشد `Value` نال است و هرگز صفر جعلی ذخیره نمی‌شود؛ فرمت‌های نمایش (`Format`, `FormatPair`) هم اینجاست. |
| `SensorValueValidator.cs` | 40 | دروازه plausibility: load ۰–۱۰۰، دما `0 < T ≤ 150`، clock > 0، power > 0، voltage ۰–۵، memory/throughput ≥ 0؛ NaN/±Inf رد می‌شود. |
| `SensorSelection.cs` | 149 | انتخاب سنسور کانونی: اولویت نام‌ها (`CPU Total`, `CPU Package`, `GPU Core`…) → هر خواندن موجود → state ناموجود. `SelectMax` برای بالاترین کلاک هسته، `SelectExact` که اجازه نمی‌دهد حافظه اشتراکی iGPU به‌جای VRAM نمایش داده شود. |
| `SensorFailureClassifier.cs` | 33 | تبدیل استثنای خواندن سنسور به availability (مثلاً `AccessDenied`). |
| `HardwareAvailability.cs` | 96 | تجمیع availability یک گروه خواندن: همه ok → Available، بعضی → Partial، هیچ‌کدام → دلیل غالب. |
| `HardwareSnapshot.cs` | 44 | تصویر immutable یک لحظه: CPU، لیست GPU، Memory، Volumeها، StorageDeviceها، زمان، مدت خواندن. |
| `Telemetry.cs` | 141 | رکوردهای `CpuTelemetry` (load/temp/clock/power/voltage/cores/نام)، `GpuTelemetry` (با `Id` پایدار برای چنده‌گرافیکی و هیبریدی)، `MemoryTelemetry`، `VolumeTelemetry`، `StorageDeviceTelemetry`. |
| `HardwareCompatibilityReport.cs` | 35 | enum `HardwareCompatibilityLevel` (`Full, Partial, Limited, Unavailable`…) + رکورد گزارش سطح + یادداشت‌ها و محدودیت‌های مجوز. |
| `HardwareCapabilityReport.cs` | 73 | شِما‌ی فایل `capability-report.json`: `CapabilitySensor(Device, Measurement, State, Reason)`، فهرست sensors، features پشتیبانی‌شده/نشده، محدودیت‌های دسترسی و سخت‌افزاری، ماشین، نسخه اپ، وضعیت elevation، زمان اولین تولید. |
| `ChartSeries.cs` | 39 | `ChartMetric` (CpuLoad, CpuTemperature, GpuLoad, GpuTemperature, MemoryUsage, …) + `ChartSeries` با نقاط، بازه، واحد، state و متن «چرا خالی است». |

**`Models/Inventory/`**

| فایل | خط | محتوا |
|---|---|---|
| `InventoryFact.cs` | 110 | یک مقدار برچسب‌دار inventory؛ حالت «ناموجود» first-class است و `—` با دلیل در tooltip رندر می‌شود. `InventoryFormat` کمک‌کننده‌های قالب‌بندی (بایت/گیگابایت/تاریخ/آپ‌تایم/سرعت شبکه). |
| `SystemInventory.cs` | 89 | رکوردهای `InventoryEntry` (بخش‌ها با عنوان/زیرعنوان/آیکون/ردیف‌ها)، `VolumeInfo`، `DriverInfo` و `SystemInventory` (شامل بخش‌ها + حجم‌ها + درایورها + diagnostics). |

**`Models/Operations/`**

| فایل | خط | محتوا |
|---|---|---|
| `OperationDefinition.cs` | 116 | enum `OperationKind` (`ReadOnly, Diagnostic, Repair, AdvancedRepair` + extensionهای برچسب/توضیح)، `OperationDefinition` (Id، عنوان، فایل مطلق، آرگومان ثابت، kind، نیاز به elevation، timeout، عملیات تأییدکننده)، `OperationGroup`، `OperationOutcome` + extensionها، `OperationResult`. |
| `OperationCatalog.cs` | 223 | تنها کاتالوگ مجاز دستورات: `sfc.verifyonly`, `sfc.scannow`, `dism.checkhealth`, `dism.scanhealth`, `dism.restorehealth`, `chkdsk.scan` (به‌علاوه دستورهای مدیریت تسک autostart). مسیرها مطلق در System32 و آرگومان‌ها ثابت‌اند؛ UI نمی‌تواند خط فرمان دلخواه بسازد. |
| `ExitCodeInterpreter.cs` | 89 | ترجمه exit code به outcome و پیام قابل اقدام: SFC `0/1/2/3`، DISM `0/3010/0x800F081F/0x800F0906/87/5`، CHKDSK `0/1/2/3`. کد ناشناس عیناً گزارش می‌شود، نه «موفق». |
| `OperationHistoryEntry.cs` | 32 | یک رکورد تاریخچه اجرا (Id، OperationId، عنوان، شروع/مدت، خروج، خلاصه). |

**`Models/Safety/`**

| فایل | خط | محتوا |
|---|---|---|
| `ChangeRecord.cs` | 192 | کل مدل journal: `ChangeKind` (`Registry, PowerPlan, GameMode, ProcessPriority, GpuPreference, LaunchApp, …`)، `ChangeState` (`Pending, Applied, Reverted, Skipped, Failed`)، `ChangeRestoreMode` (`OnDisable, OnExit, Manual`)، `ChangeSessionState`، `ChangeRecord` (session/record id، owner/title، target، value name/format، prior existed/prior/requested، state، timestamps، process identity، نیاز به elevation؛ مقادیر باینری رجیستری Base64)، `ChangeSession`، `ChangeRevertResult`. |

**`Models/Settings/`**

| فایل | خط | محتوا |
|---|---|---|
| `SettingsEnums.cs` | 44 | `AppThemePreference` (فقط `Dark` رندر می‌شود)، `AccentPalette` (Purple/Blue/Teal/Amber/Rose)، `SettingsLoadResult`، `SettingsChangedEventArgs`. |
| `AppSettings.cs` | 155 | شِما `CurrentSchemaVersion = 4` با فیلدهای: Theme، Accent، `PollingIntervalMs` (پیش‌فرض ۱۰۰۰)، `StartMinimized`، `ConfirmSensitiveOperations`، `GamingModeAutoDetect`، `GameProfiles`، و کل تنظیمات اورلی (`OverlayEnabledMetrics`, `OverlayMetricSelectionInitialized`, `OverlayFollowGamingMode`, `OverlayVisibleOnLaunch`, `OverlayOpacity=0.94`, `OverlayScale=1.0`, `OverlayLayout=Compact`, `OverlayAnchor=TopRight`, `OverlayOffsetX/Y=18`). `Normalize()` همه را clamp/snap می‌کند (تم غیر Dark → Dark، پولینگ ۵۰۰/۱۰۰۰/۲۰۰۰/۵۰۰۰، شفافیت ۰.۳–۱.۰، مقیاس ۰.۷۵–۱.۵، anchor مجاز، پروفایل‌های نامعتبر حذف و Id تکراری بازتولید)، `Clone()` کپی عمیق پروفایل‌ها. |

**`Models/Gaming/`**

| فایل | خط | محتوا |
|---|---|---|
| `GameProfileSettings.cs` | 165 | پروفایل ذخیره‌شده بازی: Id، نام، مسیر اجرایی، فعال‌بودن tweakها (power plan، Game Mode flag، اولویت، GPU preference)، `Normalize()` برای اعتبارسنجی کامل؛ `GpuPreferenceString` ساخت/ادغام رشته `UserGpuPreferences` را انجام می‌دهد (توکن‌های دیگر حفظ می‌شوند). |
| `GamingGraphicsTopology.cs` | 27 | `IsHybrid(graphics)`: آیا سیستم هم آداپتر مجتمع و هم مجزا دارد؟ گیت فعال‌شدن tweak مربوط به GPU preference. |
| `GamingProcessClassifier.cs` | 18 | فهرست نام فرایندهای غیربازی (ابزارهای Steam، Wallpaper Engine و …) تا به‌عنوان «بازی» پیشنهاد نشوند. |

**`Models/Startup/`**

| فایل | خط | محتوا |
|---|---|---|
| `StartupEntry.cs` | 286 | enumهای `StartupScope` (User/Machine)، `StartupSource` (Run, Run32, StartupFolder, ScheduledTask)، `StartupSignatureStatus`؛ `StartupRawEntry`؛ `StartupEntry` نمایشی (فرمان، exe resolve شده، scope، publisher/امضا، enabled، isRunning، مسیر)؛ `StartupCommandParser` (پارس quoted/unquoted command line و env varها)؛ `StartupApproved` (ساخت/خواندن payload باینری `StartupApproved\Run`). |

### 6.3 `Services/` (منطق، I/O و دسترسی به سیستم)

**ریشه `Services/`**

| فایل | خط | نقش |
|---|---|---|
| `IAppInfoService.cs` | 14 | قرارداد نام محصول/نسخه/پیکربندی بیلد. |
| `AppInfoService.cs` | 53 | خواندن همان مقادیر از متادیتای اسمبلی (بدون hard-code نسخه در UI). |
| `INavigationService.cs` | 13 | `NavigateTo(AppSection)` برای صفحاتی که کاربر را جای دیگر می‌فرستند (میان‌برهای داشبورد). |
| `ISystemInfoProvider.cs` | 15 | تأمین `SystemSnapshot` برای داشبورد. |
| `SystemInfoProvider.cs` | 41 | فقط APIهای عمومی .NET (`RuntimeInformation`, `Environment`, `WindowsIdentity`). بدون WMI/رجیستری/سنسور. |
| `SectionIconCatalog.cs` | 32 | نگاشت `AppSection → Icon key`، منبع واحد برای سایدبار و هدر صفحات. |

**`Services/Hardware/`**

| فایل | خط | نقش |
|---|---|---|
| `IHardwareTelemetryProvider.cs` | 15 | خواندن یک‌باره تلمتری + `IsElevated`؛ تایمر در provider نیست. |
| `LibreHardwareTelemetryProvider.cs` | 194 | پیاده‌سازی روی LibreHardwareMonitorLib: ساخت تنبل `Computer` (حدود ۰.۹s، تلاش مجدد بعد از ۳۰s شکست)، سریال‌سازی خواندن‌ها با `SemaphoreSlim`، فعال‌سازی CPU/GPU/Memory/Storage، هر شکست → availability. |
| `HardwareSensorReader.cs` | 341 | ترجمه درخت LibreHardwareMonitor به مدل تلمتری: dedupe سنسورهای بی‌نام و تکراری `(kind,name)`، اعمال validator و selection، ساخت CPU/هر GPU/Memory/Storage. |
| `StorageVolumeReader.cs` | 110 | خواندن volumeهای fixed از OS (`DriveInfo`): همیشه در دسترس، بدون elevation؛ storage به تشخیص دیسک توسط کتابخانه وابسته نیست. |
| `HardwarePollingService.cs` | 239 | مالک cadence: `PeriodicTimer` با فاصله انتخابی (کف ۲۵۰ms)، بدون همپوشانی خواندن‌ها، شکست → اسنپ‌شات unavailable؛ نگهداشت ۱۵ دقیقه‌ای (`RetentionWindow`) با محاسبه ظرفیت بافر از interval (۱۲۰–۳۶۰۰ نمونه). |
| `HardwareSampleHistory.cs` | 98 | بافر حلقه‌ای thread-safe و immutable-محور؛ `Retarget` هنگام تغییر interval. |
| `HardwareTelemetryMonitor.cs` | 126 | نقطه ورود سراسری: provider + polling + compatibility، مارشال اسنپ‌شات روی UI thread با `DispatcherSynchronizationContext.Post`، رویداد `Updated(HardwareSnapshotEventArgs)`. |
| `HardwareCompatibilityService.cs` | 95 | ساخت `HardwareCompatibilityReport`: چه چیزی گزارش می‌شود، چه چیزی نه و چرا (داستان «نیاز به elevation»). |
| `ElevationDetector.cs` | 26 | آیا فرایند می‌تواند به رابط‌های نیازمند ادمین (MSR) دست بزند. |
| `ChartSeriesBuilder.cs` | 200 | ساخت سری‌ها از همان history (بدون سمپلر دوم): پنجره ۱/۵/۱۵ دقیقه، سنسور مسدود → شکاف نه صفر، محور دما auto-scale، decimation. |
| `HardwareCapabilityReportService.cs` | 292 | تولید `%LOCALAPPDATA%\Iritool\capability-report.json` از اولین اسنپ‌شات، نوشتن atomic + `.bak`، قرنطینه فایل خراب، «Regenerate» از آخرین نمونه با حفظ timestamp اصلی. |

**`Services/Inventory/`**

| فایل | خط | نقش |
|---|---|---|
| `ISystemInventoryProvider.cs` | 15 | قرارداد inventory یک‌باره و read-only؛ بدون elevation. |
| `WindowsInventoryProvider.cs` | 695 | پیاده‌سازی ویندوزی: WMI، رجیستری، `NetworkInterface`، `DriveInfo`. هر بخش ایزوله؛ کوئری درایور موازی اجرا می‌شود (کل اسنپ‌شات ۱.۳–۱.۶s). بخش‌ها: OS، Processor، Graphics (VRAM واقعی از `HardwareInformation.qwMemorySize`)، Memory (هر ماژول)، Motherboard/BIOS/UEFI، Storage (مدل، SSD/HDD، NVMe/SATA/USB، سریال، حروف volume)، Network (بدون آداپتر مجازی/NDIS filter)، Drivers (حداکثر ۴۰ ردیف با کد مشکل Device Manager). |
| `WmiQuery.cs` | 139 | پوشش دفاعی `ManagementObjectSearcher`: timeout، سقف ردیف، تبدیل خطا به نتیجه خالی + diagnostic (`WmiQueryException`). |
| `InventoryRow.cs` | 149 | نمای فقط‌خواندنی یک ردیف WMI با accessorهای تحمل‌کننده null و تبدیل نوع (`Text`, `Number`, `UInt64`…). |
| `InventoryParser.cs` | 531 | تبدیل ردیف‌های خام به `InventoryEntry`؛ بدون WMI و بدون نوع UI تا کاملاً unit-test شود (تاریخ/آپ‌تایم، هسته/کش، VRAM مجزا در برابر اشتراکی، چند GPU، ماژول حافظه، firmware، placeholderهای WMI مثل `(Standard disk drives)`, volumeها، قالب شبکه، مشکل درایور، جدول کدها). |

**`Services/Operations/`**

| فایل | خط | نقش |
|---|---|---|
| `CommandRunner.cs` | 456 | **تنها جایی که پروسه اجرا می‌شود.** `CommandRequest/CommandResult/IConfiguration`: مسیر مطلق، آرگومان ثابت، نیاز به elevation. حالت unelevated: pipe، خطوط زنده، exit code دقیق، کنسل → kill درخت پروسه. حالت elevated: `cmd.exe /c "... > log 2>&1"` با verb `runas` و tail لاگ؛ انتظار UAC خارج از UI thread؛ کنسل ارائه نمی‌شود (والد unelevated نمی‌تواند بکشد) و UI همین را می‌گوید. |
| `LogTailReader.cs` | 74 | خواندن افزایشی فایل لاگ دستور elevated با `FileShare.ReadWrite` و حفظ موقعیت خواندن. |
| `ConsoleTextDecoder.cs` | 215 | تشخیص انکودینگ خروجی redirect شده (UTF-16LE با BOM برای SFC/DISM، UTF-8، OEM code page) + `ConsoleLineBuffer` برای chunkهای نصفه و خطوط پیشرفت `\r`. |
| `IConfirmationService.cs` / `ConfirmationService.cs` | 14 / 38 | پرسش تأیید قبل از عملیات سیستمی؛ adapter WPF با owner پنجره فعال؛ اگر دیالوگ نمایش داده نشود پاسخ «خیر» است. |
| `WindowsToolService.cs` | 169 | ارکستراسیون: یک عملیات در هر لحظه، resolve خط فرمان ثابت، اجرا با runner، ترجمه exit با `ExitCodeInterpreter`، رد اجرای هم‌زمان/تکراری. |
| `OperationHistoryStore.cs` | 105 | `%LOCALAPPDATA%\Iritool\operation-history.json`، ۶۰ رکورد آخر، newest first، نوشتن atomic. |

**`Services/Safety/` (لایه تغییر سیستم)**

| فایل | خط | نقش |
|---|---|---|
| `ChangeSessionService.cs` | 448 | مالک journal و چرخه apply/restore. سشن با همه رکوردها در حالت `Pending` **قبل از** اولین تغییر روی دیسک نوشته می‌شود (write-ahead) و رکورد بعد از موفقیت executor به `Applied` می‌رود. Restore معکوس، خطا per-record، بررسی مقدار فعلی قبل از بازنویسی (تغییر خارجی → Skip با دلیل)، بازیابی `Pending` پس از crash. `RestoreAllActiveAsync(mode)` برای OnExit و SessionEnding. |
| `ChangeJournalStore.cs` | 144 | `change-journal.json` با schema 1، atomic + `.bak`، قرنطینه فایل خراب، هرس تاریخچه پایان‌یافته (سشن‌های فعال هرگز هرس نمی‌شوند). |
| `ChangeExecutor.cs` | 374 | انجام/برگرداندن یک `ChangeRecord` برای هر `ChangeKind`: رجیستری، power plan، Game Mode flag، اولویت پروسه، GPU preference، بستن/بازگرداندن اپ. هر revert مقدار زنده را دوباره می‌خواند. |
| `RegistryTarget.cs` | 205 | مدل مکان رجیستری: `RegistryRoot`، `RegistryValueFormat`، `RegistryTarget` (مسیر نرمال‌شده/اعتبارسنجی‌شده)، `RegistryValueData`، `RegistryReadResult`، `RegistryAccessDeniedException`، `RegistryPath` (normalize). |
| `RegistryAccessor.cs` | 207 | نگهبان allow-list دقیق: HKCU `Software\Microsoft\GameBar`، HKCU `Software\Microsoft\DirectX\UserGpuPreferences`، و `StartupApproved\Run|Run32|StartupFolder` زیر HKCU/HKLM. تطبیق case-insensitive، رد مسیرهای trick و کلیدهای فرزند قبل از باز کردن handle. |
| `ChangePreview.cs` | 38 | `ChangePreview(Title, Summary, Changes)` + `IChangePreviewService`. |
| `ChangePreviewService.cs` | 41 | adapter WPF: دیالوگ `ChangePreviewDialog`؛ بستن/شکست نمایش = «خیر». |
| `ElevationService.cs` | 131 | handshake ری‌استارت با `runas`، mutex single-instance، پارس `--restart-elevated` و `--autostart`، فهرست نام فرایندهای legacy برای فعال‌سازی نمونه در حال اجرا. |
| `AdministratorService.cs` | 71 | وضعیت واقعی elevation، ری‌استارت با `runas` (`RestartOutcome.Started/Declined/Failed`) و مدیریت تسک `Iritool Autostart`. |
| `AutostartTaskService.cs` | 187 | ساخت/حذف/پرسش تسک زمان‌بندی‌شده `Iritool Autostart` از طریق `WindowsToolService`/`CommandRunner`: XML تولیدی، trigger ورود کاربر جاری، بالاترین سطح دسترسی، بدون timeout، `IgnoreNew`، آرگومان ثابت `--autostart`؛ همه مقادیر XML-escape. |

**`Services/Gaming/`**

| فایل | خط | نقش |
|---|---|---|
| `GamingModeService.cs` | 524 | سرویس اصلی Gaming Mode: `GamingTweakSelection`، اعمال previewed+journalled (power plan موجود، Game Mode flag، اولویت تا High، GPU preference، بستن ملایم اپ‌های انتخاب‌شده)، بازگردانی روی OFF/خروج/پایان سشن، گیت هیبرید-GPU برای GPU preference، رد کردن tweak اگر PID/start-time/path دیگر مطابق نباشد. |
| `GameDetectionService.cs` | 265 | شناسایی بازی از پوشه‌های کتابخانه Steam و پروسه‌های در حال اجرا زیر `steamapps\common` + پروفایل‌های دستی؛ apply/restore خودکار اختیاری (پیش‌فرض خاموش) با polling محدود و dispose. |
| `SteamLibrary.cs` | 207 | `SteamLibraryParser` برای `libraryfolders.vdf` (KeyValues کوچک، فقط مسیرها) + `ISteamLibraryLocator` / `SteamLibraryLocator` برای پیدا کردن ریشه Steam از رجیستری/مسیر پیش‌فرض. |

**`Services/Startup/`**

| فایل | خط | نقش |
|---|---|---|
| `WindowsStartupEntryProvider.cs` | 311 | شمارش read-only: Run در HKCU/HKLM هر دو view (۳۲/۶۴)، Startup folder کاربر و عمومی، و تسک‌های زمان‌بندی‌شده با trigger ورود. شکست‌ها → diagnostics. |
| `StartupEntryBuilder.cs` | 182 | ساخت `StartupEntry` از ردیف خام، `SignatureResult`/`IFileSignatureVerifier`، `StartupApprovedState`، `StartupProtectionPolicy` (ردیف‌های ویندوزی/امنیتی محافظت‌شده)، `StartupTogglePlanner` (برنامه تغییر فقط روی `StartupApproved`). |
| `AuthenticodeVerifier.cs` | 266 | تأیید امضای Authenticode با `WinVerifyTrust` (هش فایل، نه فقط وجود گواهی)، بررسی revocation آفلاین از کش محلی تا روی سیستم بدون اینترنت hang نشود. |
| `ScheduledTaskXmlParser.cs` | 173 | پارس XML تسک‌های زمان‌بندی‌شده (`schtasks /query /xml`) به `ScheduledTaskInfo` (path، command، args، trigger). |

**`Services/Platform/`**

| فایل | خط | نقش |
|---|---|---|
| `PowerPlanService.cs` | 172 | لیست و پلن فعال power plan با `powercfg` (`PowerPlanInfo(Guid, Name)`)، تغییر پلن و بازگردانی. |
| `ProcessController.cs` | 364 | snapshot پروسه‌ها (PID، start time، path، RAM)، `ProcessPriorityLevel` (Idle..High؛ Realtime ارائه نمی‌شود)، تغییر اولویت، بستن ملایم/اجبار، بازگرداندن با مسیر. |

**`Services/Benchmark/`**

| فایل | خط | نقش |
|---|---|---|
| `CpuRenderBenchmarkService.cs` | ~135 | بار کاری CPU render محدود و قابل کنسل (`CpuBenchmarkResult`: مدت، پیکسل، پیکسل/ثانیه، دمای پیک، `HasThermalGuard`) + `BenchmarkSafetyException`. دمای پیک اگر سنسور MSR در دسترس نباشد (پنجره غیرادمین) فقط با پارامتر `acceptMissingThermalGuard` رد می‌شود، وگرنه اجرا رد می‌شود. مستقل و غیرقابل مقایسه با Cinebench. |
| `PowerSourceProvider.cs` | 39 | `IPowerSourceProvider` / `WindowsPowerSourceProvider.IsOnBattery` برای دروازه تأیید باتری. |

**`Services/Settings/`**

| فایل | خط | نقش |
|---|---|---|
| `ISettingsService.cs` | 32 | قرارداد: `Current`, `SettingsPath`, `LastLoad`, `Changed`, `LoadAsync`, `SaveAsync`, reset/import/export. |
| `IResetConfirmation.cs` | 12 | قرارداد تأیید پیش از کار برگشت‌ناپذیر (reset)؛ جدا از VM نگه داشته شده تا تست‌پذیر بماند. |
| `ResetConfirmation.cs` | 33 | آداپتور WPF: دیالوگ تأیید را با owner پنجره فعال نشان می‌دهد؛ اگر دیالوگ باز نشود «نه» برمی‌گرداند تا ریست فرض نشود. |
| `SettingsService.cs` | 307 | فروشگاه JSON در `%LOCALAPPDATA%\Iritool\settings.json`: نوشتن atomic (temp + replace) با `.bak` چرخشی، قرنطینه فایل غیرقابل خواندن (نه حذف)، عدم بازنویسی فایل سالم در صورت خطای load، schema versioned، نادیده‌گرفتن property ناشناس، `SemaphoreSlim` برای سریال‌سازی، و مارشال کردن `Changed` روی `SynchronizationContext` زمان ساخت (رفع خطای cross-thread). سازنده با پارامتر `directory` برای تست. |

**`Services/Storage/`**

| فایل | خط | نقش |
|---|---|---|
| `AtomicJsonFile.cs` | 104 | نوشتن/خواندن atomic مشترک (capability report، history، journal): temp + replace، `.bak`، کنار گذاشتن فایل خراب. |

**`Services/Theming/`**

| فایل | خط | نقش |
|---|---|---|
| `IThemeService.cs` | 59 | قرارداد + `AccentDefinition(Deep, Main, Neon)` + `ThemePalette` + رویداد `Applied`. |
| `ThemeService.cs` | ~375 | اعمال تم/اکسنت در زمان اجرا با تعویض پالت در `Application.Resources`؛ همه‌ی brushها در `Colors.xaml` و `Styles.xaml` با `DynamicResource` ارجاع داده می‌شوند، بنابراین یک تعویض دیکشنری کل پنجره را در جا repaint می‌کند (بدون بازسازی پنجره، بدون بازخوانی `Styles.xaml`)؛ کلیدهای Fluent (`SystemAccentColor*`, `AccentFillColor*`) هر بار به‌روز می‌شوند ولی دیکشنری‌های Fluent فقط یک‌بار لود می‌شوند. |

### 6.4 `ViewModels/` (MVVM — همه `ObservableObject`)

| فایل | خط | نقش |
|---|---|---|
| `MainViewModel.cs` | 343 | VM شِل و `INavigationService`: ساخت ۹ صفحه، `NavigationItems`، `SelectedItem`/`CurrentPage`، `IsSidebarExpanded`، بنر بازیابی سشن (`HasRecovery`, `RecoveryTitle/Detail`, `RestoreRecoveryCommand`, `DismissRecoveryCommand`)، `EnsureLoadedAsync` صفحات پرخرج (System Information، Startup، Gaming) و `SetActive` روی مانیتور. |
| `NavigationItemViewModel.cs` | 33 | یک ردیف سایدبار: `Section`, `Title`, `IconKey`, نمونه `Page`، `IsSelected`. |
| `DashboardViewModel.cs` | 358 | داشبورد: احوال‌پرسی زمانی، تاریخ، چهار `MetricTileViewModel`، overview محیطی، وضعیت Gaming Mode، `QuickTools` که واقعاً ناوبری می‌کنند، فوتر نسخه؛ `EnsureLoadedAsync`. |
| `MetricTileViewModel.cs` | 87 | یک کاشی داشبورد: درصد اصلی، نوار انیمیشنی، سه خواندن ثانویه، chip وضعیت (LIVE/PARTIAL/ELEVATION/ERROR)، timestamp و مدت خواندن. |
| `HardwareMonitorViewModel.cs` | 343 | صفحه مانیتور: کارت هر دستگاه (`DeviceCardViewModel`)، خواندن‌ها (`ReadingRowViewModel`)، نمودارها (`HardwareChartViewModel`)، گزینه‌های بازه ۱/۵/۱۵، یادداشت elevation؛ سابسکرایب فقط وقتی صفحه فعال است. |
| `DeviceCardViewModel.cs` | 46 | یک کارت دستگاه (CPU، هر GPU، حافظه) با ردیف‌های ثابت که in-place به‌روز می‌شوند. |
| `ReadingRowViewModel.cs` | 94 | یک خواندن برچسب‌دار با reuse نمونه‌ها؛ `AvailabilityText` متن دلیل/واحد را می‌سازد. |
| `VolumeRowViewModel.cs` | 27 | ردیف یک volume با نوار استفاده. |
| `HardwareChartViewModel.cs` | 129 | یک نمودار زنده: مقدار سرتیتر، سری بازه انتخابی، state صریح وقتی چیزی برای رسم نیست؛ + `ChartRangeOptionViewModel`. تایمر ندارد. |
| `WindowsToolsViewModel.cs` | 601 | صفحه Windows Tools: فقط یک عملیات فعال، استریم خروجی در کنسول مشترک، تاریخچه ماندگار، `ToolSectionViewModel`/`ToolCardViewModel`، تأیید عملیات تعمیری، عدم کنسل برای اجرای elevated با توضیح. |
| `GamingModeViewModel.cs` | 911 | صفحه Gaming Mode: سوییچ opt-in، انتخاب پروفایل/بازی از `Detection`، انتخاب power plan (`PowerPlanOptionViewModel`)، اولویت، GPU preference (گیت هیبرید از inventory)، بستن اپ‌های پس‌زمینه (`BackgroundAppOptionViewModel`)، پروفایل‌ها (`GameProfileRowViewModel`)، هشدار باتری، پیش‌نمایش و بازگردانی. |
| `StartupManagerViewModel.cs` | 443 | صفحه Startup Manager: شمارش read-only، فعال/غیرفعال از طریق لایه ایمنی، قفل ردیف‌های HKLM بدون ادمین، هشدار ردیف محافظت‌شده، Open file location / Copy command (`StartupRowViewModel`). |
| `SystemInformationViewModel.cs` | 324 | صفحه System Information: inventory روی اولین بازدید/Refresh، کارت capability (PARTIAL · 9 of 16)، Regenerate، diagnostics. |
| `DesktopOverlayViewModel.cs` | 344 | صفحه اورلی + پروجکشن HUD: ساخت/نمایش `OverlayWindow`، متریک‌های انتخاب‌شده (`OverlayMetricViewModel`)، قفل/ویرایش، چیدمان Compact/Vertical، opacity/scale، anchor، hotkey status، پیروی از Gaming Mode، اندازه‌گیری مدت پروجکشن. |
| `BenchmarkViewModel.cs` | ~215 | صفحه Benchmarks & Tests: شروع/توقف بار کاری، `CanStartNow` برای دکمه‌ها، دروازه‌های ایمنی (نمونهٔ تازه، تأیید باتری، تأیید صریح اجرا بدون گارد حرارتی وقتی دما خوانا نیست)، `GateText` جدا از پیام اجرا، نوار پیشرفت تخمینی با DispatcherTimer، نمایش نتیجهٔ workload-specific. |
| `SettingsViewModel.cs` | 526 | صفحه Settings: تم (فقط Dark)، پنج اکسنت (`AccentOptionViewModel`)، بازه پولینگ (`PollingOptionViewModel`)، دو کلید (`StartMinimized`, `ConfirmSensitiveOperations`)، Gaming auto-detect، کارت فایل تنظیمات (Reset/Export/Import/Show in folder)، restart as administrator، وضعیت autostart؛ اعمال فوری + ذخیره اتمیک، coalesce تغییر تم+اکسنت. |
| `SettingsOptions.cs` | 48 | VMهای گزینه‌ای اکسنت و بازه پولینگ (مقدار enum/int را حمل می‌کنند، نه نمونه option). |
| `PlaceholderPageViewModel.cs` | 48 | قرارداد صفحه‌های اسکلت فاز ۱. **در حال حاضر هیچ صفحه‌ای از آن ارث نمی‌برد** (همه صفحات واقعی شده‌اند). |

### 6.5 `Views/` (XAML + code-behind نازک)

| فایل | خط | نقش |
|---|---|---|
| `DashboardView.xaml(.cs)` | 366 / 30 | هدر خوش‌آمد، ۴ کاشی، overview، کارت Gaming، دسترسی سریع، فوتر. code-behind: اسکرول/تمرکز جزئی. |
| `WindowsToolsView.xaml(.cs)` | 320 / 59 | کارت‌های ابزار با chip طبقه‌بندی، خط فرمان resolve‌شده، کنسول خروجی، تاریخچه. code-behind: رفتار کنسول (اسکرول به انتها/انتخاب). |
| `GamingModeView.xaml(.cs)` | 332 / 13 | سوییچ اصلی، انتخاب‌ها، پروفایل‌ها، هشدارها. |
| `HardwareMonitorView.xaml(.cs)` | 319 / 29 | کارت دستگاه‌ها، نمودارها (`MetricChart`)، chipهای بازه. |
| `DesktopOverlayView.xaml(.cs)` | 100 / 13 | کنترل‌های HUD (نمایش/قفل/چیدمان/opacity/scale/anchor/hotkey). |
| `BenchmarkView.xaml(.cs)` | ~68 / 9 | کنترل اجرای بنچمارک، هشدار گارد حرارتی/باتری با چک‌باکس تأیید، نوار پیشرفت، کارت‌های نتیجه و تاریخچه. |
| `StartupManagerView.xaml(.cs)` | 217 / 13 | جدول ردیف‌های startup و دکمه‌ها. |
| `SystemInformationView.xaml(.cs)` | 476 / 13 | بخش‌های inventory، کارت capability، تاریخچه/تشخیص. |
| `SettingsView.xaml(.cs)` | ~292 / 30 | کارت‌های تنظیمات؛ ردیف‌های responsive با ستون label/control جدا (رفع هم‌پوشانی)؛ نوار وضعیت pinned پایین صفحه با رنگ خطا/موفقیت، پاک‌شدن خودکار و `LiveSetting` برای screen reader. |
| `OverlayWindow.xaml(.cs)` | 31 / 218 | پنجره HUD مستقل: topmost، شفاف، `WS_EX_TOOLWINDOW|WS_EX_NOACTIVATE`، در حالت قفل `WS_EX_TRANSPARENT`، DragMove در حالت ویرایش، Clamp به work area مانیتور، ذخیره anchor-relative، `RegisterHotKey(Ctrl+Alt+O)` و آزادسازی hook/هات‌کی در `Closed`. |
| `ConfirmOperationDialog.xaml(.cs)` | 72 / 47 | تأیید عملیات تعمیری: طبقه‌بندی، دستور دقیق، دلیل elevation، timeout. |
| `ConfirmResetDialog.xaml(.cs)` | ~95 / 33 | تأیید تنها کار برگشت‌ناپذیر صفحه تنظیمات: فایل جایگزین‌شده، هشدار از دست رفتن مقادیر؛ Cancel دکمه پیش‌فرض است. |
| `ChangePreviewDialog.xaml(.cs)` | 100 / 43 | پیش‌نمایش تغییر سیستم: مقدار فعلی و درخواستی + نیاز به elevation؛ بستن = بدون تغییر. |

### 6.6 `Controls/`

| فایل | خط | نقش |
|---|---|---|
| `MetricChart.cs` | 198 | `FrameworkElement` که در `OnRender` از `ChartSeries` رسم می‌کند: یک invalidate در هر ثانیه، بدون ساخت visual tree؛ نمونه ناموجود = شکاف و baseline خط‌چین (نه خط صفر). |
| `UsageBar.xaml(.cs)` | 22 / 72 | نوار استفاده با انیمیشن مقدار (DPI-safe، بدون بیت‌مپ). |
| `PhasePlaceholder.xaml(.cs)` | 90 / 16 | ارائه مشترک صفحه‌های اسکلت فاز ۱. **اکنون ارجاعی ندارد** (همه صفحات پیاده‌سازی شده‌اند). |

### 6.7 `Converters/`

| فایل | خط | نقش |
|---|---|---|
| `IconKeyToGeometryConverter.cs` | 36 | کلید آیکون (مثل `"Monitor"`) → `Icon.Monitor` از `Assets/Icons.xaml`؛ VM از نوع WPF پاک می‌ماند. |
| `AvailabilityBrushConverter.cs` | 36 | `SensorAvailability` → رنگ وضعیت. |
| `CompatibilityLevelBrushConverter.cs` | 35 | سطح کلی سازگاری → سبز/کهربایی/قرمز (جدا از converter سنسور). |
| `OperationConverters.cs` | 52 | `ThemeBrush` (resolve کلید با fallback) + `OperationKindBrushConverter` + `OperationOutcomeBrushConverter`. |
| `VisibilityConverters.cs` | 54 | `CountToVisibilityConverter`, `NullToVisibilityConverter`, `InverseBooleanConverter`. |

---

## 7. پروژه تست — `iritool.Tests/`

`iritool.Tests.csproj` (۲۶ خط): `net10.0-windows`، `UseWPF`، `x64`، `IsPackable=false`،
`ProjectReference` به اپ با `SetConfiguration="Configuration=Debug"`.

| فایل | تعداد `[Fact]/[Theory]` | پوشش |
|---|---|---|
| `OperationTests.cs` | 35 | مسیر/آرگومان‌ها/طبقه‌بندی/لینک تأیید کاتالوگ، جدول exit code، یک‌عملیات-در-یک‌زمان، رد elevation، متن کنسول. |
| `InventoryParserTests.cs` | 24 | تاریخ/آپ‌تایم OS، ردیف خالی، هسته/کش، VRAM مجزا vs اشتراکی، چند vendor GPU، چیدمان ماژول، firmware، placeholder دیسک، volume، قالب شبکه، مشکل درایور، جدول کد. |
| `GamingModeTests.cs` | 21 | چرخه Gaming Mode، فیلتر ابزارهای Steam، پردازش کهنه، گیت هیبرید-GPU، merge/restore ترجیح GPU، مایگریشن پروفایل. |
| `SettingsTests.cs` | 20 | پیش‌فرض، round-trip، backup، قرنطینه خراب، مایگریشن، property ناشناس، snap interval، export/import، reset، notification، دستورهای ظهور. |
| `StartupManagerTests.cs` | 16 | payloadهای `StartupApproved`، ردیف‌های محافظت‌شده، escape در XML تسک، شمارش. |
| `CommandExecutionTests.cs` | 15 | echo/exit code/stderr/فایل ناموجود/کنسل/timeout، UTF-16+BOM، split شدن chunk، خطوط `\r`، tail لاگ elevated. |
| `ChangeJournalTests.cs` | 13 | ترتیب write-ahead، بازیابی بعد از crash قبل از marker، restore معکوس و idempotent، محافظت از تغییر خارجی، restore مقدار ناموجود، قرنطینه journal خراب، رد مسیر خارج از allow-list، پیش‌نمایش رد‌شده. |
| `ChartSeriesBuilderTests.cs` | 12 | history خالی، فیلتر پنجره، سنسور غایب بدون صفر، محور دما، چند GPU، GPU ناشناس، آخرین مقدار، decimation. |
| `CapabilityReportTests.cs` | 10 | تفکیک پشتیبانی‌شده/نشده، متن مجوز، حالت بدون GPU، پایداری اجرای اول، regenerate، فایل خراب، round-trip. |
| `SensorSelectionTests.cs` | 8 | انتخاب نام‌دار/fallback/`SelectExact`/`SelectMax`، حذف تکراری. |
| `SensorValueValidatorTests.cs` | 7 | دروازه plausibility به تفکیک نوع سنسور، NaN/Inf. |
| `HardwarePollingServiceTests.cs` | 7 | cadence بدون همپوشانی، کنسل، مهار provider خطادار. |
| `HardwareAvailabilityTests.cs` | 7 | تجمیع availability و دلیل غالب. |
| `HardwareCompatibilityServiceTests.cs` | 6 | سطوح سازگاری از جمله حالت unelevated. |
| `BenchmarkSafetyTests.cs` | 4 | نبود دما، تأیید باتری، توقف حرارتی، کنسل. |
| `InventoryProbeTests.cs` | 3 | پروب‌های واقعی/سبک خواندن inventory. |
| `InstanceActivationTests.cs` | 3 | پوشش نام‌های فرایند جاری/legacy، de-duplication، نام نامعتبر. |
| `HardwareSampleHistoryTests.cs` | 3 | مرزهای بافر، retarget. |
| `InventorySmokeTests.cs` | 2 | smoke روی پیاده‌سازی واقعی. |
| `StartupMarkupTests.cs` | 1 | تست STA: لود منابع اپ، شِل و همه ۹ صفحه (گارد رگرسیون `XamlParseException`). |
| `TestInfrastructure.cs` | — | زیرساخت مشترک (فایل موقت/پوشه ایزوله). |
| `TestDoubles.cs` | — | fakeهای provider تلمتری/سرویس‌ها. |
| `Phase3TestDoubles.cs` | — | fakeهای رجیستری، power plan، پروسه، preview، Steam برای فاز ۳. |

قواعد مجموعه تست (طبق NOTES): هیچ دستور elevated اجرا نمی‌شود، هیچ تنظیم واقعی سیستم تغییر نمی‌کند،
تنها پروسه‌های اجراشده `cmd.exe /c echo`، `exit 3` و یک `ping` لوپ‌بک برای تست timeout/کنسل هستند.

---

## 8. آنچه در زمان اجرا روی دیسک نوشته/خوانده می‌شود

| مسیر | نویسنده | توضیح |
|---|---|---|
| `%LOCALAPPDATA%\Iritool\settings.json` | `SettingsService` | تنظیمات، schema 4، atomic + `.bak`، فایل خراب قرنطینه می‌شود. |
| `%LOCALAPPDATA%\Iritool\capability-report.json` | `HardwareCapabilityReportService` | گزارش قابلیت اولین اجرا؛ دستی regenerate می‌شود. |
| `%LOCALAPPDATA%\Iritool\operation-history.json` | `OperationHistoryStore` | ۶۰ اجرای آخر ابزارهای ویندوز. |
| `%LOCALAPPDATA%\Iritool\change-journal.json` | `ChangeJournalStore` | journal تغییرات سیستم (write-ahead) برای Gaming Mode و Startup Manager. |
| فایل لاگ موقت | `CommandRunner` (حالت elevated) | خروجی `cmd /c ... > log` که `LogTailReader` دنبال می‌کند. |
| رجیستری (نوشتن، فقط از allow-list) | `RegistryAccessor` | `HKCU\Software\Microsoft\GameBar`، `HKCU\Software\Microsoft\DirectX\UserGpuPreferences`، `StartupApproved\Run|Run32|StartupFolder` زیر HKCU/HKLM. |
| Scheduled Task | `AutostartTaskService` | تسک ثابت `Iritool Autostart` با آرگومان `--autostart`. |
| پوشه‌های Steam (فقط خواندن) | `SteamLibraryLocator` | `libraryfolders.vdf` و `steamapps\common`. |
| `%LOCALAPPDATA%\Programs\iritool` | نصب‌کننده | محل نصب per-user. |

---

## 9. مدل ایمنی و مجوزها (خلاصه سراسری)

- مانیفست همیشه `asInvoker` است؛ برنامه هرگز خودش UAC نمی‌خواهد. Elevation فقط per-action است
  (`runas` برای SFC/DISM/CHKDSK و restart).
- هر تغییر سیستم از `ChangeSessionService` می‌گذرد: پیش‌نمایش → journal `Pending` → اجرا → `Applied`
  → بازگردانی معکوس با بررسی مقدار فعلی.
- خط فرمان‌ها فقط از `OperationCatalog` می‌آیند؛ مسیر System32 مطلق، آرگومان ثابت.
- رجیستری فقط از allow-list دقیق؛ مسیر trick/کلید فرزند رد می‌شود.
- چیزهایی که عمداً وجود ندارند: `chkdsk /f`، اولویت Realtime، ساخت power plan، توقف/غیرفعال‌سازی
  سرویس ویندوز، تزریق در بازی، FPS counter، رندر روی Fullscreen انحصاری.
- مقدار ناموجود همیشه `—` با دلیل است؛ هیچ‌جا صفر جعلی نمایش داده نمی‌شود.

---

## 10. تاریخ فازها (خلاصه `NOTES.md`)

| فاز | محتوا | وضعیت ثبت‌شده |
|---|---|---|
| 1 | شِل Fluent، سایدبار ۸ بخشی، داشبورد با placeholder، صفحات اسکلت | انجام و تأیید شده |
| 2A | موتور تلمتری (LibreHardwareMonitor), ۵۵ تست | کامل |
| 2B | inventory (WMI)، نمودارهای زنده، گزارش قابلیت | کامل روی همین ماشین |
| 2C | ذخیره تنظیمات + تم/اکسنت (Schema 1)، رفع باگ‌های thread/style/palette | کامل |
| 2D | Windows Tools (SFC/DISM/CHKDSK scan) + تأییدیه + تاریخچه | کامل، به‌جز یک اجرای واقعی elevated |
| 3 | لایه journal، Startup Manager، Gaming Mode (۲۷۹ تست) | کامل، چک‌های دستی باقی |
| 5 | بنچمارک CPU render، سابسکرایب شرطی Hardware Monitor، اورلی کامل‌تر (۲۸۹ تست) | کامل |
| — | تحویل متن‌باز: نام `iritool`، نسخه `1.0`، README انگلیسی، MIT، نصب‌کننده Inno | کامل |
| هات‌فیکس | single-instance با نام‌های legacy، `XamlParseException` آیکون با pack URI site-of-origin (۲۹۰ تست) | رفع شد |
| پایداری | binding یک‌طرفه `AutostartTaskName`، ردیف‌های responsive، فیلتر ابزار Steam، گیت پردازش کهنه/هیبرید | رفع شد |
| 4 | حذف Light/System (Schema 4)، اورلی واقعی، هات‌کی `Ctrl+Alt+O`، a11y، `PROJECT_GRAPH.md` | کامل، چک دستی مولتی‌مانیتور/DPI باقی |

---

## 11. بدهی فنی و فایل‌های بدون ارجاع

| مورد | وضعیت |
|---|---|
| `Models/HardwareMetric.cs` | هیچ ارجاعی ندارد؛ از فاز ۲A باقی مانده و طبق NOTES عمداً حذف نشده است. |
| `ViewModels/PlaceholderPageViewModel.cs` | هیچ صفحه‌ای از آن ارث نمی‌برد (همه صفحات واقعی شده‌اند). |
| `Controls/PhasePlaceholder.xaml(.cs)` | هیچ Viewی از آن استفاده نمی‌کند. |
| چک‌های دستی باقی‌مانده | پذیرش/رد UAC، ساخت/حذف تسک autostart، نوشتن واقعی `StartupApproved` و power plan، رفتار مولتی‌مانیتور/DPI در اورلی، درگیری هات‌کی با اپ دیگر. |
| سخت‌افزار تأییدنشده | AMD CPU/GPU، Intel Arc، دسکتاپ چند GPU مجزا، سیستم بدون داده SMBIOS. |
| CPU temp/clock/power بدون ادمین | `AccessDenied` است و برنامه عمداً elevation نمی‌خواهد؛ بنچمارک روی چنین ماشینی اجرا نمی‌شود. |

---

## 12. `ui-ux-pro-max-skill-2.15.0/` — پوشه مرجع (جزو محصول نیست)

یک اسکیل طراحی UI/UX با ۶۶۲ فایل و حدود ۲۳ مگابایت که فقط به‌عنوان مرجع طراحی در مخزن قرار گرفته
است. هیچ فایل اپ به آن ارجاع نمی‌دهد و در بیلد/نصب استفاده نمی‌شود. ساختار اصلی:

```
ui-ux-pro-max-skill-2.15.0/
└─ ui-ux-pro-max-skill-2.15.0/
   ├─ README.md / README.zh.md / SKILL.md / skill.json   معرفی و مانيفست اسکیل
   ├─ CLAUDE.md / CONTRIBUTING.md / CODE_OF_CONDUCT.md / SECURITY.md
   ├─ cli/                    ابزار CLI + assets (data, scripts, skills, templates/platforms)
   ├─ src/ui-ux-pro-max/      نسخه منبع همان پکیج (data, scripts, templates, skills)
   ├─ gallery/                اپ گالری (Next.js) برای مرور کامپوننت‌ها
   ├─ stack/                  استک مرجع + مثال `juniper-audit`
   ├─ docs/journals/          ژورنال‌های برنامه‌ریزی داده جست‌وجو و فازهای ۶/۷
   ├─ projects/               نمونه پروژه‌ها (healthcare-dashboard، portfolio-dark، saas-landing)
   ├─ preview/, screenshots/, scripts/
   └─ .claude/, .claude-plugin/, .github/
```

دسته‌های مهارتی داخل `cli/assets/skills/`: `banner-design`، `brand`، `design`، `design-system`،
`slides`، `ui-styling` — هر کدام با `SKILL.md` و `references/`.

---

## 13. نصب‌کننده (installer/)

نصب‌کننده از خود اپ جدا است و رابط کاربری آن WPF است؛ Inno Setup فقط به‌عنوان مسیر قدیمی در مخزن مانده و
در ساخت فعلی به کار نمی‌رود (ویزارد خودش قابل جایگزینی با WPF نیست و در حالت silent هیچ API پیشرفتی
برای درصد نصب ارائه نمی‌دهد).

| فایل | نقش |
|---|---|
| `installer/build-installer.ps1` | ۵ مرحله: publish اپ → publish uninstaller → ساخت `payload.zip` → publish رابط WPF → تولید `artifacts/installer/iritool-Setup-<v>-x64.exe` |
| `iritool-installer/App.xaml(.cs)` | نقطه ورود، مسیر `--capture <dir>` برای اسکرین‌شات خودکار هر صفحه |
| `iritool-installer/MainWindow.xaml(.cs)` | شل دو‌ستونه ۹۰۰×۵۸۰، chrome سفارشی، گوشه گرد DWM، شش صفحه |
| `iritool-installer/Theme/Palette.xaml` | پالت تیره `#171719` / `#202026` / `#29282F` + اکسنت `#8055FF`/`#A56BFF` |
| `iritool-installer/Theme/Styles.xaml` | دکمه‌ها، سوییچ، TextBox، ProgressBar بنفش، کارت، کپشن‌باتن‌ها |
| `iritool-installer/ViewModels/InstallerViewModel.cs` | وضعیت صفحه‌ها، اعتبار مسیر، هشدار نسخه نصب‌شده/اپ در حال اجرا، درصد نرم‌شده |
| `iritool-installer/Services/InstallEngine.cs` | نصب واقعی: استخراج zip داخلی به پوشه staging، سپس move؛ درصد بر اساس بایت‌های نوشته‌شده |
| `iritool-installer/Services/ShellShortcut.cs` | ساخت `.lnk` با `IShellLink` و آیکون رسمی |
| `iritool-installer/Services/UninstallRegistration.cs` | کلید HKCU Uninstall\iritool (DisplayIcon، UninstallString، QuietUninstallString) |
| `iritool-uninstaller/Program.cs` | حذف فایل‌ها/شورتکات‌ها/کلید رجیستری، خودکپی به `%TEMP%` برای حذف کامل؛ `%LOCALAPPDATA%\Iritool` دست‌نخورده می‌ماند |
| `Assets/icon.ico` (هر دو پروژه) | کپی بایت‌به‌بایت `icon.ico` ریشه؛ هیچ آیکونی بازتولید نشده |

---

## 14. دستورهای کار روزمره

```powershell
# بیلد و تست کل سولوشن
dotnet build .\iritool\iritool.slnx -c Release
dotnet test  .\iritool\iritool.slnx -c Release

# فقط تست‌ها
cd .\iritool\iritool.Tests
dotnet test

# اجرای خروجی
.\iritool\iritool\bin\Release\net10.0-windows\iritool.exe

# ساخت نصب‌کننده (خروجی: artifacts/installer/iritool-Setup-<version>-x64.exe)
powershell -ExecutionPolicy Bypass -File .\installer\build-installer.ps1
# با -SkipAppPublish از publish موجود اپ استفاده می‌شود
```

---

## 15. قراردادهای نام‌گذاری و نگهداری

- نام پروژه، سولوشن، نیم‌اسپیس، پوشه‌ها و فایل exe تا فاز انتشار نهایی `iritool`
  می‌ماند و فقط `AssemblyName`/`Product` روی `iritool` تنظیم شده است.
- هر صفحه = یک `XxxViewModel` + یک `XxxView.xaml`، نگاشت در `App.xaml` با `DataTemplate`.
- سرویس‌ها از طریق سازنده تزریق می‌شوند و هیچ singleton سراسری وجود ندارد؛ `MainViewModel`
  تنها جای ساخت VMهای صفحه است.
- تغییر رنگ/تم فقط با کلیدهای `DynamicResource` و پالت `Colors.xaml`.
- هر فیچر سیستمی باید: opt-in، قابل پیش‌نمایش، قابل بازگشت و پوشش‌داده‌شده با تست fake-backed باشد.
- بعد از هر فاز، `NOTES.md` و این `map.md` باید به‌روز شوند.


<!-- en -->

# Complete Project Map — iritool / iritool
 
> This file is a line-by-line "map" of the entire repository contents: every folder, every file, its
> role, its relationships with the others, and the paths it touches at runtime. The map is based on
> reading the source directly and counting lines at the time of writing. If a file is added or
> removed, this file must be updated.
 
---
 
## 1. The Project at a Glance
 
| Item | Value |
|---|---|
| Product/output name | `iritool` (version `1.0`) |
| Project/solution/namespace name | `iritool` / `iritool` |
| App type | Windows desktop application (WPF) |
| Framework | `net10.0-windows`, `PlatformTarget = x64` |
| Architecture | MVVM (CommunityToolkit.Mvvm) with a manual Composition Root in `App.xaml.cs` |
| Theme | Dark only (charcoal/black + purple), WPF-UI 4.3.0 |
| Number of menu sections | 9 sections |
| Tests | Separate xUnit project (`iritool.Tests`) |
| Installation | Inno Setup, per-user, no admin required |
 
NuGet packages (only 4 in the app + 3 in tests):
 
| Package | Version | Purpose |
|---|---|---|
| `WPF-UI` | 4.3.0 | Fluent window, title bar, Fluent controls, theme dictionary |
| `CommunityToolkit.Mvvm` | 8.4.2 | `ObservableObject`, `[ObservableProperty]`, `[RelayCommand]` |
| `LibreHardwareMonitorLib` | 0.9.6 | Reading CPU/GPU/RAM/disk sensors |
| `System.Management` | 10.0.2 | WMI queries for System Information |
| `xunit` / `xunit.runner.visualstudio` / `Microsoft.NET.Test.Sdk` | 2.9.3 / 2.8.2 / 18.12.0 | Test project only |
 
---
 
## 2. Complete Repository Tree
 
```
D:\iritool\                 ← repository root (git)
├─ README.md                                 Product introduction (EN + Persian section)
├─ NOTES.md                                  Phase-by-phase development history and decisions
├─ LICENSE                                   MIT, Copyright (c) 2026 iridevill
├─ .gitignore                                bin/obj/.vs/.zwork etc.
├─ icon.ico / icon.png                       Installer icon (Inno Setup source)
├─ docs/
│  └─ architecture/
│     └─ PROJECT_GRAPH.md                    Mermaid graph of service wiring
├─ installer/
│  ├─ build-installer.ps1                    Setup file build pipeline
│  ├─ iritool-installer/                     WPF installer app (new UI)
│  ├─ iritool-uninstaller/                   Small per-user uninstaller
│  ├─ iritool.iss / scripts/iritool.iss      Legacy Inno Setup path (leftover)
│  └─ assets/*.bmp                           Previous Inno artwork (unused)
├─ .freebuff/                                Tool metadata (project-id)
├─ .zwork/                                   Temporary/test/release workspace (git-ignored)
├─ ui-ux-pro-max-skill-2.15.0/               UI/UX design skill (reference, not part of the app)
└─ iritool/                  Solution folder
   ├─ iritool.slnx           Solution (app + tests)
   ├─ .vs/                                   Visual Studio cache (ignored)
   ├─ .zwork/                                App-specific obj/publish output (ignored)
   ├─ iritool/               WPF project  ← the heart of the program
   └─ iritool.Tests/         xUnit test project
```
 
Important note about the nesting: the real path of the app project is
`iritool/iritool/`, and the tests live in
`iritool/iritool.Tests/` beside it.
 
---
 
## 3. Program Execution Flow (from start to page)
 
```
iritool.exe
   │
   ▼
App.OnStartup (App.xaml.cs)
   ├─ ElevationService.WasRestartRequested / WasStartedFromAutostart  ← argument parsing
   ├─ ElevationService.TryAcquireInstance(mutex)  ── failed? → ActivateRunningInstance() → Shutdown(0)
   ├─ Error handlers: DispatcherUnhandledException / AppDomain / UnobservedTask / SessionEnding
   ├─ SettingsService.LoadAsync()  →  AppSettings (on error: defaults)
   ├─ ThemeService.Apply(theme, accent)  →  Applied event (palette applied to the Visual Tree right there)
   ├─ LibreHardwareTelemetryProvider  →  HardwareCompatibilityService  →  HardwareTelemetryMonitor
   ├─ WindowsToolService(new CommandRunner(isElevated))
   ├─ RegistryAccessor + PowerPlanService + ProcessController
   ├─ ChangeSessionService(journal, executor).InitializeAsync()  ← recovers previous sessions
   ├─ ChangePreviewService / WindowsInventoryProvider
   ├─ GamingModeService / GameDetectionService / AdministratorService / AutostartTaskService
   ├─ WindowsStartupEntryProvider(new AuthenticodeVerifier())
   ├─ MainViewModel(...)   ← all services are injected here
   ├─ MainWindow.Show()    ← shell, DataContext = MainViewModel
   └─ HardwareTelemetryMonitor.Start()  ← first snapshot after the window exists
          │
          └─ Updated → first run? → HardwareCapabilityReportService.EnsureFirstRunAsync(...)
 
App.OnExit
   ├─ unsubscribe (theme, telemetry, SessionEnding)
   ├─ GameDetectionService.Stop()
   ├─ RestoreActiveSessions()          ← reverts OnExit changes with a 10-second deadline
   ├─ Dispose: MainViewModel, monitor, settings, changes
   └─ ReleaseMutex + Dispose
```
 
Key point: `App.xaml.cs` is the only composition root; no service is created in XAML or in page
code-behind. A theme change causes the whole shell window to be rebuilt (`OnThemeApplied`) because
WPF does not re-read sealed styles with the new palette; the shared ViewModel is kept so page
state/selection/telemetry is not lost.
 
---
 
## 4. Repository Root Files
 
| File | Lines | Role |
|---|---|---|
| `README.md` | 58 | Product intro, feature list, limitations (no in-game injection, no FPS, no exclusive-fullscreen rendering), prerequisites (Windows 10/11 x64), Build/Test commands, contribution guide, Telegram link `@iridevill`, and a Persian section. |
| `NOTES.md` | 944 | The project's engineering ledger. Phase 1 to Phase 5 + hotfixes. The reference for the "why" behind every decision, build/test results, observed hardware limitations, and manual verification checklists. |
| `LICENSE` | — | MIT, holder: `iridevill`, year 2026. |
| `.gitignore` | 20 | `/vs`, `**/bin/`, `**/obj/`, `*.user`, `*.suo`, `*.cache`, `*.log`, and `.zwork/` (tool workspace). |
| `icon.ico` / `icon.png` | — | Installer/brand icon at the root; `installer/iritool.iss` points to `icon.ico` via `SetupIconFile`. |
 
### `docs/architecture/PROJECT_GRAPH.md`
A Mermaid graph of service wiring (App → Settings/Theme/Provider/Main/Gaming, etc.) together with an
"Overlay boundary" section: the overlay subscribes to `HardwareTelemetryMonitor.Updated`, builds no
second sensor loop, `RegisterHotKey` owns only `Ctrl+Alt+O`, locked mode uses `WS_EX_TRANSPARENT`,
and there is no exclusive-fullscreen rendering / game hooking / FPS.
 
### `installer/iritool.iss`
Inno Setup script:
- Variables: `AppName=iritool`, `AppVersion=1.0`, `AppExeName=iritool.exe`,
  `PublishDir=..\iritool\iritool\.zwork\publish`
- Installs to `{localappdata}\Programs\iritool`, `PrivilegesRequired=lowest`, `x64os` only
- Output in `..\.zwork\installer` named `iritool-Setup-1.0-x64.exe`
- `lzma2/ultra64` compression, `*.pdb` excluded from the package, permanent Start Menu shortcut and
  optional Desktop shortcut (`Tasks: desktopicon`, unchecked by default), optional launch after install.
### `.freebuff/`
Tool metadata only (`project-id` and its own `.gitignore`). It contains no product code.
 
### `.zwork/` (at the root)
Transient, git-ignored workspace: `attachments/` (pasted texts), PowerShell scripts for manual UI
Automation tests (`click.ps1`, `final-check.ps1`, `gaming-test.ps1`, `hwnd-probe.ps1`,
`dialog-probe.ps1`, `crash-test.ps1`, `consent-test.ps1`, `gaming-off.ps1`), `dotnet-test.log`, a
local copy of Inno Setup (`inno/` with `Default.isl` and Examples) used to compile the installer,
and the `publish`/`installer` outputs. It is not needed to run the program.
 
### `iritool/iritool.slnx`
New XML-format solution (`.slnx`) with two projects:
```
<Project Path="iritool/iritool.csproj" />
<Project Path="iritool.Tests/iritool.Tests.csproj" />
```
So `dotnet test` on the solution runs the whole test suite (this was not the case in Phase 2; it was added in Phase 3).
 
---
 
## 5. App Project — Project Root Files
 
| File | Lines | Role |
|---|---|---|
| `iritool.csproj` | 39 | `WinExe`, `net10.0-windows`, `UseWPF`, `x64`, `Nullable`, product metadata (`Product/AssemblyTitle=iritool`, `Version=1.0`), `ApplicationManifest=Assets\app.manifest`, `ApplicationIcon=Assets\app.ico`, embeds `app.ico`+`app.png` via `<Resource>` (not copied to output), 4 PackageReferences. |
| `AssemblyInfo.cs` | 10 | Assembly settings (WPF themes, `ThemeInfo`). |
| `App.xaml` | 51 | `ShutdownMode=OnMainWindowClose`; merged dictionaries: `ui:ThemesDictionary Theme="Dark"`, `ui:ControlsDictionary`, `Assets/Colors.xaml`, `Assets/Icons.xaml`, `Assets/Styles.xaml`; and 9 `DataTemplate`s mapping ViewModel → View (Dashboard, WindowsTools, GamingMode, HardwareMonitor, DesktopOverlay, Benchmark, StartupManager, SystemInformation, Settings). |
| `App.xaml.cs` | 458 | Composition root + single-instance mutex (`Local\Iritool.SingleInstance`) + activation of the running instance (including legacy names) + global error handler + settings loading + shell rebuild on theme change + `RestartElevated` + `RestoreActiveSessions`. |
| `MainWindow.xaml` | 294 | Shell: `ui:FluentWindow` with `ExtendsContentIntoTitleBar`, size 1340×860, minimum 1080×720, `WindowCornerPreference=Round`, three ambient-light Ellipse layers, 44px `ui:TitleBar`, 252px sidebar (brand + navigation `ItemsControl` + version/collapse footer), page host with a 300ms entrance animation, session recovery banner. The icon and logo are read from embedded resources via `pack://application:,,,/Assets/app.ico|app.png`, not from a file next to the exe. |
| `MainWindow.xaml.cs` | 66 | The shell's only code-behind: sidebar width animation `252 ↔ 76` on `IsSidebarExpanded` using a 220ms `DoubleAnimation` with `CubicEase`; subscribes/unsubscribes `PropertyChanged` in `DataContextChanged`. |
| `Assets/app.manifest` | 20 | `supportedOS` Windows 10/11, `dpiAware=true/pm` + `dpiAwareness=PerMonitorV2`, and `requestedExecutionLevel` = `asInvoker` (the program never requests elevation by itself). |
| `Assets/app.ico` / `Assets/app.png` | — | Icon and logo embedded in the assembly (`<Resource>`, read via pack URI from `application`). |
 
---
 
## 6. App Project — Folders
 
### 6.1 `Assets/` (theme and design)
 
| File | Lines | Content |
|---|---|---|
| `Colors.xaml` | 135 | 62 keys: palette (backdrop `#0B0B12→#08080C`, cards `rgba(255,255,255,0.06–0.09)`, stroke `rgba(255,255,255,0.10)`), accent `#8B5CF6` / `#6D28D9` / `#B983FF`, semantic green/amber/red, texts, fonts (`Segoe UI Variable Display/Text`, `Cascadia Mono`) and shadows. |
| `Icons.xaml` | 50 | 18 stream-geometry icons at 24×24 (no icon font and no emoji). Each icon key starts with `Icon.` and `IconKeyToGeometryConverter` resolves it. |
| `Styles.xaml` | 482 | 38 styles/keys: typography, card, chip, buttons, `WT.NavButton` (active state with a 3px pill), page entrance animation, settings rows, etc. All colors use `DynamicResource` so live palette swapping works. |
| `app.manifest` | 20 | Described in section 5. |
| `app.ico`, `app.png` | — | Runtime icon/logo resources. |
 
### 6.2 `Models/` (domain models — no WPF dependency)
 
| File | Lines | Content |
|---|---|---|
| `AppSection.cs` | 19 | Enum of nine sections: `Dashboard, WindowsTools, GamingMode, HardwareMonitor, DesktopOverlay, Benchmarks, StartupManager, SystemInformation, Settings`. Stable identifier for navigation and shortcuts. |
| `QuickTool.cs` | 11 | Dashboard shortcut tile record (`Title`, `Description`, `IconKey`, `Target`). |
| `SystemFact.cs` | 8 | Simple `(Label, Value)` record for the "System overview" panel. |
| `SystemSnapshot.cs` | 17 | System environment from .NET: OS description, machine name, architecture, runtime version, user, logical processor count, capture time. |
| `HardwareMetric.cs` | 17 | Phase 1 leftover (placeholder tile with `IsPlaceholder`). **Not referenced in the current code** and not removed. |
 
**`Models/Hardware/` — the heart of the telemetry model**
 
| File | Lines | Content |
|---|---|---|
| `SensorKind.cs` | 14 | Sensor type enum: `Load, Temperature, Clock, Power, Voltage, Memory, Throughput, Fan, …`. |
| `SensorAvailability.cs` | 30 | Taxonomy: `Available, Partial, SensorUnavailable, DeviceUnavailable, AccessDenied, ReadFailed, DeviceNotDetected`. Distinguishes "no sensor" from "is zero" and from "requires elevation". |
| `SensorReading.cs` | 106 | One reading: value + unit + availability + `Detail`. If not available, `Value` is null and a fake zero is never stored; display formats (`Format`, `FormatPair`) are also here. |
| `SensorValueValidator.cs` | 40 | Plausibility gate: load 0–100, temperature `0 < T ≤ 150`, clock > 0, power > 0, voltage 0–5, memory/throughput ≥ 0; NaN/±Inf rejected. |
| `SensorSelection.cs` | 149 | Canonical sensor selection: name priority (`CPU Total`, `CPU Package`, `GPU Core`…) → any available reading → unavailable state. `SelectMax` for the highest core clock, `SelectExact` which prevents an iGPU's shared memory from being shown instead of VRAM. |
| `SensorFailureClassifier.cs` | 33 | Converts a sensor-read exception to an availability (e.g. `AccessDenied`). |
| `HardwareAvailability.cs` | 96 | Aggregates availability of a group of readings: all ok → Available, some → Partial, none → the dominant reason. |
| `HardwareSnapshot.cs` | 44 | Immutable picture of a moment: CPU, GPU list, Memory, Volumes, StorageDevices, time, read duration. |
| `Telemetry.cs` | 141 | Records `CpuTelemetry` (load/temp/clock/power/voltage/cores/name), `GpuTelemetry` (with a stable `Id` for multi-GPU and hybrid), `MemoryTelemetry`, `VolumeTelemetry`, `StorageDeviceTelemetry`. |
| `HardwareCompatibilityReport.cs` | 35 | `HardwareCompatibilityLevel` enum (`Full, Partial, Limited, Unavailable`…) + level report record + notes and permission limitations. |
| `HardwareCapabilityReport.cs` | 73 | Schema of the `capability-report.json` file: `CapabilitySensor(Device, Measurement, State, Reason)`, sensors list, supported/unsupported features, access and hardware limitations, machine, app version, elevation state, first-generation time. |
| `ChartSeries.cs` | 39 | `ChartMetric` (CpuLoad, CpuTemperature, GpuLoad, GpuTemperature, MemoryUsage, …) + `ChartSeries` with points, range, unit, state, and a "why it's empty" text. |
 
**`Models/Inventory/`**
 
| File | Lines | Content |
|---|---|---|
| `InventoryFact.cs` | 110 | One labeled inventory value; the "unavailable" state is first-class and is rendered as `—` with the reason in a tooltip. `InventoryFormat` formatting helpers (bytes/gigabytes/date/uptime/network speed). |
| `SystemInventory.cs` | 89 | Records `InventoryEntry` (sections with title/subtitle/icon/rows), `VolumeInfo`, `DriverInfo`, and `SystemInventory` (including sections + volumes + drivers + diagnostics). |
 
**`Models/Operations/`**
 
| File | Lines | Content |
|---|---|---|
| `OperationDefinition.cs` | 116 | `OperationKind` enum (`ReadOnly, Diagnostic, Repair, AdvancedRepair` + label/description extensions), `OperationDefinition` (Id, title, absolute file, fixed argument, kind, elevation requirement, timeout, confirming operation), `OperationGroup`, `OperationOutcome` + extensions, `OperationResult`. |
| `OperationCatalog.cs` | 223 | The only permitted command catalog: `sfc.verifyonly`, `sfc.scannow`, `dism.checkhealth`, `dism.scanhealth`, `dism.restorehealth`, `chkdsk.scan` (plus the autostart task management commands). Paths are absolute in System32 and arguments are fixed; the UI cannot build an arbitrary command line. |
| `ExitCodeInterpreter.cs` | 89 | Translates exit codes into an outcome and actionable message: SFC `0/1/2/3`, DISM `0/3010/0x800F081F/0x800F0906/87/5`, CHKDSK `0/1/2/3`. An unknown code is reported verbatim, not as "success". |
| `OperationHistoryEntry.cs` | 32 | One run-history record (Id, OperationId, title, start/duration, exit, summary). |
 
**`Models/Safety/`**
 
| File | Lines | Content |
|---|---|---|
| `ChangeRecord.cs` | 192 | The whole journal model: `ChangeKind` (`Registry, PowerPlan, GameMode, ProcessPriority, GpuPreference, LaunchApp, …`), `ChangeState` (`Pending, Applied, Reverted, Skipped, Failed`), `ChangeRestoreMode` (`OnDisable, OnExit, Manual`), `ChangeSessionState`, `ChangeRecord` (session/record id, owner/title, target, value name/format, prior existed/prior/requested, state, timestamps, process identity, elevation requirement; binary registry values as Base64), `ChangeSession`, `ChangeRevertResult`. |
 
**`Models/Settings/`**
 
| File | Lines | Content |
|---|---|---|
| `SettingsEnums.cs` | 44 | `AppThemePreference` (only `Dark` is rendered), `AccentPalette` (Purple/Blue/Teal/Amber/Rose), `SettingsLoadResult`, `SettingsChangedEventArgs`. |
| `AppSettings.cs` | 155 | Schema `CurrentSchemaVersion = 4` with fields: Theme, Accent, `PollingIntervalMs` (default 1000), `StartMinimized`, `ConfirmSensitiveOperations`, `GamingModeAutoDetect`, `GameProfiles`, and all overlay settings (`OverlayEnabledMetrics`, `OverlayMetricSelectionInitialized`, `OverlayFollowGamingMode`, `OverlayVisibleOnLaunch`, `OverlayOpacity=0.94`, `OverlayScale=1.0`, `OverlayLayout=Compact`, `OverlayAnchor=TopRight`, `OverlayOffsetX/Y=18`). `Normalize()` clamps/snaps everything (non-Dark theme → Dark, polling 500/1000/2000/5000, opacity 0.3–1.0, scale 0.75–1.5, allowed anchor, invalid profiles removed and duplicate Ids regenerated); `Clone()` deep-copies profiles. |
 
**`Models/Gaming/`**
 
| File | Lines | Content |
|---|---|---|
| `GameProfileSettings.cs` | 165 | Saved game profile: Id, name, executable path, enabled tweaks (power plan, Game Mode flag, priority, GPU preference), `Normalize()` for full validation; `GpuPreferenceString` builds/merges the `UserGpuPreferences` string (other tokens are preserved). |
| `GamingGraphicsTopology.cs` | 27 | `IsHybrid(graphics)`: does the system have both an integrated and a discrete adapter? Gate for enabling the GPU-preference tweak. |
| `GamingProcessClassifier.cs` | 18 | List of non-game process names (Steam tools, Wallpaper Engine, etc.) so they aren't suggested as a "game". |
 
**`Models/Startup/`**
 
| File | Lines | Content |
|---|---|---|
| `StartupEntry.cs` | 286 | Enums `StartupScope` (User/Machine), `StartupSource` (Run, Run32, StartupFolder, ScheduledTask), `StartupSignatureStatus`; `StartupRawEntry`; display `StartupEntry` (command, resolved exe, scope, publisher/signature, enabled, isRunning, path); `StartupCommandParser` (parses quoted/unquoted command lines and env vars); `StartupApproved` (builds/reads the binary `StartupApproved\Run` payload). |
 
### 6.3 `Services/` (logic, I/O and system access)
 
**`Services/` root**
 
| File | Lines | Role |
|---|---|---|
| `IAppInfoService.cs` | 14 | Contract for product name/version/build configuration. |
| `AppInfoService.cs` | 53 | Reads the same values from assembly metadata (no hard-coded version in the UI). |
| `INavigationService.cs` | 13 | `NavigateTo(AppSection)` for pages that send the user elsewhere (dashboard shortcuts). |
| `ISystemInfoProvider.cs` | 15 | Supplies `SystemSnapshot` for the dashboard. |
| `SystemInfoProvider.cs` | 41 | Public .NET APIs only (`RuntimeInformation`, `Environment`, `WindowsIdentity`). No WMI/registry/sensors. |
| `SectionIconCatalog.cs` | 32 | `AppSection → Icon key` mapping, the single source for the sidebar and page headers. |
 
**`Services/Hardware/`**
 
| File | Lines | Role |
|---|---|---|
| `IHardwareTelemetryProvider.cs` | 15 | One-shot telemetry read + `IsElevated`; no timer in the provider. |
| `LibreHardwareTelemetryProvider.cs` | 194 | Implementation on LibreHardwareMonitorLib: lazy `Computer` construction (about 0.9s, retry after 30s on failure), reads serialized with `SemaphoreSlim`, enables CPU/GPU/Memory/Storage, every failure → availability. |
| `HardwareSensorReader.cs` | 341 | Translates the LibreHardwareMonitor tree into the telemetry model: dedupes unnamed and duplicate `(kind,name)` sensors, applies validator and selection, builds CPU/each GPU/Memory/Storage. |
| `StorageVolumeReader.cs` | 110 | Reads fixed volumes from the OS (`DriveInfo`): always available, no elevation; storage doesn't depend on the library's disk detection. |
| `HardwarePollingService.cs` | 239 | Cadence owner: `PeriodicTimer` with selectable interval (floor 250ms), no overlapping reads, failure → unavailable snapshot; 15-minute retention (`RetentionWindow`) with buffer capacity computed from the interval (120–3600 samples). |
| `HardwareSampleHistory.cs` | 98 | Thread-safe, immutable-oriented ring buffer; `Retarget` when the interval changes. |
| `HardwareTelemetryMonitor.cs` | 126 | Global entry point: provider + polling + compatibility, marshals the snapshot to the UI thread with `DispatcherSynchronizationContext.Post`, `Updated(HardwareSnapshotEventArgs)` event. |
| `HardwareCompatibilityService.cs` | 95 | Builds `HardwareCompatibilityReport`: what is reported, what isn't, and why (the "requires elevation" story). |
| `ElevationDetector.cs` | 26 | Can the process touch interfaces that require admin (MSR)? |
| `ChartSeriesBuilder.cs` | 200 | Builds series from the same history (no second sampler): 1/5/15-minute window, blocked sensor → gap, not zero, temperature axis auto-scale, decimation. |
| `HardwareCapabilityReportService.cs` | 292 | Generates `%LOCALAPPDATA%\Iritool\capability-report.json` from the first snapshot, atomic write + `.bak`, quarantines a corrupt file, "Regenerate" from the latest sample while keeping the original timestamp. |
 
**`Services/Inventory/`**
 
| File | Lines | Role |
|---|---|---|
| `ISystemInventoryProvider.cs` | 15 | One-shot, read-only inventory contract; no elevation. |
| `WindowsInventoryProvider.cs` | 695 | Windows implementation: WMI, registry, `NetworkInterface`, `DriveInfo`. Each section is isolated; the driver query runs in parallel (whole snapshot 1.3–1.6s). Sections: OS, Processor, Graphics (real VRAM from `HardwareInformation.qwMemorySize`), Memory (each module), Motherboard/BIOS/UEFI, Storage (model, SSD/HDD, NVMe/SATA/USB, serial, volume letters), Network (no virtual/NDIS filter adapters), Drivers (max 40 rows with Device Manager problem code). |
| `WmiQuery.cs` | 139 | Defensive wrapper around `ManagementObjectSearcher`: timeout, row cap, error → empty result + diagnostic (`WmiQueryException`). |
| `InventoryRow.cs` | 149 | Read-only view of one WMI row with null-tolerant accessors and type conversion (`Text`, `Number`, `UInt64`…). |
| `InventoryParser.cs` | 531 | Converts raw rows to `InventoryEntry`; no WMI and no UI types so it is fully unit-testable (date/uptime, cores/cache, discrete vs shared VRAM, multi-GPU, memory module, firmware, WMI placeholders such as `(Standard disk drives)`, volumes, network format, driver problem, code table). |
 
**`Services/Operations/`**
 
| File | Lines | Role |
|---|---|---|
| `CommandRunner.cs` | 456 | **The only place a process is launched.** `CommandRequest/CommandResult/IConfiguration`: absolute path, fixed arguments, elevation requirement. Unelevated mode: pipe, live lines, exact exit code, cancel → kill process tree. Elevated mode: `cmd.exe /c "... > log 2>&1"` with the `runas` verb and log tail; UAC wait off the UI thread; cancel is not offered (an unelevated parent cannot kill it) and the UI says so. |
| `LogTailReader.cs` | 74 | Incrementally reads the elevated command's log file with `FileShare.ReadWrite` and preserves the read position. |
| `ConsoleTextDecoder.cs` | 215 | Detects the encoding of redirected output (UTF-16LE with BOM for SFC/DISM, UTF-8, OEM code page) + `ConsoleLineBuffer` for partial chunks and `\r` progress lines. |
| `IConfirmationService.cs` / `ConfirmationService.cs` | 14 / 38 | Confirmation prompt before system operations; WPF adapter with the active window as owner; if the dialog isn't shown the answer is "no". |
| `WindowsToolService.cs` | 169 | Orchestration: one operation at a time, resolves the fixed command line, runs with the runner, translates exit with `ExitCodeInterpreter`, rejects concurrent/duplicate runs. |
| `OperationHistoryStore.cs` | 105 | `%LOCALAPPDATA%\Iritool\operation-history.json`, last 60 records, newest first, atomic write. |
 
**`Services/Safety/` (system-change layer)**
 
| File | Lines | Role |
|---|---|---|
| `ChangeSessionService.cs` | 448 | Owner of the journal and the apply/restore cycle. A session with all records in `Pending` is written to disk **before** the first change (write-ahead) and a record goes to `Applied` after the executor succeeds. Reverse restore, per-record errors, current-value check before overwriting (external change → Skip with reason), `Pending` recovery after a crash. `RestoreAllActiveAsync(mode)` for OnExit and SessionEnding. |
| `ChangeJournalStore.cs` | 144 | `change-journal.json` with schema 1, atomic + `.bak`, quarantines a corrupt file, prunes finished history (active sessions are never pruned). |
| `ChangeExecutor.cs` | 374 | Performs/reverts one `ChangeRecord` for each `ChangeKind`: registry, power plan, Game Mode flag, process priority, GPU preference, closing/restoring an app. Every revert re-reads the live value. |
| `RegistryTarget.cs` | 205 | Registry location model: `RegistryRoot`, `RegistryValueFormat`, `RegistryTarget` (normalized/validated path), `RegistryValueData`, `RegistryReadResult`, `RegistryAccessDeniedException`, `RegistryPath` (normalize). |
| `RegistryAccessor.cs` | 207 | Exact allow-list guard: HKCU `Software\Microsoft\GameBar`, HKCU `Software\Microsoft\DirectX\UserGpuPreferences`, and `StartupApproved\Run|Run32|StartupFolder` under HKCU/HKLM. Case-insensitive matching, rejects trick paths and child keys before opening a handle. |
| `ChangePreview.cs` | 38 | `ChangePreview(Title, Summary, Changes)` + `IChangePreviewService`. |
| `ChangePreviewService.cs` | 41 | WPF adapter: `ChangePreviewDialog`; closing/failure to show = "no". |
| `ElevationService.cs` | 131 | Restart handshake with `runas`, single-instance mutex, parses `--restart-elevated` and `--autostart`, legacy process-name list for activating the running instance. |
| `AdministratorService.cs` | 71 | Real elevation state, restart with `runas` (`RestartOutcome.Started/Declined/Failed`) and management of the `Iritool Autostart` task. |
| `AutostartTaskService.cs` | 187 | Create/delete/query the `Iritool Autostart` scheduled task through `WindowsToolService`/`CommandRunner`: generated XML, current-user logon trigger, highest privileges, no timeout, `IgnoreNew`, fixed `--autostart` argument; all values XML-escaped. |
 
**`Services/Gaming/`**
 
| File | Lines | Role |
|---|---|---|
| `GamingModeService.cs` | 524 | Main Gaming Mode service: `GamingTweakSelection`, previewed+journalled application (existing power plan, Game Mode flag, priority up to High, GPU preference, graceful closing of selected apps), restore on OFF/exit/session end, hybrid-GPU gate for GPU preference, skips a tweak if PID/start-time/path no longer match. |
| `GameDetectionService.cs` | 265 | Detects a game from Steam library folders and running processes under `steamapps\common` + manual profiles; optional automatic apply/restore (default off) with bounded polling and dispose. |
| `SteamLibrary.cs` | 207 | `SteamLibraryParser` for `libraryfolders.vdf` (small KeyValues, paths only) + `ISteamLibraryLocator` / `SteamLibraryLocator` to find the Steam root from the registry/default path. |
 
**`Services/Startup/`**
 
| File | Lines | Role |
|---|---|---|
| `WindowsStartupEntryProvider.cs` | 311 | Read-only enumeration: Run in HKCU/HKLM both views (32/64), user and common Startup folders, and scheduled tasks with a logon trigger. Failures → diagnostics. |
| `StartupEntryBuilder.cs` | 182 | Builds `StartupEntry` from a raw row, `SignatureResult`/`IFileSignatureVerifier`, `StartupApprovedState`, `StartupProtectionPolicy` (protected Windows/security rows), `StartupTogglePlanner` (change plan only on `StartupApproved`). |
| `AuthenticodeVerifier.cs` | 266 | Authenticode signature verification with `WinVerifyTrust` (file hash, not just presence of a certificate), offline revocation check from the local cache so it doesn't hang on systems without internet. |
| `ScheduledTaskXmlParser.cs` | 173 | Parses scheduled-task XML (`schtasks /query /xml`) into `ScheduledTaskInfo` (path, command, args, trigger). |
 
**`Services/Platform/`**
 
| File | Lines | Role |
|---|---|---|
| `PowerPlanService.cs` | 172 | Lists and gets the active power plan with `powercfg` (`PowerPlanInfo(Guid, Name)`), changes and restores the plan. |
| `ProcessController.cs` | 364 | Process snapshot (PID, start time, path, RAM), `ProcessPriorityLevel` (Idle..High; Realtime not offered), priority change, graceful/forced close, restore by path. |
 
**`Services/Benchmark/`**
 
| File | Lines | Role |
|---|---|---|
| `CpuRenderBenchmarkService.cs` | ~135 | Bounded, cancellable CPU render workload (`CpuBenchmarkResult`: duration, pixels, pixels/second, peak temperature, `HasThermalGuard`) + `BenchmarkSafetyException`. If the MSR sensor is unavailable (non-admin window), the peak temperature is accepted missing only with the `acceptMissingThermalGuard` parameter, otherwise the run is refused. Independent of, and not comparable with, Cinebench. |
| `PowerSourceProvider.cs` | 39 | `IPowerSourceProvider` / `WindowsPowerSourceProvider.IsOnBattery` for the battery confirmation gate. |
 
**`Services/Settings/`**
 
| File | Lines | Role |
|---|---|---|
| `ISettingsService.cs` | 32 | Contract: `Current`, `SettingsPath`, `LastLoad`, `Changed`, `LoadAsync`, `SaveAsync`, reset/import/export. |
| `IResetConfirmation.cs` | 12 | Contract for confirmation before an irreversible action (reset); kept separate from the VM to remain testable. |
| `ResetConfirmation.cs` | 33 | WPF adapter: shows the confirmation dialog with the active window as owner; if the dialog can't open it returns "no" so reset isn't assumed. |
| `SettingsService.cs` | 307 | JSON store at `%LOCALAPPDATA%\Iritool\settings.json`: atomic write (temp + replace) with a rotating `.bak`, quarantines an unreadable file (doesn't delete), doesn't overwrite a healthy file on load error, versioned schema, ignores unknown properties, `SemaphoreSlim` for serialization, and marshals `Changed` onto the construction-time `SynchronizationContext` (fixes the cross-thread error). Constructor with a `directory` parameter for tests. |
 
**`Services/Storage/`**
 
| File | Lines | Role |
|---|---|---|
| `AtomicJsonFile.cs` | 104 | Shared atomic write/read (capability report, history, journal): temp + replace, `.bak`, sets a corrupt file aside. |
 
**`Services/Theming/`**
 
| File | Lines | Role |
|---|---|---|
| `IThemeService.cs` | 59 | Contract + `AccentDefinition(Deep, Main, Neon)` + `ThemePalette` + `Applied` event. |
| `ThemeService.cs` | ~375 | Applies theme/accent at runtime by swapping the palette in `Application.Resources`; all brushes in `Colors.xaml` and `Styles.xaml` are referenced with `DynamicResource`, so a single dictionary swap repaints the whole window in place (no window rebuild, no re-reading `Styles.xaml`); Fluent keys (`SystemAccentColor*`, `AccentFillColor*`) are updated each time but the Fluent dictionaries are loaded only once. |
 
### 6.4 `ViewModels/` (MVVM — all `ObservableObject`)
 
| File | Lines | Role |
|---|---|---|
| `MainViewModel.cs` | 343 | Shell VM and `INavigationService`: builds the 9 pages, `NavigationItems`, `SelectedItem`/`CurrentPage`, `IsSidebarExpanded`, session recovery banner (`HasRecovery`, `RecoveryTitle/Detail`, `RestoreRecoveryCommand`, `DismissRecoveryCommand`), `EnsureLoadedAsync` for heavy pages (System Information, Startup, Gaming) and `SetActive` on the monitor. |
| `NavigationItemViewModel.cs` | 33 | One sidebar row: `Section`, `Title`, `IconKey`, `Page` instance, `IsSelected`. |
| `DashboardViewModel.cs` | 358 | Dashboard: time-based greeting, date, four `MetricTileViewModel`s, ambient overview, Gaming Mode status, `QuickTools` that really navigate, version footer; `EnsureLoadedAsync`. |
| `MetricTileViewModel.cs` | 87 | One dashboard tile: main percentage, animated bar, three secondary readings, status chip (LIVE/PARTIAL/ELEVATION/ERROR), timestamp and read duration. |
| `HardwareMonitorViewModel.cs` | 343 | Monitor page: per-device card (`DeviceCardViewModel`), readings (`ReadingRowViewModel`), charts (`HardwareChartViewModel`), 1/5/15 range options, elevation note; subscribes only when the page is active. |
| `DeviceCardViewModel.cs` | 46 | One device card (CPU, each GPU, memory) with fixed rows updated in place. |
| `ReadingRowViewModel.cs` | 94 | One labeled reading with instance reuse; `AvailabilityText` builds the reason/unit text. |
| `VolumeRowViewModel.cs` | 27 | One volume row with a usage bar. |
| `HardwareChartViewModel.cs` | 129 | One live chart: headline value, series for the selected range, explicit state when there's nothing to draw; + `ChartRangeOptionViewModel`. No timer. |
| `WindowsToolsViewModel.cs` | 601 | Windows Tools page: only one active operation, output streamed into a shared console, persistent history, `ToolSectionViewModel`/`ToolCardViewModel`, confirmation for repair operations, no cancel for elevated runs with an explanation. |
| `GamingModeViewModel.cs` | 911 | Gaming Mode page: opt-in switch, profile/game selection from `Detection`, power plan selection (`PowerPlanOptionViewModel`), priority, GPU preference (hybrid gate from inventory), closing background apps (`BackgroundAppOptionViewModel`), profiles (`GameProfileRowViewModel`), battery warning, preview and restore. |
| `StartupManagerViewModel.cs` | 443 | Startup Manager page: read-only enumeration, enable/disable through the safety layer, locks HKLM rows without admin, protected-row warning, Open file location / Copy command (`StartupRowViewModel`). |
| `SystemInformationViewModel.cs` | 324 | System Information page: inventory on first visit/Refresh, capability card (PARTIAL · 9 of 16), Regenerate, diagnostics. |
| `DesktopOverlayViewModel.cs` | 344 | Overlay page + HUD projection: builds/shows `OverlayWindow`, selected metrics (`OverlayMetricViewModel`), lock/edit, Compact/Vertical layout, opacity/scale, anchor, hotkey status, follow Gaming Mode, projection duration measurement. |
| `BenchmarkViewModel.cs` | ~215 | Benchmarks & Tests page: start/stop workload, `CanStartNow` for buttons, safety gates (fresh sample, battery confirmation, explicit confirmation to run without a thermal guard when temperature isn't readable), `GateText` separate from the run message, estimated progress bar with a DispatcherTimer, workload-specific result display. |
| `SettingsViewModel.cs` | 526 | Settings page: theme (Dark only), five accents (`AccentOptionViewModel`), polling interval (`PollingOptionViewModel`), two switches (`StartMinimized`, `ConfirmSensitiveOperations`), Gaming auto-detect, settings-file card (Reset/Export/Import/Show in folder), restart as administrator, autostart status; immediate apply + atomic save, coalesces theme+accent change. |
| `SettingsOptions.cs` | 48 | Option VMs for accent and polling interval (they carry the enum/int value, not an option instance). |
| `PlaceholderPageViewModel.cs` | 48 | Contract for Phase 1 skeleton pages. **Currently no page inherits from it** (all pages are now real). |
 
### 6.5 `Views/` (XAML + thin code-behind)
 
| File | Lines | Role |
|---|---|---|
| `DashboardView.xaml(.cs)` | 366 / 30 | Welcome header, 4 tiles, overview, Gaming card, quick access, footer. Code-behind: minor scroll/focus. |
| `WindowsToolsView.xaml(.cs)` | 320 / 59 | Tool cards with category chip, resolved command line, output console, history. Code-behind: console behavior (scroll to end/selection). |
| `GamingModeView.xaml(.cs)` | 332 / 13 | Main switch, selections, profiles, warnings. |
| `HardwareMonitorView.xaml(.cs)` | 319 / 29 | Device cards, charts (`MetricChart`), range chips. |
| `DesktopOverlayView.xaml(.cs)` | 100 / 13 | HUD controls (show/lock/layout/opacity/scale/anchor/hotkey). |
| `BenchmarkView.xaml(.cs)` | ~68 / 9 | Benchmark run controls, thermal-guard/battery warning with confirmation checkbox, progress bar, result and history cards. |
| `StartupManagerView.xaml(.cs)` | 217 / 13 | Startup rows table and buttons. |
| `SystemInformationView.xaml(.cs)` | 476 / 13 | Inventory sections, capability card, history/diagnostics. |
| `SettingsView.xaml(.cs)` | ~292 / 30 | Settings cards; responsive rows with separate label/control columns (fixes overlap); status bar pinned to the bottom of the page with error/success color, auto-clear and `LiveSetting` for screen readers. |
| `OverlayWindow.xaml(.cs)` | 31 / 218 | Standalone HUD window: topmost, transparent, `WS_EX_TOOLWINDOW|WS_EX_NOACTIVATE`, `WS_EX_TRANSPARENT` when locked, DragMove in edit mode, clamp to the monitor work area, anchor-relative save, `RegisterHotKey(Ctrl+Alt+O)` and release of hook/hotkey in `Closed`. |
| `ConfirmOperationDialog.xaml(.cs)` | 72 / 47 | Repair-operation confirmation: classification, exact command, elevation reason, timeout. |
| `ConfirmResetDialog.xaml(.cs)` | ~95 / 33 | Confirmation of the settings page's only irreversible action: file to be replaced, warning about losing values; Cancel is the default button. |
| `ChangePreviewDialog.xaml(.cs)` | 100 / 43 | System-change preview: current and requested value + elevation requirement; closing = no change. |
 
### 6.6 `Controls/`
 
| File | Lines | Role |
|---|---|---|
| `MetricChart.cs` | 198 | A `FrameworkElement` that draws from `ChartSeries` in `OnRender`: one invalidate per second, no visual tree built; an unavailable sample = gap and a dashed baseline (not a zero line). |
| `UsageBar.xaml(.cs)` | 22 / 72 | Usage bar with value animation (DPI-safe, no bitmaps). |
| `PhasePlaceholder.xaml(.cs)` | 90 / 16 | Shared presentation for Phase 1 skeleton pages. **Now unreferenced** (all pages are implemented). |
 
### 6.7 `Converters/`
 
| File | Lines | Role |
|---|---|---|
| `IconKeyToGeometryConverter.cs` | 36 | Icon key (e.g. `"Monitor"`) → `Icon.Monitor` from `Assets/Icons.xaml`; the VM stays free of WPF types. |
| `AvailabilityBrushConverter.cs` | 36 | `SensorAvailability` → status color. |
| `CompatibilityLevelBrushConverter.cs` | 35 | Overall compatibility level → green/amber/red (separate from the sensor converter). |
| `OperationConverters.cs` | 52 | `ThemeBrush` (key resolve with fallback) + `OperationKindBrushConverter` + `OperationOutcomeBrushConverter`. |
| `VisibilityConverters.cs` | 54 | `CountToVisibilityConverter`, `NullToVisibilityConverter`, `InverseBooleanConverter`. |
 
---
 
## 7. Test Project — `iritool.Tests/`
 
`iritool.Tests.csproj` (26 lines): `net10.0-windows`, `UseWPF`, `x64`, `IsPackable=false`,
`ProjectReference` to the app with `SetConfiguration="Configuration=Debug"`.
 
| File | `[Fact]/[Theory]` count | Coverage |
|---|---|---|
| `OperationTests.cs` | 35 | Catalog paths/arguments/classification/confirmation link, exit-code table, one-operation-at-a-time, elevation rejection, console text. |
| `InventoryParserTests.cs` | 24 | OS date/uptime, empty row, cores/cache, discrete vs shared VRAM, multi-vendor GPU, module layout, firmware, disk placeholder, volume, network format, driver problem, code table. |
| `GamingModeTests.cs` | 21 | Gaming Mode cycle, Steam tool filter, stale process, hybrid-GPU gate, GPU preference merge/restore, profile migration. |
| `SettingsTests.cs` | 20 | Defaults, round-trip, backup, corrupt quarantine, migration, unknown property, interval snap, export/import, reset, notification, appearance commands. |
| `StartupManagerTests.cs` | 16 | `StartupApproved` payloads, protected rows, escaping in task XML, enumeration. |
| `CommandExecutionTests.cs` | 15 | echo/exit code/stderr/missing file/cancel/timeout, UTF-16+BOM, chunk splitting, `\r` lines, elevated log tail. |
| `ChangeJournalTests.cs` | 13 | Write-ahead ordering, recovery after a crash before the marker, reverse and idempotent restore, external-change protection, restoring a nonexistent value, corrupt journal quarantine, rejecting a path outside the allow-list, rejected preview. |
| `ChartSeriesBuilderTests.cs` | 12 | Empty history, window filter, absent sensor without zero, temperature axis, multi-GPU, unknown GPU, last value, decimation. |
| `CapabilityReportTests.cs` | 10 | Supported/unsupported split, permission text, no-GPU case, first-run stability, regenerate, corrupt file, round-trip. |
| `SensorSelectionTests.cs` | 8 | Named/fallback selection/`SelectExact`/`SelectMax`, duplicate removal. |
| `SensorValueValidatorTests.cs` | 7 | Plausibility gate by sensor type, NaN/Inf. |
| `HardwarePollingServiceTests.cs` | 7 | Non-overlapping cadence, cancel, containing a faulting provider. |
| `HardwareAvailabilityTests.cs` | 7 | Availability aggregation and dominant reason. |
| `HardwareCompatibilityServiceTests.cs` | 6 | Compatibility levels including the unelevated case. |
| `BenchmarkSafetyTests.cs` | 4 | Missing temperature, battery confirmation, thermal stop, cancel. |
| `InventoryProbeTests.cs` | 3 | Real/light inventory-reading probes. |
| `InstanceActivationTests.cs` | 3 | Coverage of current/legacy process names, de-duplication, invalid name. |
| `HardwareSampleHistoryTests.cs` | 3 | Buffer bounds, retarget. |
| `InventorySmokeTests.cs` | 2 | Smoke test on the real implementation. |
| `StartupMarkupTests.cs` | 1 | STA test: loads app resources, shell and all 9 pages (`XamlParseException` regression guard). |
| `TestInfrastructure.cs` | — | Shared infrastructure (temp file/isolated folder). |
| `TestDoubles.cs` | — | Fake telemetry provider/services. |
| `Phase3TestDoubles.cs` | — | Fakes for registry, power plan, process, preview, Steam for Phase 3. |
 
Test-suite rules (per NOTES): no elevated command is run, no real system setting is changed, and
the only processes launched are `cmd.exe /c echo`, `exit 3`, and a loopback `ping` for the
timeout/cancel test.
 
---
 
## 8. What Is Written/Read on Disk at Runtime
 
| Path | Writer | Description |
|---|---|---|
| `%LOCALAPPDATA%\Iritool\settings.json` | `SettingsService` | Settings, schema 4, atomic + `.bak`, a corrupt file is quarantined. |
| `%LOCALAPPDATA%\Iritool\capability-report.json` | `HardwareCapabilityReportService` | First-run capability report; regenerated manually. |
| `%LOCALAPPDATA%\Iritool\operation-history.json` | `OperationHistoryStore` | Last 60 runs of Windows tools. |
| `%LOCALAPPDATA%\Iritool\change-journal.json` | `ChangeJournalStore` | System-change (write-ahead) journal for Gaming Mode and Startup Manager. |
| Temporary log file | `CommandRunner` (elevated mode) | Output of `cmd /c ... > log` that `LogTailReader` follows. |
| Registry (write, allow-list only) | `RegistryAccessor` | `HKCU\Software\Microsoft\GameBar`, `HKCU\Software\Microsoft\DirectX\UserGpuPreferences`, `StartupApproved\Run|Run32|StartupFolder` under HKCU/HKLM. |
| Scheduled Task | `AutostartTaskService` | Fixed task `Iritool Autostart` with the `--autostart` argument. |
| Steam folders (read only) | `SteamLibraryLocator` | `libraryfolders.vdf` and `steamapps\common`. |
| `%LOCALAPPDATA%\Programs\iritool` | Installer | Per-user install location. |
 
---
 
## 9. Safety and Permissions Model (global summary)
 
- The manifest is always `asInvoker`; the program never requests UAC by itself. Elevation is only
  per-action (`runas` for SFC/DISM/CHKDSK and restart).
- Every system change passes through `ChangeSessionService`: preview → journal `Pending` → execute →
  `Applied` → reverse restore with a current-value check.
- Command lines come only from `OperationCatalog`; absolute System32 path, fixed arguments.
- Registry only through the exact allow-list; trick paths/child keys are rejected.
- Things that deliberately do not exist: `chkdsk /f`, Realtime priority, creating power plans,
  stopping/disabling Windows services, in-game injection, FPS counter, rendering over exclusive
  fullscreen.
- An unavailable value is always `—` with a reason; a fake zero is never shown anywhere.
---
 
## 10. Phase History (summary of `NOTES.md`)
 
| Phase | Content | Recorded status |
|---|---|---|
| 1 | Fluent shell, 8-section sidebar, dashboard with placeholders, skeleton pages | Done and verified |
| 2A | Telemetry engine (LibreHardwareMonitor), 55 tests | Complete |
| 2B | Inventory (WMI), live charts, capability report | Complete on this machine |
| 2C | Settings persistence + theme/accent (Schema 1), fixes for thread/style/palette bugs | Complete |
| 2D | Windows Tools (SFC/DISM/CHKDSK scan) + confirmation + history | Complete, except one real elevated run |
| 3 | Journal layer, Startup Manager, Gaming Mode (279 tests) | Complete, manual checks remain |
| 5 | CPU render benchmark, conditional Hardware Monitor subscription, fuller overlay (289 tests) | Complete |
| — | Open-source handoff: `iritool` name, version `1.0`, English README, MIT, Inno installer | Complete |
| Hotfix | Single-instance with legacy names, icon `XamlParseException` with site-of-origin pack URI (290 tests) | Fixed |
| Stability | One-way binding for `AutostartTaskName`, responsive rows, Steam tool filter, stale-process/hybrid gate | Fixed |
| 4 | Removal of Light/System (Schema 4), real overlay, `Ctrl+Alt+O` hotkey, a11y, `PROJECT_GRAPH.md` | Complete, manual multi-monitor/DPI check remains |
 
---
 
## 11. Technical Debt and Unreferenced Files
 
| Item | Status |
|---|---|
| `Models/HardwareMetric.cs` | No references; left over from Phase 2A and, per NOTES, deliberately not removed. |
| `ViewModels/PlaceholderPageViewModel.cs` | No page inherits from it (all pages are real). |
| `Controls/PhasePlaceholder.xaml(.cs)` | No View uses it. |
| Remaining manual checks | UAC accept/decline, creating/deleting the autostart task, real `StartupApproved` and power-plan writes, overlay multi-monitor/DPI behavior, hotkey conflict with another app. |
| Unverified hardware | AMD CPU/GPU, Intel Arc, desktops with multiple discrete GPUs, systems without SMBIOS data. |
| CPU temp/clock/power without admin | `AccessDenied`; the program deliberately does not request elevation; the benchmark does not run on such a machine. |
 
---
 
## 12. `ui-ux-pro-max-skill-2.15.0/` — Reference Folder (not part of the product)
 
A UI/UX design skill with 662 files and about 23 MB, kept in the repository only as a design
reference. No app file references it and it is not used in build/install. Main structure:
 
```
ui-ux-pro-max-skill-2.15.0/
└─ ui-ux-pro-max-skill-2.15.0/
   ├─ README.md / README.zh.md / SKILL.md / skill.json   Skill intro and manifest
   ├─ CLAUDE.md / CONTRIBUTING.md / CODE_OF_CONDUCT.md / SECURITY.md
   ├─ cli/                    CLI tool + assets (data, scripts, skills, templates/platforms)
   ├─ src/ui-ux-pro-max/      Source version of the same package (data, scripts, templates, skills)
   ├─ gallery/                Gallery app (Next.js) for browsing components
   ├─ stack/                  Reference stack + `juniper-audit` example
   ├─ docs/journals/          Planning journals for search data and phases 6/7
   ├─ projects/               Sample projects (healthcare-dashboard, portfolio-dark, saas-landing)
   ├─ preview/, screenshots/, scripts/
   └─ .claude/, .claude-plugin/, .github/
```
 
Skill categories inside `cli/assets/skills/`: `banner-design`, `brand`, `design`, `design-system`,
`slides`, `ui-styling` — each with `SKILL.md` and `references/`.
 
---
 
## 13. Installer (installer/)
 
The installer is separate from the app itself and its UI is WPF; Inno Setup remains in the repository
only as a legacy path and is not used in the current build (its wizard can't be replaced with WPF, and
in silent mode it offers no advanced API for install percentage).
 
| File | Role |
|---|---|
| `installer/build-installer.ps1` | 5 steps: publish app → publish uninstaller → build `payload.zip` → publish WPF UI → produce `artifacts/installer/iritool-Setup-<v>-x64.exe` |
| `iritool-installer/App.xaml(.cs)` | Entry point, `--capture <dir>` path for automatic screenshots of each page |
| `iritool-installer/MainWindow.xaml(.cs)` | Two-column 900×580 shell, custom chrome, DWM rounded corners, six pages |
| `iritool-installer/Theme/Palette.xaml` | Dark palette `#171719` / `#202026` / `#29282F` + accent `#8055FF`/`#A56BFF` |
| `iritool-installer/Theme/Styles.xaml` | Buttons, switch, TextBox, purple ProgressBar, card, caption buttons |
| `iritool-installer/ViewModels/InstallerViewModel.cs` | Page state, path validation, warning for installed version/running app, smoothed percentage |
| `iritool-installer/Services/InstallEngine.cs` | Real install: extracts the embedded zip to a staging folder, then move; percentage based on bytes written |
| `iritool-installer/Services/ShellShortcut.cs` | Creates `.lnk` with `IShellLink` and the official icon |
| `iritool-installer/Services/UninstallRegistration.cs` | HKCU Uninstall\iritool key (DisplayIcon, UninstallString, QuietUninstallString) |
| `iritool-uninstaller/Program.cs` | Removes files/shortcuts/registry key, self-copies to `%TEMP%` for complete removal; `%LOCALAPPDATA%\Iritool` is left untouched |
| `Assets/icon.ico` (both projects) | Byte-for-byte copy of the root `icon.ico`; no icon was regenerated |
 
---
 
## 14. Everyday Commands
 
```powershell
# Build and test the whole solution
dotnet build .\iritool\iritool.slnx -c Release
dotnet test  .\iritool\iritool.slnx -c Release
 
# Tests only
cd .\iritool\iritool.Tests
dotnet test
 
# Run the output
.\iritool\iritool\bin\Release\net10.0-windows\iritool.exe
 
# Build the installer (output: artifacts/installer/iritool-Setup-<version>-x64.exe)
powershell -ExecutionPolicy Bypass -File .\installer\build-installer.ps1
# With -SkipAppPublish the existing app publish is used
```
 
---
 
## 15. Naming and Maintenance Conventions
 
- The project, solution, namespace, folders and exe file name stay `iritool` until the final
  release phase, and only `AssemblyName`/`Product` is set to `iritool`.
- Each page = one `XxxViewModel` + one `XxxView.xaml`, mapped in `App.xaml` with a `DataTemplate`.
- Services are injected through constructors and there is no global singleton; `MainViewModel` is
  the only place that builds page VMs.
- Color/theme changes only via `DynamicResource` keys and the `Colors.xaml` palette.
- Every system feature must be: opt-in, previewable, reversible, and covered by fake-backed tests.
- After each phase, `NOTES.md` and this `map.md` must be updated.
