# Maintainer: Benjamin Schneider <ben@bens.haus>

buildarch=8

pkgname=linux-a3700-lts
pkgver=6.12.15
pkgrel=1
pkgdesc='Kernel and modules for Marvell Armada A3700 SoC'
arch=(aarch64)
url='https://www.kernel.org/'
license=('GPL-2.0 WITH Linux-syscall-note')
makedepends=(
  bc
  tar
  xz
)
install=$pkgname.install
options=('!strip')
_srcname=linux-$pkgver
source=(
  https://cdn.kernel.org/pub/linux/kernel/v${pkgver%%.*}.x/${_srcname}.tar.{xz,sign}
  config
  cpufreq.patch
)
validpgpkeys=(
  'ABAF11C65A2970B130ABE3C479BE3E4300411886'  # Linus Torvalds
  '647F28654894E3BD457199BE38DBBDC86092693E'  # Greg Kroah-Hartman
)
# https://www.kernel.org/pub/linux/kernel/v6.x/sha256sums.asc
sha256sums=('5ff5bd84ea0e22c53437302db5d394d0a92d8b8b1a88ce20d1098298e9f7630a'
            'SKIP'
            'd3a2d5e63c4d2e3552e8e6f38445099de02146ae1247dcbb53ce63f5078c4429'
            'a1514b9bf05a2b25a2737971f034feb2ec650e8c9b102afac0f3c47080267e46')
prepare() {
  cd $_srcname

  echo "Setting version..."
  echo "-$pkgrel" > localversion.10-pkgrel
  echo "${pkgbase#linux}" > localversion.20-pkgname

  local src
  for src in "${source[@]}"; do
    src="${src%%::*}"
    src="${src##*/}"
    src="${src%.zst}"
    [[ $src = *.patch ]] || continue
    patch -Np1 < "../$src"
  done

  echo "Setting config..."
  cp ../config .config
  make olddefconfig
  diff -u ../config .config || :

  make -s kernelrelease > version
  echo "Prepared $pkgbase version $(<version)"
}

build() {
  cd ${_srcname}

  unset LDFLAGS
  make ${MAKEFLAGS} Image modules
}

package() {
  pkgdesc="$pkgdesc"
  depends=(
    coreutils
    kmod
  )
  optdepends=(
    'linux-firmware-marvell: wifi and bluetooth driver (mwifiex)'
    'wireless-regdb: to set the correct wireless channels of your country'
  )
  provides=(WIREGUARD-MODULE)

  cd $_srcname
  local kernver="$(<version)"
  local modulesdir="$pkgdir/usr/lib/modules/$kernver"

  echo "Installing boot image..."
  install -Dm644 arch/arm64/boot/Image -t "$modulesdir"

  echo "Installing modules..."
  make INSTALL_MOD_PATH="$pkgdir/usr" INSTALL_MOD_STRIP=1 \
    DEPMOD=/doesnt/exist modules_install  # Suppress depmod

  # remove build link
  rm "$modulesdir"/build
}
