# Maintainer: ohfp/lsf <lsf at pfho dot net>

# run pgo build or not; with X(vfb) or wayland
: ${_build_profiled:=true}
: ${_build_profiled_xvfb:=false}

pkgname=librewolf
_pkgname=LibreWolf
epoch=1
pkgver=141.0.0_1
_fixedfirefoxver="${pkgver%_*}" # Version of Firefox this LibreWolf version is based on, but the Firefox patch number is always included
_librewolfver="${pkgver#*_}"
_firefoxver="${_fixedfirefoxver%.0}" # Removes ".0" from the end. For "136.0.0" this will result in "136.0" but for "136.0.1" won't do anything.
pkgrel=1
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
  ffmpeg
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
  mold
  llvm
  mesa
  nasm
  nodejs
  pciutils
  lld
  python
  python-setuptools
  rust
  unzip
  'wasi-compiler-rt'
  'wasi-libc++'
  'wasi-libc++abi'
  'wasi-libc'
  yasm
  zip
  ) # pciutils: only to avoid some PGO warning(?)
optdepends=(
  'hunspell-dictionary: Spell checking'
  'libnotify: Notification integration'
  'networkmanager: Location detection via available WiFi networks'
  'speech-dispatcher: Text-to-Speech'
  'xdg-desktop-portal: Screensharing with Wayland'
)

if [[ "${_build_profiled}" == "true" ]]; then
  if [[ "${_build_profiled_xvfb}" == "true" ]]; then
    makedepends+=(
      xorg-server-xvfb
    )
  else
    makedepends+=(
      weston
      xorg-xwayland
      wlheadless-run # aur/xwayland-run-git
    )
  fi
fi

backup=('usr/lib/librewolf/librewolf.cfg'
        'usr/lib/librewolf/distribution/policies.json')
options=(
  !debug
  !emptydirs
  !makeflags
)

install='librewolf.install'
source=(
  https://gitlab.com/api/v4/projects/32320088/packages/generic/librewolf-source/$_firefoxver-$_librewolfver/librewolf-$_firefoxver-$_librewolfver.source.tar.gz # {,.sig} sig files are currently broken, it seems
  $pkgname.desktop
  "default192x192.png"
)

sha256sums=('59bbd2e298e56c96dd8879df2b2220d385f40d9e49d7aa843f036e9bdff61e1f'
            '7d01d317b7db7416783febc18ee1237ade2ec86c1567e2c2dd628a94cbf2f25d'
            '959c94c68cab8d5a8cff185ddf4dca92e84c18dccc6dc7c8fe11c78549cdc2f1')

validpgpkeys=('034F7776EF5E0C613D2F7934D29FBD5F93C0CFC3') # maltej(?)


