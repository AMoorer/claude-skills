---
name: github-folder-downloader
description: Downloads a specific folder and all its contents from any GitHub repository as a ZIP archive for local use
---

# GitHub Folder Downloader

This skill downloads a specific folder (and all its contents) from any GitHub repository, making it easy to grab documentation, examples, assets, or any subset of a repo without cloning the entire repository.

## Purpose

Downloads a targeted folder from a GitHub repository with all its contents preserved in their original structure. Perfect for:
- Grabbing example code or templates
- Downloading documentation folders
- Extracting asset libraries
- Getting specific project subdirectories
- Archiving portions of repositories

## When to Use This Skill

Use this skill when the user:
- Wants to download a specific folder from GitHub
- Mentions getting a directory/folder from a repo
- Provides a GitHub URL pointing to a folder (tree view)
- Wants folder contents without cloning entire repo
- Asks to "download", "grab", "get", or "fetch" a GitHub folder

## Required Information

1. **GitHub URL** - Either:
   - Full folder URL: `https://github.com/user/repo/tree/branch/path/to/folder`
   - Repository + folder path: `user/repo` and `path/to/folder`

2. **Target Location** - Where to save the downloaded folder (e.g., `C:\Users\andym\Downloads`)

If not provided, ask for both.

## Execution Process

### Step 1: Parse GitHub URL

Extract components from the URL:
```powershell
# From: https://github.com/user/repo/tree/main/folder/subfolder
$owner = "user"
$repo = "repo"
$branch = "main"
$folderPath = "folder/subfolder"
```

Handle various URL formats:
- `https://github.com/user/repo/tree/branch/path`
- `https://github.com/user/repo/blob/branch/path` (file URL - get parent folder)
- Short form: `user/repo` + separate folder path

Default branch to `main` if not specified, try `master` if `main` fails.

### Step 2: Download Repository Archive

```powershell
$repoZipUrl = "https://github.com/$owner/$repo/archive/refs/heads/$branch.zip"
$tempDir = "$env:TEMP\github-download-$(Get-Random)"
$zipFile = "$tempDir\repo.zip"
$extractDir = "$tempDir\extracted"

# Create temp directories
New-Item -ItemType Directory -Path $tempDir -Force | Out-Null
New-Item -ItemType Directory -Path $extractDir -Force | Out-Null

# Download
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
$webClient = New-Object System.Net.WebClient
$webClient.DownloadFile($repoZipUrl, $zipFile)

# Extract
Expand-Archive -Path $zipFile -DestinationPath $extractDir -Force
```

### Step 3: Locate Target Folder

```powershell
# Find the extracted repo folder (named like "repo-main")
$repoFolder = Get-ChildItem -Path $extractDir -Directory | Select-Object -First 1

# Navigate to target folder
$targetFolder = Join-Path $repoFolder.FullName $folderPath

if (-not (Test-Path $targetFolder)) {
    Write-Error "Folder not found: $folderPath"
    # Suggest similar paths if possible
    exit 1
}
```

### Step 4: Copy to Destination

```powershell
$folderName = Split-Path $folderPath -Leaf
$destinationPath = Join-Path $targetLocation $folderName

# Check if destination exists
if (Test-Path $destinationPath) {
    # Ask user or append timestamp
    $destinationPath = "$destinationPath-$(Get-Date -Format 'yyyyMMdd-HHmmss')"
}

# Copy the entire folder
Copy-Item -Path $targetFolder -Destination $destinationPath -Recurse -Force

Write-Host "✓ Downloaded to: $destinationPath"
```

### Step 5: Optional - Create ZIP Archive

```powershell
# If user wants a ZIP of the downloaded folder
$zipOutput = "$destinationPath.zip"
Compress-Archive -Path $destinationPath -DestinationPath $zipOutput -CompressionLevel Optimal
Write-Host "✓ Created archive: $zipOutput"
```

### Step 6: Clean Up and Report

```powershell
# Remove temporary files
Remove-Item -Path $tempDir -Recurse -Force

# Report statistics
$fileCount = (Get-ChildItem -Path $destinationPath -Recurse -File).Count
$folderSize = (Get-ChildItem -Path $destinationPath -Recurse | Measure-Object -Property Length -Sum).Sum
```

## Output Format

Provide clear feedback:

