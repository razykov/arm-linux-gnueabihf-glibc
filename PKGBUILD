# Contributor: Alexander 'hatred' Drozdov <adrozdoff@gmail.com>
# Contributor: toha257 <toha257@gmail.com>
# Contributor: Allan McRae <allan@archlinux.org>
# Contributor: Bartłomiej Piotrowski <bpiotrowski@archlinux.org>
# Contributor: Kevin Mihelich <kevin@archlinuxarm.org>
# Maintainer: Tavian Barnes <tavianator@tavianator.com>

_target="arm-linux-gnueabihf"
pkgname=${_target}-glibc
pkgver=2.28
pkgrel=5
pkgdesc="GNU C Library (${_target})"
arch=('any')
url="http://www.gnu.org/software/libc/"
license=(GPL LGPL)
depends=("${_target}-linux-api-headers>=4.17.11-1")
makedepends=("${_target}-gcc-stage2>=8.2.1+20181127" gperf)
provides=("${_target}-glibc-headers=${pkgver}" "${_target}-eglibc")
conflicts=("${_target}-glibc-headers" "${_target}-eglibc")
replaces=("${_target}-glibc-headers")
options=(!buildflags !strip staticlibs)
_commit=5a74abda201907cafbdabd1debf98890313ff71e
#source=(git+https://sourceware.org/git/glibc.git#commit=$_commit
source=(https://ftp.gnu.org/gnu/glibc/glibc-$pkgver.tar.xz{,.sig}
        glibc-${_commit}.patch)
validpgpkeys=(7273542B39962DF7B299931416792B4EA25340F8) # Carlos O'Donell
md5sums=('c81d2388896379997bc359d4f2084239'
         'SKIP'
         'b64d9921601d1e25cca2c802f15d6dcf')

prepare() {
  mkdir -p glibc-build

  [[ -d glibc-$pkgver ]] && ln -s glibc-$pkgver glibc
  cd glibc

  local i; for i in ${source[@]}; do
    case ${i%::*} in
      *.patch)
        msg2 "Applying ${i}"
        patch -p1 -i "$srcdir/${i}"
        ;;
    esac
  done
}

build() {
  local _configure_flags=(
      --prefix=/
      --with-headers=/usr/${_target}/include
      --enable-add-ons
      --enable-bind-now
      --enable-lock-elision
      --disable-multi-arch
      --enable-stack-protector=strong
      --enable-stackguard-randomization
      --disable-profile
      --disable-werror
  )

  cd glibc-build

  echo "slibdir=/lib" >> configparms
  echo "rtlddir=/lib" >> configparms
  echo "sbindir=/bin" >> configparms
  echo "rootsbindir=/bin" >> configparms

  # remove fortify for building libraries
  CPPFLAGS=${CPPFLAGS/-D_FORTIFY_SOURCE=2/}

  export BUILD_CC=gcc
  export CC=${_target}-gcc
  export CXX=${_target}-g++
  export AR=${_target}-ar
  export RANLIB=${_target}-ranlib

  "$srcdir/glibc-$pkgver/configure" \
      --libdir=/lib \
      --libexecdir=/lib \
      ${_configure_flags[@]} \
      --target=${_target} \
      --host=${_target} \
      --build=${CHOST}

  echo "build-programs=no" >> configparms
  make
}

package() {
  cd glibc-build

  make install_root="$pkgdir/usr/$_target" install

  mkdir -p "$pkgdir/usr/$_target/usr"
  ln -s ../{include,lib} "$pkgdir/usr/$_target/usr"

  # Remove unneeded for compilation files
  rm -rf "$pkgdir/usr/$_target/"{bin,sbin,etc,share,var}
}
