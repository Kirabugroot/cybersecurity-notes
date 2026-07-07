# networking notes

## ports i always need to look up

| port | service | notes |
|------|---------|-------|
| 21 | FTP | try anonymous login, check for writable dirs |
| 22 | SSH | version matters, old versions have vulns |
| 23 | Telnet | cleartext, rare but still exists |
| 25 | SMTP | email, check for open relay |
| 53 | DNS | zone transfer worth trying |
| 80/443 | HTTP/HTTPS | obvious |
| 110 | POP3 | old email |
| 139/445 | SMB | check for eternal blue, null sessions |
| 389 | LDAP | AD related, try null bind |
| 1433 | MSSQL | |
| 3306 | MySQL | |
| 3389 | RDP | |
| 5432 | PostgreSQL | |
| 5900 | VNC | often no auth or weak passwords |
| 6379 | Redis | no auth by default in old versions |
| 8080/8443 | alt HTTP | dev servers, admin panels |
| 27017 | MongoDB | no auth by default sometimes |

---

## nmap

basic scan:
```bash
nmap -sV -sC -p- TARGET
```

- `-sV` service version detection
- `-sC` default scripts
- `-p-` all 65535 ports (slow but thorough)
- `-T4` faster timing (careful on unstable networks)
- `-oN output.txt` save output

faster initial:
```bash
nmap -p- --min-rate 5000 -Pn TARGET
```
then do targeted `-sV -sC` on open ports only

UDP scan (slow, but DNS, SNMP, DHCP live there):
```bash
sudo nmap -sU --top-ports 100 TARGET
```

specific scripts:
```bash
nmap --script smb-vuln* -p 445 TARGET
nmap --script http-enum TARGET
nmap --script dns-zone-transfer TARGET
```

---

## DNS

zone transfer (often blocked but worth trying):
```bash
dig axfr @NAMESERVER domain.com
```

subdomains from cert: `crt.sh` website, search `%.domain.com`

brute force:
```bash
gobuster dns -d domain.com -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt
```

reverse lookup:
```bash
dig -x IP
```

---

## SMB

null session (older windows):
```bash
smbclient -L //TARGET -N
```

list shares:
```bash
smbclient //TARGET/share -N
crackmapexec smb TARGET
```

check for eternal blue (MS17-010):
```bash
nmap --script smb-vuln-ms17-010 -p 445 TARGET
```

---

## tunneling / pivoting

when you're inside a network and need to reach stuff

SSH local port forward:
```bash
ssh -L 8080:INTERNAL_HOST:80 user@JUMP_HOST
# now localhost:8080 reaches INTERNAL_HOST:80
```

SSH dynamic (SOCKS proxy):
```bash
ssh -D 9050 user@JUMP_HOST
# then use proxychains with tools
```

`/etc/proxychains.conf` - add `socks5 127.0.0.1 9050`

then: `proxychains nmap -sT TARGET`

chisel for when you don't have SSH:
```bash
# attacker
./chisel server -p 8000 --reverse

# victim
./chisel client ATTACKER:8000 R:8080:127.0.0.1:80
```

---

## HTTP stuff

curl one liners:
```bash
# follow redirects, show headers
curl -IL https://target.com

# POST with data
curl -X POST -d 'user=admin&pass=admin' https://target.com/login

# with cookie
curl -b 'session=abc123' https://target.com/admin

# ignore SSL
curl -k https://target.com

# custom header
curl -H 'X-Forwarded-For: 127.0.0.1' https://target.com
```

---

## subnetting (quick reference)

/24 = 255.255.255.0 = 254 hosts
/25 = 126 hosts
/16 = 65534 hosts

CIDR calc: just use `ipcalc` honestly
```bash
ipcalc 192.168.1.0/24
```

---

## stuff i learned the hard way

- always check UDP, SNMP especially (community string "public" is default)
- internal IP ranges: 10.x.x.x, 172.16-31.x.x, 192.168.x.x
- don't forget IPv6 during scans (`-6` flag in nmap)
- Wireshark/tcpdump on internal network captures way more than you expect
- ARP scanning finds hosts that block ICMP: `nmap -PR -sn 192.168.1.0/24`
