# Maintainer: Giancarlo Razzolini <grazzolini@archlinux.org>
# Maintainer: Frederik Schwan <freswa at archlinux dot org>
# Contributor: Bartłomiej Piotrowski <bpiotrowski@archlinux.org>
# Contributor: Allan McRae <allan@archlinux.org>

# toolchain build order: linux-api-headers->glibc->binutils->gcc->glibc->binutils->gcc
# NOTE: valgrind requires rebuilt with each major glibc version

pkgbase=glibc
pkgname=(glibc lib32-glibc)
pkgver=2.38
_commit=f6445dc94da185b3d1ee283f0ca0a34c4e1986cc
pkgrel=5
arch=(x86_64)
url='https://www.gnu.org/software/libc'
license=(GPL LGPL)
makedepends=(git gd lib32-gcc-libs python)
options=(staticlibs !lto)
source=(git+https://sourceware.org/git/glibc.git#commit=${_commit}
        locale.gen.txt
        locale-gen
        lib32-glibc.conf
        sdt.h sdt-config.h
        reenable_DT_HASH.patch
)
validpgpkeys=(7273542B39962DF7B299931416792B4EA25340F8 # Carlos O'Donell
              BC7C7372637EC10C57D7AA6579C43DFBF1CF2187) # Siddhesh Poyarekar
b2sums=('SKIP'
        'c859bf2dfd361754c9e3bbd89f10de31f8e81fd95dc67b77d10cb44e23834b096ba3caa65fbc1bd655a8696c6450dfd5a096c476b3abf5c7e125123f97ae1a72'
        '04fbb3b0b28705f41ccc6c15ed5532faf0105370f22133a2b49867e790df0491f5a1255220ff6ebab91a462f088d0cf299491b3eb8ea53534cb8638a213e46e3'
        '7c265e6d36a5c0dff127093580827d15519b6c7205c2e1300e82f0fb5b9dd00b6accb40c56581f18179c4fbbc95bd2bf1b900ace867a83accde0969f7b609f8a'
        'a6a5e2f2a627cc0d13d11a82458cfd0aa75ec1c5a3c7647e5d5a3bb1d4c0770887a3909bfda1236803d5bc9801bfd6251e13483e9adf797e4725332cd0d91a0e'
        '214e995e84b342fe7b2a7704ce011b7c7fc74c2971f98eeb3b4e677b99c860addc0a7d91b8dc0f0b8be7537782ee331999e02ba48f4ccc1c331b60f27d715678'
        '35e03ed912e1b0cd23783ab83ce919412885c141344905b8b67bbad4a86c48cf3e893806060e48d5737514ff80cea0b58b0e1f15707c32224579c416dcd810c0')

prepare() {
  mkdir -p glibc-build lib32-glibc-build

  [[ -d glibc-$pkgver ]] && ln -s glibc-$pkgver glibc
  cd glibc

  # Re-enable `--hash-style=both` for building shared objects due to issues with EPIC's EAC
  # which relies on DT_HASH to be present in these libs.
  # reconsider 2023-01
  patch -Np1 -i "${srcdir}"/reenable_DT_HASH.patch
}

build() {
  local _configure_flags=(
      --prefix=/usr
      --with-headers=/usr/include
      --with-bugurl=https://bugs.archlinux.org/
      --enable-bind-now
      --enable-cet
      --enable-fortify-source
      --enable-kernel=4.4
      --enable-multi-arch
      --enable-stack-protector=strong
      --enable-systemtap
      --disable-nscd
      --disable-profile
      --disable-werror
  )

  (
    cd glibc-build

    echo "slibdir=/usr/lib" >> configparms
    echo "rtlddir=/usr/lib" >> configparms
    echo "sbindir=/usr/bin" >> configparms
    echo "rootsbindir=/usr/bin" >> configparms

    # Credits @allanmcrae
    # https://github.com/allanmcrae/toolchain/blob/f18604d70c5933c31b51a320978711e4e6791cf1/glibc/PKGBUILD
    # remove fortify for building libraries
    # CFLAGS=${CFLAGS/-Wp,-D_FORTIFY_SOURCE=2/}

    "${srcdir}"/glibc/configure \
        --libdir=/usr/lib \
        --libexecdir=/usr/lib \
        "${_configure_flags[@]}"

    make -O

    # build info pages manually for reproducibility
    make info
  )

  (
    cd lib32-glibc-build
    export CC="gcc -m32 -mstackrealign"
    export CXX="g++ -m32 -mstackrealign"

    echo "slibdir=/usr/lib32" >> configparms
    echo "rtlddir=/usr/lib32" >> configparms
    echo "sbindir=/usr/bin" >> configparms
    echo "rootsbindir=/usr/bin" >> configparms

    "${srcdir}"/glibc/configure \
        --host=i686-pc-linux-gnu \
        --libdir=/usr/lib32 \
        --libexecdir=/usr/lib32 \
        "${_configure_flags[@]}"

    make -O
  )

  # pregenerate C.UTF-8 locale until it is built into glibc
  # https://sourceware.org/glibc/wiki/Proposals/C.UTF-8
  # https://bugs.archlinux.org/task/74864
  #
  # do this here instead of in package_glibc()
  # because localedef does not like fakeroot
  make -C "${srcdir}"/glibc/localedata objdir="${srcdir}"/glibc-build \
    DESTDIR="${srcdir}"/locales install-files-C.UTF-8/UTF-8
}

