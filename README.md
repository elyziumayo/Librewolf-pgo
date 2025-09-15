# 🌩️ librewolf-pgo

High-performance LibreWolf with PGO + BOLT optimizations and multi-benchmark profiling.

## ✨ Features

- 👤 **PGO Profiling**: Default + JetStream 2.0 + MotionMark
- ⚡ **BOLT Optimization**: Post-link binary optimization
- 🏠 **Native Optimization**: Auto-tuned for your specific CPU when building from source
- 📦 **Pre-built x86_64-v3**: Available from custom repository for modern processors

## 💾 Repo Installation
> [!WARNING]
> Pre-built packages are compiled for x86_64-v3. Make sure your CPU supports x86_64-v3 before proceeding.
> > **🔍 Quick CPU Support Check** 
> ```bash
> grep -q "avx\|avx2\|bmi1\|bmi2\|f16c\|fma\|lzcnt\|movbe\|xsave" /proc/cpuinfo && echo "✅ x86_64-v3 supported" || echo "❌ x86_64-v3 not supported"
> ```

> ### 🛰️ Quick Repo Install
```bash
sudo bash -c 'echo -e "\n[Erevos]\nServer = https://erevos.elyzium.net/\$arch-v3\nSigLevel = Never" >> /etc/pacman.conf && pacman -Syy && pacman -S librewolf'
```

> ### **⚒️ Manual Repo Install**

**Step 1: Add repository to pacman.conf**
```bash
sudo nano /etc/pacman.conf
```
> Add these lines at the end of the file:
```
[Erevos]
Server = https://erevos.elyzium.net/$arch-v3
SigLevel = Never
```

**Step 2: Update package database and install**
```bash
sudo pacman -Syy && sudo pacman -S librewolf
```

## 🔨 Build from Source
Clone and build (auto-optimized for your CPU):
```bash
git clone https://github.com/elyziumayo/Librewolf-pgo.git
cd Librewolf-pgo
makepkg -si
```

