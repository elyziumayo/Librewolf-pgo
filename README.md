# 🔥 LibreWolf-PGO

📌 Version: 141.0.0_1

A performance-optimized build of LibreWolf for Arch Linux that's been hitting the compiler gym. Think of it as LibreWolf after a few cups of espresso. ☕

## ⚡ What Makes It Faster

| Feature | Benefit |
|---------|---------|
| **PGO (Profile-Guided Optimization)** | Uses real browsing patterns to optimize hot code paths |
| **LTO (Link-Time Optimization)** | Optimizes across module boundaries for better performance |
| **-O3** | Higher optimization level than default -O2 |
| **x86-64-v3** | Targets modern CPUs with AVX/AVX2/BMI/etc. support |
| **Mold Linker** | Up to 5× faster linking than the default LLD |
| **SIMD Optimizations** | Vectorized operations for parallel data processing |
| **Rust Performance** | Optimized Rust components with reduced codegen units |

## 🎭 Installation
### Install the pre-built package
```bash
sudo pacman -U librewolf-1:141.0.0_1-1.1-x86_64.pkg.tar.zst
```
> 💡 **Tip:** Review the PKGBUILD file for detailed optimization info

## ✨ Inspiration

Inspired by [Mercury Browser](https://github.com/Alex313031/Mercury) - a Firefox fork that proved optimized compiler settings can make browsers significantly faster.



## ⚠️ Important Notes

This is an unofficial optimization project. While it works well on modern hardware, your mileage may vary. Requires a CPU with AVX2 support.

---

## 📄 License

MPL-2.0 (Same as LibreWolf) 
