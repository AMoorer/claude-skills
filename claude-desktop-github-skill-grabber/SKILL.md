---
name: claude-desktop-github-skill-grabber
description: Downloads Claude skills from GitHub repositories with flexible search options - single skill, top-level only, or recursive discovery through folder structures
---

# Claude Desktop GitHub Skill Grabber

This skill automates the process of downloading Claude skills from GitHub repositories and preparing them for upload to Claude Desktop.

## Purpose

When users want to download multiple skills from a GitHub repository containing Claude skills, this skill:
1. Downloads the entire repository (or specific folder)
2. Identifies folders containing SKILL.md files using flexible search modes
3. Creates individual ZIP files for each skill
4. Saves them to a user-specified directory
5. Provides a summary of what was downloaded

## When to Use This Skill

Use this skill when the user:
- Wants to download skills from a GitHub repository
- Mentions getting skills from git/GitHub
- Asks to "grab skills" or "download skills" from a repository
- Wants to prepare skills for upload to Claude Desktop
- Provides a GitHub repository URL and wants to extract skills from it
- Wants a specific skill, all top-level skills, or all skills in nested folders

## Required Information

To execute this skill, you need:
1. **Repository URL** - The GitHub repository containing Claude skills (e.g., `https://github.com/user/repo`)
2. **Target Directory** - Where to save the zipped skills (e.g., `Z:\LIBRARY\SOFTWARE\CLAUDE\SKILLS`)
3. **Search Mode** (optional) - Defaults to "recursive" if not specified

If the user doesn't provide either of these, ask for them.

## Search Modes

This skill supports three search modes:

### 1. Single Skill Mode
**When to use**: User specifies a particular skill folder name or URL points to a specific skill folder

**Example**: 
- "Grab the 'file-organizer' skill from that repo"
- URL: `https://github.com/user/repo/tree/main/skills/file-organizer`

**Behavior**: 
- Downloads only the specified folder
- Validates it contains SKILL.md
- Creates a single ZIP file

### 2. Top-Level Only Mode
**When to use**: User wants only skills at the root/top level of the repository or specified directory

**Example**:
- "Get the top-level skills from that repo"
- "Download skills from the main directory only"

**Behavior**:
- Searches only immediate child folders
- Ignores nested subdirectories
- Good for repositories with flat structure

### 3. Recursive Mode (Default)
**When to use**: User wants all skills regardless of folder depth, or doesn't specify

**Example**:
- "Grab all skills from that repo"
- "Download every skill you can find"
- No search mode specified

**Behavior**:
- Searches through all subdirectories
- Finds skills at any nesting level
- Most comprehensive option

## Execution Process

### Step 1: Determine Search Mode

Ask the user or infer from context:
```
Single skill: User mentions specific skill name or URL points to specific folder
Top-level: User says "top level", "main directory", "root skills"
Recursive: User says "all skills", "every skill", or doesn't specify (DEFAULT)
```

### Step 2: Validate Input
- Confirm the GitHub repository URL is valid
- Confirm the target directory path
- If target directory doesn't exist, create it
- Extract skill name if in single-skill mode

### Step 3: Download Repository
Use PowerShell commands to:
```powershell
$repoZipUrl = "https://github.com/[user]/[repo]/archive/refs/heads/[branch].zip"
# Download the repository as a ZIP file
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
$webClient = New-Object System.Net.WebClient
$webClient.DownloadFile($repoZipUrl, $tempZipFile)
```

**Important**: For GitHub repository URLs:
- Convert `https://github.com/user/repo` to `https://github.com/user/repo/archive/refs/heads/master.zip`
- Or use `main` instead of `master` if that's the default branch
- Handle URLs pointing to specific folders: `https://github.com/user/repo/tree/main/folder-name`

### Step 4: Extract and Identify Skills

**For Single Skill Mode**:
```powershell
# Navigate to the specific skill folder
$skillFolder = Get-Item "$extractedRepo\$skillName"
if (Test-Path "$($skillFolder.FullName)\SKILL.md") {
    # Process this single skill
} else {
    Write-Error "Specified folder does not contain SKILL.md"
}
```

