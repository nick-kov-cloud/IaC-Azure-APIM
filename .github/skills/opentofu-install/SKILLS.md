As of early 2026, the current stable version is v1.11.0, which introduced features like ephemeral resources and native S3 state locking.

**Choose the method that fits your system below:**


## 💻 macOS

The easiest way to install and keep OpenTofu updated on Mac is via Homebrew.
```
Bash
brew update
brew install opentofu
Verify with: tofu -version
```
## 🐧 Linux
You can use the official installer script or your distribution's package manager.

1. Using the Installer Script (Universal)
This is the quickest "one-liner" for most Linux distros:
```
Bash
curl --proto '=https' --tlsv1.2 -fsSL https://get.opentofu.org/install-opentofu.sh -o install-opentofu.sh
chmod +x install-opentofu.sh
./install-opentofu.sh --install-method standalone
rm install-opentofu.sh
2. Debian/Ubuntu (APT)
Bash
# Install required tooling
sudo apt-get update && sudo apt-get install -y apt-transport-https ca-certificates curl gnupg

# Add the OpenTofu GPG keys
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://get.opentofu.org/opentofu.gpg | sudo tee /etc/apt/keyrings/opentofu.gpg >/dev/null
curl -fsSL https://packages.opentofu.org/opentofu/tofu/gpgkey | sudo gpg --no-tty --batch --dearmor -o /etc/apt/keyrings/opentofu-repo.gpg >/dev/null

# Add the repository
echo "deb [signed-by=/etc/apt/keyrings/opentofu.gpg,/etc/apt/keyrings/opentofu-repo.gpg] https://packages.opentofu.org/opentofu/tofu/any/ any main" | sudo tee /etc/apt/sources.list.d/opentofu.list > /dev/null

# Install
sudo apt-get update
sudo apt-get install -y tofu
```

## 🪟 Windows
You can use a package manager like Winget or Scoop.

### Option A: Winget (Recommended)

```
PowerShell
winget install --exact --id=OpenTofu.Tofu
Option B: Scoop
PowerShell
scoop bucket add main
scoop install main/opentofu
```


## 🐳 Docker
If you prefer running OpenTofu in a container, you can pull the official minimal image from GitHub Packages:
```
Bash
docker pull ghcr.io/opentofu/opentofu:minimal
Note: Starting with v1.10, the official images are intended as base images for multi-stage builds rather than direct entry points.
```

## ✅ Next Step
Once installed, you can initialize your first project by running:
```
Bash
tofu init
```