prepare() {
  mkdir -p mozbuild
  cd librewolf-$_firefoxver-$_librewolfver

  mv mozconfig ../mozconfig

  cat >>../mozconfig <<END

ac_add_options --enable-linker=mold

ac_add_options --prefix=/usr

ac_add_options --disable-bootstrap
ac_add_options --enable-jemalloc

export CC='clang'
export CXX='clang++'

# Compiler, Linker, and Rust flags
export CFLAGS="-O3 -ffp-contract=fast -march=x86-64-v3"
export CPPFLAGS="-O3 -ffp-contract=fast -march=x86-64-v3"
export CXXFLAGS="-O3 -ffp-contract=fast -march=x86-64-v3"
export LDFLAGS="-Wl,-O3 -Wl,-mllvm,-fp-contract=fast -march=x86-64-v3 -Wl,--threads=8 -Wl,--gc-sections"
export MOZ_LTO_LDFLAGS="-Wl,-mllvm,-polly"
export RUSTFLAGS="-C target-cpu=x86-64-v3 -C target-feature=+avx2,+fma,+bmi2,+f16c -C codegen-units=1 -C opt-level=3"
export POLLY="-mllvm -polly -mllvm -polly-2nd-level-tiling -mllvm -polly-loopfusion-greedy -mllvm -polly-pattern-matching-based-opts -mllvm -polly-position=before-vectorizer -mllvm -polly-vectorizer=stripmine"
export VERBOSE=1

# Enable Mozilla-specific LTO flag (safe to enable globally)
export MOZ_LTO=1
ac_add_options MOZ_LTO=1

# Branding
ac_add_options --with-app-name=${pkgname}
# TODO: re-evaluate
ac_add_options --enable-update-channel=release

export MOZ_APP_REMOTINGNAME=${pkgname}

# System libraries
ac_add_options --with-system-nspr
ac_add_options --with-system-nss

# Features
# keep alsa option in here until merged upstream
ac_add_options --enable-alsa
ac_add_options --enable-jack
ac_add_options --enable-pulseaudio

# wasi
ac_add_options --with-wasi-sysroot=/usr/share/wasi-sysroot
# Use lld specifically for WASM while keeping mold for the main linking
export WASM_LD="/usr/bin/wasm-ld"

# options for controlled parallelism to avoid memory exhaustion
mk_add_options MOZ_MAKE_FLAGS="-j6"
# ac_add_options --enable-linker=gold

# optimizations
# Set -Copt-level=3
export OPT_LEVEL="3"
ac_add_options OPT_LEVEL="3"
export RUSTC_OPT_LEVEL="3"
ac_add_options RUSTC_OPT_LEVEL="3"

# Disable debug
ac_add_options --disable-debug
ac_add_options --disable-debug-symbols
ac_add_options --disable-debug-js-modules

# Strip debug symbols for smaller binaries and faster loading
ac_add_options --enable-strip
ac_add_options --enable-install-strip
export STRIP_FLAGS="--strip-debug --strip-unneeded"

# Enable automatic clobber to prevent build errors
mk_add_options AUTOCLOBBER=1
export AUTOCLOBBER=1.

# Enable additional optimizations
ac_add_options --enable-wasm-avx          # Enable AVX instructions for WebAssembly
ac_add_options --enable-optimize="-O3 -march=x86-64-v3"  # Target modern x86-64 CPUs with better instruction set
ac_add_options --enable-rust-simd         # Enable SIMD optimizations in Rust code
ac_add_options --disable-tests
ac_add_options --disable-dmd
END

if [[ "${CARCH}" == "aarch64" ]]; then
  cat >>../mozconfig <<END
# TODO: re-evaluate (is used by ALARM, but why?)
ac_add_options --enable-optimize="-g0 -O2"

ac_add_options --enable-lto=full
END

  export MOZ_DEBUG_FLAGS=" "
  export CFLAGS+=" -g0"
  export CXXFLAGS+=" -g0"
  export RUSTFLAGS="-Cdebuginfo=0"

else

  cat >>../mozconfig <<END
# Arch upstream has it in their PKGBUILD, ALARM does not for aarch64:
ac_add_options --disable-elf-hack

ac_add_options --enable-lto=full
END
fi

  # reduce chance of builds failing during linking due to running out of memory
  export LDFLAGS+=" -Wl,--no-keep-memory"

  # upstream Arch fixes
  #
}


