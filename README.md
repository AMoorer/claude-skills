# Andy's Claude Skills Collection

A curated collection of custom Claude skills developed by Andy Moorer for enhancing workflows across Claude Desktop, Claude Code, and the Claude API.

**License**: Creative Commons Attribution 4.0 International (CC BY 4.0) - Free to use with attribution

## What Are Claude Skills?

Claude Skills are customizable workflows that teach Claude how to perform specific tasks according to your unique requirements. Skills enable Claude to execute tasks in a repeatable, standardized manner across all Claude platforms.

- **Composable** - Combine multiple skills for complex workflows
- **Portable** - Use the same skill across Claude.ai, Claude Code, and the API
- **Efficient** - Claude loads only what's needed for optimal performance
- **Powerful** - Include executable code for technical reliability

## Skills in This Repository

### 🔧 Claude Desktop GitHub Skill Grabber
**[Download v1.0](https://github.com/AMoorer/claude-skills/releases/latest/download/claude-desktop-github-skill-grabber.zip)** | **[View Source](./claude-desktop-github-skill-grabber/)**

Automates downloading Claude skills from GitHub repositories with flexible search options.

**Features**:
- Three search modes: single skill, top-level only, or recursive discovery
- Downloads and zips skills automatically
- Supports any GitHub repository with skills
- Handles nested folder structures

**Usage**: 
- "Grab all skills from [repo URL]" → Recursive search (default)
- "Get just the [skill-name] skill" → Single skill download
- "Download top-level skills only" → Non-recursive search

---

### 📂 GitHub Folder Downloader
**[Download v1.0](https://github.com/AMoorer/claude-skills/releases/latest/download/github-folder-downloader.zip)** | **[View Source](./github-folder-downloader/)**

Downloads any specific folder from GitHub repositories without cloning the entire repo.

**Features**:
- Extracts targeted folders from GitHub repos
- Preserves folder structure and contents
- Optional ZIP creation of downloaded folder
- Works with any public GitHub repository

**Usage**:
- "Download the [folder-path] from [repo URL]"
- "Get the docs folder from [repo] and save to [location]"

## Getting Started

### Quick Install (Claude Desktop)

**Option 1: From Releases (Recommended)**
1. Go to [Releases](https://github.com/AMoorer/claude-skills/releases/latest)
2. Download the skill ZIP file(s) you want
3. Open Claude Desktop → Settings → Capabilities → Skills
4. Click "Upload skill" and select the ZIP file
5. Toggle the skill ON

**Option 2: Build from Source**
1. Clone this repository or download the skill folder
2. ZIP the skill folder (ensure folder structure: `skill-name/SKILL.md`)
3. Upload to Claude Desktop as above

### For Claude Code (CLI)

```bash
# Clone this repository
git clone https://github.com/AMoorer/claude-skills.git

# Copy skill to your project or user skills directory
cd ~/.claude/skills/
cp -r /path/to/claude-skills/[skill-name] .

# Or for project-specific skills
cd your-project/.claude/skills/
cp -r /path/to/claude-skills/[skill-name] .

# Start Claude Code
claude
```

Skills load automatically when Claude determines they're relevant to your task.

### For Claude API

```python
import anthropic

client = anthropic.Client(api_key="your-api-key")

# Load skill from file
with open('skill-folder/SKILL.md', 'r') as f:
    skill_content = f.read()

response = client.messages.create(
    model="claude-sonnet-4-20250514",
    skills=[skill_content],
    messages=[{"role": "user", "content": "Your prompt"}]
)
```

See the [Skills API documentation](https://docs.claude.com/en/api/skills-guide) for details.

## Skill Structure

Each skill is a folder containing a SKILL.md file with YAML frontmatter:

```
skill-name/
├── SKILL.md          # Required: Skill instructions and metadata
├── scripts/          # Optional: Helper scripts
├── templates/        # Optional: Document templates
└── resources/        # Optional: Reference files
```

### SKILL.md Format

```markdown
---
name: my-skill-name
description: A clear description of what this skill does and when to use it.
---

# My Skill Name

Detailed instructions for Claude on how to execute this skill...

## When to Use This Skill
...

## Execution Process
...
```

## Release Philosophy

Skills in this repository follow a release paradigm:
- **Development**: Skills are developed and tested in the main branch
- **Releases**: Stable versions are tagged and include pre-built ZIP files
- **Easy Installation**: Download ZIPs directly from releases for Claude Desktop

This makes it easy to:
- Get stable, tested versions of skills
- Install skills without building from source
- Track version history and changes

## Contributing

Suggestions and improvements are welcome!

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/new-skill`)
3. Add your skill following the structure above
4. Test your skill across platforms (Claude Desktop, Code, and/or API)
5. Submit a pull request with clear documentation

### Skill Quality Guidelines

- Focus on specific, repeatable tasks
- Include clear examples and edge cases  
- Write instructions for Claude, not end users
- Document prerequisites and dependencies
- Include error handling guidance
- Test across multiple use cases

## Resources

### Official Documentation
- [Claude Skills Overview](https://www.anthropic.com/news/skills) - Official announcement
- [Skills User Guide](https://support.claude.com/en/articles/12512180-using-skills-in-claude) - How to use skills
- [Creating Custom Skills](https://support.claude.com/en/articles/12512198-creating-custom-skills) - Development guide
- [Skills API Documentation](https://docs.claude.com/en/api/skills-guide) - API integration
- [Agent Skills Blog](https://anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills) - Engineering deep dive

### Other Skill Repositories
- [anthropics/skills](https://github.com/anthropics/skills) - Official Anthropic skills
- [ComposioHQ/awesome-claude-skills](https://github.com/ComposioHQ/awesome-claude-skills) - Community collection
- [travisvn/awesome-claude-skills](https://github.com/travisvn/awesome-claude-skills) - Curated list with resources

## License

This repository is licensed under the **Creative Commons Attribution 4.0 International (CC BY 4.0)**.

You are free to:
- **Share** — copy and redistribute the skills
- **Adapt** — remix, transform, and build upon the skills
- **Commercial use** — use for any purpose, including commercially

Under these terms:
- **Attribution** — You must give appropriate credit, provide a link to the license, and indicate if changes were made

See [LICENSE](LICENSE) for full details.

### How to Attribute

When using or adapting skills from this repository:

```markdown
Based on [skill-name] by Andy Moorer
https://github.com/AMoorer/claude-skills
```

## Author

**Andy Moorer**
- Senior Technical Artist at Meta Reality Labs
- Focus: Real-time simulation, procedural systems, PopcornFX
- GitHub: [@AMoorer](https://github.com/AMoorer)

---

*Claude Skills work across Claude.ai, Claude Code, and the Claude API. Once you create a skill, it's portable across all platforms, making your workflows consistent everywhere you use Claude.*
