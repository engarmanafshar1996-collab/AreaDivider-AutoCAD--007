# مقایسه تغییرات - قبل و بعد از رفع مشکل

## 1. فایل Workflow

### ❌ قبل (با خطا)

```yaml
name: Build AutoCAD Add-in

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]
  workflow_dispatch:

jobs:
  build:
    runs-on: windows-latest

    steps:
    - uses: actions/checkout@v4

    - name: Setup MSBuild
      uses: microsoft/setup-msbuild@v2
      with:
        vs-version: '2019'  # ❌ این پارامتر باعث خطا می‌شود

    - name: Build Solution
      run: msbuild AreaDivider/AreaDivider.csproj /p:Configuration=Release /p:Platform="Any CPU"

    - name: Upload Artifact
      uses: actions/upload-artifact@v4
      with:
        name: AreaDivider-Release
        path: AreaDivider/bin/Release/AreaDivider.dll
```

### ✅ بعد (اصلاح شده)

```yaml
name: Build AutoCAD Add-in

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]
  workflow_dispatch:

jobs:
  build:
    runs-on: windows-latest

    steps:
    - uses: actions/checkout@v4

    # ✅ حذف پارامتر vs-version
    - name: Setup MSBuild
      uses: microsoft/setup-msbuild@v2

    # ✅ اضافه شدن NuGet restore
    - name: Setup NuGet
      uses: nuget/setup-nuget@v1
      
    - name: Restore NuGet packages
      run: nuget restore AreaDivider/AreaDivider.csproj -PackagesDirectory packages

    # ✅ اضافه شدن TargetFrameworkVersion
    - name: Build Solution
      run: msbuild AreaDivider/AreaDivider.csproj /p:Configuration=Release /p:Platform="Any CPU" /p:TargetFrameworkVersion=v4.8

    - name: Upload Artifact
      uses: actions/upload-artifact@v4
      with:
        name: AreaDivider-Release
        path: AreaDivider/bin/Release/AreaDivider.dll
```

---

## 2. فایل AreaDivider.csproj

### ❌ قبل (با مسیر مطلق)

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFramework>net48</TargetFramework>
    <PlatformTarget>x64</PlatformTarget>
    <OutputPath>bin\$(Configuration)\</OutputPath>
    <AppendTargetFrameworkToOutputPath>false</AppendTargetFrameworkToOutputPath>
    <RootNamespace>AreaDivider</RootNamespace>
    <AssemblyName>AreaDivider</AssemblyName>
    <GenerateAssemblyInfo>false</GenerateAssemblyInfo>
  </PropertyGroup>

  <ItemGroup>
    <!-- ❌ مسیرهای مطلق که در CI کار نمی‌کنند -->
    <Reference Include="AcCoreMgd">
      <HintPath>C:\Program Files\Autodesk\AutoCAD 2026\AcCoreMgd.dll</HintPath>
      <Private>False</Private>
    </Reference>
    <Reference Include="AcDbMgd">
      <HintPath>C:\Program Files\Autodesk\AutoCAD 2026\AcDbMgd.dll</HintPath>
      <Private>False</Private>
    </Reference>
    <Reference Include="AcMgd">
      <HintPath>C:\Program Files\Autodesk\AutoCAD 2026\AcMgd.dll</HintPath>
      <Private>False</Private>
    </Reference>
    <Reference Include="System" />
    <Reference Include="System.Core" />
    <Reference Include="System.Data" />
    <Reference Include="System.Xml" />
  </ItemGroup>