# Credits for _skip_test() and check() @allanmcrae
# https://github.com/allanmcrae/toolchain/blob/f18604d70c5933c31b51a320978711e4e6791cf1/glibc/PKGBUILD
_skip_test() {
  test=${1}
  file=${2}
  sed -i "/\b${test} /d" "${srcdir}"/glibc/${file}
}

check() (
  cd glibc-build

  # adjust/remove buildflags that cause false-positive testsuite failures
  sed -i '/FORTIFY/d' configparms                                     # failure to build testsuite
  sed -i 's/-Werror=format-security/-Wformat-security/' config.make   # failure to build testsuite
  sed -i '/CFLAGS/s/-fno-plt//' config.make                           # 16 failures
  sed -i '/CFLAGS/s/-fexceptions//' config.make                       # 1 failure
  LDFLAGS=${LDFLAGS/,-z,now/}                                         # 10 failures

  # The following tests fail due to restrictions in the Arch build system
  # The correct fix is to add the following to the systemd-nspawn call:
  # --system-call-filter="@clock @memlock @pkey"
  _skip_test test-errno-linux        sysdeps/unix/sysv/linux/Makefile
  _skip_test tst-mlock2              sysdeps/unix/sysv/linux/Makefile
  _skip_test tst-ntp_gettime         sysdeps/unix/sysv/linux/Makefile
  _skip_test tst-ntp_gettimex        sysdeps/unix/sysv/linux/Makefile
  _skip_test tst-pkey                sysdeps/unix/sysv/linux/Makefile
  _skip_test tst-process_mrelease    sysdeps/unix/sysv/linux/Makefile
  _skip_test tst-adjtime             time/Makefile

  make -O check
)

package_glibc() {
  pkgdesc='GNU C Library'
  depends=('linux-api-headers>=4.10' tzdata filesystem)
  optdepends=('gd: for memusagestat'
              'perl: for mtrace')
  install=glibc.install
  backup=(etc/gai.conf
          etc/locale.gen)

  make -C glibc-build DESTDIR="${pkgdir}" install
  rm -f "${pkgdir}"/etc/ld.so.cache

  # Shipped in tzdata
  rm -f "${pkgdir}"/usr/bin/{tzselect,zdump,zic}

  cd glibc

  install -dm755 "${pkgdir}"/usr/lib/{locale,systemd/system,tmpfiles.d}

  install -m644 posix/gai.conf "${pkgdir}"/etc/gai.conf

  install -m755 "${srcdir}"/locale-gen "${pkgdir}"/usr/bin

  # Create /etc/locale.gen
  install -m644 "${srcdir}"/locale.gen.txt "${pkgdir}"/etc/locale.gen
  sed -e '1,3d' -e 's|/| |g' -e 's|\\| |g' -e 's|^|#|g' \
    localedata/SUPPORTED >> "${pkgdir}"/etc/locale.gen

  # Add SUPPORTED file to pkg
  sed -e '1,3d' -e 's|/| |g' -e 's| \\||g' \
    localedata/SUPPORTED > "${pkgdir}"/usr/share/i18n/SUPPORTED

  # install C.UTF-8 so that it is always available
  install -dm755 "${pkgdir}"/usr/lib/locale
  cp -r "${srcdir}"/locales/usr/lib/locale/C.utf8 -t "${pkgdir}"/usr/lib/locale
  sed -i '/#C\.UTF-8 /d' "${pkgdir}"/etc/locale.gen

  # Provide tracing probes to libstdc++ for exceptions, possibly for other
  # libraries too. Useful for gdb's catch command.
  install -Dm644 "${srcdir}"/sdt.h "${pkgdir}"/usr/include/sys/sdt.h
  install -Dm644 "${srcdir}"/sdt-config.h "${pkgdir}"/usr/include/sys/sdt-config.h
}

package_lib32-glibc() {
  pkgdesc='GNU C Library (32-bit)'
  depends=("glibc=$pkgver")
  options+=('!emptydirs')

  cd lib32-glibc-build

  make DESTDIR="${pkgdir}" install
  rm -rf "${pkgdir}"/{etc,sbin,usr/{bin,sbin,share},var}

  # We need to keep 32 bit specific header files
  find "${pkgdir}"/usr/include -type f -not -name '*-32.h' -delete

  # Dynamic linker
  install -d "${pkgdir}"/usr/lib
  ln -s ../lib32/ld-linux.so.2 "${pkgdir}"/usr/lib/

  # Add lib32 paths to the default library search path
  install -Dm644 "${srcdir}"/lib32-glibc.conf "${pkgdir}"/etc/ld.so.conf.d/lib32-glibc.conf

  # Symlink /usr/lib32/locale to /usr/lib/locale
  ln -s ../lib/locale "${pkgdir}"/usr/lib32/locale
}
