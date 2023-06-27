FROM ubuntu:latest AS kernel-build

ARG KERNEL_GIT=https://github.com/pohrom/linux-surface-rt
ARG KERNEL_BRANCH=ci
RUN \
    apt update &&\
    apt install git build-essential gcc-arm-linux-gnueabihf ccache libncurses-dev flex bison libssl-dev bc libelf-dev libudev-dev -y &&\
    git clone -b ${KERNEL_BRANCH} ${KERNEL_GIT} kernel --depth 1 &&\
    \
    cd /kernel &&\
    ARCH=arm make ARCH=arm tegra_defconfig &&\
    cat .github/workflows/tegra_surface_rt_defconfig >> .config &&\
    ARCH=arm yes '' | make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- -j $(nproc) &&\
    \
    mkdir /kmod &&\
    ARCH=arm make INSTALL_MOD_PATH=/kmod modules_install 

# prepare rootfs.img
FROM ubuntu:latest AS base
COPY --from=kernel-build kernel/arch/arm/boot/dts/tegra30-microsoft-surface-rt-efi.dts ./bootfs
COPY --from=kernel-build kernel/arch/arm/boot/zImage ./bootfs
COPY --from=kernel-build kmod/ ./kmod
COPY grub.cfg bootfs/
COPY mobile.cfg /
COPY pmbootstrap.cfg /
COPY shellprocess.conf /
RUN \
    apt update &&\
    apt install parted wget -y &&\
    wget http://host.docker.internal:8000/raspi_1_bullseye.img -O rootfs.img &&\
    wget https://github.com/makuhlmann/grub-2.04-nokeymod/releases/download/v1-nokeymod/grub.efi -O bootfs/grub.efi &&\
    wget https://github.com/makuhlmann/grub-2.04-nokeymod/releases/download/v1-nokeymod/unicode.pf2 -O bootfs/unicode.pf2
    wget https://www.debian.org/Pics/debian-logo-1024x576.png -O logo.png

# RUN --security=insecure \
#     dd if=/dev/zero bs=1MiB of=rootfs.img conv=notrunc oflag=append count=1000 &&\
#     LOOPDEV="$(losetup -P --show -f rootfs.img)" &&\
#     parted "${LOOPDEV}" resizepart 2 100% &&\
#     resize2fs -f "${LOOPDEV}p2"
# RUN \ 
#     mkdir -p /mnt/boot /mnt/rootfs &&\
#     mount "${LOOPDEV}p1" /mnt/boot &&\
#     mount "${LOOPDEV}p2" /mnt/rootfs &&\
#     cp grub.efi /mnt/boot/efi/boot/bootarm.efi &&\
#     cp unicode.pf2 /mnt/boot &&\
#     #cp .github/workflows/grub.cfg /mnt/boot
#     cp zImage /mnt/boot &&\
#     cp tegra30-microsoft-surface-rt-efi.dts /mnt/boot &&\
#     cp -r kmod/* /mnt/rootfs

# new image
#truncate -s 1000M rootfs.img
#parted --script /dev/loop0 mktable gpt mkpart primary 2048s 100%


# FROM ubuntu:latest AS base

# ARG IMG_URL=http://localhost:8000/raspi_1_bullseye.img
# ARG MIRROR_PMOS=https://mirrors.ustc.edu.cn/postmarketos/
# ARG MIRROR_ALPINE=http://mirrors.ustc.edu.cn/alpine/ 
# COPY ./workflows/* ./
# RUN \
#     apt update &&\
#     apt install wget -y &&\
#     wget ${IMG_URL} -O rootfs.img &&\
#     wget https://www.debian.org/Pics/debian-logo-1024x576.png -O logo.png



# git clone ${KERNEL_GIT} linux

# RUN \
#     apt update \
#     apt install python3-pip -y \
#     pip3 install pmbootstrap \

#     WHO_RESERVE="$(pip show -f pmbootstrap | awk '/^Location: / { print $2 }')/pmb/install/_install.py"
#     echo "$WHO_RESERVE"
#     sudo sed -i "s/size_reserve =/size_reserve = 1 #/g" "$WHO_RESERVE"

#     yes '' | pmbootstrap -y -c pmbootstrap.cfg -w ~/pmbootstrap --details-to-stdout init
#     yes '' | pmbootstrap -y -c pmbootstrap.cfg -w ~/pmbootstrap --details-to-stdout install \
#     --no-base --no-recommends --ondev --no-rootfs --cp \
#     logo.png:/usr/share/calamares/branding/default-mobile/logo.png \
#     rootfs.img:/var/lib/rootfs.img \
#     mobile.conf:/etc/calamares/modules/mobile.conf \
#     shellprocess.conf:/etc/calamares/modules/shellprocess.conf
