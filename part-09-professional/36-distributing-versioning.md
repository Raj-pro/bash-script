# Chapter 36: Distributing and Versioning Scripts

## Introduction

Professional script distribution requires proper versioning, packaging, and documentation. This chapter covers Git workflows, semantic versioning, and distribution methods.

---

## Version Control with Git

### Git Best Practices

```bash
# Initialize repository
git init
git add .
git commit -m "Initial commit"

# Create .gitignore
cat > .gitignore << 'EOF'
*.log
*.tmp
.env
.env.local
secrets/
coverage/
.DS_Store
EOF

# Add README
cat > README.md << 'EOF'
# My Bash Script

Description of what the script does.

## Installation

```bash
git clone https://github.com/user/script.git
cd script
./install.sh
```

## Usage

```bash
./script.sh [options]
```

## License

MIT
EOF
```

### Branching Strategy

```bash
# Main branches
git checkout -b main       # Production-ready code
git checkout -b develop    # Development branch

# Feature branches
git checkout -b feature/new-backup-method
git checkout -b fix/backup-rotation

# Release branches
git checkout -b release/v1.2.0

# Hotfix branches
git checkout -b hotfix/critical-bug
```

---

## Semantic Versioning

### Version Format

```
MAJOR.MINOR.PATCH

1.0.0  # Initial release
1.0.1  # Patch: bug fixes
1.1.0  # Minor: new features (backward compatible)
2.0.0  # Major: breaking changes
```

### Version Management Script

```bash
#!/bin/bash
# version.sh

VERSION_FILE="VERSION"
CURRENT_VERSION=$(cat "$VERSION_FILE" 2>/dev/null || echo "0.0.0")

bump_version() {
    local part=$1
    
    IFS='.' read -ra PARTS <<< "$CURRENT_VERSION"
    local major=${PARTS[0]}
    local minor=${PARTS[1]}
    local patch=${PARTS[2]}
    
    case $part in
        major)
            major=$((major + 1))
            minor=0
            patch=0
            ;;
        minor)
            minor=$((minor + 1))
            patch=0
            ;;
        patch)
            patch=$((patch + 1))
            ;;
    esac
    
    NEW_VERSION="$major.$minor.$patch"
    echo "$NEW_VERSION" > "$VERSION_FILE"
    
    # Update in script
    sed -i "s/readonly VERSION=.*/readonly VERSION=\"$NEW_VERSION\"/" script.sh
    
    # Git tag
    git add "$VERSION_FILE" script.sh
    git commit -m "Bump version to $NEW_VERSION"
    git tag -a "v$NEW_VERSION" -m "Version $NEW_VERSION"
    
    echo "Version bumped to $NEW_VERSION"
}

case "${1:-}" in
    major|minor|patch)
        bump_version "$1"
        ;;
    *)
        echo "Current version: $CURRENT_VERSION"
        echo "Usage: $0 {major|minor|patch}"
        ;;
esac
```

---

## Changelog Management

### CHANGELOG.md Format

```markdown
# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- New backup rotation feature

### Changed
- Improved error messages

### Deprecated
- Old configuration format will be removed in v2.0.0

### Removed
- Deprecated --legacy flag

### Fixed
- Fixed bug in date parsing

### Security
- Fixed path traversal vulnerability

## [1.1.0] - 2025-01-15

### Added
- Support for remote backups
- Email notifications

### Fixed
- Memory leak in log rotation

## [1.0.0] - 2025-01-01

### Added
- Initial release
- Basic backup functionality
- Configuration file support
```

### Auto-Generate Changelog

```bash
#!/bin/bash
# generate-changelog.sh

LAST_TAG=$(git describe --tags --abbrev=0 2>/dev/null || echo "")
NEW_VERSION=$1

if [ -z "$NEW_VERSION" ]; then
    echo "Usage: $0 <version>"
    exit 1
fi

echo "## [$NEW_VERSION] - $(date +%Y-%m-%d)" > CHANGELOG.new

echo "" >> CHANGELOG.new
echo "### Changes" >> CHANGELOG.new

if [ -n "$LAST_TAG" ]; then
    git log $LAST_TAG..HEAD --pretty=format:"- %s" >> CHANGELOG.new
else
    git log --pretty=format:"- %s" >> CHANGELOG.new
fi

echo "" >> CHANGELOG.new
echo "" >> CHANGELOG.new

# Prepend to existing changelog
cat CHANGELOG.md >> CHANGELOG.new
mv CHANGELOG.new CHANGELOG.md

echo "Changelog updated for version $NEW_VERSION"
```

---

## Packaging Scripts

### Simple Installer

```bash
#!/bin/bash
# install.sh

set -euo pipefail

INSTALL_DIR="/usr/local/bin"
SCRIPT_NAME="myscript"
CONFIG_DIR="$HOME/.config/$SCRIPT_NAME"

echo "Installing $SCRIPT_NAME..."

# Copy script
sudo cp "$SCRIPT_NAME.sh" "$INSTALL_DIR/$SCRIPT_NAME"
sudo chmod +x "$INSTALL_DIR/$SCRIPT_NAME"

# Create config directory
mkdir -p "$CONFIG_DIR"
cp config/default.conf "$CONFIG_DIR/config.conf"

# Create man page
sudo mkdir -p /usr/local/share/man/man1
sudo cp docs/$SCRIPT_NAME.1 /usr/local/share/man/man1/
sudo mandb

echo "Installation complete!"
echo "Run: $SCRIPT_NAME --help"
```

