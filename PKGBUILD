# Maintainer: Donald Carr <sirspudd@gmail.com>

# Set up the pi for Qt compilation. On Arch I just install chromium which pulls in all the deps
# Mount/copy this prepped rasp rootfs somewhere and set this path as the sysroot below
echo "Set your sysroot" && exit 1
_sysroot=/mnt/pi

_piver=pi
_mkspec="linux-r${_piver}-g++"
pkgname=qpi
pkgver=5.6.0
_pkgver=${pkgver}-beta
_pipkgname=qt-everywhere-opensource-src-${_pkgver}
pkgrel=1
pkgdesc="Qt for the ${_piver}, coz this shouldnt be obtuse"
arch=("x86_64" "i686")
url="http://www.qt.io"
license=("LGPL3")
makedepends=("git" "pkgconfig" "gcc" "arm-bcm2708-linux-gnueabi")
source=("git://github.com/sirspudd/mkspecs.git" "https://download.qt.io/development_releases/qt/5.6/${_pkgver}/single/${_pipkgname}.tar.gz")
sha256sums=("SKIP" "d69103ec34b3775edfa47581b14ee9a20789d4b0d7d26220fb92f2cd32eb06f9")
#sha256sums=("SKIP" "eb7c430f9f73d8f9d1a0d328e8a77549ffcf3b9915bee0c3dd6ae9ceffb86ef9")

build() {
  local _srcdir="${srcdir}/${_pipkgname}"
  local _bindir="${_srcdir}-build"

  # Qt tries to do the right thing and stores these, breaking cross compilation
  unset LDFLAGS
  unset CFLAGS
  unset CXXFLAGS

  # Get our mkspec
  cp -r "${srcdir}/mkspecs/${_mkspec}" ${_srcdir}/qtbase/mkspecs/devices

  mkdir -p ${_bindir}
  cd ${_bindir}

  # skipping because of errors: qtwayland
  # skipping on principle: qtscript
  # skipping because of the target in question: widgets qtwebengine qtwebchannel

  ${_srcdir}/configure \
    -release \
    -confirm-license \
    -opensource \
    -prefix /opt/qt-${_pkgver}-${_piver} \
    -opengl es2 \
    \
    -no-widgets \
    -make libs \
    \
    -skip qtscript \
    -skip qtwebengine \
    -skip qtwebchannel \
    -skip qtwayland \
    \
    -sysroot ${_sysroot} \
    -device ${_mkspec} \
    -device-option CROSS_COMPILE=/opt/arm-bcm2708hardfp-linux-gnueabi/bin/arm-bcm2708hardfp-linux-gnueabi-

  make
}

package() {
  local _srcdir="${srcdir}/${_pipkgname}"
  local _bindir="${_srcdir}-build"

  cd "${_bindir}"
  INSTALL_ROOT="$pkgdir" make install
}
