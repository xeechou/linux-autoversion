#This PKGBUILD file is design to build kernel from a existing src tree,
#or install precompiled kernel binary files directly.

#So we will have a clean src tree
pkgbase=linux-test
_kernel_bin=kernel_build

#the variable you have to provide
_builddir=kernel_build
kernel_src_dir='/home/developer/Courses/kernel-base'
_srcname=kernel_tree
#end the variable you have to provide

pkgver=3.8.1
pkgrel=1
pkgdesc="The Linux kernel and modules"
depends=('coreutils' 'linux-firmware' 'kmod' 'mkinitcpio>=0.7')
makedepends=('xmlto' 'docbook-xsl' 'kmod' 'inetutils' 'bc')
optdepends=('crda: to set the correct wireless channels of your country')
provides=("kernel26${_kernelname}=${pkgver}")
conflicts=("kernel26${_kernelname}")
replaces=("kernel26${_kernelname}")
arch=('i686' 'x86_64')
url="http://www.kernel.org/"
license=('GPL2')

source=(#if we provide this, means kernel compile progress is already done
	"${_kernel_bin}.tar.xz"
	'linux.preset'
	)
sha256sums=('65847bc847344434657db729d2dde4a408e303ea29ae1409520cecee8da6fc3d'
            '2c2e8428e2281babcaf542e246c2b63dea599abb7ae086fa482081580f108a98')

#this one strip the linux off
_kernelname=${pkgbase#linux}
prepare() {
  #XXX:checked
  #build dir has to be the same as kernel_bin files, then builddir is created
  #automatically by tar
  if [ "${kernel_src_dir}" == "" ];then
    return 1
  fi

  #provide kernel source tree for compile and move modules
  ln -s ${kernel_src_dir} ${srcdir}/${_srcname}

  mkdir -p "${srcdir}/${_srcname}"

  #we need to check here if there exist kernel bin files
  if [ "${_kernel_bin}" == "" ]; then 
    make O="${srcdir}/${_builddir}" menuconfig
  fi
}

build() {
  #XXX:checked
  cd "${srcdir}/${_srcname}"

  #we need to check here if there exist kernel bin files
  if [ "${_kernel_bin}" == "" ]; then 
    #return 1
    make O="${srcdir}/${_builddir}" bzImage modules
  fi
  #otherwise this step is done already done
}

_package() {
  #we dont need to worry about mkinitcpio, depmod thing, They are done by
  #install script, we need to provide a preset and install file instead.
  
  #we build kernel objs on _builddir, and install them in pkgdir
  #install binary files, this means we have a compiled binary tree
  cd "${srcdir}/${_srcname}"
  #echo "$(pwd)"

  KARCH=x86
  install=linux.install
  # get kernel version
  _kernver="$(make O="${srcdir}/${_builddir}" kernelrelease)"
  _kernver=$(echo "${_kernver}" | sed -n 2p -)
  #strip the -dirty away
  _kernver=${_kernver%-*}		
  _basekernel=${_kernver%%-*}
  _basekernel=${_basekernel%.*}

  mkdir -p "${pkgdir}"/{lib/modules,lib/firmware,boot}
  make O="${srcdir}/${_builddir}" INSTALL_MOD_PATH="${pkgdir}" modules_install
  cp "${srcdir}/${_builddir}"/arch/$KARCH/boot/bzImage "${pkgdir}/boot/vmlinuz-${pkgbase}"

  # set correct depmod command for install
  cp -f "${startdir}/${install}" "${startdir}/${install}.pkg"
  true && install=${install}.pkg
  sed -e "s/KERNEL_NAME=.*/KERNEL_NAME=${_kernelname}/" -i "${startdir}/${install}"
  sed "s/KERNEL_VERSION=.*/KERNEL_VERSION=${_kernver}/" -i "${startdir}/${install}"

  # install mkinitcpio preset file for kernel
  install -D -m644 "${srcdir}/linux.preset" "${pkgdir}/etc/mkinitcpio.d/${pkgbase}.preset"
  sed \
    -e "1s|'linux.*'|'${pkgbase}'|" \
    -e "s|ALL_kver=.*|ALL_kver=\"/boot/vmlinuz-${pkgbase}\"|" \
    -e "s|default_image=.*|default_image=\"/boot/initramfs-${pkgbase}.img\"|" \
    -i "${pkgdir}/etc/mkinitcpio.d/${pkgbase}.preset"

  # remove build and source links
  rm -f "${pkgdir}"/lib/modules/${_kernver}/{source,build}
  # remove the firmware
  rm -rf "${pkgdir}/lib/firmware"
  # gzip -9 all modules to save 100MB of space
  find "${pkgdir}" -name '*.ko' -exec gzip -9 {} \;
  # make room for external modules
  ln -s "../extramodules-${_basekernel}${_kernelname:--ARCH}" "${pkgdir}/lib/modules/${_kernver}/extramodules"
  # add real version for building modules and running depmod from post_install/upgrade
  mkdir -p "${pkgdir}/lib/modules/extramodules-${_basekernel}${_kernelname:--ARCH}"
  echo "${_kernver}" > "${pkgdir}/lib/modules/extramodules-${_basekernel}${_kernelname:--ARCH}/version"

  # Now we call depmod...
  #echo "Call Depmod"
  cp "${srcdir}/${_builddir}/System.map" System.map
  depmod -b "${pkgdir}" -F System.map "${_kernver}"
  #echo "Called Depmod"

  # move module tree /lib -> /usr/lib
  mkdir -p "${pkgdir}/usr"
  mv "${pkgdir}/lib" "${pkgdir}/usr/"

  # add vmlinux
  install -D -m644 "${srcdir}/${_builddir}/"vmlinux "${pkgdir}/usr/lib/modules/${_kernver}/build/vmlinux" 
}

pkgname=("${pkgbase}")
for _p in ${pkgname[@]}; do
  eval "package_${_p}() {
    _package${_p#${pkgbase}}
  }"
done
