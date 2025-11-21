# تحلیل مشکل MSBuild در GitHub Actions

## مشکل اصلی
خطای "Unable to find MSBuild" هنگام استفاده از `microsoft/setup-msbuild@v2` با پارامتر `vs-version: '2019'`

## علت احتمالی
بر اساس مستندات و issue های GitHub:

1. **Runner های GitHub-hosted معمولاً نسخه‌های pre-release ندارند**
2. **پارامتر vs-version باید به صورت range مشخص شود نه string ساده**
3. **Windows-latest ممکن است Visual Studio 2019 نداشته باشد**

## راه‌حل‌های پیشنهادی

### راه‌حل 1: حذف پارامتر vs-version (توصیه شده)
```yaml
- name: Setup MSBuild
  uses: microsoft/setup-msbuild@v2
```
این به action اجازه می‌دهد که آخرین نسخه موجود MSBuild را پیدا کند.

### راه‌حل 2: استفاده از range syntax
```yaml
- name: Setup MSBuild
  uses: microsoft/setup-msbuild@v2
  with:
    vs-version: '[16.0,17.0)'
```

### راه‌حل 3: استفاده از windows-2019 به جای windows-latest
```yaml
runs-on: windows-2019
```

### راه‌حل 4: نصب دستی Build Tools
اضافه کردن step برای نصب Visual Studio Build Tools

## توضیحات فنی

- Visual Studio 2019 = نسخه 16.x
- Visual Studio 2022 = نسخه 17.x
- windows-latest معمولاً آخرین نسخه stable را دارد (احتمالاً VS 2022)
- پارامتر vs-version باید از syntax vswhere استفاده کند: `[min,max)`
  - `[` = inclusive
  - `(` = exclusive
