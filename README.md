# AutoCAD Add-in - Area Divider

## GitHub Actions MSBuild Fix

This repository contains the fixed configuration for building an AutoCAD .NET add-in using GitHub Actions.

## Problem

The original workflow failed with:
```
Error: Unable to find MSBuild.
```

## Solution

The issue was caused by:
1. Incorrect usage of `vs-version` parameter in `setup-msbuild`
2. Absolute paths in `.csproj` file that don't exist in CI environment
3. Missing AutoCAD DLL files in the repository

## Project Structure

```
your-repo/
├── .github/
│   └── workflows/
│       ├── build.yml                    # Recommended solution
│       ├── build-alternative.yml        # Alternative with windows-2019
│       └── build-with-buildtools.yml    # Advanced solution
├── AreaDivider/
│   ├── AreaDivider.csproj              # Fixed with relative paths
│   ├── Commands.cs
│   └── AreaDividerForm.cs
├── References/
│   ├── AcCoreMgd.dll                   # AutoCAD reference DLLs
│   ├── AcDbMgd.dll
│   └── AcMgd.dll
├── SOLUTION_GUIDE.md                    # Detailed guide in Persian
└── README.md                            # This file
```

## Quick Start

### Option 1: Recommended (Simple)

Use the default workflow without specifying VS version:

**File**: `.github/workflows/build.yml`

```yaml
- name: Setup MSBuild
  uses: microsoft/setup-msbuild@v2
```

### Option 2: Use Windows 2019 Runner

If you specifically need Visual Studio 2019:

**File**: `.github/workflows/build-alternative.yml`

```yaml
jobs:
  build:
    runs-on: windows-2019
```

### Option 3: Install Build Tools

For maximum control, install Build Tools manually:

**File**: `.github/workflows/build-with-buildtools.yml`

## Key Changes

### 1. Fixed `.csproj` File

Changed from absolute paths:
```xml
<HintPath>C:\Program Files\Autodesk\AutoCAD 2026\AcCoreMgd.dll</HintPath>
```

To relative paths:
```xml
<HintPath>..\References\AcCoreMgd.dll</HintPath>
```

### 2. Fixed Workflow

Removed incorrect `vs-version: '2019'` parameter:

**Before** (Wrong):
```yaml
- name: Setup MSBuild
  uses: microsoft/setup-msbuild@v2
  with:
    vs-version: '2019'
```

**After** (Correct):
```yaml
- name: Setup MSBuild
  uses: microsoft/setup-msbuild@v2
```

## Setup Instructions

1. **Copy the fixed files** to your repository:
   - `AreaDivider/AreaDivider.csproj`
   - `.github/workflows/build.yml` (choose one of the three options)

2. **Create References folder** and add AutoCAD DLLs:
   ```bash
   mkdir References
   # Copy AcCoreMgd.dll, AcDbMgd.dll, AcMgd.dll to this folder
   ```

3. **Commit and push**:
   ```bash
   git add .
   git commit -m "Fix MSBuild configuration for GitHub Actions"
   git push
   ```

4. **Check the Actions tab** in your GitHub repository to see the build result.

## Troubleshooting

### Still getting MSBuild error?

Add debug output to your workflow:

```yaml
- name: Debug MSBuild
  run: |
    vswhere -all
    msbuild -version
```

### Missing references?

Make sure:
- DLL files are in the `References` folder
- DLL files are committed to git
- Paths in `.csproj` match the folder structure

### Build fails with other errors?

Check:
- Target Framework is correctly set to `net48`
- Platform target matches your needs (`x64` or `Any CPU`)
- All source files are present

## Technical Details

- **Target Framework**: .NET Framework 4.8
- **Platform**: x64
- **AutoCAD Version**: Compatible with AutoCAD 2026 (and other versions)
- **Build Tool**: MSBuild (from Visual Studio)

## Files Included

1. **AreaDivider.csproj** - Fixed project file with relative paths
2. **build.yml** - Recommended workflow (no version specification)
3. **build-alternative.yml** - Alternative workflow (windows-2019)
4. **build-with-buildtools.yml** - Advanced workflow (manual installation)
5. **SOLUTION_GUIDE.md** - Detailed guide in Persian
6. **README.md** - This file

## Additional Resources

- [microsoft/setup-msbuild](https://github.com/microsoft/setup-msbuild)
- [GitHub Actions Runner Images](https://github.com/actions/runner-images)
- [MSBuild Documentation](https://docs.microsoft.com/en-us/visualstudio/msbuild/)

## License

This configuration is provided as-is for educational purposes.