### Tarball Distribution

```bash
#!/bin/bash
# create-release.sh

VERSION=$1
PROJECT="myscript"
RELEASE_DIR="releases"

if [ -z "$VERSION" ]; then
    echo "Usage: $0 <version>"
    exit 1
fi

mkdir -p "$RELEASE_DIR"

# Create archive
tar -czf "$RELEASE_DIR/${PROJECT}-${VERSION}.tar.gz" \
    --exclude='.git' \
    --exclude='releases' \
    --exclude='*.log' \
    --transform "s|^|${PROJECT}-${VERSION}/|" \
    .

# Create checksum
cd "$RELEASE_DIR"
sha256sum "${PROJECT}-${VERSION}.tar.gz" > "${PROJECT}-${VERSION}.tar.gz.sha256"

echo "Release created: $RELEASE_DIR/${PROJECT}-${VERSION}.tar.gz"
```

---

## Docker Distribution

### Dockerfile for Bash Script

```dockerfile
FROM alpine:latest

# Install dependencies
RUN apk add --no-cache bash curl jq

# Copy script and dependencies
COPY script.sh /usr/local/bin/script
COPY lib/ /usr/local/lib/script/

# Make executable
RUN chmod +x /usr/local/bin/script

# Create volume for data
VOLUME /data

# Set entrypoint
ENTRYPOINT ["/usr/local/bin/script"]
CMD ["--help"]
```

### Build and Publish

```bash
#!/bin/bash
# docker-release.sh

VERSION=$1
IMAGE_NAME="myorg/myscript"

if [ -z "$VERSION" ]; then
    echo "Usage: $0 <version>"
    exit 1
fi

# Build image
docker build -t "$IMAGE_NAME:$VERSION" .
docker tag "$IMAGE_NAME:$VERSION" "$IMAGE_NAME:latest"

# Push to registry
docker push "$IMAGE_NAME:$VERSION"
docker push "$IMAGE_NAME:latest"

echo "Docker image published: $IMAGE_NAME:$VERSION"
```

---

## GitHub Releases

### Create Release Script

```bash
#!/bin/bash
# create-github-release.sh

set -euo pipefail

VERSION=$1
REPO="user/repo"
GITHUB_TOKEN="${GITHUB_TOKEN:-}"

if [ -z "$VERSION" ] || [ -z "$GITHUB_TOKEN" ]; then
    echo "Usage: GITHUB_TOKEN=xxx $0 <version>"
    exit 1
fi

# Create release
release_id=$(curl -s -X POST \
    -H "Authorization: token $GITHUB_TOKEN" \
    -H "Content-Type: application/json" \
    -d "{
        \"tag_name\": \"v$VERSION\",
        \"name\": \"Release $VERSION\",
        \"body\": \"$(cat CHANGELOG.md | sed -n '/## \['$VERSION'\]/,/## \[/p' | head -n -1)\",
        \"draft\": false,
        \"prerelease\": false
    }" \
    "https://api.github.com/repos/$REPO/releases" | \
    jq -r '.id')

# Upload assets
for file in releases/*; do
    curl -s -X POST \
        -H "Authorization: token $GITHUB_TOKEN" \
        -H "Content-Type: application/octet-stream" \
        --data-binary @"$file" \
        "https://uploads.github.com/repos/$REPO/releases/$release_id/assets?name=$(basename $file)"
done

echo "GitHub release created: v$VERSION"
```

---

## License Management

### Add License

```bash
#!/bin/bash
# add-license.sh

LICENSE_TYPE=${1:-MIT}

case $LICENSE_TYPE in
    MIT)
        cat > LICENSE << 'EOF'
MIT License

Copyright (c) 2025 Your Name

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
EOF
        ;;
    *)
        echo "Unknown license: $LICENSE_TYPE"
        exit 1
        ;;
esac

echo "License added: $LICENSE_TYPE"
```

---

## Documentation

### Generate Documentation

```bash
#!/bin/bash
# generate-docs.sh

# Extract help text from script
./script.sh --help > docs/usage.txt

# Generate man page
cat > docs/myscript.1 << 'EOF'
.TH MYSCRIPT 1 "2025-01-01" "1.0.0" "User Commands"
.SH NAME
myscript \- Description of the script
.SH SYNOPSIS
.B myscript
[\fB\-h\fR]
[\fB\-v\fR]
.SH DESCRIPTION
Detailed description of what the script does.
.SH OPTIONS
.TP
.BR \-h ", " \-\-help
Display help message
.TP
.BR \-v ", " \-\-version
Display version information
.SH AUTHOR
Written by Your Name
.SH SEE ALSO
bash(1)
EOF

echo "Documentation generated"
```

---

## Best Practices

1. **Use semantic versioning**
2. **Maintain a changelog**
3. **Tag releases in Git**
4. **Provide installation instructions**
5. **Include license file**
6. **Write comprehensive README**
7. **Create man pages for tools**
8. **Test releases before publishing**
9. **Sign releases (GPG)**
10. **Provide checksums**

---

## Summary

- Version scripts with semantic versioning
- Use Git for version control
- Maintain changelog
- Package scripts properly
- Distribute via multiple channels
- Document installation and usage
- Include appropriate license

---

## Congratulations!

You've completed the comprehensive Bash scripting guide. You now have the knowledge to write professional, maintainable, and efficient Bash scripts for system administration, DevOps, and automation tasks.

Keep practicing, keep learning, and happy scripting! 🚀
