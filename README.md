# 🔥 LibreWolf-PGO 🔥

📌 Version: 138.0.3_1

A performance-optimized build of LibreWolf for Arch Linux that's been hitting the compiler gym. Think of it as LibreWolf after a few cups of espresso. ☕


## 📋 About This Project

This repo contains a faster version of LibreWolf, inspired by Mercury browser. I've tinkered with the build settings until the compiler started sweating. The browser didn't know what hit it.

## ⚡ What Makes It Faster

| Feature | Benefit |
|---------|---------|
| **PGO** | Uses your browsing patterns to optimize code paths |
| **-O3** | Higher optimization level than default -O2 |
| **AVX2** | Uses modern CPU instructions for faster processing |
| **Mold** | Faster linker than default LLD |
| **Parallel Build** | Uses multiple CPU cores for faster compiling |
| **Rust Optimizations** | Better settings for Rust components |

## 📦 Installation

### Pre-built Binary

1. Download the package from the Releases section
2. Install with:
   ```bash
   sudo pacman -U librewolf-1:138.0.3_1-1-x86_64.pkg.tar.zst
   ```
> 💡 **Tip:** Review the PKGBUILD file for detailed optimization info

## ✨ Inspiration

Inspired by [Mercury Browser](https://github.com/Alex313031/Mercury) - a Firefox fork that proved optimized compiler settings can make browsers significantly faster. 

## ⚠️ Important Notes

Unofficial rookie project. Works on my machine™. Needs AVX2. Use at your own risk!

---

## 📄 License

MPL-2.0 (Same as LibreWolf)
