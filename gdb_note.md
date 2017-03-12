##gdb usage

###UI interface
##### ctrl + x + a, window for source code
##### ctrl + x + 2, window for assembly or register dump
##### ctrl + x + 1, back to source code window
##### ctrl + l , refresh screen
##### ctrl + x + o, switch active window
##### ctrl + p, ctrl + n, up and down

###core dump
##### ulimit -a, check "core file size" is not 0
##### ulimit -c, show "core file size"
##### ulimit -c ulimited, enable core dump in this shell
##### gdb [exec file] [core file], reproduce erro


### Setup an arm chroot
##### sudo qemu-debootstrap --arch armhf vivid eabi-chroot

### remote gdb with arm chroot
##### chroot eabi-chroot
##### build helloc.c and compile with -g
##### mount -t proc proc /proc
##### mount -t sysfs sys /sys
##### apt-get install gdbserver
##### ~~gdbserver localhost:hello~~ --> qemu: Unsupported syscall: 26
##### qemu-arm-static -g 222 hello
##### [host] arm-none-eabi-gdb ./hello
##### [host] target remote localhost:222
