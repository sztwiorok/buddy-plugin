# Buddy CLI Installation Guide

Complete installation instructions for the Buddy CLI (bdy) across all supported platforms.

## Table of Contents
- [NPM Installation (Cross-Platform)](#npm-installation-cross-platform)
- [Linux Installation](#linux-installation)
- [macOS Installation](#macos-installation)
- [Windows Installation](#windows-installation)
- [Verifying Installation](#verifying-installation)
- [Updating bdy](#updating-bdy)
- [Troubleshooting](#troubleshooting)

## NPM Installation (Cross-Platform)

**Recommended for all platforms** - Works on Linux, macOS, and Windows.

### Requirements
- Node.js version 20 or higher

### Installation

```bash
npm install -g bdy
```

**Verify installation:**
```bash
bdy version
```

**Update:**
```bash
npm update -g bdy
```

**Uninstall:**
```bash
npm uninstall -g bdy
```

## Linux Installation

### Linux x64 (amd64)

#### Method 1: APT Package Manager (Recommended for Debian/Ubuntu)

```bash
# Install prerequisites
sudo apt-get update && sudo apt-get install -y software-properties-common

# Add Buddy GPG key
sudo gpg --homedir /tmp --no-default-keyring \
  --keyring /usr/share/keyrings/buddy.gpg \
  --keyserver hkp://keyserver.ubuntu.com:80 \
  --recv-keys eb39332e766364ca6220e8dc631c5a16310cc0ad

# Add Buddy repository
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/buddy.gpg] https://es.buddy.works/bdy/apt-repo prod main" | \
  sudo tee /etc/apt/sources.list.d/buddy.list > /dev/null

# Install bdy
sudo apt-get update && sudo apt-get install -y bdy
```

#### Method 2: Direct Download

```bash
# Download and extract
curl https://es.buddy.works/bdy/prod/1.16.12/linux-x64.tar.gz -o bdy.tar.gz
sudo tar -zxf bdy.tar.gz -C /usr/local/bin/

# Verify installation
bdy version
```

### Linux ARM64

#### Method 1: APT Package Manager (Recommended for Debian/Ubuntu)

```bash
# Install prerequisites
sudo apt-get update && sudo apt-get install -y software-properties-common

# Add Buddy GPG key
sudo gpg --homedir /tmp --no-default-keyring \
  --keyring /usr/share/keyrings/buddy.gpg \
  --keyserver hkp://keyserver.ubuntu.com:80 \
  --recv-keys eb39332e766364ca6220e8dc631c5a16310cc0ad

# Add Buddy repository
echo "deb [arch=arm64 signed-by=/usr/share/keyrings/buddy.gpg] https://es.buddy.works/bdy/apt-repo prod main" | \
  sudo tee /etc/apt/sources.list.d/buddy.list > /dev/null

# Install bdy
sudo apt-get update && sudo apt-get install -y bdy
```

#### Method 2: Direct Download

```bash
# Download and extract
curl https://es.buddy.works/bdy/prod/1.16.12/linux-arm64.tar.gz -o bdy.tar.gz
sudo tar -zxf bdy.tar.gz -C /usr/local/bin/

# Verify installation
bdy version
```

## macOS Installation

### Method 1: Homebrew (Recommended)

```bash
# Add Buddy tap
brew tap buddy/bdy

# Install bdy
brew install bdy
```

**Update via Homebrew:**
```bash
brew update
brew upgrade bdy
```

### Method 2: Direct Download

```bash
# Create directory if it doesn't exist
sudo mkdir -p -m 755 /usr/local/bin

# Download and extract (ARM64 for Apple Silicon)
curl https://es.buddy.works/bdy/prod/1.16.12/darwin-arm64.tar.gz -o bdy.tar.gz
sudo tar -zxf bdy.tar.gz -C /usr/local/bin/

# Verify installation
bdy version
```

**Note:** For Intel Macs, use `darwin-x64.tar.gz` instead of `darwin-arm64.tar.gz`.

## Windows Installation

### Method 1: Chocolatey (Recommended)

```powershell
choco install bdy --version=1.16.12-prod --pre
```

### Method 2: Direct Download

```powershell
# Download
curl https://es.buddy.works/bdy/prod/1.16.12/win-x64.zip -o bdy.zip

# Extract
tar -xf bdy.zip

# Add to PATH manually or move to a directory in PATH
```

**Add to PATH:**
1. Right-click "This PC" → Properties → Advanced system settings
2. Click "Environment Variables"
3. Edit PATH and add the directory containing bdy.exe

## Verifying Installation

After installation, verify bdy is correctly installed:

```bash
# Check version
bdy version

# View help
bdy --help
```

**Expected output:**
```
bdy version 1.16.12
```

## Updating bdy

### NPM (All Platforms)

```bash
npm update -g bdy
```

### Homebrew (macOS)

```bash
brew update
brew upgrade bdy
```

### APT (Debian/Ubuntu Linux)

```bash
sudo apt-get update
sudo apt-get upgrade bdy
```

### Chocolatey (Windows)

```powershell
choco upgrade bdy
```

### Manual Update

Download and install the latest version using the same method as initial installation.

## Troubleshooting

### Command Not Found

**Symptom:** `bdy: command not found` after installation

**Solutions:**
1. Verify installation completed without errors
2. Check binary is in PATH:
   ```bash
   which bdy
   ```
3. Try restarting terminal/shell
4. Manually add to PATH:
   ```bash
   export PATH=$PATH:/usr/local/bin
   ```

### Permission Denied

**Symptom:** Permission errors when running bdy

**Solutions:**
1. Ensure binary has execute permissions:
   ```bash
   chmod +x $(which bdy)
   ```
2. If installed in system directory, use sudo during installation
3. For personal installation, install to user directory:
   ```bash
   mkdir -p ~/.local/bin
   mv bdy ~/.local/bin/
   export PATH=$PATH:~/.local/bin
   ```

### SSL/TLS Certificate Errors

**Symptom:** Certificate validation errors when running commands

**Solutions:**
1. Update system certificates:
   ```bash
   # macOS
   brew install ca-certificates

   # Ubuntu/Debian
   sudo apt install ca-certificates

   # Red Hat/CentOS
   sudo yum install ca-certificates
   ```
2. Verify system time is correct (certificate validation depends on accurate time)

### Installation Script Fails

**Symptom:** Install script exits with errors

**Solutions:**
1. Check internet connectivity
2. Verify curl is installed:
   ```bash
   curl --version
   ```
3. Try manual download method instead
4. Check system architecture is supported:
   ```bash
   uname -m
   ```

### Binary Architecture Mismatch

**Symptom:** "cannot execute binary file" or "exec format error"

**Solutions:**
1. Download correct binary for your architecture:
   - x86_64/amd64 for most modern systems
   - arm64 for Apple Silicon Macs
   - i386 for older 32-bit systems
2. Check system architecture:
   ```bash
   # Linux/macOS
   uname -m

   # Windows PowerShell
   $env:PROCESSOR_ARCHITECTURE
   ```

## Platform-Specific Notes

### NPM Installation

- **Recommended for all platforms** - Consistent experience across operating systems
- **Requires Node.js v20 or higher** - Check with `node --version`
- **Automatic updates** - Use `npm update -g bdy` to update
- **Easy uninstall** - Use `npm uninstall -g bdy` to remove

### macOS

- **Homebrew is recommended** for macOS-native package management
- **Apple Silicon (M1/M2/M3)** - Use `darwin-arm64.tar.gz` binary
- **Intel Macs** - Use `darwin-x64.tar.gz` binary
- **Gatekeeper may block** the binary on first run. If this occurs:
  ```bash
  xattr -d com.apple.quarantine $(which bdy)
  ```

### Linux

- **APT recommended for Debian/Ubuntu** - Integrated with system package manager
- **ARM64 support** - Full support for ARM-based systems (Raspberry Pi, etc.)
- **x64 (amd64)** - Standard 64-bit Intel/AMD processors
- **systemd-based distributions** (Ubuntu, Debian, Fedora) are fully supported

### Windows

- **Chocolatey recommended** for Windows package management
- **PowerShell execution policy** may need adjustment for scripts:
  ```powershell
  Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
  ```
- **Windows Subsystem for Linux (WSL)** can use Linux installation methods
- **Direct download** provides standalone executable without dependencies

## Offline Installation

For environments without internet access:

1. Download binary on a connected machine
2. Transfer binary to target machine via USB/network
3. Place in appropriate directory and make executable
4. Configure authentication using environment variables or config files

## Docker/Container Usage

Run bdy in containers using NPM installation:

```dockerfile
FROM node:20-alpine

# Install bdy via NPM
RUN npm install -g bdy

CMD ["bdy", "--help"]
```

**Ubuntu-based container:**
```dockerfile
FROM ubuntu:22.04

# Install Node.js and bdy
RUN apt-get update && \
    apt-get install -y curl && \
    curl -fsSL https://deb.nodesource.com/setup_20.x | bash - && \
    apt-get install -y nodejs && \
    npm install -g bdy && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

CMD ["bdy", "--help"]
```

**Direct binary installation (smaller image):**
```dockerfile
FROM ubuntu:22.04

RUN apt-get update && \
    apt-get install -y curl && \
    curl https://es.buddy.works/bdy/prod/1.16.12/linux-x64.tar.gz -o bdy.tar.gz && \
    tar -zxf bdy.tar.gz -C /usr/local/bin/ && \
    rm bdy.tar.gz && \
    apt-get remove -y curl && \
    apt-get autoremove -y && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

CMD ["bdy", "--help"]
```

## CI/CD Integration

### GitHub Actions

```yaml
- name: Install Buddy CLI
  run: |
    npm install -g bdy
    bdy version
```

**Using setup-node action:**
```yaml
- uses: actions/setup-node@v4
  with:
    node-version: '20'
- name: Install Buddy CLI
  run: npm install -g bdy
```

### GitLab CI

```yaml
image: node:20

before_script:
  - npm install -g bdy
  - bdy version
```

### Jenkins

```groovy
stage('Install Buddy CLI') {
    steps {
        sh 'npm install -g bdy'
        sh 'bdy version'
    }
}
```

### CircleCI

```yaml
version: 2.1
jobs:
  build:
    docker:
      - image: node:20
    steps:
      - checkout
      - run: npm install -g bdy
      - run: bdy version
```

## Uninstallation

### Homebrew

```bash
brew uninstall bdy
brew untap buddy-works/tap
```

### Manual Installation

```bash
# Remove binary
sudo rm /usr/local/bin/bdy

# Remove config (optional)
rm -rf ~/.buddy
```

### NPM

```powershell
npm uninstall -g @buddy-works/cli
```

### APT (Debian/Ubuntu)

```bash
sudo apt-get remove bdy
sudo rm /etc/apt/sources.list.d/buddy.list
sudo rm /usr/share/keyrings/buddy.gpg
```

### Chocolatey (Windows)

```powershell
choco uninstall bdy
```