```
✓ Repository: user/repo (branch: main)
✓ Folder: path/to/folder
✓ Downloaded to: C:\Users\andym\Downloads\folder

Contents:
  • [X] files
  • [Y] subdirectories
  • Total size: [Z] MB

Location: C:\Users\andym\Downloads\folder
[Optional] ZIP archive: C:\Users\andym\Downloads\folder.zip
```

## User Options

Support these optional parameters:
- **Create ZIP**: Automatically create a ZIP archive of downloaded folder
- **Rename**: Allow custom name for downloaded folder
- **Overwrite**: Handle existing folders (overwrite, rename with timestamp, or ask)
- **Include metadata**: Option to include a README with download info (repo, branch, date, URL)

## Error Handling

Handle these scenarios:

1. **Invalid GitHub URL**
   - Parse fails or doesn't match GitHub pattern
   - Ask for clarification or correct URL

2. **Repository Not Found**
   - 404 error when downloading
   - Check if repo is private or doesn't exist
   - Suggest checking URL

3. **Branch Not Found**
   - Try alternate branches: `main` → `master` → `develop`
   - List available branches if possible
   - Ask user for correct branch

4. **Folder Not Found**
   - Specified path doesn't exist in repo
   - Show available folders at parent level
   - Suggest similar paths (fuzzy matching)

5. **Network Errors**
   - Timeout or connection issues
   - Suggest retry or check internet connection

6. **Permission Errors**
   - Cannot write to target location
   - Suggest alternate location or check permissions

## Example Usage

### Example 1: Full URL
**User**: "Download the examples folder from https://github.com/microsoft/TypeScript/tree/main/tests/cases/compiler"

**Claude**:
```
✓ Repository: microsoft/TypeScript (branch: main)
✓ Folder: tests/cases/compiler
✓ Downloaded to: C:\Users\andym\Downloads\compiler

Contents:
  • 1,234 files
  • 45 subdirectories
  • Total size: 12.5 MB
```

### Example 2: Short Form
**User**: "Get the docs folder from anthropics/anthropic-sdk-python and save it to C:\Temp"

**Claude**:
```
Downloading: anthropics/anthropic-sdk-python/docs
Branch: main (default)
Target: C:\Temp

✓ Downloaded to: C:\Temp\docs
```

### Example 3: With Options
**User**: "Download the assets folder from that repo and make a ZIP of it"

**Claude**:
```
✓ Downloaded to: C:\Users\andym\Downloads\assets
✓ Created archive: C:\Users\andym\Downloads\assets.zip

You now have:
  • Folder: C:\Users\andym\Downloads\assets
  • Archive: C:\Users\andym\Downloads\assets.zip (2.3 MB)
```

## Advanced Features

### Metadata File
Create a `_DOWNLOAD_INFO.txt` in the downloaded folder:
```
Downloaded from GitHub
Repository: user/repo
Branch: main
Folder: path/to/folder
Download Date: 2025-11-09 12:34:56
URL: https://github.com/user/repo/tree/main/path/to/folder
```

### Smart Path Detection
If user provides unclear path:
```powershell
# Search for folders matching pattern
$matches = Get-ChildItem -Path $repoFolder.FullName -Directory -Recurse | 
    Where-Object { $_.Name -like "*$searchTerm*" }

if ($matches.Count -gt 1) {
    # Present options to user
    Write-Host "Found multiple matches:"
    $matches | ForEach-Object { Write-Host "  - $($_.FullName)" }
}
```

## Technical Notes

- Uses GitHub's archive API (no authentication needed for public repos)
- Preserves original folder structure and permissions
- Handles symbolic links appropriately
- Efficient for large folders (downloads only once, extracts targeted folder)
- Works with any branch, tag, or commit SHA
- Temp files are always cleaned up, even on errors

## Platform Compatibility

Designed for Windows with PowerShell. Adapt for other platforms:

**macOS/Linux**:
```bash
# Use curl and unzip
curl -L "https://github.com/$owner/$repo/archive/refs/heads/$branch.zip" -o repo.zip
unzip repo.zip
cp -r "repo-$branch/$folderPath" "$destination/"
```

## Safety Considerations

- Only downloads from public repositories (no authentication)
- Preview folder size before downloading large folders
- Warn if folder contains executables or scripts
- Always verify source repository is trustworthy
- Don't execute any downloaded code without inspection

## Related Skills

Works well with:
- **Claude Desktop GitHub Skill Grabber** - Specifically for Claude skills
- **File Organizer** - To organize downloaded contents
- **Documentation tools** - If downloading docs
