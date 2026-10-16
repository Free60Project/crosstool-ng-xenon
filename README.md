# Building the Xenon toolchain with crosstool-NG

bare-metal `powerpc-xenon-elf` cross toolchain (alias `xenon-*`),

binutils 2.47, GCC 16.2.0 (C/C++), newlib 4.6.0.

## Contents

| File | Purpose |
|------|---------|
| `xenon.defconfig` | crosstool-NG config (PPC, 32-bit, big-endian, bare-metal, newlib, `--with-cpu=cell --enable-altivec`) |
| `patches/gcc/16.2.0/0001-xenon-disable-broken-altivec.patch` | disables ~20 AltiVec instructions |
| `patches/gcc/16.2.0/0002-xenon-libstdcxx-newlib-workarounds.patch` | forces off `getentropy`/`arc4random`/runpath autodetection in libstdc++ |


## 1. Host prerequisites

Debian/Ubuntu:
```bash
apt install build-essential gcc g++ gperf bison flex texinfo help2man make \
  libncurses5-dev python3-dev autoconf automake libtool libtool-bin gawk wget bzip2 \
  xz-utils unzip patch libstdc++6 rsync git meson ninja-build
```
Fedora:

```
dnf install gcc gcc-c++ gperf bison flex texinfo help2man make ncurses-devel python3-devel autoconf automake libtool gawk wget bzip2 xz unzip patch rsync git`
```

## 2. Get crosstool-NG (>= 1.29)

```bash
git clone https://github.com/crosstool-ng/crosstool-ng ~/crosstool-ng-src
cd ~/crosstool-ng-src
./bootstrap && ./configure --prefix=$HOME/ctng && make -j$(nproc) && make install
```

## 3. Build

```bash
export CT_NG=$HOME/ctng/bin/ct-ng
# Create build config (./.config) from defconfig
DEFCONFIG=xenon.defconfig "$CT_NG" defconfig
# The environment variable allows restarting from the last successful step
CT_DEBUG_CT_SAVE_STEPS=y \
  $CT_NG build
```

Resume after a failure with `cd work && ct-ng build`.

## 4. Verify

```bash
export PATH=/usr/local/xenon/bin:$PATH
powerpc-xenon-elf-gcc --version && xenon-gcc --version
echo 'int main(void){return 0;}' > t.c
powerpc-xenon-elf-gcc -mcpu=cell -m32 -c t.c && echo OK
```
Then libxenon itself builds against it after setting:

```
export DEVKITXENON=/usr/local/xenon
```
