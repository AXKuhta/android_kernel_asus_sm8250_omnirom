## Custom kernel for Zenfone 7 Omnirom 13

Here's how to build it in Docker on Github Codespaces or on your own machine if you're ok with downloading a lot of stuff:

```sh
#
# fresh start
#
docker run -it ubuntu

#
# dependencies
#
apt update
apt upgrade
apt install wget curl git nano make build-essential bison flex libssl-dev bc cpio kmod zip

#
# clang
#
cd ~
mkdir x
cd x
wget https://android.googlesource.com/platform/prebuilts/clang/host/linux-x86/+archive/fb69815b96ce8dd4821078dd36ac92dde80a23e1/clang-r383902c.tar.gz
tar -xf clang-r383902c.tar.gz

#
# gcc
#
cd ~
mkdir y
cd y
wget https://android.googlesource.com/platform/prebuilts/gcc/linux-x86/aarch64/aarch64-linux-android-4.9/+archive/f95a7e464dcbf49032954036703e0fe35730d429.tar.gz
tar -xf f95a7e464dcbf49032954036703e0fe35730d429.tar.gz

#
# repo
#
cd ~
git clone https://github.com/AXKuhta/android_kernel_asus_sm8250_omnirom/ --depth=1 --branch improvements
cd android_kernel_asus_sm8250_omnirom

#
# environment
#
export PATH="$HOME/x/bin:$PATH"
export ARCH=arm64
export SUBARCH=arm64
export CROSS_COMPILE=~/y/bin/aarch64-linux-android-
export CLANG_TRIPLE=aarch64-linux-gnu-
export DTC_EXT=~/android_kernel_asus_sm8250_omnirom/dtc-aosp

#
# load defconfig, then build
#
mkdir out
make CC=clang AR=llvm-ar NM=llvm-nm OBJCOPY=llvm-objcopy OBJDUMP=llvm-objdump STRIP=llvm-strip O=out -j16 vendor/zf7_defconfig
make CC=clang AR=llvm-ar NM=llvm-nm OBJCOPY=llvm-objcopy OBJDUMP=llvm-objdump STRIP=llvm-strip O=out -j16

#
# gzip image
#
gzip -9 -k -f out/arch/arm64/boot/Image

#
# clone AnyKernel3
#
git clone https://github.com/osm0sis/AnyKernel3.git
cd AnyKernel3

# devicecheck=0
# block=boot
# is_slot_device=auto
# leave only those commands:
# dumb_boot;
# write_boot;
nano anykernel.sh

cp ../out/arch/arm64/boot/Image.gz .

#
# package into a flashable zip
#
zip -r9 ZF7-Omni13-custom-kernel.zip * -x .git README.md *placeholder

#
# upload it
#
curl --upload-file ./ZF7-Omni13-custom-kernel.zip https://free.keep.sh

#
# [optional] generate a qr code then scan it on the phone
#
apt install qrencode
qrencode -t ansi ...link...
```
