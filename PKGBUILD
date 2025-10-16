# === Maintainer ===
# ohfp/lsf <lsf at pfho dot net>

: ${_build_profiled:=true}

pkgname=librewolf
_pkgname=LibreWolf
epoch=1
pkgver=144.0.0_1
_fixedfirefoxver="${pkgver%_*}"
_librewolfver="${pkgver#*_}"
_firefoxver="${_fixedfirefoxver%.0}"
pkgrel=1.1
pkgdesc="Community-maintained fork of Firefox, focused on privacy, security and freedom."
url="https://librewolf.net/"
arch=(x86_64 aarch64)
license=(MPL-2.0)

depends=(
  dbus
  alsa-lib
  at-spi2-core
  bash
  cairo
  ffmpeg4.4
  fontconfig
  freetype2
  gcc-libs
  gdk-pixbuf2
  glib2
  glibc
  gtk3
  hicolor-icon-theme
  libpulse
  libx11
  libxcb
  libxcomposite
  libxdamage
  libxext
  libxfixes
  libxrandr
  libxss
  libxt
  mime-types
  nspr
  nss
  pango
  ttf-font
)
makedepends=(
  binutils
  cbindgen
  clang
  diffutils
  git
  imake
  inetutils
  jack
  jq
  lld
  llvm
  perf
  mesa
  nasm
  nodejs
  pciutils
  python
  python-setuptools
  rust
  unzip
  'wasi-compiler-rt'
  'wasi-libc++'
  'wasi-libc++abi'
  'wasi-libc'
  llvm-bolt
  yasm
  zip
)
optdepends=(
  'hunspell-dictionary: Spell checking'
  'libnotify: Notification integration'
  'networkmanager: Location detection via available WiFi networks'
  'speech-dispatcher: Text-to-Speech'
  'xdg-desktop-portal: Screensharing with Wayland'
)

backup=(
  'usr/lib/librewolf/librewolf.cfg'
  'usr/lib/librewolf/distribution/policies.json'
)
options=(
  !debug
  !emptydirs
  !makeflags
)

install='librewolf.install'
source=(
  https://gitlab.com/api/v4/projects/32320088/packages/generic/librewolf-source/$_firefoxver-$_librewolfver/librewolf-$_firefoxver-$_librewolfver.source.tar.gz
  $pkgname.desktop
  "default192x192.png"
)

sha256sums=('7dbf8ebee436fd3efc5895b5151af0e23063ef1d3a47ff3da6d55dfcc1b047c6'
            '7d01d317b7db7416783febc18ee1237ade2ec86c1567e2c2dd628a94cbf2f25d'
            '959c94c68cab8d5a8cff185ddf4dca92e84c18dccc6dc7c8fe11c78549cdc2f1')

validpgpkeys=('034F7776EF5E0C613D2F7934D29FBD5F93C0CFC3')


prepare() {
  mkdir -p mozbuild
  cd librewolf-$_firefoxver-$_librewolfver

# === Benchmark Repository Setup ===
  if [[ "${_build_profiled}" == "true" ]]; then
    echo "Cloning repositories to Patchers folder..."
    if [[ ! -d "${startdir}/Patchers" ]]; then
      mkdir -p "${startdir}/Patchers"
    fi
    
    if [[ ! -d "${startdir}/Patchers/JetStream" ]]; then
      git clone -b JetStream2.2 --single-branch https://github.com/WebKit/JetStream.git "${startdir}/Patchers/JetStream"
    fi
    
    if [[ ! -d "${startdir}/Patchers/MotionMark" ]]; then
      git clone https://github.com/elyziumayo/MotionMark.git "${startdir}/Patchers/MotionMark"
    fi
  fi

  mv mozconfig ../mozconfig

  cat >>../mozconfig <<END

# === Clang + LLVM Toolchain ===
export CC=clang
export CXX=clang++
export AR=llvm-ar
export NM=llvm-nm
export STRIP=llvm-strip
export OBJCOPY=llvm-objcopy
export OBJDUMP=llvm-objdump
export READELF=llvm-readelf
export RANLIB=llvm-ranlib
export HOSTCC=clang
export HOSTCXX=clang++
export HOSTAR=llvm-ar
ac_add_options --enable-linker=lld
ac_add_options --target=x86_64-unknown-linux-gnu

# === Installation Path ===
ac_add_options --prefix=/usr

# === Build Configuration ===
ac_add_options --disable-bootstrap
ac_add_options --enable-jemalloc

# === Compiler Flags ===
export CFLAGS="-O3 -ffp-contract=fast -march=native -fno-semantic-interposition -fno-plt  -ffunction-sections -fdata-sections"
export CPPFLAGS="-O3 -ffp-contract=fast -march=native"
export CXXFLAGS="-O3 -ffp-contract=fast -march=native -fno-semantic-interposition -fno-plt -ffunction-sections -fdata-sections"

# === Linker Flags ===
export LDFLAGS="-Wl,-O3 -Wl,-mllvm,-fp-contract=fast -march=native -Wl,--threads=12 -Wl,--gc-sections -Wl,--icf=all -Wl,--as-needed"
export MOZ_LTO_LDFLAGS="-Wl,-mllvm,-polly"

# === Rust Optimizations ===
export RUSTFLAGS="-C target-cpu=native -C codegen-units=1 -C opt-level=3"

# === Polly Loop Optimizations ===
export POLLY="-mllvm -polly -mllvm -polly-2nd-level-tiling -mllvm -polly-loopfusion-greedy -mllvm -polly-pattern-matching-based-opts -mllvm -polly-position=before-vectorizer -mllvm -polly-vectorizer=stripmine"

# === Build System ===
export VERBOSE=1
mk_add_options MOZ_MAKE_FLAGS="-j12"
mk_add_options AUTOCLOBBER=1
export AUTOCLOBBER=1

# === Mozilla LTO Configuration ===
export MOZ_LTO=1
ac_add_options MOZ_LTO=1

# === Branding ===
ac_add_options --with-app-name=${pkgname}
ac_add_options --enable-update-channel=release
export MOZ_APP_REMOTINGNAME=${pkgname}

# === System Libraries ===
ac_add_options --with-system-nspr
ac_add_options --with-system-nss

# === Audio Support ===
ac_add_options --enable-alsa
ac_add_options --enable-jack
ac_add_options --enable-pulseaudio

# === WebAssembly Support ===
ac_add_options --with-wasi-sysroot=/usr/share/wasi-sysroot
ac_add_options --enable-wasm-avx

# === Optimization Levels ===
export OPT_LEVEL="3"
ac_add_options OPT_LEVEL="3"
export RUSTC_OPT_LEVEL="3"
ac_add_options RUSTC_OPT_LEVEL="3"
ac_add_options --enable-optimize="-O3 -march=native"

# === Debug Configuration ===
ac_add_options --disable-debug
ac_add_options --disable-debug-symbols
ac_add_options --disable-debug-js-modules

# === Additional Optimizations ===
ac_add_options --enable-rust-simd

# === Locale Configuration ===
ac_add_options --with-l10n-base=.
ac_add_options --enable-ui-locale=en-US

# === Disable Unnecessary Components ===
ac_add_options --disable-tests
ac_add_options --disable-dmd
ac_add_options --disable-webspeech
END

if [[ "${CARCH}" == "aarch64" ]]; then
  cat >>../mozconfig <<END
# === AArch64 Specific Configuration ===
ac_add_options --enable-optimize="-g0 -O3 -march=native"
ac_add_options --enable-lto
END

  export MOZ_DEBUG_FLAGS=" "
  export CFLAGS+=" -g0"
  export CXXFLAGS+=" -g0"
  export RUSTFLAGS="-Cdebuginfo=0"

else
  cat >>../mozconfig <<END
# === x86_64 Specific Configuration ===
ac_add_options --disable-elf-hack
ac_add_options --enable-lto=full
END
fi

# === Memory Management ===
  export LDFLAGS+=" -Wl,--no-keep-memory"
}


build() {
  cd librewolf-$_firefoxver-$_librewolfver

# === Build Environment Setup ===
  export MACH_BUILD_PYTHON_NATIVE_PACKAGE_SOURCE=pip
  export MOZBUILD_STATE_PATH="$srcdir/mozbuild"
  export MOZ_BUILD_DATE="$(date -u${SOURCE_DATE_EPOCH:+d @$SOURCE_DATE_EPOCH} +%Y%m%d%H%M%S)"
  export MOZ_NOSPAM=1

# === Compiler Flag Adjustments ===
  CFLAGS="${CFLAGS/_FORTIFY_SOURCE=3/_FORTIFY_SOURCE=2}"
  CXXFLAGS="${CXXFLAGS/_FORTIFY_SOURCE=3/_FORTIFY_SOURCE=2}"
  CFLAGS="${CFLAGS/-fexceptions/}"
  CXXFLAGS="${CXXFLAGS/-fexceptions/}"

# === System Limits ===
  ulimit -n 8192

# === PGO Build Process ===
  if [[ "${_build_profiled}" == "true" ]]; then
    # === Profile Generation Configuration ===
    if [[ "${CARCH}" == "aarch64" ]]; then
      cat >.mozconfig ../mozconfig - <<END
ac_add_options --enable-profile-generate
export MOZ_ENABLE_FULL_SYMBOLS=1
ac_add_options --enable-strip
ac_add_options --enable-install-strip
export STRIP_FLAGS="--strip-debug --strip-unneeded"
END
    else
      cat >.mozconfig ../mozconfig - <<END
ac_add_options --enable-profile-generate=cross
export MOZ_ENABLE_FULL_SYMBOLS=1
ac_add_options --enable-strip
ac_add_options --enable-install-strip
export STRIP_FLAGS="--strip-debug --strip-unneeded"
END
    fi

# === Disable Extensions for Profiling ===
    cp "lw/policies.json" "$srcdir/policies.json"
    jq 'del(.policies.Extensions.Install)' "$srcdir/policies.json" > "lw/policies.json"

    echo "Building instrumented browser..."
    ./mach build --priority normal

# === Enhanced Profiling Setup ===
    echo "Profiling instrumented browser..."
    ./mach package
    cp -f "${startdir}/Patchers/enhance.py" build/pgo/profileserver.py
    cp -f "${startdir}/Patchers/index.html" build/pgo/index.html
    cp -rf "${startdir}/Patchers/JetStream" third_party/webkit/PerformanceTests/
    cp -rf "${startdir}/Patchers/MotionMark/MotionMark" third_party/webkit/PerformanceTests/
    grep -n "add_host(.*primary" build/pgo/profileserver.py | cat || true
    LLVM_PROFDATA=llvm-profdata JARLOG_FILE="$PWD/jarlog" ./mach python build/pgo/profileserver.py

# === Profile Use Configuration ===
    echo "Removing instrumented browser..."
    ./mach clobber objdir

    echo "Building optimized browser..."

    if [[ -s merged.profdata ]]; then
      stat -c "Profile data found (%s bytes)" merged.profdata
      if [[ "${CARCH}" == "x86_64" ]]; then
        cat >.mozconfig ../mozconfig - <<END
ac_add_options --enable-profile-use
END
      else
        cat >.mozconfig ../mozconfig - <<END
ac_add_options --enable-profile-use=cross
END
      fi
      cat >> .mozconfig - << END
ac_add_options --with-pgo-profile-path=${PWD@Q}/merged.profdata
END
    else
      echo "Profile data not found."
    fi

    if [[ -s jarlog ]]; then
      stat -c "Jar log found (%s bytes)" jarlog
      cat >> .mozconfig - << END
ac_add_options --with-pgo-jarlog=${PWD@Q}/jarlog
END
    else
      echo "Jar log not found."
    fi

# === Re-enable Extensions ===
    cp "$srcdir/policies.json" "lw/policies.json"

  else
    cat >.mozconfig ../mozconfig
  fi

  # === Final Build Configuration ===
  cat >> .mozconfig <<END
export LDFLAGS="$LDFLAGS -Wl,--emit-relocs"
END
  echo "Final build with relocations enabled: LDFLAGS=$LDFLAGS"
  ./mach build --priority normal

# === BOLT Optimization ===
  if command -v perf >/dev/null 2>&1 && command -v perf2bolt >/dev/null 2>&1 && command -v llvm-bolt >/dev/null 2>&1; then
      echo "Collecting BOLT profile on optimized browser..."
      MOZ_DISABLE_PACKAGE_STRIP=1 ./mach package
      PERF_OUT="$PWD/bolt-perf.data"
      rm -f "$PERF_OUT"
      perf record -e cycles:u -j any,u -o "$PERF_OUT" -- ./mach python build/pgo/profileserver.py || true

      BIN="$(find . -path "./obj*" -type f -executable \( -name librewolf -o -name firefox \) | grep "/dist/" | head -n1)"
      if [[ -n "$BIN" && -s "$PERF_OUT" ]]; then
        if readelf -S "$BIN" | grep -q -E 'rel(a)?\.text|reloc|RELA|REL'; then
          echo "Relocations found in binary - BOLT will work with full optimizations"
        else
          echo "WARNING: No relocations found - BOLT will use limited optimizations"
        fi
        echo "Converting perf data to BOLT format..."
        perf2bolt -o bolt.fdata -p "$PERF_OUT" "$BIN"

        echo "Running llvm-bolt optimization..."
        llvm-bolt "$BIN" \
          -o "$BIN.bolted" \
          -data=bolt.fdata \
          -reorder-blocks=ext-tsp \
          -reorder-functions=cdsort \
          -split-functions \
          -split-all-cold \
          -split-strategy=profile2 \
          -jump-tables=aggressive \
          -icf=all \
          -group-stubs \
          -eliminate-unreachable \
          -peepholes=all \
          -shorten-instructions \
          -strip-rep-ret \
          -x86-strip-redundant-address-size \
          -jt-footprint-reduction \
          -simplify-rodata-loads \
          -inline-small-functions \
          -frame-opt=hot

        if [[ -s "$BIN.bolted" ]]; then
                 chmod +x "$BIN.bolted"
                 mv -f "$BIN.bolted" "$BIN"
                 echo "BOLT optimization applied to $BIN"
                 echo "Stripping final bolted binary..."
                 strip --strip-all "$BIN" || true

        echo "Removing extra unused sections aggressively with llvm-objcopy..."
                 llvm-objcopy \
                      --remove-section=.comment \
                      --remove-section=.eh_frame \
                      --remove-section=.note.gnu.build-id \
                      "$BIN" "$BIN.trimmed"

      if [[ -s "$BIN.trimmed" ]]; then
                      mv -f "$BIN.trimmed" "$BIN"
             echo "Binary aggressively trimmed with llvm-objcopy"
         else
            echo "llvm-objcopy trimming failed or produced empty file, skipping"
      fi
        else
            echo "BOLT output missing; skipping replacement"
        fi
      else
        echo "Skipping BOLT: binary or perf data missing"
      fi

  else
    echo "ERROR: BOLT tools not available - perf, perf2bolt, or llvm-bolt missing"
    echo "BOLT optimization is required for this build"
    exit 1
  fi
}