**For Top-Level Only Mode**:
```powershell
# Search only immediate children
$skillFolders = Get-ChildItem -Path $repoFolder.FullName -Directory | 
    Where-Object { Test-Path "$($_.FullName)\SKILL.md" }
```

**For Recursive Mode**:
```powershell
# Search all subdirectories recursively
$skillFolders = Get-ChildItem -Path $repoFolder.FullName -Directory -Recurse | 
    Where-Object { Test-Path "$($_.FullName)\SKILL.md" }
```

### Step 5: Zip Each Skill
```powershell
foreach ($folder in $skillFolders) {
    $skillName = $folder.Name
    $zipPath = Join-Path $targetDir "$skillName.zip"
    
    # Remove existing zip if it exists
    if (Test-Path $zipPath) {
        Remove-Item $zipPath -Force
    }
    
    # Compress the folder
    Compress-Archive -Path $folder.FullName -DestinationPath $zipPath -CompressionLevel Optimal
    
    Write-Host "[$currentSkill/$totalSkills] Zipped: $skillName"
}
```

### Step 6: Clean Up and Report
- Remove temporary files
- Report the search mode used
- Report the number of skills found and processed
- List all the skill names that were zipped
- Remind the user how to upload them to Claude Desktop

## Output Format

After processing, provide output like:

```
Search Mode: [Single/Top-Level/Recursive]
✓ Downloaded repository: [repo-name]
✓ Found [X] skill(s)
✓ Zipped all skills to: [target-directory]

Skills downloaded:
  • skill-name-1
  • skill-name-2 (from subfolder/path)
  • skill-name-3
  ...

Next steps:
1. Open Claude Desktop
2. Go to Settings → Capabilities → Skills
3. Click "Upload skill" and select the ZIP files from:
   [target-directory]
```

## Error Handling

Handle these common errors:
- **Invalid GitHub URL**: Ask the user to provide a valid GitHub repository URL
- **Repository doesn't contain skills**: Inform the user that no folders with SKILL.md were found
- **Specific skill not found** (single mode): Let user know the skill doesn't exist or path is wrong
- **Target directory not accessible**: Verify the path is valid and writable
- **Network errors**: Inform the user if download fails
- **Duplicate skill names**: If multiple skills have the same name (in recursive mode), append path info to ZIP name

## Example Usage

### Example 1: Single Skill
**User**: "Grab just the file-organizer skill from https://github.com/ComposioHQ/awesome-claude-skills"

**Claude**: 
1. Identifies single-skill mode
2. Downloads repo
3. Finds and zips only "file-organizer" folder
4. Reports: "Found 1 skill: file-organizer"

### Example 2: Top-Level Only
**User**: "Get the skills from the main directory of that repo, don't go into subfolders"

**Claude**:
1. Identifies top-level mode
2. Downloads repo
3. Searches only immediate children
4. Reports skills found at root level only

### Example 3: Recursive (Default)
**User**: "Grab all skills from https://github.com/ComposioHQ/awesome-claude-skills and put them in Z:\LIBRARY\SOFTWARE\CLAUDE\SKILLS"

**Claude**: 
1. Uses recursive mode (default)
2. Downloads repo
3. Searches all subdirectories
4. Reports all skills found at any depth

## Technical Notes

- Use Windows PowerShell commands (user is on Windows)
- Use `$env:TEMP` for temporary file operations
- Always clean up temporary files after processing
- Default to `master` branch but try `main` if master fails
- Handle both full repo URLs and tree/directory URLs if possible
- In recursive mode, watch for duplicate skill names in different folders
- Consider adding folder context to ZIP names if duplicates found: `skill-name-folder1.zip`, `skill-name-folder2.zip`

## Platform Compatibility

This skill is designed for Windows systems with PowerShell. For other platforms, adapt the commands accordingly:
- **macOS/Linux**: Use bash with `curl`, `unzip`, and `zip` commands
- Detect the platform using `$IsWindows`, `$IsMacOS`, `$IsLinux` PowerShell variables

## Safety Considerations

- Only process repositories from trusted sources
- Preview the skills found before zipping (show the list)
- Don't execute any code from the downloaded skills, only package them
- Warn users to audit skills before uploading to Claude Desktop
- Be careful with recursive mode on very large repositories
