# Maintainer: Florian Mounier <paradoxxx.zero at gmail dot com>

# This package rebuild only the i915 module with this patch applied: https://gitlab.freedesktop.org/drm/intel/-/issues/1627
# Must use --overwrite "*/i915.ko.zst" as it replaces the module, for a lack of a better solution

pkgname=linux-i915-module-patched
pkgbase=linux-i915-module-patched
pkgver=5.16.10
pkgrel=1
pkgdesc='Linux i915 module with this patch applied: https://gitlab.freedesktop.org/drm/intel/-/issues/1627 /!\\ Must use --overwrite "*/i915.ko.zst" as it replaces the module '
_srctag=v${pkgver%.*}-${pkgver##*.}
url="https://github.com/archlinux/linux/commits/$_srctag"
arch=(x86_64)
license=(GPL2)
options=('!strip')
source=(
  patch.patch
)
validpgpkeys=(
  'ABAF11C65A2970B130ABE3C479BE3E4300411886'  # Linus Torvalds
  '647F28654894E3BD457199BE38DBBDC86092693E'  # Greg Kroah-Hartman
  'A2FF3A36AAA56654109064AB19802F8B0D70FC30'  # Jan Alexander Steffens (heftig)
  'C7E7849466FE2358343588377258734B41C31549'  # David Runge <dvzrv@archlinux.org>
)
sha256sums=(
            'c5e09bf109f6291727cdd4ba65f0d8160656026ec3a87f9549ebef7aeff98686')

export KBUILD_BUILD_HOST=archlinux
export KBUILD_BUILD_USER=$pkgbase
export KBUILD_BUILD_TIMESTAMP="$(date -Ru${SOURCE_DATE_EPOCH:+d @$SOURCE_DATE_EPOCH})"

prepare() {
  ver=v$pkgver-arch$pkgrel

  # Clone repo if it hasn't been done already
  # We manually clone repo instead of setting `source`,
  # because linux repo is gigantic and we want to clone only 
  # the tags we want (~200 MB vs 5GB)
  if [ ! -d linux ]; then
    git clone --depth 1 --branch $ver https://github.com/archlinux/linux.git
  fi

  cd $srcdir/linux

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
  cd $srcdir/linux
  make clean
  make -j8 scripts
  make -j8 prepare
  make -j8 modules_prepare
  make -j8 M=drivers/gpu/drm/i915
}

package() {
  cd $srcdir/linux
  local kernver="$(<version)"
  local modulesdir="$pkgdir/usr/lib/modules/$kernver"

  echo "Installing i915 module... ${src} ${srcdir} ${_srcname} ${pkgdir} "
  zstd -z -19 "./drivers/gpu/drm/i915/i915.ko"
  install -D -m644 "./drivers/gpu/drm/i915/i915.ko.zst" "${modulesdir}/kernel/drivers/gpu/drm/i915/i915.ko.zst"
}
