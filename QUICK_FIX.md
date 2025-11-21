# راه‌حل سریع - خطای MSBuild

## مشکل
```
Error: Unable to find MSBuild.
```

## راه‌حل (3 مرحله ساده)

### 1️⃣ تغییر workflow

فایل `.github/workflows/build.yml` را باز کنید و این قسمت:

```yaml
- name: Setup MSBuild
  uses: microsoft/setup-msbuild@v2
  with:
    vs-version: '2019'  # ❌ این خط را حذف کنید
```

را به این تغییر دهید:

```yaml
- name: Setup MSBuild
  uses: microsoft/setup-msbuild@v2
  # ✅ هیچ پارامتری نمی‌خواهد
```

### 2️⃣ تغییر مسیرها در csproj

فایل `AreaDivider/AreaDivider.csproj` را باز کنید و مسیرهای مطلق را تغییر دهید:

**قبل**:
```xml
<HintPath>C:\Program Files\Autodesk\AutoCAD 2026\AcCoreMgd.dll</HintPath>
```

**بعد**:
```xml
<HintPath>..\References\AcCoreMgd.dll</HintPath>
```

این کار را برای هر سه DLL انجام دهید.

### 3️⃣ اضافه کردن فایل‌های DLL

یک پوشه `References` در root پروژه ایجاد کنید و این فایل‌ها را در آن قرار دهید:
- `AcCoreMgd.dll`
- `AcDbMgd.dll`
- `AcMgd.dll`

## ساختار نهایی

```
your-repo/
├── .github/workflows/build.yml
├── AreaDivider/
│   └── AreaDivider.csproj
└── References/
    ├── AcCoreMgd.dll
    ├── AcDbMgd.dll
    └── AcMgd.dll
```

## تمام! 🎉

حالا commit و push کنید:

```bash
git add .
git commit -m "Fix MSBuild error"
git push
```

---

برای جزئیات بیشتر، فایل `SOLUTION_GUIDE.md` را مطالعه کنید.
