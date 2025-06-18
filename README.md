# Promptr

**Transform AI prompts into executable scripts.** 

Promptr revolutionizes how we think about AI automation by introducing the concept of **AI scripts** - executable files that contain AI prompts with variable substitution, requirement validation, and security controls. Whether you're a developer automating code reviews or a content creator generating reports, Promptr makes AI workflows as simple as running a shell script.

## 🎯 Why Promptr?

- **For Technical Users**: Create reusable AI automation scripts with proper dependency management and security controls
- **For Non-Technical Users**: Execute complex AI workflows with simple command-line arguments - no programming required
- **For Teams**: Share standardized AI prompts that anyone can run with consistent results
- **For Workflows**: Chain AI scripts together or integrate them into existing automation pipelines

Promptr takes prompt files (`.md`, `.prompt.yaml`, or `.txt`), processes variable substitutions, validates requirements, and executes them using Claude CLI with appropriate permissions.

## Features

- **📜 Executable prompts**: Add `#!/usr/bin/env promptr` shebang to any prompt file and execute it directly like a script
- **Multiple file formats**: Support for `.prompt.yaml` (following [GitHub's specification](https://docs.github.com/en/github-models/use-github-models/storing-prompts-in-github-repositories)), `.md`, and `.txt` files with YAML frontmatter support
- **Variable substitution**: Dynamic content using `{{variable_name}}` syntax
- **Requirements validation**: Automatic checking for required commands, tools, environment variables, and directories
- **Security controls**: Fine-grained tool permissions via requirements specification
- **Interactive help**: Built-in help system for prompts and usage examples
- **Live output**: Verbose mode with streaming JSON output from Claude
- **Flexible variable input**: Support for command-line args, JSON, YAML files, and file contents

## Installation

1. Ensure you have Ruby installed
2. Make the script executable: `chmod +x promptr`
3. Add to your PATH: `export PATH=/path/to/promptr:$PATH`
4. Ensure you have Claude CLI installed and configured

## Usage

### Basic Commands

```bash
# List available prompts
promptr list

# Execute a prompt
promptr <command> [options]

# Show help for a specific prompt
promptr <command> help
promptr help <command>

# Show general help
promptr help
```

### Global Flags

```bash
-v, --verbose    Show live output instead of spinner
```

### File Resolution

Promptr resolves files in the following order:

1. If the argument has an extension: treats as direct file path
2. If no extension: searches `./prompts` directory for:
   - `<command>.prompt.yaml`
   - `<command>.md`
   - `<command>.txt`

### Variable Input Methods

#### Command-line arguments

```bash
promptr my-prompt key1=value1 key2=value2
```

#### File contents

```bash
promptr my-prompt config=@config.json
```

#### JSON input

```bash
promptr my-prompt --json '{"key1":"value1","key2":"value2"}'
```

#### File-based variables

```bash
promptr my-prompt --file variables.json
promptr my-prompt --file variables.yaml
```

### YAML Frontmatter Support

**New feature!** Markdown and text files now support YAML frontmatter for requirements specification. This allows you to add the same powerful validation and tool permissions to simple `.md` and `.txt` files:

```markdown
---
name: "Advanced Code Reviewer" 
description: "Comprehensive code analysis with security focus"
requirements:
  commands: ["git", "eslint", "cargo"]
  tools: ["Read", "Edit", "Grep", "Bash"]
  environment: ["GITHUB_TOKEN"]
  directories: ["./src", "./tests"]
---

# Advanced Code Review

Please perform a comprehensive review of {{file_path}}...
```

This brings the full power of requirements validation to simple markdown prompts!

## File Formats

### YAML Prompts (`.prompt.yaml`)

Following GitHub's prompt specification with additional `requirements` section:

```yaml
name: "Code Reviewer"
description: "Reviews code and provides feedback"
model: "claude-4-sonnet"
requirements:
  commands: ["git", "npm"]           # Required executables
  tools: ["Edit", "Write", "Read"]   # Claude Code tools to allow
  environment: ["API_KEY"]           # Required env variables
  directories: ["./src"]             # Required directories
messages:
  - role: "system"
    content: "You are a code reviewer."
  - role: "user"
    content: "Please review this code: {{code}}"
```

### Markdown/Text Files (`.md`, `.txt`)

Simple files with variable substitution. **NEW**: Now supports YAML frontmatter for requirements!

#### Basic Markdown/Text
```markdown
# Code Review Request

Please review the following code for {{language}}:

{{code}}

Focus on:
- Performance
- Security  
- Best practices
```

#### Markdown with YAML Frontmatter
```markdown
---
name: "Bug Fixer"
description: "Find and fix bugs in code"
requirements:
  commands: ["cargo", "git"]
  tools: ["Read", "Edit", "Bash"]
  directories: ["."]
---

# Bug Analysis

Find and fix bugs in {{language}} code at {{file_path}}.
Focus on {{bug_type}} issues and provide fixes.
```

## Requirements Validation

Promptr validates requirements before execution:

- **Commands**: Checks if executables exist in PATH or as absolute/relative paths
- **Tools**: Specifies which Claude Code tools are allowed (automatically added to `--allowedTools`)
- **Environment**: Validates required environment variables are set
- **Directories**: Ensures required directories exist

### Security Model

Commands listed in `requirements.commands` are automatically granted access via Claude's `--allowedTools` parameter as `Bash(command:*)`. Tools listed in `requirements.tools` are granted specific permissions. If no tools are specified, safe defaults are used: `["Read", "Glob", "Grep", "LS"]`.

#### Available Claude Code Tools

**Read-only tools:**
- `Read`: Read files from filesystem
- `Glob`: Fast file pattern matching
- `Grep`: Content search with regex
- `LS`: List directory contents

**File editing tools:**
- `Edit`: Make exact string replacements in files
- `MultiEdit`: Make multiple edits to a single file

**File creation tools:**
- `Write`: Create new files or overwrite existing ones

**External tools:**
- `WebFetch`: Fetch and process web content
- `WebSearch`: Search the web for information

**Task management tools:**
- `TodoRead`: Read current todo list
- `TodoWrite`: Create and manage task lists

**Development tools:**
- `Task`: Launch new agent with tools access
- `exit_plan_mode`: Exit planning mode

## Examples

### Real-World AI Script Scenarios

```bash
# Technical: Automated code review AI script
./scripts/code-review.md file=@src/components/Button.tsx language="TypeScript"

# Content: Generate blog post from outline  
./content/blog-generator.prompt.yaml outline=@post-outline.md tone="professional"

# DevOps: Infrastructure documentation generator
./ops/doc-generator.md service="auth-api" environment="production"

# Marketing: Social media content creator
./marketing/social-posts.md product="new-feature" platform="twitter"
```

### Basic Usage

```bash
# Execute a simple prompt
promptr code-review code="console.log('hello')" language="javascript"

# Use verbose mode to see live output
promptr -v code-review code=@myfile.js language="javascript"

# Load variables from file
promptr code-review --file review-config.yaml
```

### 🚀 Creating AI Scripts (Shebang Support)

**The killer feature**: Transform any prompt into an AI script! Add a shebang to any prompt file and execute it directly like a traditional shell script. This democratizes AI automation - technical users can create sophisticated AI workflows, while non-technical users can execute them with simple commands.

```bash
# Create an executable prompt file
cat > my-prompt.md << 'EOF'
#!/usr/bin/env promptr
# Code Review Prompt

Please review this {{language}} code:

{{code}}

Focus on best practices and potential issues.
EOF

# Make it executable
chmod +x my-prompt.md

# Execute directly with variables
./my-prompt.md code="console.log('hello')" language="JavaScript"
```

This works with any supported file format:

```yaml
#!/usr/bin/env promptr
name: "Code Reviewer"
description: "Reviews code and provides feedback"
messages:
  - role: "user"
    content: "Review this {{language}} code: {{code}}"
```

```bash
chmod +x review.prompt.yaml
./review.prompt.yaml code=@myfile.js language="JavaScript"
```

### Creating Executable Prompts (PATH method)

If `promptr` is in your PATH, you can also make prompts directly executable without shebangs:

```bash
# Add to PATH
export PATH=/path/to/promptr:$PATH

# Execute prompt directly
./prompts/code-review.prompt.yaml code="console.log('hello')"
```

### Getting Help

```bash
# Show help for a specific prompt
promptr code-review help

# Example output:
📋 Prompt: Code Reviewer
📝 Description: Reviews code and provides feedback
🤖 Model: claude-4-sonnet

⚙️  Requirements:
   Commands: git, npm
   Tools: Edit, Write, Read

🔧 Variables:
   code=<value>
   language=<value>

💡 Usage examples:
   promptr code-review code=value language=value
   promptr code-review --json '{"code":"value","language":"value"}'
```

## Error Handling

Promptr provides clear error messages for common issues:

- Missing prompt files
- Unsupported file types
- Missing required variables
- Failed requirement validation
- Invalid JSON/YAML syntax

## Development

The main script is a single Ruby file (`promptr`) with the following key components:

- **File discovery and resolution**
- **Variable parsing and substitution**
- **Requirements validation**
- **Claude command building and execution**
- **Streaming output handling**

## Contributing

This is a prototype tool. Contributions welcome!

## License

[Add your license here]
