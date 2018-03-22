# Maintainer: Donald Carr <sirspudd at gmail dot com>

# set -x

# Documentation

# Set up the pi for Qt compilation.
# For a comprehensive set of deps I just install chromium which pulls in everything

#* I had to removed xcomposite as code path breaks
#* Remove 2 (mesa) pkgconfig files we allow to screw our mkspec
# * rm /usr/lib/pkgconfig/glesv2.pc
# * rm /usr/lib/pkgconfig/egl.pc

# I use NFS to develop against my sysroot personally: sudo mount qpi2.local:/ /mnt/pi

# NB: Mandatory edit: set this variable to point to your raspberry pi's sysroot

# Arch: build dependencies for the target device documented in PKGBUILD.libs
# Fedora: systemd-devel mesa-*-devel wayland*-devel fontconfig-devel libinput-devel freetype-devel qt5-qtdeclarative-devel

pkgname="qt-sdk"

# Sanity check
_building=true

# Options
_target_host=false
_sysroot=""
_piver=""
_use_mesa=false
_float=false
_shadow_build=true
# automatically disabled if you are building webengine
_debug=true
_skip_qtscript=true
_skip_qtwebengine=false
_skip_qtwidgets=false
_static_build=false
_build_from_head=false
_patching=true
_minimal=true
_uberminimal=false
_testing=false

if [[ -z ${startdir} ]]; then
  _building=false;
fi

if [[ -f target-host ]]; then
  unset LOCAL_PI_VER
  _target_host=true
fi

if [[ -f static ]]; then
  _static_build=true
fi

if [[ -f testing ]]; then
  _testing=true
fi

if [[ -f full-build ]]; then
  _minimal=false
fi

# Sanity check options
if $_building && ! $_target_host; then
  if [[ -z $_piver ]] && [[ -n $LOCAL_PI_VER ]]; then _piver=$LOCAL_PI_VER; fi
  _sysroot=/mnt/pi${_piver}
  if [[ -z "${_piver}" ]]; then
    echo "You have to set a pi version (_piver) to build"
    exit 1
  fi

  if [[ ! -d "${_sysroot}/etc" ]]; then
    echo "You have to set a valid sysroot to proceed with the build"
    exit 1
  fi
fi

# vars
_local_qt5_repo="/opt/dev/src/qtproject/qt5"
if $_testing; then
  _pkgvermajmin="5.11"
  _pkgverpatch=".0"
  # {alpha/beta/beta2/rc}
  _dev_suffix="beta2"
  pkgrel=3
else
  _pkgvermajmin="5.10"
  _pkgverpatch=".0"
  # {alpha/beta/beta2/rc}
  _dev_suffix=""
  pkgrel=4
fi
pkgver="${_pkgvermajmin}${_pkgverpatch}"
$_build_from_head && pkgver=6.6.6
_pkgver=${pkgver}
_release_type="development_releases"
_profiled_gpu_fn=qpi-proprietary.sh

__pkgconfigpath=${_sysroot}/usr/lib/pkgconfig
__eglpkgconfigpath="${__pkgconfigpath}/egl.pc"
__glespkgconfigpath="${__pkgconfigpath}/glesv2.pc"

case ${_piver} in
1)
  _toolchain_name=armv6-rpi-linux-gnueabihf
  _toolchain="/opt/${_toolchain_name}/bin/${_toolchain_name}-"
  _mkspec="linux-rpi${_piver}-g++"
  # too problematic for me to care about
  #_float=true
;;
2)
  _toolchain_name=armv7-rpi2-linux-gnueabihf
  _toolchain="/opt/${_toolchain_name}/bin/${_toolchain_name}-"
  _mkspec="linux-rpi${_piver}-g++"
;;
3)
  _toolchain_name=aarch64-rpi3-linux-gnu
  _toolchain="/opt/x-tools/${_toolchain_name}/bin/${_toolchain_name}-"
  _use_mesa=true
  _mkspec="linux-rpi${_piver}-g++"
;;
4)
  # yuck; here lies tinkerboard until I find a better way of generalizing this
  _toolchain_name=armv7-rpi2-linux-gnueabihf
  _toolchain="/opt/gcc-linaro-7.1.1-2017.08-x86_64_arm-linux-gnueabihf/bin/arm-linux-gnueabihf-"
  _use_mesa=true
  _mkspec="linux-tinker-g++"
;;
esac

if $_building && $_uberminimal; then
  _skip_qtwidgets=true
fi

if $_building && $_minimal; then
  _skip_qtscript=true
  _skip_qtwebengine=true