package() {
  cd librewolf-$_firefoxver-$_librewolfver
  DESTDIR="$pkgdir" ./mach install

# === Vendor Configuration ===
  local vendorjs="$pkgdir/usr/lib/$pkgname/browser/defaults/preferences/vendor.js"
  install -Dvm644 /dev/stdin "$vendorjs" <<END
pref("spellchecker.dictionary_path", "/usr/share/hunspell");
END

# === Distribution Configuration ===
  local distini="$pkgdir/usr/lib/$pkgname/distribution/distribution.ini"
  install -Dvm644 /dev/stdin "$distini" <<END
[Global]
id=io.gitlab.${pkgname}-community
version=1.0
about=LibreWolf

[Preferences]
app.distributor="LibreWolf Community"
app.distributor.channel=$pkgname
app.partner.librewolf=$pkgname
END

# === Icon Installation ===
  for i in 16 32 48 64 128; do
    install -Dvm644 browser/branding/${pkgname}/default$i.png \
      "$pkgdir/usr/share/icons/hicolor/${i}x${i}/apps/$pkgname.png"
  done
  install -Dvm644 ${srcdir}/default192x192.png \
    "$pkgdir/usr/share/icons/hicolor/192x192/apps/$pkgname.png"
  install -Dvm644 browser/branding/${pkgname}/default16.png \
    "$pkgdir/usr/share/icons/hicolor/symbolic/apps/$pkgname-symbolic.png"
  install -Dvm644 ../$pkgname.desktop \
    "$pkgdir/usr/share/applications/$pkgname.desktop"

# === Binary Wrapper ===
  install -Dvm755 /dev/stdin "$pkgdir/usr/bin/$pkgname" <<END
#!/bin/sh
exec /usr/lib/$pkgname/librewolf "\$@"
END
  ln -srfv "$pkgdir/usr/bin/$pkgname" "$pkgdir/usr/lib/$pkgname/librewolf-bin"

# === System Certificates ===
  local nssckbi="$pkgdir/usr/lib/$pkgname/libnssckbi.so"
  if [[ -e $nssckbi ]]; then
    ln -srfv "$pkgdir/usr/lib/libnssckbi.so" "$nssckbi"
  fi
}
