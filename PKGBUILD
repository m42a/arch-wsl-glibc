# Maintainer: Jan de Groot <jgc@archlinux.org>
# Maintainer: Andreas Radke <andyrtr@archlinux.org>

pkgname=glibc
pkgver=2.9
pkgrel=2
_glibcdate=20081119
install=glibc.install
backup=(etc/locale.gen)
pkgdesc="GNU C Library"
arch=(i686 x86_64)
license=('GPL' 'LGPL')
url="http://www.gnu.org/software/libc"
groups=('base')
depends=('sh' 'kernel-headers>=2.6.27.6' 'tzdata' 'texinfo')
makedepends=('gcc>=4.3.2-2')
replaces=('glibc-xen')
source=(ftp://ftp.archlinux.org/other/glibc/${pkgname}-${pkgver}_${_glibcdate}.tar.bz2
	ftp://ftp.archlinux.org/other/glibc/glibc-patches-${pkgver}-2.tar.gz
	nscd
	locale.gen.txt
	locale-gen)
md5sums=('1f7cc590a7a9bbef8b09fe89af69fb8c'
         '7679e2bcd981847efccb2bad9e57fee3'
         'b587ee3a70c9b3713099295609afde49'
         '07ac979b6ab5eeb778d55f041529d623'
         '476e9113489f93b348b21e144b6a8fcf')

build() {

  # for cvs checkout
  mkdir ${srcdir}/glibc-${_glibcdate}
  cd ${srcdir}/glibc-${_glibcdate}
  export _TAG=glibc-2_9-branch
  export 'CVSROOT=:pserver:anoncvs@sources.redhat.com:/cvs/glibc'
#  cvs -z9 co -r $_TAG libc || return 1
#  tar -cvjf ../../glibc-2.9_${_glibcdate}.tar.bz2 libc
#  return 1

  cd ${srcdir}/libc

  # patch from Debian
  patch -Np1 -i ${srcdir}/glibc-patches/glibc-2.5-localedef_segfault-1.patch || return 1 # still needed?

  # Upstream fixes. See sources.redhat.com bugzilla
  patch -Np1 -i ${srcdir}/glibc-patches/glibc-2.7-bz4781.patch || return 1

  # Gentoo fixes
  patch -Np1 -i ${srcdir}/glibc-patches/glibc-dont-build-timezone.patch || return 1

  # fixes taken from FC10 2.9-3 rpm, fixes FS#12215
  # see http://sources.redhat.com/bugzilla/show_bug.cgi?id=7060 
  # see https://bugzilla.redhat.com/show_bug.cgi?id=459756
  patch -Np1 -i ${srcdir}/glibc-patches/glibc-nss_dns-gethostbyname4-disable.patch || return 1
  patch -Np1 -i ${srcdir}/glibc-patches/glibc-fixes1.patch || return 1

  install -m755 -d ${pkgdir}/etc
  touch ${pkgdir}/etc/ld.so.conf

  mkdir glibc-build
  cd glibc-build

  if [ "${CARCH}" = "i686" ]; then
    # Hack to fix NPTL issues with Xen, only required on 32bit platforms
    export CFLAGS="${CFLAGS} -mno-tls-direct-seg-refs"
  fi

  echo "slibdir=/lib" >> configparms

  ../configure --prefix=/usr \
      --enable-add-ons=nptl,libidn --without-cvs \
      --enable-kernel=2.6.16 --disable-profile \
      --with-headers=/usr/include --libexecdir=/usr/lib \
      --enable-bind-now --with-tls --with-__thread \
      --libdir=/usr/lib --without-gd
    
  make || return 1
  make install_root=${pkgdir} install || return 1

  rm -f ${pkgdir}/etc/ld.so.cache ${pkgdir}/etc/ld.so.conf ${pkgdir}/etc/localtime

  install -m755 -d ${pkgdir}/etc/rc.d
  install -m755 -d ${pkgdir}/usr/sbin
  install -m755 -d ${pkgdir}/usr/lib/locale
  install -m644 ${srcdir}/libc/nscd/nscd.conf ${pkgdir}/etc/nscd.conf
  install -m755 ${srcdir}/nscd ${pkgdir}/etc/rc.d/nscd
  install -m755 ${srcdir}/locale-gen ${pkgdir}/usr/sbin

  sed -i -e 's/^\tserver-user/#\tserver-user/' ${pkgdir}/etc/nscd.conf || return 1

  # create /etc/locale.gen
  install -m644 ${srcdir}/locale.gen.txt ${pkgdir}/etc/locale.gen
  sed -i "s|/| |g" ${srcdir}/libc/localedata/SUPPORTED
  sed -i 's|\\| |g' ${srcdir}/libc/localedata/SUPPORTED
  sed -i "s|SUPPORTED-LOCALES=||" ${srcdir}/libc/localedata/SUPPORTED
  cat ${srcdir}/libc/localedata/SUPPORTED >> ${pkgdir}/etc/locale.gen
  sed -i "s|^|#|g" ${pkgdir}/etc/locale.gen

  if [ "${CARCH}" = "x86_64" ]; then
    # fix for the linker
    sed -i '/RTLDLIST/s%/ld-linux.so.2 /lib64%%' ${pkgdir}/usr/bin/ldd
    #Comply with multilib binaries, they look for the linker in /lib64
    mkdir ${pkgdir}/lib64
    cd ${pkgdir}/lib64
    ln -v -s ../lib/ld* .
  fi

  rm -f ${pkgdir}/usr/share/info/dir
}