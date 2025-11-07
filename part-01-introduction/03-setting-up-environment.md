# Chapter 3: Setting Up Your Environment

## Table of Contents
1. [Installing and Updating Bash](#installing-and-updating-bash)
2. [Using Bash on Different Operating Systems](#using-bash-on-different-operating-systems)
3. [Setting Up Editors for Scripting](#setting-up-editors-for-scripting)
4. [Essential Tools and Utilities](#essential-tools-and-utilities)
5. [Summary](#summary)

---

## Installing and Updating Bash

### Check Current Bash Version

```bash
# Check Bash version
bash --version
# Output: GNU bash, version 5.1.16(1)-release (x86_64-pc-linux-gnu)

# Check default shell
echo $SHELL
# Output: /bin/bash

# Check Bash location
which bash
# Output: /usr/bin/bash

# List all installed shells
cat /etc/shells
```

### Linux Installation

#### Ubuntu/Debian

```bash
# Update package list
sudo apt update

# Install Bash (usually pre-installed)
sudo apt install bash

# Install latest version
sudo apt install --only-upgrade bash

# Check version
bash --version
```

#### CentOS/RHEL/Fedora

```bash
# Update system
sudo yum update

# Install Bash
sudo yum install bash

# Or using dnf (Fedora)
sudo dnf install bash

# Update to latest
sudo dnf upgrade bash
```

#### Arch Linux

```bash
# Update and install
sudo pacman -Syu bash

# Check version
pacman -Q bash
```

#### From Source (Latest Version)

```bash
# Install dependencies
sudo apt install build-essential

# Download latest Bash source
cd /tmp
wget https://ftp.gnu.org/gnu/bash/bash-5.2.tar.gz

# Extract
tar -xzf bash-5.2.tar.gz
cd bash-5.2

# Configure, compile, and install
./configure
make
sudo make install

# Verify installation
/usr/local/bin/bash --version

# Make it default (optional)
sudo chsh -s /usr/local/bin/bash $USER
```

### macOS Installation

#### Using Built-in Bash

```bash
# macOS comes with Bash (older version)
bash --version
# Output: GNU bash, version 3.2.57 (older)

# Note: macOS Catalina+ uses Zsh as default
echo $SHELL
# Output: /bin/zsh (if default) or /bin/bash
```

#### Install Latest Bash with Homebrew

```bash
# Install Homebrew (if not installed)
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install latest Bash
brew install bash

# Check new version
/usr/local/bin/bash --version
# Output: GNU bash, version 5.2.x

# Add to allowed shells
sudo echo /usr/local/bin/bash >> /etc/shells

# Change default shell
chsh -s /usr/local/bin/bash

# Restart terminal and verify
echo $SHELL
bash --version
```

### Windows Installation

#### Windows Subsystem for Linux (WSL) - Recommended

**Install WSL 2:**

```powershell
# Open PowerShell as Administrator

# Install WSL
wsl --install

# Or install specific distribution
wsl --install -d Ubuntu

# List available distributions
wsl --list --online

# Set WSL 2 as default
wsl --set-default-version 2

# Restart computer
```

**After Installation:**

```bash
# Launch Ubuntu (or your chosen distro)
# Bash is pre-installed

# Update Bash
sudo apt update
sudo apt install --only-upgrade bash

# Check version
bash --version
```

#### Git Bash (Lightweight Alternative)

```bash
# Download from: https://git-scm.com/downloads
# Run installer
# Git Bash provides Bash environment on Windows

# After installation, launch Git Bash
# Check version
bash --version
```

#### Cygwin (Full POSIX Environment)

```bash
# Download from: https://www.cygwin.com/
# Run setup-x86_64.exe
# Select packages including bash
# Complete installation
```

---

## Using Bash on Different Operating Systems

### Linux (Native Environment)

**Advantages:**
- ✅ Native Bash support
- ✅ All features available
- ✅ Best performance
- ✅ Direct system access

**Setup:**

```bash
# Already installed on most distros
# Verify installation
bash --version

# Make sure bash is your default shell
chsh -s /bin/bash

# Reload shell configuration
source ~/.bashrc
```

### macOS

**Default Shell Transition:**

macOS Catalina (10.15) and later use Zsh as default shell, but Bash is still available.

**Using Bash on macOS:**

```bash
# Switch to Bash temporarily
bash

# Make Bash default permanently
chsh -s /bin/bash

# Or use Homebrew's newer Bash
chsh -s /usr/local/bin/bash

# Create/Edit .bash_profile
touch ~/.bash_profile

# macOS-specific: Source .bashrc from .bash_profile
echo 'if [ -f ~/.bashrc ]; then source ~/.bashrc; fi' >> ~/.bash_profile
```

**macOS-Specific Considerations:**

```bash
# Some commands differ from Linux
# Use GNU coreutils for Linux-like behavior

# Install GNU tools
brew install coreutils findutils gnu-tar gnu-sed gawk gnutls gnu-indent gnu-getopt grep

# Add to PATH in ~/.bashrc
export PATH="/usr/local/opt/coreutils/libexec/gnubin:$PATH"
export PATH="/usr/local/opt/findutils/libexec/gnubin:$PATH"
export PATH="/usr/local/opt/gnu-sed/libexec/gnubin:$PATH"

# Now you can use Linux-style commands
ls --color=auto  # Works with GNU ls
```

### Windows (WSL)

**WSL Best Practices:**

```bash
# Access Windows files from WSL
cd /mnt/c/Users/YourName/Documents

# Access WSL files from Windows
# Navigate to: \\wsl$\Ubuntu\home\username

# Set up proper line endings
git config --global core.autocrlf input

# Configure Windows Terminal (recommended)
# Install from Microsoft Store
# Add WSL profile for quick access
```

**WSL Configuration (~/.bashrc):**

```bash
# WSL-specific settings

# Set display for GUI apps (WSL 2 with WSLg)
export DISPLAY=:0

# Windows interoperability
# Run Windows commands from WSL
alias explorer='explorer.exe'
alias code='code.exe'

# Convenient aliases for crossing file systems
alias cdwin='cd /mnt/c/Users/YourName'
alias cdd='cd /mnt/d'

# Fix permission issues
if grep -q microsoft /proc/version; then
    # WSL-specific umask
    umask 022
fi
```

**Performance Tip:**

```bash
# Work within WSL filesystem for better performance
# Avoid /mnt/c for heavy operations

# Good (fast):
~/projects/myapp

# Bad (slow):
/mnt/c/Users/YourName/projects/myapp
```

### Cross-Platform Script Compatibility

```bash
#!/bin/bash
# Cross-platform compatible script

# Detect operating system
detect_os() {
    case "$(uname -s)" in
        Linux*)     OS=Linux;;
        Darwin*)    OS=Mac;;
        CYGWIN*)    OS=Cygwin;;
        MINGW*)     OS=MinGw;;
        *)          OS="UNKNOWN:${unameOut}"
    esac
    echo $OS
}

OS=$(detect_os)
echo "Running on: $OS"

# OS-specific logic
case $OS in
    Linux)
        # Linux-specific commands
        sudo apt update
        ;;
    Mac)
        # macOS-specific commands
        brew update
        ;;
    *)
        echo "Unsupported OS"
        exit 1
        ;;
esac
```

---

## Setting Up Editors for Scripting

### Vim (Terminal-Based)

**Installation:**

```bash
# Linux
sudo apt install vim          # Ubuntu/Debian
sudo yum install vim          # CentOS/RHEL
sudo pacman -S vim            # Arch

# macOS
brew install vim

# Already installed on most systems
vim --version
```

**Basic Vim Configuration (~/.vimrc):**

```vim
" Enable syntax highlighting
syntax on

" Show line numbers
set number

" Set tab width
set tabstop=4
set shiftwidth=4
set expandtab

" Auto-indent
set autoindent
set smartindent

" Highlight search results
set hlsearch
set incsearch

" Enable mouse support
set mouse=a

" Show matching brackets
set showmatch

" File type detection
filetype plugin indent on

" Bash-specific settings
autocmd FileType sh setlocal tabstop=4 shiftwidth=4 expandtab
```

**Vim Basics for Bash Scripting:**

```bash
# Create/Edit a script
vim myscript.sh

# Vim modes:
# Press 'i' - Insert mode (start typing)
# Press 'Esc' - Normal mode (navigate)
# Press ':' - Command mode

# Common commands:
:w          # Save file
:q          # Quit
:wq         # Save and quit
:q!         # Quit without saving
:set number # Show line numbers
/pattern    # Search for pattern
:%s/old/new/g  # Replace all occurrences

# Make script executable from within vim
:!chmod +x %

# Run script from within vim
:!bash %
```

### Nano (Beginner-Friendly)

**Installation:**

```bash
# Usually pre-installed
# If not:
sudo apt install nano       # Ubuntu/Debian
```

**Basic Nano Configuration (~/.nanorc):**

```bash
# Enable syntax highlighting
include /usr/share/nano/*.nanorc

# Set tab size
set tabsize 4

# Convert tabs to spaces
set tabstospaces

# Enable mouse support
set mouse

# Auto-indent
set autoindent

# Line numbers
set linenumbers

# Smooth scrolling
set smooth
```

**Nano Basics:**

```bash
# Edit a file
nano myscript.sh

# Common shortcuts (shown at bottom):
Ctrl+O  # Save (WriteOut)
Ctrl+X  # Exit
Ctrl+K  # Cut line
Ctrl+U  # Paste (Uncut)
Ctrl+W  # Search
Ctrl+\  # Replace
Ctrl+G  # Help
```

### VS Code (GUI Editor)

**Installation:**

```bash
# Linux - Download from https://code.visualstudio.com/
# Or via package manager:
sudo snap install code --classic

# macOS
brew install --cask visual-studio-code

# Windows - Download installer from website
```

**Essential Extensions for Bash:**

```bash
# Install extensions via VS Code marketplace or command:
code --install-extension timonwong.shellcheck
code --install-extension foxundermoon.shell-format
code --install-extension mads-hartmann.bash-ide-vscode
code --install-extension rogalmic.bash-debug
```

**VS Code Settings for Bash (settings.json):**

```json
{
    "files.associations": {
        "*.sh": "shellscript",
        ".bashrc": "shellscript",
        ".bash_profile": "shellscript"
    },
    "shellformat.flag": "-i 4 -bn -ci -sr",
    "[shellscript]": {
        "editor.defaultFormatter": "foxundermoon.shell-format",
        "editor.formatOnSave": true,
        "editor.tabSize": 4,
        "editor.insertSpaces": true
    },
    "shellcheck.enable": true,
    "shellcheck.run": "onType"
}
```

**Useful VS Code Features:**

```bash
# Command Palette (Ctrl+Shift+P / Cmd+Shift+P)
> Shell Format: Format Document

# Integrated Terminal (Ctrl+` / Cmd+`)
# Run scripts directly from editor

# Debugging
# Set breakpoints in bash scripts
# Use Bash Debug extension

# Snippets
# Type 'bash' and select template
# Creates script with shebang automatically
```

### Sublime Text

**Installation & Setup:**

```bash
# macOS
brew install --cask sublime-text

# Add Bash syntax highlighting
# Preferences → Settings

# Install Package Control
# Then install: BashSupport, SublimeLinter-shellcheck
```

### Atom

**Installation:**

```bash
# macOS
brew install --cask atom

# Install packages:
apm install language-shellscript
apm install linter-shellcheck
```

---

## Essential Tools and Utilities

### ShellCheck (Linter)

**Installation:**

```bash
# Ubuntu/Debian
sudo apt install shellcheck

# macOS
brew install shellcheck

# Fedora
sudo dnf install shellcheck

# From binary
curl -sS https://webi.sh/shellcheck | sh
```

**Usage:**

```bash
# Check a script
shellcheck myscript.sh

# Example output:
# In myscript.sh line 5:
# echo $var
#      ^--^ SC2086: Double quote to prevent globbing and word splitting.

# Fix automatically (in some editors)
# or manually apply suggestions
```

### shfmt (Formatter)

**Installation:**

```bash
# Using go
go install mvdan.cc/sh/v3/cmd/shfmt@latest

# macOS
brew install shfmt

# Binary download
curl -sS https://webi.sh/shfmt | sh
```

**Usage:**

```bash
# Format a script
shfmt -i 4 -bn -ci -sr myscript.sh

# Format in place
shfmt -i 4 -bn -ci -sr -w myscript.sh

# Format all scripts in directory
shfmt -i 4 -bn -ci -sr -w .
```

### Git (Version Control)

**Setup for Bash Projects:**

```bash
# Install git
sudo apt install git        # Linux
brew install git            # macOS

# Configure
git config --global user.name "Your Name"
git config --global user.email "your@email.com"

# Initialize repository
mkdir bash-scripts
cd bash-scripts
git init

# Create .gitignore
cat > .gitignore << 'EOF'
# Ignore temporary files
*.tmp
*.swp
*~

# Ignore logs
*.log

# Ignore OS files
.DS_Store
Thumbs.db
EOF

# Create README
echo "# My Bash Scripts" > README.md

# First commit
git add .
git commit -m "Initial commit"
```

### Environment Setup Script

Create a setup script to automate environment configuration:

```bash
#!/bin/bash
# setup_bash_env.sh - Setup Bash development environment

set -e

echo "Setting up Bash development environment..."

# Install essential tools
if command -v apt &> /dev/null; then
    sudo apt update
    sudo apt install -y vim git shellcheck curl wget
elif command -v brew &> /dev/null; then
    brew install vim git shellcheck
fi

# Create directory structure
mkdir -p ~/bash-scripts/{bin,lib,tests,docs}

# Setup .bashrc
cat >> ~/.bashrc << 'EOF'

# Custom Bash Development Settings
export BASH_SCRIPTS_HOME="$HOME/bash-scripts"
export PATH="$BASH_SCRIPTS_HOME/bin:$PATH"

# Useful aliases
alias bashreload='source ~/.bashrc'
alias bashedit='vim ~/.bashrc'
alias checksh='shellcheck'

# Development functions
newscript() {
    local name="$1"
    cat > "$name" << 'SCRIPT'
#!/bin/bash
# Description:
# Author:
# Date: $(date +%Y-%m-%d)

set -euo pipefail

main() {
    echo "Script started"
}

main "$@"
SCRIPT
    chmod +x "$name"
    vim "$name"
}
EOF

echo "✅ Environment setup complete!"
echo "Run: source ~/.bashrc"
```

---

## Summary

### Key Takeaways

1. **Bash** is available on all major platforms (Linux, macOS, Windows via WSL)
2. **WSL** is the best option for Bash on Windows
3. **ShellCheck** is essential for catching errors
4. **Choose an editor** that suits your workflow (Vim, Nano, VS Code)
5. **Version control** (Git) is important for managing scripts

### Quick Start Checklist

- [ ] Verify Bash is installed: `bash --version`
- [ ] Set Bash as default shell: `chsh -s /bin/bash`
- [ ] Install ShellCheck: `sudo apt install shellcheck`
- [ ] Setup editor (VS Code, Vim, or Nano)
- [ ] Create ~/bash-scripts directory
- [ ] Configure .bashrc with useful aliases
- [ ] Install Git for version control
- [ ] Create your first script

### Recommended Setup

```bash
# Minimal professional setup
1. Latest Bash (5.x+)
2. VS Code with ShellCheck extension
3. Git for version control
4. WSL 2 (if on Windows)
5. Basic .bashrc configuration
```

### Next Steps

Now that your environment is set up, you're ready to start learning Bash fundamentals!

---

**Next Chapter:** [Basic Shell Commands →](../part-02-fundamentals/04-basic-shell-commands.md)

---

*Practice Exercise: Complete the Quick Start Checklist above. Create a simple "Hello World" script and run it. Verify ShellCheck is working by checking the script.*
