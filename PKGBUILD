# Maintainer: binarycraft007 <keep.it.sns@gmail.com>

pkgname=libcronet
pkgdesc="Networking stack of Chromium put into a library"
pkgver=135.0.7049.95
pkgrel=1
arch=('x86_64')
url='https://github.com/binarycraft007/libcronet'
license=('BSD-3-Clause')
depends=("gcc-libs" "glibc")
makedepends=("ninja" "gn" "ccache" "python")
checkdepends=("python" "openssl")

_DEBUG=0
_PGO_PATH='chrome-linux-7049-1743506729-8d2e173f8aa03b70a1de8aad9812aea978e9660f-85bda532d69f3240f1a2f324c1687e9ccee408e9.profdata'
_clang_path='clang-llvmorg-21-init-1655-g7b473dfe-1.tar.xz'

options=('!strip' '!lto')
source=(
  "${pkgname}-${pkgver}.tar.gz::https://github.com/binarycraft007/libcronet/archive/refs/tags/v${pkgver}.tar.gz"
  "${_PGO_PATH}::https://storage.googleapis.com/chromium-optimization-profiles/pgo_profiles/${_PGO_PATH}"
  "${_clang_path}::https://commondatastorage.googleapis.com/chromium-browser-clang/Linux_x64/${_clang_path}"
  "0001-cronet-Add-cert-net-fetcher.patch"
  "0002-cronet-Use-fixed-proxy-resolution-from-experimental-.patch"
  "0003-cronet-Support-setting-feature-list-from-experimenta.patch"
  "0004-net-grpc_support-Set-NetworkIsolationKey-from-header.patch"
  "0005-cronet-Support-setting-socket-limits-from-experiment.patch"
  "0006-cronet-fix-crash-we-have-no-command-line-arguments.patch"
)

noextract=(
  "${_clang_path}"
)

sha256sums=('d5f9d74f28369c40cce7acc220438e59e264f4bdb953dad314c82b111129a23e'
            'a9afb105df6125c42512c9a0f68776c6849e9b72c06f8f3bfc22d9b2031994a9'
            '5d94230fdb20386df002b32046139c05a1f0f9f98451b202abacdaf918fb3fe8'
            'f728850bbf680f154d6b482d2f62e0030998ae72e3437d4c25f01a7725402245'
	    'c343cc1e4dc226410310afaa83b0c56c1e1f92b4fb2322416a5ff21a4a9ffb60'
	    'f9357a0fd230751b86b65c9607ee2d599c2ab9c3bab1dccea0919b43338e6e64'
	    '507baa735fa2974a9d9c0dcd051ed8bb8a080ebe0407447c67a9f6519625c159'
	    '056be302a4fa1c837cbbb12a51def17f398d4ae5b5fb2062ae7643b64c5815cc'
	    'cbee443f6b4508cf894aac4f515c27bd6fd5c7330539da6c45948327091b1e3f')

provides=('libcronet')
conflicts=('libcronet-git' 'libcronet-bin')

prepare() {
  SRC_DIR="${srcdir}/${pkgname}-${pkgver}/src"

  mkdir -p "${SRC_DIR}/chrome/build/pgo_profiles"
  cp ${_PGO_PATH} "${SRC_DIR}/chrome/build/pgo_profiles/"

  mkdir -p "${SRC_DIR}/third_party/llvm-build/Release+Asserts"
  tar xJf ${_clang_path} -C "${SRC_DIR}/third_party/llvm-build/Release+Asserts/"

  cd ${srcdir}/${pkgname}-${pkgver}
  for patch in $(ls ../[0-9][0-9][0-9][0-9]-*.patch | sort); do
    echo "Applying patch: $(basename "$patch")"
    patch -p1 -i "$patch"
  done
}

build() {
  SRC_DIR="${srcdir}/${pkgname}-${pkgver}/src"

  cd "${SRC_DIR}"

  export TMPDIR="$PWD/tmp"
  rm -rf "$TMPDIR"
  mkdir -p "$TMPDIR"

  out=out/Release

  PYTHON=$(which python3 2>/dev/null)

  export CCACHE_SLOPPINESS=time_macros
  export CCACHE_BASEDIR="$PWD"
  export CCACHE_CPP2=yes
  CCACHE=ccache

  flags="
    cc_wrapper=\"$CCACHE\""

  if (( _DEBUG )); then
    flags="$flags"'
      is_debug=true
      is_component_build=true
      chrome_pgo_phase=0
    '
  else
    flags="$flags"'
      is_official_build=true
      exclude_unwind_tables=true
      enable_resource_allowlist_generation=false
      symbol_level=0
      chrome_pgo_phase=2
    '
  fi

  flags="$flags"'
    is_clang=true
    use_sysroot=false

    fatal_linker_warnings=false
    treat_warnings_as_errors=false

    is_cronet_build=true

    enable_base_tracing=false
    use_udev=false
    use_aura=false
    use_ozone=false
    use_gio=false
    use_platform_icu_alternatives=true
    use_glib=false

    disable_file_support=true
    enable_websockets=false
    use_kerberos=false
    disable_file_support=true
    disable_zstd_filter=false
    enable_mdns=false
    enable_reporting=false
    include_transport_security_state_preload_list=false
    enable_device_bound_sessions=false
    enable_bracketed_proxy_uris=true

    use_nss_certs=false
  
    enable_backup_ref_ptr_support=false
    enable_dangling_raw_ptr_checks=false

    disable_histogram_support=true
  '

  # Disable CFI icall for linux x64
  # See https://github.com/llvm/llvm-project/issues/86430
  flags="$flags"'
    use_cfi_icall=false'

  rm -rf "./$out"
  mkdir -p out

  export DEPOT_TOOLS_WIN_TOOLCHAIN=0

  gn gen "$out" "--args=$flags $EXTRA_FLAGS"

  ninja -C "$out" cronet_package
}

package() {
  SRC_DIR="${srcdir}/${pkgname}-${pkgver}/src"

  cd "${SRC_DIR}"

  install -Dm755 "out/Release/libcronet.$pkgver.so" \
    "$pkgdir/usr/lib/libcronet.$pkgver.so"
  ln -s "libcronet.$pkgver.so" "$pkgdir/usr/lib/libcronet.so"
  install -Dm644 out/Release/obj/components/cronet/libcronet_static.a \
    "$pkgdir/usr/lib/libcronet_static.a"

  install -d "$pkgdir/usr/include/cronet"
  install -m644 out/Release/cronet/include/*.h \
    "$pkgdir/usr/include/cronet/"
}