fi

if $_target_host; then
  _use_mesa=true
else
  if ! $_testing; then
    depends=("qpi${_piver}-toolchain")
  fi
fi

if [[ -z "${_dev_suffix}" ]]; then _release_type="official_releases"; fi

$_build_from_head && _patching=false && _shadow_build=true

# PKGBUILD vars

install=qpi.install

rm $install
touch $install

if [[ -n ${_piver} ]] || ! $_building; then
  pkgname="${pkgname}-raspberry-pi${_piver}"
fi
provides=($pkgname)

if $_static_build; then
  pkgname="${pkgname}-static"
  _debug=false
  _skip_qtwidgets=true
  _skip_qtscript=true
  _skip_qtwebengine=true
fi

$_skip_qtwebengine && _additional_configure_flags="$_additional_configure_flags -skip qtwebengine -no-icu"
$_skip_qtscript && _additional_configure_flags="$_additional_configure_flags -skip qtscript"
$_skip_qtwidgets && _additional_configure_flags="$_additional_configure_flags -no-widgets"
$_static_build && _additional_configure_flags="$_additional_configure_flags -static"
$_float && _additional_configure_flags="$_additional_configure_flags -qreal float"
$_debug && _additional_configure_flags="$_additional_configure_flags -force-debug-info -separate-debug-info"

_libspkgname="${pkgname}-target-libs"
_libsdebugpkgname="${pkgname}-target-libs-debug"
_packaginguser=$(whoami)
_qt_package_name_prefix="qt-everywhere-src"
if [[ -n ${_dev_suffix} ]]; then
    _pkgver=${pkgver}-${_dev_suffix}
fi
_source_package_name=${_qt_package_name_prefix}-${_pkgver}
_baseprefix=/opt
_installprefix=${_baseprefix}/${pkgname}

pkgdesc="Qt SDK for the Raspberry Pi 1/2/3"
arch=("x86_64")
url="http://www.chaos-reins.com/qpi/"
license=("LGPL3" "GPL3")
optdepends=('qtcreator: Integrated Raspberry Pi IDE development')
makedepends=("git" "pkgconfig" "gcc" "gperf" "python")
#_provider=http://qt.mirror.constant.com/
_provider=https://download.qt.io
_tmpfs_dir=/vortex

_arch_specific_configure_options="\
    -prefix /usr \
    -docdir /usr/share/doc/qt \
    -headerdir /usr/include/qt \
    -archdatadir /usr/lib/qt \
    -datadir /usr/share/qt \
    -sysconfdir /etc/xdg \
    -examplesdir /usr/share/doc/qt/examples \
    -no-rpath \
"

if $_static_build; then
    if $_target_host; then
        _additional_configure_flags="$_additional_configure_flags -no-eglfs"
    else
        _additional_configure_flags="$_additional_configure_flags -no-xcb"
    fi
    _exhaustive_static_specific_configure_options="\
        -no-direct2d \
        -no-directfb \
        -no-gbm \
        -no-kms \
        -no-linuxfb \
        -no-mirclient \
        -no-cups \
        -no-iconv \
        -no-accessibility \
        -no-gif \
        -no-sql-mysql \
        -no-sql-psql \
        -no-tslib \
        -no-feature-bearermanagement \
        -no-qml-debug \
    "

    _additional_configure_flags="$_additional_configure_flags $_exhaustive_static_specific_configure_options"
    _additional_configure_flags="$_additional_configure_flags
        -ltcg \
        -no-ico \
        -no-glib \
        -no-fontconfig \
        -qt-freetype \
        -qt-harfbuzz \
    "
else
    _additional_configure_flags="$_additional_configure_flags \
        -hostprefix ${_installprefix} \
        -fontconfig \
        -system-sqlite \
        -system-freetype \
        -system-harfbuzz \
    "
fi

# Seems to be creating a large amount of breakage
_core_configure_options="\
                 -prefix ${_installprefix} \
                 -optimized-qmake \
                 -optimized-tools \
                 -optimize-size \
                 -confirm-license \
                 -opensource \
                 -v \
                 -silent \
                 -release \
                 -pch \
                 -journald \
                 -make libs \
                 -nomake tools \
                 \
                 -reduce-exports"

if $_testing; then
  _tar_xz_sha256="9482538af151454f79def3df1f4f76fc9475372b96cc9ca8515d7b2112a7d8cf"
else
  _tar_xz_sha256="936d4cf5d577298f4f9fdb220e85b008ae321554a5fcd38072dc327a7296230e"
fi

