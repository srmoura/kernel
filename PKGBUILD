# Maintainer: Ranieri Althoff <ranisalt+kernel@gmail.com>

#pkgbase=linux               # Build stock -ARCH kernel
pkgbase=linux-custom       # Build kernel with a different name
_srcname=linux-4.1
pkgver=4.1.6
pkgrel=1
arch=('i686' 'x86_64')
url="http://www.kernel.org/"
license=('GPL2')
makedepends=('xmlto' 'docbook-xsl' 'kmod' 'inetutils' 'bc')
options=('!strip')
#_uksmvernel="0.1.2.4-beta"
#_uksmname="linux-v4.0"
_ckpatchversion=2
_ckpatchname="patch-4.1-ck${_ckpatchversion}"
_gcc_patch="enable_additional_cpu_optimizations_for_gcc_v4.9+_kernel_v3.15+.patch"
_bfqpath="http://algo.ing.unimo.it/people/paolo/disk_sched/patches/4.1.0-v7r8"

# Unwanted DRM drivers, split with vertical bar |
udrm='AST|BOCHS|CIRRUS_QEMU|GMA500|MGA|MGAG200|NOUVEAU|QXL|RADEON|R128|SAVAGE|TDFX|VIA|VMWGFX'

# Unwanted FB drivers, split with vertical bar |
ufb='HYPERV|OPENCORES|VIA|VOODOO1'

source=("https://www.kernel.org/pub/linux/kernel/v4.x/${_srcname}.tar.xz"
        "https://www.kernel.org/pub/linux/kernel/v4.x/${_srcname}.tar.sign"
        "https://www.kernel.org/pub/linux/kernel/v4.x/patch-${pkgver}.xz"
        "https://www.kernel.org/pub/linux/kernel/v4.x/patch-${pkgver}.sign"
        "http://repo-ck.com/source/gcc_patch/${_gcc_patch}.gz"
        #"http://kerneldedup.org/download/uksm/beta/uksm-${_uksmvernel}-for-${_uksmname}.patch"
        "http://ck.kolivas.org/patches/4.0/4.1/4.1-ck${_ckpatchversion}/${_ckpatchname}.bz2"
        "${_bfqpath}/0001-block-cgroups-kconfig-build-bits-for-BFQ-v7r8-4.1.patch"
        "${_bfqpath}/0002-block-introduce-the-BFQ-v7r8-I-O-sched-for-4.1.patch"
        "${_bfqpath}/0003-block-bfq-add-Early-Queue-Merge-EQM-to-BFQ-v7r8-for-4.1.0.patch"
        # the main kernel config files
        'config' 'config.x86_64'
        # standard config files for mkinitcpio ramdisk
        'linux.preset'
        'change-default-console-loglevel.patch')
sha256sums=('caf51f085aac1e1cea4d00dbbf3093ead07b551fc07b31b2a989c05f8ea72d9f'
            'SKIP'
            '64e4deb16a279e233b0c91463b131bd0f3de6aabdb49efded8314bcf5dbfe070'
            'SKIP'
            '819961379909c028e321f37e27a8b1b08f1f1e3dd58680e07b541921282da532'
            #'440a76585338fd57f93f381bf19260347ffe82fa1520f5cf57b6343807db634d'
            '87726411f583862e456156fe82ef51b188e5d92e7a4bd944e01a091cd7c46428'
            'ec0ca3c8051ea6d9a27a450998af8162464c224299deefc29044172940e96975'
            'c5c2c48638c2a8180948bd118ffcc33c8b7ff5f9f1e4b04c8e2cafeca2bde87b'
            '4f30f76adbdf49aec8d41ac27ad212734500c272f3cba594f134a7bc263820d9'
            'b5d6829dcb75d99fea401d9579e859a6ebb9bc09b2d6992dde171e8f05d5cbcf'
            'ee55d469a4c00b6fb4144549f2a9c5b84d9fe7948c7cbd2637dce72227392b4f'
            'c3f70dbd79420ab82d85d79c8a98f8e14d33a6155abcf52c4f4515518cdcace1'
            '1256b241cd477b265a3c2d64bdc19ffe3c9bbcee82ea3994c590c2c76e767d99')
validpgpkeys=(
              'ABAF11C65A2970B130ABE3C479BE3E4300411886' # Linus Torvalds
              '647F28654894E3BD457199BE38DBBDC86092693E' # Greg Kroah-Hartman
             )

_kernelname=${pkgbase#linux}

prepare() {
  cd "${srcdir}/${_srcname}"

  # add upstream patch
  patch -Np1 -i "${srcdir}/patch-${pkgver}"

  # add latest fixes from stable queue, if needed
  # http://git.kernel.org/?p=linux/kernel/git/stable/stable-queue.git

  # set DEFAULT_CONSOLE_LOGLEVEL to 4 (same value as the 'quiet' kernel param)
  # remove this when a Kconfig knob is made available by upstream
  # (relevant patch sent upstream: https://lkml.org/lkml/2011/7/26/227)
  patch -Np1 -i "${srcdir}/change-default-console-loglevel.patch"

  # Patch source with UKSM
  #msg "Patching with UKSM"
  #patch -Np1 -i "${srcdir}/uksm-${_uksmvernel}-for-${_uksmname}.patch"

  # patch source with ck patchset with BFS
  # fix double name in EXTRAVERSION
  sed -i -re "s/^(.EXTRAVERSION).*$/\1 = /" "${srcdir}/${_ckpatchname}"
  msg "Patching source with ck1 including BFS v0.462"
  patch -Np1 -i "${srcdir}/${_ckpatchname}"

  # Patch source with BFQ scheduler"
  msg "Patching source with BFQ patches"
  for p in $(ls ${srcdir}/000{1,2,3}-block*.patch); do
    patch -Np1 -i "$p"
  done

  # Patch source to enable more gcc CPU optimizatons via the make nconfig
  msg "Patching source with gcc patch to enable more cpus types"
  patch -Np1 -i "${srcdir}/${_gcc_patch}"

  make mrproper

  if [ "${CARCH}" = "x86_64" ]; then
    cat "${srcdir}/config.x86_64" > ./.config
  else
    cat "${srcdir}/config" > ./.config
  fi
  echo 'CONFIG_HZ_250_NODEFAULT=n' >> ./.config

  sed -r -i -e 's/CONFIG_ACCESSIBILITY=.*/# CONFIG_ACCESSIBILITY is not set/' \
    -i -e 's/CONFIG_AGP=.*/# CONFIG_AGP is not set/' \
    -i -e 's/CONFIG_CHROME_PLATFORMS=.*/# CONFIG_CHROME_PLATFORMS is not set/' \
    -i -e "s/CONFIG_DRM_(${udrw})=.*/# CONFIG_DRM_\1 is not set/g" \
    -i -e "s/CONFIG_FB_(${ufb})=.*/# CONFIG_FB_\1 is not set/g" \
    -i -e 's/CONFIG_FIREWIRE=.*/# CONFIG_FIREWIRE is not set/' \
    -i -e 's/CONFIG_HAMRADIO=.*/# CONFIG_HAMRADIO is not set/' \
    -i -e 's/CONFIG_HYPERVISOR_GUEST=.*/# CONFIG_HYPERVISOR_GUEST is not set/' \
    -i -e 's/CONFIG_INFINIBAND=.*/# CONFIG_INFINIBAND is not set/' \
    -i -e 's/CONFIG_INPUT_TOUCHSCREEN=.*/# CONFIG_INPUT_TOUCHSCREEN is not set/' \
    -i -e 's/CONFIG_INPUT_MISC=.*/# CONFIG_INPUT_MISC is not set/' \
    -i -e 's/CONFIG_IRDA=.*/# CONFIG_IRDA is not set/' \
    -i -e 's/CONFIG_MACINTOSH_DRIVERS=.*/# CONFIG_MACINTOSH_DRIVERS is not set/' \
    -i -e 's/CONFIG_MD=.*/# CONFIG_MD is not set/' \
    -i -e 's/CONFIG_MEMSTICK=.*/# CONFIG_MEMSTICK is not set/' \
    -i -e 's/CONFIG_NFC=.*/# CONFIG_NFC is not set/' \
    -i -e 's/CONFIG_PARPORT=.*/# CONFIG_PARPORT is not set/' \
    -i -e 's/CONFIG_PARTITION_ADVANCED=.*/# CONFIG_PARTITION_ADVANCED is not set/' \
    -i -e 's/CONFIG_SND_SOC=.*/# CONFIG_SND_SOC is not set/' \
    -i -e 's/CONFIG_STAGING=.*/# CONFIG_STAGING is not set/' \
    -i -e 's/CONFIG_WATCHDOG=.*/# CONFIG_WATCHDOG is not set/' \
    -i -e 's/CONFIG_WIMAX=.*/# CONFIG_WIMAX is not set/' ./.config

  msg "Enabling native optimizations..."
  sed -i -e 's/CONFIG_GENERIC_CPU=y/# CONFIG_GENERIC_CPU is not set\nCONFIG_MNATIVE=y/' ./.config

  if [ "${_kernelname}" != "" ]; then
    sed -i "s|CONFIG_LOCALVERSION=.*|CONFIG_LOCALVERSION=\"${_kernelname}\"|g" ./.config
    sed -i "s|CONFIG_LOCALVERSION_AUTO=.*|CONFIG_LOCALVERSION_AUTO=n|" ./.config
  fi

  # set extraversion to pkgrel
  sed -ri "s|^(EXTRAVERSION =).*|\1 -${pkgrel}|" Makefile

  # compress kernel with fazter LZ4 algorithm
  msg "Changing kernel compression to LZ4"
  sed -i -e 's/CONFIG_KERNEL_GZIP=y/# CONFIG_KERNEL_GZIP is not set/' \
    -i -e 's/# CONFIG_KERNEL_LZ4 is not set/CONFIG_KERNEL_LZ4=y/' ./.config

  #if [ "${CARCH}" = "x86_64" ]; then
  #  msg "Disabling NUMA from kernel config..."
  #  sed -i -e 's/CONFIG_NUMA=y/# CONFIG_NUMA is not set/' \
  #    -i -e '/CONFIG_AMD_NUMA=y/d' \
  #    -i -e '/CONFIG_X86_64_ACPI_NUMA=y/d' \
  #    -i -e '/CONFIG_NODES_SPAN_OTHER_NODES=y/d' \
  #    -i -e '/# CONFIG_NUMA_EMU is not set/d' \
  #    -i -e '/CONFIG_NODES_SHIFT=6/d' \
  #    -i -e '/CONFIG_NEED_MULTIPLE_NODES=y/d' \
  #    -i -e '/# CONFIG_MOVABLE_NODE is not set/d' \
  #    -i -e '/CONFIG_USE_PERCPU_NUMA_NODE_ID=y/d' \
  #    -i -e '/CONFIG_ACPI_NUMA=y/d' ./.config
  #fi

  #msg "Enabling Ultra-KSM for page merging..."
  #sed -i -e 's/\(CONFIG_KSM=y\)/\1\nCONFIG_UKSM=y\n# CONFIG_KSM_LEGACY is not set/' ./.config

  msg "Enabling BFS CPU scheduler..."
  echo CONFIG_SCHED_BFS=y >> ./.config
  echo CONFIG_SMT_NICE=y >> ./.config

  msg "Enabling BFQ and setting as default I/O scheduler..."
  sed -i -e 's/\(CONFIG_CFQ_GROUP_IOSCHED=.*\)/\1\nCONFIG_IOSCHED_BFQ=y\nCONFIG_CGROUP_BFQIO=y/' \
    -i -e '/CONFIG_DEFAULT_IOSCHED/ s,cfq,bfq,' \
    -i -e 's/CONFIG_DEFAULT_CFQ=y/# CONFIG_DEFAULT_CFQ is not set\nCONFIG_DEFAULT_BFQ=y/' ./.config

  # don't run depmod on 'make install'. We'll do this ourselves in packaging
  sed -i '2iexit 0' scripts/depmod.sh

  # get kernel version
  make prepare

  # load configuration
  # Configure the kernel. Replace the line below with one of your choice.
  #make menuconfig # CLI menu for configuration
  #make nconfig # new CLI menu for configuration
  #make xconfig # X-based configuration
  make oldconfig # using old config from previous kernel version
  # ... or manually edit .config

  # rewrite configuration
  yes "" | make config >/dev/null
}

build() {
  cd "${srcdir}/${_srcname}"

  make ${MAKEFLAGS} LOCALVERSION= bzImage modules
}

_package() {
  pkgdesc="The ${pkgbase/linux/Linux} kernel and modules"
  [ "${pkgbase}" = "linux" ] && groups=('base')
  depends=('coreutils' 'linux-firmware' 'kmod' 'mkinitcpio>=0.7')
  optdepends=('crda: to set the correct wireless channels of your country')
  provides=("kernel26${_kernelname}=${pkgver}")
  conflicts=("kernel26${_kernelname}")
  replaces=("kernel26${_kernelname}")
  backup=("etc/mkinitcpio.d/${pkgbase}.preset")
  install=linux.install

  cd "${srcdir}/${_srcname}"

  KARCH=x86

  # get kernel version
  _kernver="$(make LOCALVERSION= kernelrelease)"
  _basekernel=${_kernver%%-*}
  _basekernel=${_basekernel%.*}

  mkdir -p "${pkgdir}"/{lib/modules,lib/firmware,boot}
  make LOCALVERSION= INSTALL_MOD_PATH="${pkgdir}" modules_install
  cp arch/$KARCH/boot/bzImage "${pkgdir}/boot/vmlinuz-${pkgbase}"

  # set correct depmod command for install
  cp -f "${startdir}/${install}" "${startdir}/${install}.pkg"
  true && install=${install}.pkg
  sed \
    -e  "s/KERNEL_NAME=.*/KERNEL_NAME=${_kernelname}/" \
    -e  "s/KERNEL_VERSION=.*/KERNEL_VERSION=${_kernver}/" \
    -i "${startdir}/${install}"

  # install mkinitcpio preset file for kernel
  install -D -m644 "${srcdir}/linux.preset" "${pkgdir}/etc/mkinitcpio.d/${pkgbase}.preset"
  sed \
    -e "1s|'linux.*'|'${pkgbase}'|" \
    -e "s|ALL_kver=.*|ALL_kver=\"/boot/vmlinuz-${pkgbase}\"|" \
    -e "s|default_image=.*|default_image=\"/boot/initramfs-${pkgbase}.img\"|" \
    -e "s|fallback_image=.*|fallback_image=\"/boot/initramfs-${pkgbase}-fallback.img\"|" \
    -i "${pkgdir}/etc/mkinitcpio.d/${pkgbase}.preset"

  # remove build and source links
  rm -f "${pkgdir}"/lib/modules/${_kernver}/{source,build}
  # remove the firmware
  rm -rf "${pkgdir}/lib/firmware"
  # make room for external modules
  ln -s "../extramodules-${_basekernel}${_kernelname:--ARCH}" "${pkgdir}/lib/modules/${_kernver}/extramodules"
  # add real version for building modules and running depmod from post_install/upgrade
  mkdir -p "${pkgdir}/lib/modules/extramodules-${_basekernel}${_kernelname:--ARCH}"
  echo "${_kernver}" > "${pkgdir}/lib/modules/extramodules-${_basekernel}${_kernelname:--ARCH}/version"

  # Now we call depmod...
  depmod -b "${pkgdir}" -F System.map "${_kernver}"

  # move module tree /lib -> /usr/lib
  mkdir -p "${pkgdir}/usr"
  mv "${pkgdir}/lib" "${pkgdir}/usr/"

  # add vmlinux
  install -D -m644 vmlinux "${pkgdir}/usr/lib/modules/${_kernver}/build/vmlinux" 
}

_package-headers() {
  pkgdesc="Header files and scripts for building modules for ${pkgbase/linux/Linux} kernel"
  provides=("kernel26${_kernelname}-headers=${pkgver}")
  conflicts=("kernel26${_kernelname}-headers")
  replaces=("kernel26${_kernelname}-headers")

  install -dm755 "${pkgdir}/usr/lib/modules/${_kernver}"

  cd "${srcdir}/${_srcname}"
  install -D -m644 Makefile \
    "${pkgdir}/usr/lib/modules/${_kernver}/build/Makefile"
  install -D -m644 kernel/Makefile \
    "${pkgdir}/usr/lib/modules/${_kernver}/build/kernel/Makefile"
  install -D -m644 .config \
    "${pkgdir}/usr/lib/modules/${_kernver}/build/.config"

  mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/include"

  for i in acpi asm-generic config crypto drm generated keys linux math-emu \
    media net pcmcia scsi sound trace uapi video xen; do
    cp -a include/${i} "${pkgdir}/usr/lib/modules/${_kernver}/build/include/"
  done

  # copy arch includes for external modules
  mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/arch/x86"
  cp -a arch/x86/include "${pkgdir}/usr/lib/modules/${_kernver}/build/arch/x86/"

  # copy files necessary for later builds, like nvidia and vmware
  cp Module.symvers "${pkgdir}/usr/lib/modules/${_kernver}/build"
  cp -a scripts "${pkgdir}/usr/lib/modules/${_kernver}/build"

  # fix permissions on scripts dir
  chmod og-w -R "${pkgdir}/usr/lib/modules/${_kernver}/build/scripts"
  mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/.tmp_versions"

  mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/arch/${KARCH}/kernel"

  cp arch/${KARCH}/Makefile "${pkgdir}/usr/lib/modules/${_kernver}/build/arch/${KARCH}/"

  if [ "${CARCH}" = "i686" ]; then
    cp arch/${KARCH}/Makefile_32.cpu "${pkgdir}/usr/lib/modules/${_kernver}/build/arch/${KARCH}/"
  fi

  cp arch/${KARCH}/kernel/asm-offsets.s "${pkgdir}/usr/lib/modules/${_kernver}/build/arch/${KARCH}/kernel/"

  # add docbook makefile
  install -D -m644 Documentation/DocBook/Makefile \
    "${pkgdir}/usr/lib/modules/${_kernver}/build/Documentation/DocBook/Makefile"

  # add dm headers
  mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/md"
  cp drivers/md/*.h "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/md"

  # add inotify.h
  mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/include/linux"
  cp include/linux/inotify.h "${pkgdir}/usr/lib/modules/${_kernver}/build/include/linux/"

  # add wireless headers
  mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/net/mac80211/"
  cp net/mac80211/*.h "${pkgdir}/usr/lib/modules/${_kernver}/build/net/mac80211/"

  # add dvb headers for external modules
  # in reference to:
  # http://bugs.archlinux.org/task/9912
  mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/media/dvb-core"
  cp drivers/media/dvb-core/*.h "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/media/dvb-core/"
  # and...
  # http://bugs.archlinux.org/task/11194
  mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/include/config/dvb/"
  cp include/config/dvb/*.h "${pkgdir}/usr/lib/modules/${_kernver}/build/include/config/dvb/"

  # add dvb headers for http://mcentral.de/hg/~mrec/em28xx-new
  # in reference to:
  # http://bugs.archlinux.org/task/13146
  mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/media/dvb-frontends/"
  cp drivers/media/dvb-frontends/lgdt330x.h "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/media/dvb-frontends/"
  mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/media/i2c/"
  cp drivers/media/i2c/msp3400-driver.h "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/media/i2c/"

  # add dvb headers
  # in reference to:
  # http://bugs.archlinux.org/task/20402
  mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/media/usb/dvb-usb"
  cp drivers/media/usb/dvb-usb/*.h "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/media/usb/dvb-usb/"
  mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/media/dvb-frontends"
  cp drivers/media/dvb-frontends/*.h "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/media/dvb-frontends/"
  mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/media/tuners"
  cp drivers/media/tuners/*.h "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/media/tuners/"

  # add xfs and shmem for aufs building
  mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/fs/xfs"
  mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/mm"
  # removed in 3.17 series
  # cp fs/xfs/xfs_sb.h "${pkgdir}/usr/lib/modules/${_kernver}/build/fs/xfs/xfs_sb.h"

  # copy in Kconfig files
  for i in $(find . -name "Kconfig*"); do
    mkdir -p "${pkgdir}"/usr/lib/modules/${_kernver}/build/`echo ${i} | sed 's|/Kconfig.*||'`
    cp ${i} "${pkgdir}/usr/lib/modules/${_kernver}/build/${i}"
  done

  chown -R root.root "${pkgdir}/usr/lib/modules/${_kernver}/build"
  find "${pkgdir}/usr/lib/modules/${_kernver}/build" -type d -exec chmod 755 {} \;

  # strip scripts directory
  find "${pkgdir}/usr/lib/modules/${_kernver}/build/scripts" -type f -perm -u+w 2>/dev/null | while read binary ; do
    case "$(file -bi "${binary}")" in
      *application/x-sharedlib*) # Libraries (.so)
        /usr/bin/strip ${STRIP_SHARED} "${binary}";;
      *application/x-archive*) # Libraries (.a)
        /usr/bin/strip ${STRIP_STATIC} "${binary}";;
      *application/x-executable*) # Binaries
        /usr/bin/strip ${STRIP_BINARIES} "${binary}";;
    esac
  done

  # remove unneeded architectures
  rm -rf "${pkgdir}"/usr/lib/modules/${_kernver}/build/arch/{alpha,arc,arm,arm26,arm64,avr32,blackfin,c6x,cris,frv,h8300,hexagon,ia64,m32r,m68k,m68knommu,metag,mips,microblaze,mn10300,openrisc,parisc,powerpc,ppc,s390,score,sh,sh64,sparc,sparc64,tile,unicore32,um,v850,xtensa}
}

_package-docs() {
  pkgdesc="Kernel hackers manual - HTML documentation that comes with the ${pkgbase/linux/Linux} kernel"
  provides=("kernel26${_kernelname}-docs=${pkgver}")
  conflicts=("kernel26${_kernelname}-docs")
  replaces=("kernel26${_kernelname}-docs")

  cd "${srcdir}/${_srcname}"

  mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build"
  cp -al Documentation "${pkgdir}/usr/lib/modules/${_kernver}/build"
  find "${pkgdir}" -type f -exec chmod 444 {} \;
  find "${pkgdir}" -type d -exec chmod 755 {} \;

  # remove a file already in linux package
  rm -f "${pkgdir}/usr/lib/modules/${_kernver}/build/Documentation/DocBook/Makefile"
}

pkgname=("${pkgbase}" "${pkgbase}-headers" "${pkgbase}-docs")
for _p in ${pkgname[@]}; do
  eval "package_${_p}() {
    $(declare -f "_package${_p#${pkgbase}}")
    _package${_p#${pkgbase}}
  }"
done

# vim:set ts=8 sts=2 sw=2 et:
