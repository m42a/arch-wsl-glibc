# Maintainer: Allan McRae <allan@archlinux.org>

# toolchain build order: linux-api-headers->glibc->binutils->gcc->binutils->glibc
# NOTE: valgrind requires rebuilt with each new glibc version

pkgname=glibc
pkgver=2.14
pkgrel=6
_glibcdate=20110908
pkgdesc="GNU C Library"
arch=('i686' 'x86_64')
url="http://www.gnu.org/software/libc"
license=('GPL' 'LGPL')
groups=('base')
depends=('linux-api-headers>=3.0' 'tzdata')
makedepends=('gcc>=4.6')
backup=(etc/gai.conf
        etc/locale.gen
        etc/nscd.conf)
options=('!strip')
install=glibc.install
source=(ftp://ftp.archlinux.org/other/glibc/${pkgname}-${pkgver}_${_glibcdate}.tar.xz
        glibc-2.10-dont-build-timezone.patch
        glibc-2.10-bz4781.patch
        glibc-__i686.patch
        glibc-2.12.1-static-shared-getpagesize.patch
        glibc-2.12.2-ignore-origin-of-privileged-program.patch
        glibc-2.13-futex.patch
        glibc-2.13-dlclose-search-reset.patch
        glibc-2.14-libdl-crash.patch
        glibc-2.14-avoid-assertion-on-empty-dns-answer.patch
        glibc-2.14-reexport-rpc-interface.patch
        glibc-2.14-reinstall-nis-rpc-headers.patch
        nscd
        locale.gen.txt
        locale-gen)
md5sums=('069069eb9100cc7affd7ad884cb3c3e9'
         '4dadb9203b69a3210d53514bb46f41c3'
         '0c5540efc51c0b93996c51b57a8540ae'
         '40cd342e21f71f5e49e32622b25acc52'
         'a3ac6f318d680347bb6e2805d42b73b2'
         'b042647ea7d6f22ad319e12e796bd13e'
         '7d0154b7e17ea218c9fa953599d24cc4'
         '22d09c58718fb3d1d31c3a6c14ca6886'
         'cea62cc6b903d222c5f26e05a3c0e0e6'
         '13728807283f111e5d9d38cf38e0a461'
         'c5de2a946215d647c8af5432ec4b0da0'
         '55febbb72139ac7b65757df085024b83'
         'b587ee3a70c9b3713099295609afde49'
         '07ac979b6ab5eeb778d55f041529d623'
         '476e9113489f93b348b21e144b6a8fcf')


