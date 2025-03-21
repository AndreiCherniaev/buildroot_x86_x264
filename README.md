How to build Linux image with x264.

## Compare with qemu_x86_defconfig
Config [qemu_x86_x264_defconfig](my_external_tree/configs/qemu_x86_openh264_defconfig) is based on [qemu_x86_defconfig](https://github.com/buildroot/buildroot/blob/e82217622ea4778148de82a4b77972940b5e9a9e/configs/qemu_x86_defconfig).

## Clone
```
git clone --remote-submodules --recurse-submodules -j8 https://github.com/AndreiCherniaev/buildroot_x86_x264.git
cd buildroot_x86_x264
```
## Synthesize MJPEG video
input.yuvj422p video I plan use as input to test in x264's CLI. This step is optional, by default you will use already existing `input.y4m` file from this repo. This step is if you want to synthesize video again.
```
# sudo apt install ffmpeg
ffmpeg -f lavfi -i testsrc=size=384x288:rate=1:duration=3 -vcodec wrapped_avframe -pix_fmt yuv420p my_external_tree/board/my_company/my_board/fs-overlay/root/input.y4m
```

## Make image
In example I use qemu_x86_x264_defconfig
```
make clean -C buildroot
make BR2_EXTERNAL=$PWD/my_external_tree -C $PWD/buildroot qemu_x86_x264_defconfig
make -C buildroot
```
## Save non-default buildroot .config
To save non-default buildroot's buildroot/.config to $PWD/my_external_tree/configs/qemu_x86_x264_defconfig
```
make -C $PWD/buildroot savedefconfig BR2_DEFCONFIG=$PWD/my_external_tree/configs/qemu_x86_x264_defconfig
```
## Start in QEMU
This code is based on emulation [script1](https://github.com/buildroot/buildroot/blob/02540771bccf7b10c7daecce5f0e1e41a73c1e07/boot/grub2/readme.txt#L4) and [script2](https://github.com/buildroot/buildroot/blob/9e3d572ff532df945fbc282fed22d10098e5718b/board/pc/readme.txt), for qemu_x86_openh264_defconfig run the emulation with:
```
qemu-system-i386 -M pc -kernel buildroot/output/images/bzImage -drive file=buildroot/output/images/rootfs.ext2,if=virtio,format=raw -append "rootwait root=/dev/vda console=tty1 console=ttyS0" -serial stdio -net nic,model=virtio -net user
```
Note: Optionally add `-nographic` to see output not in extra screen but in console terminal. Or `-display curses` to pseudographic.

Test
```
x264 --crf 24 -o o.mkv input.yuvj422p
```

## Problems
In case of problems to fix it and then build again
```
make x264-dirclean -C buildroot
make x264-rebuild -C buildroot 2>&1 |tee x264.log
```
Then see log in `x264.log`
