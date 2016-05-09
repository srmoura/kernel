#!/usr/bin/env bash
# Maintainer: Ranieri Althoff <ranisalt+kernel@gmail.com>

#pkgbase=linux              # Build stock -ARCH kernel
pkgbase=linux-custom        # Build kernel with a different name
_srcname=linux-4.5
pkgver=4.5.3
pkgrel=1
arch=('i686' 'x86_64')
url="http://www.kernel.org/"
license=('GPL2')
makedepends=('xmlto' 'docbook-xsl' 'kmod' 'inetutils' 'bc')
options=('!strip')

_kver=${_srcname#linux-}

# ck patchset
_ckver=1
_ckpatch="patch-${_kver}-ck${_ckver}"

# graysky's gcc patch
_gccpatch="enable_additional_cpu_optimizations_for_gcc_v4.9+_kernel_v3.15+.patch"

# paolo's bfq i/o scheduler
_bfqver="v7r11"
_bfqpath="http://algo.ing.unimo.it/people/paolo/disk_sched/patches/${_kver}.0-${_bfqver}"

# Unwanted DRM drivers, split with vertical bar |
udrm='AST|BOCHS|CIRRUS_QEMU|GMA500|MGA|MGAG200|NOUVEAU|QXL|RADEON|R128|SAVAGE|TDFX|VIA|VMWGFX'

# Unwanted FB drivers, split with vertical bar |
ufb='HYPERV|OPENCORES|VIA|VOODOO1'

source=("https://www.kernel.org/pub/linux/kernel/v4.x/${_srcname}.tar.xz"
        "https://www.kernel.org/pub/linux/kernel/v4.x/${_srcname}.tar.sign"
        "https://www.kernel.org/pub/linux/kernel/v4.x/patch-${pkgver}.xz"
        "https://www.kernel.org/pub/linux/kernel/v4.x/patch-${pkgver}.sign"
        # graysky's gcc patch file
        "http://repo-ck.com/source/gcc_patch/${_gccpatch}.gz"
        # ck patchset file
        "http://ck.kolivas.org/patches/4.0/${_kver}/${_kver}-ck${_ckver}/${_ckpatch}.xz"
        # paolo's bfq i/o scheduler files
        "${_bfqpath}/0001-block-cgroups-kconfig-build-bits-for-BFQ-${_bfqver}-${_kver}.0.patch"
        "${_bfqpath}/0002-block-introduce-the-BFQ-${_bfqver}-I-O-sched-for-${_kver}.0.patch"
        "${_bfqpath}/0003-block-bfq-add-Early-Queue-Merge-EQM-to-BFQ-${_bfqver}-for.patch"
        # the main kernel config files
        'config' 'config.x86_64'
        # standard config files for mkinitcpio ramdisk
        'linux.preset'
        'change-default-console-loglevel.patch')
sha256sums=('a40defb401e01b37d6b8c8ad5c1bbab665be6ac6310cdeed59950c96b31a519c'
            'SKIP'
            '02a1a3fb190031130f6a304a1d3ebb72ce0b4e6dcc608c9e423780e4df10dec7'
            'SKIP'
            'cf0f984ebfbb8ca8ffee1a12fd791437064b9ebe0712d6f813fd5681d4840791'
            '582faf80f7ee1e6d9844c598893101d0cf941afa92fc2981e909f1382a36710a'
            '5d19ecb91320a64f0abb6c8e70205fef848ada967093faa94e4c0c39c340d0c8'
            '9c1e11772ff29d37dacc9246f63e24d5154eb61682ba2b7e175a9ccbdc7116e1'
            'e0c9474431b60ca9fc3da04e7610748219da143440f1d7f5152572c7c63b52e0'
            '2355efbab340d16c1b60a7805b987a78e57266809ba6c986ceef68ef7ce71db0'
            'cee1781f96e55a909757c4533cdacb57c3ffe6f6f01f709e8a5a837dc4a68bba'
            'dfcd3b919ef8bb3eff6d86b9df4b94f3db6f2386c25ad961edf52b6d1e2fb205'
            '1256b241cd477b265a3c2d64bdc19ffe3c9bbcee82ea3994c590c2c76e767d99')
validpgpkeys=(
              'ABAF11C65A2970B130ABE3C479BE3E4300411886' # Linus Torvalds
              '647F28654894E3BD457199BE38DBBDC86092693E' # Greg Kroah-Hartman
             )

#_kernelname=${pkgbase#linux}
_kernelname="-GLaDOS"

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
  sed -i -re "s/^(.EXTRAVERSION).*$/\1 = /" "${srcdir}/${_ckpatch}"
  msg "Patching source with ck${_ckver}"
  patch -Np1 -i "${srcdir}/${_ckpatch}"

  # Patch source with BFQ scheduler
  msg "Patching source with BFQ patches"
  for p in $(ls ${srcdir}/000{1,2,3}-block*.patch); do
    patch -Np1 -i "$p"
  done

  # Patch source to enable more gcc CPU optimizatons via the make nconfig
  msg "Patching source with gcc patch to enable more cpus types"
  patch -Np1 -i "${srcdir}/${_gccpatch}"

  make mrproper

  if [ "${CARCH}" = "x86_64" ]; then
    cat "${srcdir}/config.x86_64" > ./.config
  else
    cat "${srcdir}/config" > ./.config
  fi
  echo 'CONFIG_HZ_250_NODEFAULT=n' >> ./.config

  sed -r -i -e 's/CONFIG_ACCESSIBILITY=.*/# CONFIG_ACCESSIBILITY is not set/' \
    -i -e 's/CONFIG_AGP=.*/# CONFIG_AGP is not set/' \
    -i -e 's/CONFIG_BUG=.*/CONFIG_BUG=n/' \
    -i -e 's/CONFIG_CHROME_PLATFORMS=.*/# CONFIG_CHROME_PLATFORMS is not set/' \
    -i -e 's/CONFIG_DEBUG_FS=.*/CONFIG_DEBUG_FS=n/' \
    -i -e 's/CONFIG_DEBUG_KERNEL=.*/CONFIG_DEBUG_KERNEL=n/' \
    -i -e "s/CONFIG_DRM_(${udrw})=.*/# CONFIG_DRM_\1 is not set/g" \
    -i -e "s/CONFIG_FB_(${ufb})=.*/# CONFIG_FB_\1 is not set/g" \
    -i -e 's/CONFIG_FIREWIRE=.*/# CONFIG_FIREWIRE is not set/' \
    -i -e 's/CONFIG_HAMRADIO=.*/# CONFIG_HAMRADIO is not set/' \
    -i -e 's/CONFIG_HYPERVISOR_GUEST=.*/# CONFIG_HYPERVISOR_GUEST is not set/' \
    -i -e 's/CONFIG_IKCONFIG(_PROC)?=.*/CONFIG_IKCONFIG\1=y/' \
    -i -e 's/CONFIG_INFINIBAND=.*/# CONFIG_INFINIBAND is not set/' \
    -i -e 's/CONFIG_INPUT_TOUCHSCREEN=.*/# CONFIG_INPUT_TOUCHSCREEN is not set/' \
    -i -e 's/CONFIG_INPUT_MISC=.*/# CONFIG_INPUT_MISC is not set/' \
    -i -e 's/CONFIG_IRDA=.*/# CONFIG_IRDA is not set/' \
    -i -e 's/CONFIG_KALLSYMS=.*/CONFIG_KALLSYMS=n/' \
    -i -e 's/CONFIG_KPROBES=.*/# CONFIG_KPROBES is not set/' \
    -i -e 's/CONFIG_MACINTOSH_DRIVERS=.*/# CONFIG_MACINTOSH_DRIVERS is not set/' \
    -i -e 's/CONFIG_MD=.*/# CONFIG_MD is not set/' \
    -i -e 's/CONFIG_MEMSTICK=.*/# CONFIG_MEMSTICK is not set/' \
    -i -e 's/CONFIG_NFC=.*/# CONFIG_NFC is not set/' \
    -i -e 's/CONFIG_PARPORT=.*/# CONFIG_PARPORT is not set/' \
    -i -e 's/CONFIG_PARTITION_ADVANCED=.*/# CONFIG_PARTITION_ADVANCED is not set/' \
    -i -e 's/CONFIG_PROFILING=.*/# CONFIG_PROFILING is not set/' \
    -i -e 's/CONFIG_SFI=.*/CONFIG_SFI=n/' \
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
  #msg "Changing kernel compression to LZ4"
  #sed -i -e 's/CONFIG_KERNEL_GZIP=y/# CONFIG_KERNEL_GZIP is not set/' \
  #  -i -e 's/# CONFIG_KERNEL_LZ4 is not set/CONFIG_KERNEL_LZ4=y/' ./.config

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
  sed -i -e '/CONFIG_IOSCHED_CFQ=.*/i CONFIG_IOSCHED_BFQ=y' \
    -i -e '/CONFIG_CFQ_GROUP_IOSCHED=.*/i CONFIG_BFQ_GROUP_IOSCHED=y' \
    -i -e '/CONFIG_DEFAULT_IOSCHED/ s,cfq,bfq,' \
    -i -e 's/\(CONFIG_DEFAULT_CFQ\)=.*/# \1 is not set\nCONFIG_DEFAULT_BFQ=y/' ./.config

  # don't run depmod on 'make install'. We'll do this ourselves in packaging
  sed -i '2iexit 0' scripts/depmod.sh

  # get kernel version
  make prepare

  # load probed modules
  #sudo /usr/bin/modprobed-db recall
  #make localmodconfig

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

  # remove a files already in linux-docs package
  rm -f "${pkgdir}/usr/lib/modules/${_kernver}/build/Documentation/kbuild/Kconfig.recursion-issue-01"
  rm -f "${pkgdir}/usr/lib/modules/${_kernver}/build/Documentation/kbuild/Kconfig.recursion-issue-02"
  rm -f "${pkgdir}/usr/lib/modules/${_kernver}/build/Documentation/kbuild/Kconfig.select-break"
}

_package-docs() {
  pkgdesc="Kernel hackers manual - HTML documentation that comes with the ${pkgbase/linux/Linux} kernel"

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
