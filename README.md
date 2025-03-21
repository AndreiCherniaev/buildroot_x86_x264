How to build Linux image with x264.

## Compare with qemu_x86_defconfig
Config [qemu_x86_x264_defconfig](my_external_tree/configs/qemu_x86_x264_defconfig) is based on [qemu_x86_defconfig](https://github.com/buildroot/buildroot/blob/e82217622ea4778148de82a4b77972940b5e9a9e/configs/qemu_x86_defconfig).

## Clone
```
git clone --remote-submodules --recurse-submodules -j8 https://github.com/AndreiCherniaev/buildroot_x86_x264.git
cd buildroot_x86_x264
```
## Synthesize MJPEG video
input.y4m video I plan use as input to test in x264's CLI. This step is optional, by default you will use already existing `input.y4m` file from this repo. This step is if you want to synthesize video again.
```
# sudo apt install ffmpeg
# mkdir -p my_external_tree/board/my_company/my_board/fs-overlay/root/
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
This code is based on emulation [script1](https://github.com/buildroot/buildroot/blob/02540771bccf7b10c7daecce5f0e1e41a73c1e07/boot/grub2/readme.txt#L4) and [script2](https://github.com/buildroot/buildroot/blob/9e3d572ff532df945fbc282fed22d10098e5718b/board/pc/readme.txt), run the emulation:
```
qemu-system-i386 -M pc -kernel buildroot/output/images/bzImage -drive file=buildroot/output/images/rootfs.ext2,if=virtio,format=raw -append "rootwait root=/dev/vda console=tty1 console=ttyS0" -serial stdio -net nic,model=virtio -net user
```
Note: Optionally add `-nographic` to see output not in extra screen but in console terminal. Or `-display curses` to pseudographic.

## The test
Login in Linux as `root` and type:
```
# x264 --crf 24 -o o.mkv input.y4m 
y4m [info]: 384x288p 1:1 @ 1/1 fps (cfr)
x264 [info]: using SAR=1/1
x264 [info]: using cpu capabilities: MMX2 SSE2 SSE3 Cache64
x264 [info]: profile High, level 2.1, 4:2:0, 8-bit
x264 [info]: frame I:1     Avg QP: 6.80  size:  4466                           
x264 [info]: frame P:1     Avg QP:21.09  size:  1247
x264 [info]: frame B:1     Avg QP: 9.16  size:  1092
x264 [info]: consecutive B-frames: 33.3% 66.7%  0.0%  0.0%
x264 [info]: mb I  I16..4: 75.0%  3.9% 21.1%
x264 [info]: mb P  I16..4:  4.2%  1.6%  7.6%  P16..4:  3.2%  3.9%  0.2%  0.0%  0.0%    skip:79.2%
x264 [info]: mb B  I16..4:  2.1%  0.7%  1.6%  B16..8:  6.2%  6.5%  0.9%  direct: 1.6%  skip:80.3%  L0:53.0% L1:45.3% BI: 1.7%
x264 [info]: 8x8 transform intra:5.3% inter:68.6%
x264 [info]: coded y,uvDC,uvAC intra: 14.4% 35.8% 27.3% inter: 1.9% 7.2% 6.4%
x264 [info]: i16 v,h,dc,p: 81%  7%  7%  6%
x264 [info]: i8 v,h,dc,ddl,ddr,vr,hd,vl,hu: 72%  6% 12%  2%  0%  0%  0%  0%  8%
x264 [info]: i4 v,h,dc,ddl,ddr,vr,hd,vl,hu: 61% 18% 15%  2%  1%  1%  0%  2%  0%
x264 [info]: i8c dc,h,v,p: 48% 17% 30%  4%
x264 [info]: Weighted P-Frames: Y:0.0% UV:0.0%
x264 [info]: kb/s:18.15

encoded 3 frames, 16.20 fps, 20.03 kb/s
```
What's the problem? Look at `using cpu capabilities: MMX2 SSE2 SSE3 Cache64`, but in buildroot we selected pentium pro, so even MMX2 should not be available. See also [x264 SIMD asm](https://forum.doom9.org/showthread.php?t=185002)

## How to rebuild x264
In case of problems to fix it and then build again
```
make x264-dirclean -C buildroot
make x264-rebuild -C buildroot 2>&1 |tee x264.log
```
Then see log in `x264.log`