if ! $_build_from_head; then
  source=("git://github.com/sirspudd/mkspecs.git" "${_provider}/${_release_type}/qt/${_pkgvermajmin}/${_pkgver}/single/${_source_package_name}.tar.xz")
  sha256sums=("SKIP" "$_tar_xz_sha256")
fi

options=('!strip')

if ${_use_mesa}; then
  _profiled_gpu_fn=qpi-mesa.sh
  _additional_configure_flags="$_additional_configure_flags -gbm -kms"
else
  if $_building && ([[ -f ${__eglpkgconfigpath} ]] || [[ -f ${__glespkgconfigpath} ]]); then
    echo "Mesa is about to eat our communal poodle; delete egl.pc and glesv2.pc in your sysroot"
    exit 1
  fi
fi

# callbacks
finish() {
    if [[ -n ${startdir} ]]; then
      cd ${startdir}
      rm $install
      touch $install
    fi
}

adjust_bin_dir() {
  if [[ -n ${_srcdir} ]]; then
    _bindir="${_srcdir}"
  else
    # Probably repackaging: gonna have to make some assumptions
    _bindir="${startdir}/src/${_source_package_name}"
  fi
  if $_shadow_build; then
    _bindir="${_bindir}-build"
    if $_minimal && [[ -d $_tmpfs_dir ]]; then
      _bindir="${_tmpfs_dir}/${_bindir}"
    fi
  fi
}

adjust_src_dir() {
  if $_build_from_head; then
     if [[ -z $_local_qt5_repo ]]; then echo "Need to set a repo dir to build from head"; exit 1; fi
    _srcdir=$_local_qt5_repo
  fi
}

build() {
  # Qt tries to do the right thing and stores these, breaking cross compilation
  # unset everything set in /etc/makepkg.conf
  unset LDFLAGS
  unset CFLAGS
  unset CXXFLAGS
  unset CHOST
  unset CPPFLAGS
  unset CARCH
  unset HOSTTYPE
  unset MACHTYPE
  unset DEBUG_CFLAGS
  unset DEBUG_CXXFLAGS

  export PATH=${startdir}:${PATH}

  _srcdir="${srcdir}/${_source_package_name}"
  adjust_src_dir
  adjust_bin_dir

  local _basedir="${_srcdir}/qtbase"
  local _mkspec_dir="${_basedir}/mkspecs/devices/${_mkspec}"

  if $_static_build; then
    local _qtpro=${_srcdir}/qt.pro
    local _tmp_qtpro=$(mktemp)
    cp $_qtpro $_tmp_qtpro
    echo "QT_BUILD_MODULES = qtbase qtdeclarative" > $_qtpro
    cat $_tmp_qtpro >> $_qtpro
  fi

  cd ${_srcdir}

  # enable reduce relocations
  sed -i '/error Symbolic function binding/d' ${_srcdir}/qtbase/configure.json

if ! $_target_host; then
  # Get our mkspec
  rm -Rf $_mkspec_dir
  cp -r "${srcdir}/mkspecs/${_mkspec}" $_mkspec_dir
fi

  rm -Rf ${_bindir}
  mkdir -p ${_bindir}
  cd ${_bindir}

  # Too bleeding big
  # -developer-build \
  # -separate-debug-info \

  # Chromium requires python2 to be the system python on your build host
  # I literally symlink /usr/bin/python to /usr/bin/python2 on arch

  # Just because you can enable something doesnt mean you should
  # Prepare for breakage in all your Qt derived projects
  #-qtnamespace "Pi${_piver}" \

# -platform linux-clang \
if $_target_host; then
  local _configure_line="${_srcdir}/configure \
                 -opengl desktop \
                 ${_core_configure_options} \
                 ${_additional_configure_flags}"
# ${_arch_specific_configure_options} \
else
  local _configure_line="${_srcdir}/configure \
                 ${_core_configure_options} \
                 -qtlibinfix "Pi${_piver}" \
                 -sysroot ${_sysroot} \
                 -device ${_mkspec} \
                 -device-option CROSS_COMPILE=${_toolchain} \
                 -no-xcb \
                 -qpa eglfs \
                 -opengl es2 \
                 -egl \
                 ${_additional_configure_flags}"
fi
  local _configure_line_fn=configure_line
  echo ${_configure_line} > ${_configure_line_fn}
  set &> configure_env
  ${_configure_line} || ${_configure_line} || exit 1
  make || exit 1
}