build() {
  cd librewolf-$_firefoxver-$_librewolfver

  export MACH_BUILD_PYTHON_NATIVE_PACKAGE_SOURCE=pip
  export MOZBUILD_STATE_PATH="$srcdir/mozbuild"
  export MOZ_BUILD_DATE="$(date -u${SOURCE_DATE_EPOCH:+d @$SOURCE_DATE_EPOCH} +%Y%m%d%H%M%S)"
  export MOZ_NOSPAM=1

  # malloc_usable_size is used in various parts of the codebase
  CFLAGS="${CFLAGS/_FORTIFY_SOURCE=3/_FORTIFY_SOURCE=2}"
  CXXFLAGS="${CXXFLAGS/_FORTIFY_SOURCE=3/_FORTIFY_SOURCE=2}"

  # Breaks compilation since https://bugzilla.mozilla.org/show_bug.cgi?id=1896066
  CFLAGS="${CFLAGS/-fexceptions/}"
  CXXFLAGS="${CXXFLAGS/-fexceptions/}"

  # LTO/PGO needs more open files
  ulimit -n 4096

  # Do 3-tier PGO


  if [[ "${_build_profiled}" == "true" ]]; then
    if [[ "${CARCH}" == "aarch64" ]]; then

      cat >.mozconfig ../mozconfig - <<END
ac_add_options --enable-profile-generate
export MOZ_ENABLE_FULL_SYMBOLS=1
END

    else

      cat >.mozconfig ../mozconfig - <<END
ac_add_options --enable-profile-generate=cross
export MOZ_ENABLE_FULL_SYMBOLS=1
END

    fi

    # temporarily disable ublock-origin, interferes with profiling
    cp "lw/policies.json" "$srcdir/policies.json"
    jq 'del(.policies.Extensions.Install)' "$srcdir/policies.json" > "lw/policies.json"

    echo "Building instrumented browser..."

    ./mach build --priority normal

    echo "Profiling instrumented browser..."

    ./mach package

    local _headless_env=(
      LIBGL_ALWAYS_SOFTWARE=true \
      LLVM_PROFDATA=llvm-profdata \
        JARLOG_FILE="$PWD/jarlog" \
        dbus-run-session
    )

    if [[ "${_build_profiled_xvfb}" == "true" ]]; then
      local _headless_run=(
        xvfb-run
        -s "-screen 0 1920x1080x24 -nolisten local"
      )
    else
      local _headless_run=(
        wlheadless-run
        -c weston --width=1920 --height=1080
      )
    fi

    env "${_headless_env[@]}" "${_headless_run[@]}" -- ./mach python build/pgo/profileserver.py

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

    # reenable ublock-origin
    cp "$srcdir/policies.json" "lw/policies.json"

  else
    cat >.mozconfig ../mozconfig
  fi

  ./mach build --priority normal
}

package() {
  cd librewolf-$_firefoxver-$_librewolfver
  DESTDIR="$pkgdir" ./mach install

  local vendorjs="$pkgdir/usr/lib/$pkgname/browser/defaults/preferences/vendor.js"

  install -Dvm644 /dev/stdin "$vendorjs" <<END
// Use system-provided dictionaries
pref("spellchecker.dictionary_path", "/usr/share/hunspell");

// Don't disable extensions in the application directory
// done in librewolf.cfg
// pref("extensions.autoDisableScopes", 11);
END

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

  for i in 16 32 48 64 128; do
    install -Dvm644 browser/branding/${pkgname}/default$i.png \
      "$pkgdir/usr/share/icons/hicolor/${i}x${i}/apps/$pkgname.png"
  done
  # install -Dvm644 browser/branding/librewolf/content/about-logo.png \
    # "$pkgdir/usr/share/icons/hicolor/192x192/apps/$pkgname.png"
  install -Dvm644 ${srcdir}/default192x192.png \
    "$pkgdir/usr/share/icons/hicolor/192x192/apps/$pkgname.png"

  # arch upstream provides a separate svg for this. we don't have that, so let's re-use 16.png
  install -Dvm644 browser/branding/${pkgname}/default16.png \
    "$pkgdir/usr/share/icons/hicolor/symbolic/apps/$pkgname-symbolic.png"

  install -Dvm644 ../$pkgname.desktop \
    "$pkgdir/usr/share/applications/$pkgname.desktop"

  # Install a wrapper to avoid confusion about binary path
  install -Dvm755 /dev/stdin "$pkgdir/usr/bin/$pkgname" <<END
#!/bin/sh
exec /usr/lib/$pkgname/librewolf "\$@"
END

  # Replace duplicate binary with wrapper
  # https://bugzilla.mozilla.org/show_bug.cgi?id=658850
  ln -srfv "$pkgdir/usr/bin/$pkgname" "$pkgdir/usr/lib/$pkgname/librewolf-bin"
  # Use system certificates
  local nssckbi="$pkgdir/usr/lib/$pkgname/libnssckbi.so"
  if [[ -e $nssckbi ]]; then
    ln -srfv "$pkgdir/usr/lib/libnssckbi.so" "$nssckbi"
  fi
}
