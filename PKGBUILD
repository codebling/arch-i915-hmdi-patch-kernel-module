# Maintainer: Florian Mounier <paradoxxx.zero at gmail dot com>

# This package rebuild only the i915 module with this patch applied: https://gitlab.freedesktop.org/drm/intel/-/issues/1627
# Must use --overwrite "*/i915.ko.zst" as it replaces the module, for a lack of a better solution

pkgname=i915-hmdi-patch-kernel-module
_linver=5.16.10
_archver=arch1
_archrel=1
_archtag=${_linver}-${_archver}-${_archrel}
pkgver=${_archtag//-/_}
pkgrel=1
_gitvertag=v$_linver-$_archver
pkgdesc='Linux i915 module with this patch applied: https://gitlab.freedesktop.org/drm/intel/-/issues/1627'
url="https://github.com/archlinux/linux"
arch=(x86_64)
license=(GPL2)
source=(
  https://github.com/archlinux/linux/archive/refs/tags/$_gitvertag.tar.gz
  blacklist-i915.conf
  kernel-config
  patch.patch
)
sha256sums=(
  1152b06923f0e406e4fd291df657160d5674da152ad59cf2bfc9f73bbb24c7c6
  78d65bb0202d2594bb99256c81d8bac6212079dff168ed090f9f20ee4b2c71ea
  a7e88715c86f2ea77e80cb0535d827406676cb8227a9367dd98931f511b06f31
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

  # Adjust the kernel version to match Arch's (e.g. -arch1-1) so that modprobe does not complain
  sed -i -E 's/^EXTRAVERSION.*$/EXTRAVERSION = -'${_archver}-${_archrel}'/' Makefile

  cp ../kernel-config ./.config

  # Remove the debug info with the following command
  #sed -i -E 's/CONFIG_DEBUG_INFO *= *y/CONFIG_DEBUG_INFO=n/' ./.config

  make olddefconfig

  make clean
  make scripts
  make prepare
  make modules_prepare
  make M=drivers/gpu/drm/i915
}

package() {
  install -Dm644 blacklist-i915.conf "$pkgdir/etc/modprobe.d/blacklist-i915.conf"

  cd $srcdir/linux*
  local modulesdir="$pkgdir/usr/lib/modules/$_archtag"
  zstd -z -T0 -19 "./drivers/gpu/drm/i915/i915.ko"
  install -D -m644 "./drivers/gpu/drm/i915/i915.ko.zst" "${modulesdir}/kernel/drivers/gpu/drm/i915/i915-hdmi-patch.ko.zst"
}

post_install() {
  depmod
}