create_install_script() {
  local _install_script_location="${startdir}/${install}"

  echo _piver="${_piver}" > ${_install_script_location}
  echo _qmakepath="${_installprefix}/bin/qmake" >> ${_install_script_location}
  echo _sysroot="${_sysroot}" >> ${_install_script_location}

  cat "${startdir}/_${install}" >> "${startdir}/${install}"
}

package() {
  adjust_bin_dir

  create_install_script

  # Qt is now installed to $pkgdir/$sysroot/$prefix
  # manually generate/decompose host/target

  local _basepkgdir=${pkgdir}/${_installprefix}

  local _libsdir="${startdir}/${_libspkgname}"
  local _libsdebugdir="${startdir}/${_libsdebugpkgname}"

  local _libspkgdir="${_libsdir}/topkg"
  local _libsdebugpkgdir="${_libsdebugdir}/topkg"

  local _libspkgbuild="${_libsdir}/PKGBUILD"
  local _libsdebugpkgbuild="${_libsdebugdir}/PKGBUILD"

  local _pkgprofiled=${_libspkgdir}/etc/profile.d
  local _pkgbindir=${_libspkgdir}/usr/bin
  local _profiledfn=qpi.sh
  local _qpienvexecfn=qpi-env-exec

  local _installed_dir="${pkgdir}/${_sysroot}/${_baseprefix}"
  local _installed_dir_sans_sysroot_offset="${pkgdir}/${_baseprefix}"

  rm -Rf ${_libspkgdir} ${_libsdebugpkgdir} ${pkgdir}
  mkdir -p ${_libspkgdir} ${_libsdebugpkgdir} ${pkgdir}

  cd "${_bindir}"
  INSTALL_ROOT="$pkgdir" make install || exit 1

  cd $(dirname ${_installed_dir})
  find $(basename ${_installed_dir}) -name '*.debug' -exec cp --parents '{}' ${_libsdebugpkgdir} \; -exec rm '{}' \;

  if $_static_build; then
    if ! $_target_host; then
        mv ${_installed_dir} ${_installed_dir_sans_sysroot_offset}
    fi
  else
    mv ${_installed_dir} ${_libspkgdir}
  fi

  cp ${_bindir}/configure_line ${_bindir}/config.summary ${_basepkgdir}
  cp ${_bindir}/configure_line ${_bindir}/config.summary ${_libspkgdir}

  cp ${startdir}/PKGBUILD.libs ${_libspkgbuild}
  cp ${startdir}/PKGBUILD.libs.debug ${_libsdebugpkgbuild}

  # set correct libs version
  sed -i "s/libspkgrel/${pkgrel}/" ${_libspkgbuild} || exit 1
  sed -i "s/libspkgver/${pkgver}/" ${_libspkgbuild} || exit 1
  sed -i "s/libspkgname/${_libspkgname}/" ${_libspkgbuild} || exit 1
  sed -i "s/libspiver/${_piver}/" ${_libspkgbuild} || exit 1

  # debug
  sed -i "s/libspkgrel/${pkgrel}/" ${_libsdebugpkgbuild} || exit 1
  sed -i "s/libspkgver/${pkgver}/" ${_libsdebugpkgbuild} || exit 1
  sed -i "s/libspkgname/${_libspkgname}/" ${_libsdebugpkgbuild} || exit 1
  sed -i "s/libsdebugpkgname/${_libsdebugpkgname}/" ${_libsdebugpkgbuild} || exit 1
  sed -i "s/libspiver/${_piver}/" ${_libsdebugpkgbuild} || exit 1

  if ! ${_target_host} && ! ${_static_build}; then
    mkdir -p ${_pkgprofiled}
    mkdir -p ${_pkgbindir}
    cp -L ${startdir}/${_profiledfn} ${_pkgprofiled} || exit 1
    cp -L ${startdir}/${_profiled_gpu_fn} ${_pkgprofiled} || exit 1
    cp -L ${startdir}/${_qpienvexecfn} ${_pkgbindir} || exit 1
    sed -i "s,localpiprefix,${_installprefix}," ${_pkgprofiled}/${_profiledfn} || exit 1
  fi

if ! $_static_build; then
  cd ${_libsdir}
  runuser -l ${_packaginguser} -c 'makepkg -d -f' || exit 1
  mv ${_libsdir}/${_libspkgname}-${pkgver}-${pkgrel}-any.pkg.tar.xz ${startdir}
fi

if $_debug; then
  cd ${_libsdebugdir}
  runuser -l ${_packaginguser} -c 'makepkg -d -f' || exit 1
  mv ${_libsdebugdir}/${_libsdebugpkgname}-${pkgver}-${pkgrel}-any.pkg.tar.xz ${startdir}
fi
}