mksource() {
  git clone git://sourceware.org/git/glibc.git
  pushd glibc
  git checkout -b glibc-2.14-arch origin/release/2.14/master
  popd
  tar -cvJf glibc-${pkgver}_${_glibcdate}.tar.xz glibc/*
}

build() {
  cd ${srcdir}/glibc

  # timezone data is in separate package (tzdata)
  patch -Np1 -i ${srcdir}/glibc-2.10-dont-build-timezone.patch

  # http://sources.redhat.com/bugzilla/show_bug.cgi?id=4781
  patch -Np1 -i ${srcdir}/glibc-2.10-bz4781.patch

  # http://sources.redhat.com/bugzilla/show_bug.cgi?id=411
  # http://sourceware.org/ml/libc-alpha/2009-07/msg00072.html
  patch -Np1 -i ${srcdir}/glibc-__i686.patch

  # http://sourceware.org/bugzilla/show_bug.cgi?id=11929
  # using Fedora "fix" as patch in that bug report causes breakages...
  patch -Np1 -i ${srcdir}/glibc-2.12.1-static-shared-getpagesize.patch

  # http://www.exploit-db.com/exploits/15274/
  # http://sourceware.org/git/?p=glibc.git;a=patch;h=d14e6b09 (only fedora branch...)
  patch -Np1 -i ${srcdir}/glibc-2.12.2-ignore-origin-of-privileged-program.patch

  # http://sourceware.org/bugzilla/show_bug.cgi?id=12403
  patch -Np1 -i ${srcdir}/glibc-2.13-futex.patch

  # https://bugzilla.redhat.com/show_bug.cgi?id=593675
  # http://sourceware.org/git/?p=glibc.git;a=commitdiff;h=2f811bf8 (only fedora branch...)
  patch -Np1 -i ${srcdir}/glibc-2.13-dlclose-search-reset.patch

  # http://sourceware.org/git/?p=glibc.git;a=commitdiff;h=675155e9 (only fedora branch...)
  # http://sourceware.org/ml/libc-alpha/2011-06/msg00006.html
  patch -Np1 -i ${srcdir}/glibc-2.14-libdl-crash.patch

  # http://sourceware.org/bugzilla/show_bug.cgi?id=13013
  # http://sourceware.org/git/?p=glibc.git;a=commitdiff;h=032c0ee3 (only fedora branch...)
  patch -Np1 -i ${srcdir}/glibc-2.14-avoid-assertion-on-empty-dns-answer.patch

  # re-export RPC interface until libtirpc is ready as a replacement
  # http://sourceware.org/git/?p=glibc.git;a=commitdiff;h=acee4873 (only fedora branch...)
  patch -Np1 -i ${srcdir}/glibc-2.14-reexport-rpc-interface.patch
  # http://sourceware.org/git/?p=glibc.git;a=commitdiff;h=bdd816a3 (only fedora branch...)
  patch -Np1 -i ${srcdir}/glibc-2.14-reinstall-nis-rpc-headers.patch

  install -dm755 ${pkgdir}/etc
  touch ${pkgdir}/etc/ld.so.conf

  cd ${srcdir}
  mkdir glibc-build
  cd glibc-build

  if [[ ${CARCH} = "i686" ]]; then
    # Hack to fix NPTL issues with Xen, only required on 32bit platforms
    export CFLAGS="${CFLAGS} -mno-tls-direct-seg-refs"
  fi

  echo "slibdir=/lib" >> configparms

  # remove hardening options from CFLAGS for building libraries
  CFLAGS=${CFLAGS/-fstack-protector/}
  CFLAGS=${CFLAGS/-D_FORTIFY_SOURCE=2/}

  ${srcdir}/glibc/configure --prefix=/usr \
      --libdir=/usr/lib --libexecdir=/usr/lib \
      --with-headers=/usr/include \
      --enable-add-ons=nptl,libidn \
      --enable-kernel=2.6.27 \
      --with-tls --with-__thread \
      --enable-bind-now --without-gd \
      --without-cvs --disable-profile \
      --disable-multi-arch

  # build libraries with hardening disabled
  echo "build-programs=no" >> configparms
  make
  
  # re-enable hardening for programs
  sed -i "s#=no#=yes#" configparms
  echo "CC += -fstack-protector -D_FORTIFY_SOURCE=2" >> configparms
  echo "CXX += -fstack-protector -D_FORTIFY_SOURCE=2" >> configparms
  make

  # remove harding in preparation to run test-suite
  sed -i '2,4d' configparms
}

check() {
  cd ${srcdir}/glibc-build

  # some errors are expected - manually check log files
  make -k check || true
}

package() {
  cd ${srcdir}/glibc-build
  make install_root=${pkgdir} install

  rm -f ${pkgdir}/etc/ld.so.{cache,conf}

  install -dm755 ${pkgdir}/etc/rc.d
  install -dm755 ${pkgdir}/usr/sbin
  install -dm755 ${pkgdir}/usr/lib/locale
  install -m644 ${srcdir}/glibc/nscd/nscd.conf ${pkgdir}/etc/nscd.conf
  install -m755 ${srcdir}/nscd ${pkgdir}/etc/rc.d/nscd
  install -m755 ${srcdir}/locale-gen ${pkgdir}/usr/sbin
  install -m644 ${srcdir}/glibc/posix/gai.conf ${pkgdir}/etc/gai.conf

  sed -i -e 's/^\tserver-user/#\tserver-user/' ${pkgdir}/etc/nscd.conf

  # create /etc/locale.gen
  install -m644 ${srcdir}/locale.gen.txt ${pkgdir}/etc/locale.gen
  sed -i "s|/| |g" ${srcdir}/glibc/localedata/SUPPORTED
  sed -i 's|\\| |g' ${srcdir}/glibc/localedata/SUPPORTED
  sed -i "s|SUPPORTED-LOCALES=||" ${srcdir}/glibc/localedata/SUPPORTED
  cat ${srcdir}/glibc/localedata/SUPPORTED >> ${pkgdir}/etc/locale.gen
  sed -i "s|^|#|g" ${pkgdir}/etc/locale.gen

  if [[ ${CARCH} = "x86_64" ]]; then
    # fix for the linker
    sed -i '/RTLDLIST/s%lib64%lib%' ${pkgdir}/usr/bin/ldd
    # Comply with multilib binaries, they look for the linker in /lib64
    mkdir ${pkgdir}/lib64
    cd ${pkgdir}/lib64
    ln -v -s ../lib/ld* .
  fi
  
  # manually strip files as stripping libpthread-*.so and libthread_db.so
  # with the default $STRIP_SHARED breaks gdb and stripping ld-*.so breaks
  # valgrind on x86_64

  cd $pkgdir
  strip $STRIP_BINARIES sbin/{ldconfig,sln} \
                        usr/bin/{gencat,getconf,getent,iconv,locale} \
                        usr/bin/{localedef,pcprofiledump,rpcgen,sprof} \
                        usr/lib/getconf/* \
                        usr/sbin/{iconvconfig,nscd}
  [[ $CARCH = "i686" ]] && strip $STRIP_BINARIES usr/bin/lddlibc4

  strip $STRIP_STATIC usr/lib/*.a \
                      lib/{{ld,libpthread}-${pkgver},libthread_db-1.0}.so

  strip $STRIP_SHARED lib/{libanl,libBrokenLocale,libc,libcidn,libcrypt}-${pkgver}.so \
                      lib/libnss_{compat,dns,files,hesiod,nis,nisplus}-${pkgver}.so \
                      lib/{libdl,libm,libnsl,libresolv,librt,libutil}-${pkgver}.so \
                      lib/{libmemusage,libpcprofile,libSegFault}.so \
                      usr/lib/{pt_chown,{audit,gconv}/*.so}
}