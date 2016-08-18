# $Id: PKGBUILD 273608 2016-08-11 18:16:05Z tpowa $
# Maintainer: Tobias Powalowski <tpowa@archlinux.org>
# Maintainer: Thomas Baechler <thomas@archlinux.org>

#pkgbase=linux              # Build stock -ARCH kernel
pkgbase=linux-custom  # Build kernel with a different name
_srcname=linux-4.7
pkgver=4.7.1
_pkgbasever=4.7
pkgrel=1
arch=('i686' 'x86_64')
url="http://www.kernel.org/"
license=('GPL2')
makedepends=('xmlto' 'docbook-xsl' 'kmod' 'inetutils' 'bc' 'libelf')
options=('!strip')

_kver=${_srcname#linux-}

# ck patchset
_ckver=1
_ckpatch="patch-${_kver}-ck${_ckver}"

# paolo's bfq i/o scheduler
_bfqver="v8r2"
_bfqpath="http://algo.ing.unimo.it/people/paolo/disk_sched/patches/${_kver}.0-${_bfqver}"

# Unwanted DRM drivers
UDRM='amdgpu ast bochs cirrus_qemu gma500 mga mgag200 nouveau qxl radeon r128 savage tdfx udl via virtio_gpu vmwgfx'

# Unwanted FB drivers
UFB='hyperv opencores udl via virtual voodoo1'

# Unwanted filesystems
UFS='reiserfs jfs xfs gfs2 ocfs2 btrfs nilfs2'

# Unwanted ramdisk formats
URD='bzip2 lzma xz lzo'

source=("https://linux-libre.fsfla.org/pub/linux-libre/releases/${_pkgbasever}-gnu/linux-libre-${_pkgbasever}-gnu.tar.xz"
        "https://linux-libre.fsfla.org/pub/linux-libre/releases/${_pkgbasever}-gnu/linux-libre-${_pkgbasever}-gnu.tar.xz.sign"
        "https://linux-libre.fsfla.org/pub/linux-libre/releases/${pkgver}-gnu/patch-${_pkgbasever}-gnu-${pkgver}-gnu.xz"
        "https://linux-libre.fsfla.org/pub/linux-libre/releases/${pkgver}-gnu/patch-${_pkgbasever}-gnu-${pkgver}-gnu.xz.sign"
        # graysky's gcc patch file
        "git+https://github.com/graysky2/kernel_gcc_patch.git"
        # ck patchset file
        "http://ck.kolivas.org/patches/4.0/${_kver}/${_kver}-ck${_ckver}/${_ckpatch}.xz"
        # paolo's bfq i/o scheduler files
        "${_bfqpath}/0001-block-cgroups-kconfig-build-bits-for-BFQ-v7r11-${_pkgbasever}.0.patch"
        "${_bfqpath}/0002-block-introduce-the-BFQ-v7r11-I-O-sched-for-${_pkgbasever}.0.patch"
        "${_bfqpath}/0003-block-bfq-add-Early-Queue-Merge-EQM-to-BFQ-v7r11-for.patch"
        "${_bfqpath}/0004-block-bfq-turn-BFQ-v7r11-for-${_pkgbasever}.0-into-BFQ-${_bfqver}-for.patch"
        'zen-tune.patch'
        # the main kernel config files
        'config' 'config.x86_64'
        # standard config files for mkinitcpio ramdisk
        'linux.preset'
        'change-default-console-loglevel.patch')

sha256sums=('f483e595e0ad9a9d1b3afd20e4ecb0b798cf16eb31e79a7b99311eb9c061032a'
            'SKIP'
            'dd831fc3104057cf43f05391c40da84543a146a7ce748bb6e21132a06712a879'
            'SKIP'
            'SKIP'
            'e8d70729a7a58bac904d9a7a52ae4d46feec671afa307e6814895d74daf5ffbc'
            '1e16d406dc5b58d61198566281dbfea781fae78af0ed839ab3950255fa56aa78'
            '391b1cb6b423c427fc46fb491f85d544e4756795833c6fb2553ddad6dc658d93'
            '57d5a143de0424a5ac2b86e3f43fde57e31c101de0a029f9c40c1cf21a9a795a'
            '9620022c602f60e666ae0faa65ad33d52024219895ad1aef06701cce4d9492aa'
            'e951a1185337773b08bd433c82ee8e4a3a353945c7a033e5d7296558df90c3a5'
            'e9b7f502b5e27fc4154f0b98f3870f2988b58414df948a61a82502d84bc68a72'
            '0004f61a57696a41a1bca1a4a2e6abb03b6ae1b2e67ce73055ae062764660f2f'
            'bd24bded4327f58b0fb2619272c698504186fa0c1adbddf13038e7f6b897ce68'
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
  patch -p1 -i "${srcdir}/patch-${_pkgbasever}-gnu-${pkgver}-gnu"

  # add latest fixes from stable queue, if needed
  # http://git.kernel.org/?p=linux/kernel/git/stable/stable-queue.git

  # set DEFAULT_CONSOLE_LOGLEVEL to 4 (same value as the 'quiet' kernel param)
  # remove this when a Kconfig knob is made available by upstream
  # (relevant patch sent upstream: https://lkml.org/lkml/2011/7/26/227)
  patch -p1 -i "${srcdir}/change-default-console-loglevel.patch"

  # patch source with ck patchset with BFS
  # fix double name in EXTRAVERSION
  sed -i -re "s/^(.EXTRAVERSION).*$/\1 = /" "${srcdir}/${_ckpatch}"
  msg "Patching source with ck${_ckver}"
  patch -Np1 -i "${srcdir}/${_ckpatch}"

  # Patch source with BFQ scheduler
  msg "Patching source with BFQ patches"
  for p in "${srcdir}"/000{1,2,3,4}-block*.patch; do
    patch -Np1 -i "$p"
  done

  # Patch source to enable more gcc CPU optimizatons via the make nconfig
  msg "Patching source with gcc patch to enable more cpus types"
  patch -Np1 -i "${srcdir}/kernel_gcc_patch/enable_additional_cpu_optimizations_for_gcc_v4.9+_kernel_v3.15+.patch"

  # Patch source with Zen Tuning for improved responsiveness
  msg "Patching source with Zen Tuning"
  patch -Np1 -i "${srcdir}/zen-tune.patch"

  make mrproper

  if [ "${CARCH}" = "x86_64" ]; then
    cat "${srcdir}/config.x86_64" > ./.config
  else
    cat "${srcdir}/config" > ./.config
  fi

  msg "Disabling unused drivers, port support and protocols"
  scripts/config --disable agp \
                 --disable firewire \
                 --disable hamradio \
                 --disable hypervisor_guest \
                 --disable infiniband \
                 --disable irda \
                 --disable macintosh_drivers \
                 --disable md \
                 --disable memstick \
                 --disable nfc \
                 --disable parport \
                 --disable partition_advanced \
                 --disable sfi \
                 --disable snd_soc \
                 --disable staging \
                 --disable wimax

  msg "Disabling unused debugging features"
  scripts/config --disable debug_fs \
                 --disable debug_kernel \
                 --disable dynamic_debug \
                 --disable ftrace \
                 --disable kprobes \
                 --disable profiling \
                 --disable unused_symbols \
                 --disable watchdog

  msg "Disabling unsafe or unused module features"
  scripts/config --disable module_force_load \
                 --disable module_force_unload \
                 --disable modversions


  msg "Disabling accessibility support"
  scripts/config --disable accessibility

  msg "Enabling access to config via /proc"
  scripts/config --enable ikconfig --enable ikconfig_proc

  msg "Disabling touchscreen and miscellaneous input"
  scripts/config --disable input_touchscreen --disable input_misc

  msg "Disabling unwanted DRM drivers"
  for d in $UDRM; do
    scripts/config --disable "drm_${d}"
  done

  msg "Disabling unwanted framebuffer drivers"
  for f in $UFB; do
    scripts/config --disable "fb_${f}"
  done

  msg "Disabling unwanted file systems"
  for f in $UFS; do
    scripts/config --disable "${f}_fs"
  done

  msg "Disabling unused ramdisk formats"
  for r in $URD; do
    scripts/config --disable "rd_${r}"
  done

  if [ "${_kernelname}" != "" ]; then
    sed -i "s|CONFIG_LOCALVERSION=.*|CONFIG_LOCALVERSION=\"${_kernelname}\"|g" ./.config
    sed -i "s|CONFIG_LOCALVERSION_AUTO=.*|CONFIG_LOCALVERSION_AUTO=n|" ./.config
  fi

  # set extraversion to pkgrel
  sed -ri "s|^(EXTRAVERSION =).*|\1 -${pkgrel}|" Makefile

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

  msg "Enabling BFS CPU scheduler..."
  scripts/config --disable hz_250_nodefault \
                 --enable sched_bfs \
                 --enable smt_nice

  msg "Enabling BFQ and setting noop as default I/O scheduler..."
  scripts/config --enable bfq_group_iosched \
                 --enable iosched_bfq \
                 --disable iosched_cfq \
                 --enable default_noop

  msg "Enabling Zen tuning options..."
  scripts/config --enable zen_interactive

  msg "Setting reduced log buffer sizes"
  scripts/config --set-val log_buf_shift 18 --set-val nmi_log_buf_shift 12

  msg "Enabling processor-specific optimizations"
  scripts/config --set-val nr_cpus 4 \
                 --disable amd_iommu \
                 --disable calgary_iommu \
                 --disable gart_iommu \
                 --disable microcode_amd \
                 --disable x86_mce_amd

  # don't run depmod on 'make install'. We'll do this ourselves in packaging
  sed -i '2iexit 0' scripts/depmod.sh

  msg "Enabling compiler optimizations"
  scripts/config --disable cpu_freq_default_gov_ondemand \
                 --enable  cpu_freq_default_gov_schedutil \
                 --enable  kernel_lz4 \
                 --enable  mnative \
                 --enable  optimize_inlining \
                 --enable  trim_unused_ksyms

  # rewrite configuration
  yes "" | make config >/dev/null

  # get kernel version
  make prepare

  # load probed modules
  sudo /usr/bin/modprobed-db recall
  make localmodconfig

  msg "Disabling stubborn features"
  scripts/config --disable chrome_platforms \
                 --disable gcov \
                 --disable virtualization

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
    media net pcmcia scsi soc sound trace uapi video xen; do
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

  # add objtool for external module building and enabled VALIDATION_STACK option
  if [ -f tools/objtool/objtool ];  then
      mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/tools/objtool"
      cp -a tools/objtool/objtool ${pkgdir}/usr/lib/modules/${_kernver}/build/tools/objtool/
  fi

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

pkgname=("${pkgbase}" "${pkgbase}-headers")
for _p in ${pkgname[@]}; do
  eval "package_${_p}() {
    $(declare -f "_package${_p#${pkgbase}}")
    _package${_p#${pkgbase}}
  }"
done

# vim:set ts=8 sts=2 sw=2 et:
