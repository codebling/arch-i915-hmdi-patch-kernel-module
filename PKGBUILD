# Maintainer: Florian Mounier <paradoxxx.zero at gmail dot com>

# This package rebuild only the i915 module with this patch applied: https://gitlab.freedesktop.org/drm/intel/-/issues/1627
# Must use --overwrite "*/i915.ko.zst" as it replaces the module, for a lack of a better solution

pkgname=i915-hmdi-patch-kernel-module
pkgver=5.16.10
pkgrel=1
pkgdesc='Linux i915 module with this patch applied: https://gitlab.freedesktop.org/drm/intel/-/issues/1627'
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

prepare() {
  cd $srcdir/linux*

  local i;for i in "${source[@]}";do
    case $i in
      *.patch)
        echo "Applying patch ${i}"
        patch -p1 -i "${srcdir}/${i}"
    esac
  done
}

build() {
  cd $srcdir/linux*
  make clean
  make scripts
  make prepare
  make modules_prepare
  make M=drivers/gpu/drm/i915
}

package() {
  cd $srcdir/linux*
  local kernelversion=$(make -s kernelrelease)
  local modulesdir="$pkgdir/usr/lib/modules/$kernelversion"
  zstd -z -T0 -19 "./drivers/gpu/drm/i915/i915.ko"
  install -D -m644 "./drivers/gpu/drm/i915/i915.ko.zst" "${modulesdir}/kernel/drivers/gpu/drm/i915/i915-hdmi-patch.ko.zst"
}
