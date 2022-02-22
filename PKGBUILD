# Maintainer: Florian Mounier <paradoxxx.zero at gmail dot com>

# This package rebuild only the i915 module with this patch applied: https://gitlab.freedesktop.org/drm/intel/-/issues/1627
# Must use --overwrite "*/i915.ko.zst" as it replaces the module, for a lack of a better solution

pkgname=linux-i915-module-patched
pkgbase=linux-i915-module-patched
pkgver=5.16.10
pkgrel=1
pkgdesc='Linux i915 module with this patch applied: https://gitlab.freedesktop.org/drm/intel/-/issues/1627 /!\\ Must use --overwrite "*/i915.ko.zst" as it replaces the module '
_vertag=v$pkgver-arch$pkgrel
url="https://github.com/archlinux/linux"
arch=(x86_64)
license=(GPL2)
source=(
  https://github.com/archlinux/linux/archive/refs/tags/$_vertag.tar.gz
  patch.patch
)
sha256sums=(
  1152b06923f0e406e4fd291df657160d5674da152ad59cf2bfc9f73bbb24c7c6
  c5e09bf109f6291727cdd4ba65f0d8160656026ec3a87f9549ebef7aeff98686)

export KBUILD_BUILD_HOST=archlinux
export KBUILD_BUILD_USER=$pkgbase
export KBUILD_BUILD_TIMESTAMP="$(date -Ru${SOURCE_DATE_EPOCH:+d @$SOURCE_DATE_EPOCH})"

prepare() {
  cd $srcdir/linux*

  echo "Setting version..."
  scripts/setlocalversion --save-scmversion
  echo "-$pkgrel" > localversion.10-pkgrel
  echo "" > localversion.20-pkgname

  local src
  for src in "${source[@]}"; do
    src="${src%%::*}"
    src="${src##*/}"
    [[ $src = *.patch ]] || continue
    echo "Applying patch $src..."
    patch -Np1 < "../$src"
  done

  echo "Setting config..."
  wget -O .config "https://raw.githubusercontent.com/archlinux/svntogit-packages/packages/linux/trunk/config"
  make olddefconfig

  make -s kernelrelease > version
  echo "Prepared $pkgbase version $(<version)"
}

build() {
  cd $srcdir/linux*
  make clean
  make -j8 scripts
  make -j8 prepare
  make -j8 modules_prepare
  make -j8 M=drivers/gpu/drm/i915
}

package() {
  cd $srcdir/linux*
  local kernver="$(<version)"
  local modulesdir="$pkgdir/usr/lib/modules/$kernver"

  echo "Installing i915 module... ${src} ${srcdir} ${_srcname} ${pkgdir} "
  zstd -z -19 "./drivers/gpu/drm/i915/i915.ko"
  install -D -m644 "./drivers/gpu/drm/i915/i915.ko.zst" "${modulesdir}/kernel/drivers/gpu/drm/i915/i915.ko.zst"
}
