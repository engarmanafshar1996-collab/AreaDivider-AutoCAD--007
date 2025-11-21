# راهنمای حل مشکل MSBuild در GitHub Actions

## شرح مشکل

هنگام اجرای workflow در GitHub Actions، خطای زیر رخ می‌دهد:

```
Run microsoft/setup-msbuild@v2
C:\ProgramData\Chocolatey\bin\vswhere.exe -products * -requires Microsoft.Component.MSBuild -property installationPath -latest -version 2019
Error: Unable to find MSBuild.
```

## علت مشکل

مشکل از چند عامل ناشی می‌شود:

1. **استفاده نادرست از پارامتر `vs-version`**: مقدار `'2019'` به عنوان یک string ساده قابل قبول نیست. باید از syntax خاص vswhere استفاده شود.

2. **عدم وجود Visual Studio 2019 در runner**: Runner های `windows-latest` معمولاً آخرین نسخه Visual Studio (2022) را دارند و ممکن است VS 2019 نصب نباشد.

3. **مسیرهای مطلق در فایل csproj**: استفاده از مسیرهای مطلق مانند `C:\Program Files\Autodesk\AutoCAD 2026\` در محیط CI/CD کار نمی‌کند.

## راه‌حل‌های پیشنهادی

### ✅ راه‌حل 1: حذف پارامتر vs-version (توصیه شده)

**فایل**: `.github/workflows/build.yml`

این ساده‌ترین و بهترین راه‌حل است. به `setup-msbuild` اجازه دهید که خودش آخرین نسخه موجود MSBuild را پیدا کند:

```yaml
- name: Setup MSBuild
  uses: microsoft/setup-msbuild@v2
```

**مزایا**:
- ساده و قابل اعتماد
- با تمام runner های GitHub سازگار است
- نیازی به مشخص کردن نسخه خاص ندارد

**معایب**:
- کنترل کمتری روی نسخه MSBuild دارید

---

### ✅ راه‌حل 2: استفاده از windows-2019

**فایل**: `.github/workflows/build-alternative.yml`

اگر حتماً نیاز به Visual Studio 2019 دارید، از runner مخصوص آن استفاده کنید:

```yaml
jobs:
  build:
    runs-on: windows-2019
```

**مزایا**:
- Visual Studio 2019 از قبل نصب است
- سازگاری بیشتر با پروژه‌های قدیمی‌تر

**معایب**:
- Runner قدیمی‌تر است و ممکن است در آینده deprecated شود
- ممکن است کندتر از windows-latest باشد

---

### ⚠️ راه‌حل 3: نصب دستی Build Tools

**فایل**: `.github/workflows/build-with-buildtools.yml`

نصب دستی Visual Studio Build Tools قبل از build:

```yaml
- name: Setup Visual Studio Build Tools
  run: |
    choco install visualstudio2019buildtools --package-parameters "--add Microsoft.VisualStudio.Workload.MSBuildTools --quiet --norestart"
```

**مزایا**:
- کنترل کامل روی نسخه و component های نصب شده

**معایب**:
- زمان build بیشتر می‌شود (نصب Build Tools حدود 5-10 دقیقه طول می‌کشد)
- پیچیده‌تر است
- ممکن است با مشکلات نصب مواجه شود

---

## تغییرات لازم در فایل پروژه

### فایل `AreaDivider.csproj`

مسیرهای مطلق را با مسیرهای نسبی جایگزین کنید:

**قبل** (اشتباه):
```xml
<Reference Include="AcCoreMgd">
  <HintPath>C:\Program Files\Autodesk\AutoCAD 2026\AcCoreMgd.dll</HintPath>
  <Private>False</Private>
</Reference>
```

**بعد** (درست):
```xml
<Reference Include="AcCoreMgd">
  <HintPath>..\References\AcCoreMgd.dll</HintPath>
  <Private>False</Private>
</Reference>
```

### ساختار پوشه‌های پروژه

```
your-repo/
├── .github/
│   └── workflows/
│       └── build.yml
├── AreaDivider/
│   ├── AreaDivider.csproj
│   ├── Commands.cs
│   └── AreaDividerForm.cs
└── References/
    ├── AcCoreMgd.dll
    ├── AcDbMgd.dll
    └── AcMgd.dll
```

## مراحل پیاده‌سازی

### گام 1: آپدیت فایل csproj

فایل `AreaDivider.csproj` را با نسخه اصلاح شده جایگزین کنید که از مسیرهای نسبی استفاده می‌کند.

### گام 2: ایجاد پوشه References

در root پروژه، پوشه `References` ایجاد کنید و فایل‌های DLL اتوکد را در آن قرار دهید:

```bash
mkdir References
# کپی فایل‌های dll به این پوشه
```

### گام 3: انتخاب و اعمال workflow

یکی از سه فایل workflow را انتخاب کنید:

- **توصیه می‌شود**: `build.yml` (حذف پارامتر vs-version)
- جایگزین: `build-alternative.yml` (استفاده از windows-2019)
- پیشرفته: `build-with-buildtools.yml` (نصب دستی)

فایل انتخابی را در مسیر `.github/workflows/` قرار دهید.

### گام 4: Commit و Push

```bash
git add .
git commit -m "Fix MSBuild setup in GitHub Actions"
git push
```

### گام 5: بررسی نتیجه

به تب **Actions** در repository خود بروید و نتیجه build را بررسی کنید.

## نکات مهم

1. **فایل‌های DLL اتوکد**: حتماً فایل‌های `AcCoreMgd.dll`, `AcDbMgd.dll`, و `AcMgd.dll` را در پوشه `References` قرار دهید.

2. **Platform Target**: پروژه شما روی `x64` تنظیم شده است. اطمینان حاصل کنید که این با نیازهای شما مطابقت دارد.

3. **Target Framework**: پروژه از `.NET Framework 4.8` استفاده می‌کند که نیاز به MSBuild مخصوص .NET Framework دارد.

4. **Private=False**: این تنظیم باعث می‌شود DLL های اتوکد در خروجی کپی نشوند (چون در محیط اتوکد از قبل موجودند).

## عیب‌یابی

### اگر همچنان خطای MSBuild دریافت کردید:

1. **بررسی لاگ کامل**: در GitHub Actions، لاگ کامل را بررسی کنید تا ببینید vswhere چه چیزی پیدا می‌کند.

2. **اضافه کردن debug output**:
```yaml
- name: Check MSBuild
  run: |
    vswhere -all
    msbuild -version
```

3. **بررسی runner image**: مطمئن شوید که runner مورد استفاده شما Visual Studio یا Build Tools دارد.

### اگر خطای NuGet دریافت کردید:

ممکن است نیاز باشد packages را restore کنید:

```yaml
- name: Restore NuGet packages
  run: nuget restore AreaDivider/AreaDivider.csproj
```

### اگر خطای Reference دریافت کردید:

مطمئن شوید که:
- فایل‌های DLL در پوشه `References` موجودند
- مسیرهای HintPath در csproj صحیح هستند
- فایل‌ها در git commit شده‌اند

## منابع مفید

- [microsoft/setup-msbuild Documentation](https://github.com/microsoft/setup-msbuild)
- [GitHub Actions Runner Images](https://github.com/actions/runner-images)
- [vswhere Documentation](https://github.com/microsoft/vswhere)

## پشتیبانی

اگر همچنان با مشکل مواجه هستید، لطفاً موارد زیر را بررسی کنید:

1. لاگ کامل GitHub Actions
2. محتوای فایل csproj
3. ساختار پوشه‌های پروژه
4. نسخه runner مورد استفاده