</Project>
```

### ✅ بعد (با مسیر نسبی)

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFramework>net48</TargetFramework>
    <PlatformTarget>x64</PlatformTarget>
    <OutputPath>bin\$(Configuration)\</OutputPath>
    <AppendTargetFrameworkToOutputPath>false</AppendTargetFrameworkToOutputPath>
    <RootNamespace>AreaDivider</RootNamespace>
    <AssemblyName>AreaDivider</AssemblyName>
    <GenerateAssemblyInfo>false</GenerateAssemblyInfo>
  </PropertyGroup>

  <ItemGroup>
    <!-- ✅ مسیرهای نسبی که در CI کار می‌کنند -->
    <Reference Include="AcCoreMgd">
      <HintPath>..\References\AcCoreMgd.dll</HintPath>
      <Private>False</Private>
    </Reference>
    <Reference Include="AcDbMgd">
      <HintPath>..\References\AcDbMgd.dll</HintPath>
      <Private>False</Private>
    </Reference>
    <Reference Include="AcMgd">
      <HintPath>..\References\AcMgd.dll</HintPath>
      <Private>False</Private>
    </Reference>
    <Reference Include="System" />
    <Reference Include="System.Core" />
    <Reference Include="System.Data" />
    <Reference Include="System.Xml" />
    <!-- ✅ اضافه شدن reference های Windows Forms -->
    <Reference Include="System.Drawing" />
    <Reference Include="System.Windows.Forms" />
  </ItemGroup>

</Project>
```

---

## 3. ساختار پوشه‌ها

### ❌ قبل

```
your-repo/
├── .github/
│   └── workflows/
│       └── build.yml
└── AreaDivider/
    ├── AreaDivider.csproj
    ├── Commands.cs
    └── AreaDividerForm.cs
```

**مشکل**: فایل‌های DLL اتوکد در repository نیستند و مسیرهای مطلق در CI کار نمی‌کنند.

### ✅ بعد

```
your-repo/
├── .github/
│   └── workflows/
│       ├── build.yml                    # راه‌حل اصلی
│       ├── build-alternative.yml        # راه‌حل جایگزین
│       └── build-with-buildtools.yml    # راه‌حل پیشرفته
├── AreaDivider/
│   ├── AreaDivider.csproj              # با مسیرهای نسبی
│   ├── Commands.cs
│   └── AreaDividerForm.cs
└── References/                          # ✅ پوشه جدید
    ├── AcCoreMgd.dll
    ├── AcDbMgd.dll
    └── AcMgd.dll
```

**حل شده**: فایل‌های DLL در repository هستند و از مسیرهای نسبی استفاده می‌شود.

---

## خلاصه تغییرات

| مورد | قبل | بعد |
|------|-----|-----|
| **Workflow - vs-version** | `vs-version: '2019'` ❌ | حذف شده ✅ |
| **Workflow - NuGet** | ندارد ❌ | اضافه شده ✅ |
| **csproj - مسیر DLL** | مطلق ❌ | نسبی ✅ |
| **csproj - Windows Forms** | ندارد ❌ | اضافه شده ✅ |
| **ساختار - References** | ندارد ❌ | اضافه شده ✅ |
| **مستندات** | ندارد ❌ | کامل ✅ |

---

## چرا این تغییرات کار می‌کنند؟

### 1. حذف `vs-version: '2019'`

**مشکل**: 
- پارامتر `'2019'` به عنوان string ساده قابل قبول نیست
- باید از syntax vswhere استفاده شود: `[16.0,17.0)`
- Runner ممکن است VS 2019 نداشته باشد

**راه‌حل**:
- حذف پارامتر به action اجازه می‌دهد خودش بهترین نسخه را پیدا کند
- با تمام runner ها سازگار است

### 2. تغییر مسیرها به نسبی

**مشکل**:
- مسیر `C:\Program Files\Autodesk\AutoCAD 2026\` در CI وجود ندارد
- هر developer ممکن است نسخه متفاوتی داشته باشد

**راه‌حل**:
- مسیر نسبی `..\..\References\` همیشه کار می‌کند
- فایل‌های DLL در repository هستند

### 3. اضافه کردن NuGet restore

**مشکل**:
- ممکن است package های NuGet restore نشده باشند

**راه‌حل**:
- قبل از build، packages را restore می‌کند
- اطمینان از وجود تمام dependencies

---

## نتیجه

با این تغییرات، build در GitHub Actions بدون خطا اجرا می‌شود و فایل `AreaDivider.dll` به عنوان artifact آپلود می‌شود.
