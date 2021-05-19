This is a slight modification of the base glibc pkgbuild to decrease the minimum required kernel version to 3.13.  WSL1 [reports inconsistent kernel versions](https://github.com/microsoft/WSL/issues/3023#issuecomment-452339739), so even though `uname` reports the kernel version as 4.4.0, the vDSO reports the version as 3.13.11, which is below the 4.4 required by Arch's glibc.

In addition, since WSL1 does not support running 32-bit programs, lib32-glibc was removed to reduce build time

## Other helpful WSL1 alterations

* [Strip minimum kernel versions from Qt binaries with a pacman hook](https://github.com/microsoft/WSL/issues/3023#issuecomment-452245576)

* WSL1 does not support SYSV IPC for fakeroot, so [use TCP instead](https://aur.archlinux.org/packages/fakeroot-tcp/)
