# TZ VPN — Ubuntu 24.04 VPS-এ 3X-UI Full Installation Guide (বাংলা)

**শুরু থেকে শেষ পর্যন্ত: GitHub থেকে ZIP Download, Extract, VPS প্রস্তুতি, SSH Security, Firewall, 3X-UI Install, IP-Based Admin Panel, Subscription, VPN Inbound, DNS Fix, Troubleshooting, Backup এবং TZ VPN App।**

এই গাইডটি নতুন Ubuntu 24.04 LTS 64-bit VPS-এর জন্য। VPS Provider, 3X-UI Version এবং VPN Protocol অনুযায়ী কিছু মেনু বা সেটিংস আলাদা হতে পারে। প্রতিটি কমান্ড চালানোর আগে নির্দেশনা পড়ুন।

> **গুরুত্বপূর্ণ নিরাপত্তা:** Public HTTP Panel বা Subscription ব্যবহার করলে Username, Password, Client Token ও Subscription Data এনক্রিপ্টেড থাকে না। নিরাপত্তার জন্য HTTPS/TLS ব্যবহার করুন। Admin Panel-এর জন্য SSH Tunnel ব্যবহার করা ভালো। ZIP বা স্ক্রিপ্ট চালানোর আগে এর ফাইলগুলো যাচাই করুন।

## সূচিপত্র

1. [GitHub থেকে ZIP Download ও Extract](#১-github-থেকে-zip-download-ও-extract)
2. [VPS প্রস্তুতি](#২-vps-প্রস্তুতি)
3. [SSH দিয়ে Login](#৩-ssh-দিয়ে-login)
4. [Ubuntu Update ও Package](#৪-ubuntu-update-ও-প্রয়োজনীয়-package)
5. [SSH Port 22022](#৫-ssh-port-22022-configure)
6. [UFW ও Provider Firewall](#৬-ufw-firewall-ও-provider-firewall)
7. [Official 3X-UI Install](#৭-official-3x-ui-installation)
8. [Admin Panel](#৮-admin-panel-খোলা)
9. [Subscription Settings](#৯-subscription-settings-ip-only)
10. [VPN Inbound তৈরি](#১০-vpn-inbound-তৈরি)
11. [Client ও Subscription Test](#১১-client-তৈরি-ও-subscription-test)
12. [DNS Fix](#১২-dns-fix)
13. [Troubleshooting](#১৩-troubleshooting)
14. [Management Commands](#১৪-management-commands)
15. [Backup ও Security](#১৫-backup-ও-security)
16. [TZ VPN App](#১৬-tz-vpn-app)
17. [Final Checklist](#১৭-final-checklist)

---

## ১. GitHub থেকে ZIP Download ও Extract

এই Repository Public। সবাই ZIP Download করতে পারবে।

**ZIP Direct Download:**

https://github.com/tzalmamun/Eva/raw/refs/heads/main/tzvpn.zip

VPS-এ SSH Login করার পর নিচের কমান্ডগুলো চালান:

```bash
apt update
apt install -y curl unzip
mkdir -p /opt/tzvpn
cd /opt/tzvpn
curl -fL "https://github.com/tzalmamun/Eva/raw/refs/heads/main/tzvpn.zip" -o tzvpn.zip
unzip -o tzvpn.zip -d /opt/tzvpn
```

Extract হওয়া ফাইল দেখুন:

```bash
find /opt/tzvpn -maxdepth 3 -type f -print | head -200
```

ZIP-এর মধ্যে README বা INSTALL ফাইল থাকলে আগে পড়ুন:

```bash
find /opt/tzvpn -maxdepth 3 -type f \( -iname "README*" -o -iname "INSTALL*" \) -print
```

**গুরুত্বপূর্ণ:** ZIP Extract করলেই 3X-UI নিজে থেকে Install হয় না। Extract করা ফাইলের নির্দেশনা যাচাই করে তারপর পরবর্তী ধাপ অনুসরণ করুন। অচেনা Shell Script যাচাই না করে `bash` দিয়ে চালাবেন না।

---

## ২. VPS প্রস্তুতি

VPS Provider থেকে **Ubuntu 24.04 LTS 64-bit Server** তৈরি করুন। Provider Dashboard থেকে তথ্য সংগ্রহ করুন:

| বিষয় | উদাহরণ |
|---|---|
| Server IP | `YOUR_SERVER_IP` |
| SSH Username | `root` |
| Initial SSH Port | `22` (Provider অনুযায়ী ভিন্ন হতে পারে) |
| নতুন SSH Port | `22022` |
| Admin Panel Port | `2053` (উদাহরণ) |
| Subscription Port | `2096` (উদাহরণ) |
| VPN Inbound Port | আপনার Inbound অনুযায়ী |

`YOUR_SERVER_IP`-এর জায়গায় নিজের VPS-এর Public IP বসাবেন। `203.0.113.10`-এর মতো Documentation IP ব্যবহার করবেন না।

> OS Reinstall করলে পুরোনো সার্ভারের ডেটা মুছে যেতে পারে। প্রয়োজনীয় Backup আগে নিন।

## ৩. SSH দিয়ে Login

নিজের কম্পিউটারে Terminal / PowerShell খুলুন:

```bash
ssh root@YOUR_SERVER_IP
```

Provider-এর SSH Port 22 না হলে:

```bash
ssh -p INITIAL_SSH_PORT root@YOUR_SERVER_IP
```

Login যাচাই:

```bash
whoami
hostname
hostname -I
```

`whoami`-এর ফলাফল `root` হওয়া উচিত। অন্য User হলে প্রয়োজনীয় কমান্ডের আগে `sudo` ব্যবহার করুন।

## ৪. Ubuntu Update ও প্রয়োজনীয় Package

```bash
apt update
apt -y full-upgrade
apt -y install curl wget nano sudo ufw fail2ban ca-certificates sqlite3 unzip
timedatectl set-timezone UTC
timedatectl
```

Kernel Update-এর কারণে Reboot প্রয়োজন হলে:

```bash
reboot
```

Server চালু হলে আবার SSH দিয়ে Login করুন।

## ৫. SSH Port 22022 Configure

**সতর্কতা:** Provider Firewall-এ আগে `22022/TCP` Allow করুন। বর্তমান SSH Session বন্ধ করবেন না। নতুন Port দিয়ে Login পরীক্ষা না করে পুরোনো Port বন্ধ করবেন না।

প্রথমে UFW-তে নতুন ও পুরোনো SSH Port Allow করুন:

```bash
ufw allow 22022/tcp
ufw allow 22/tcp
```

SSH Configuration Backup:

```bash
cp -a /etc/ssh/sshd_config /etc/ssh/sshd_config.backup
mkdir -p /etc/ssh/sshd_config.d
```

Drop-in ফাইলে নতুন Port যোগ করুন:

```bash
cat > /etc/ssh/sshd_config.d/00-custom-port.conf <<'EOF'
Port 22022
EOF
```

Configuration পরীক্ষা:

```bash
sshd -t
sshd -T | grep '^port '
```

যদি Output-এ একাধিক Port দেখা যায়, SSH একাধিক Port-এ Listen করতে পারে। Ubuntu-তে মূল `sshd_config` বা অন্য Drop-in-এ আগে থেকে `Port 22` থাকতে পারে। কার্যকর Configuration যাচাই করে প্রয়োজনমতো পুরোনো Port Directive সরান; অন্ধভাবে ফাইল overwrite করবেন না।

কোনো Error না এলে:

```bash
systemctl reload ssh
```

বর্তমান Session চালু রেখেই নিজের কম্পিউটারে নতুন Terminal খুলে পরীক্ষা করুন:

```bash
ssh -p 22022 root@YOUR_SERVER_IP
```

নতুন Port দিয়ে Login সফল হওয়ার পর Provider Firewall থেকে পুরোনো SSH Port সরাতে পারেন। UFW-তে পুরোনো Rule থাকলে:

```bash
ufw delete allow 22/tcp
```

Provider-এর Initial SSH Port 22 না হলে সেই Port-ও যাচাই করুন। নতুন SSH Login নিশ্চিত না করে কখনো পুরোনো Port বন্ধ করবেন না।

## ৬. UFW Firewall ও Provider Firewall

SSH, Panel ও Subscription-এর উদাহরণ Port Allow করুন:

```bash
ufw allow 22022/tcp
ufw allow 2053/tcp
ufw allow 2096/tcp
ufw default deny incoming
ufw default allow outgoing
ufw --force enable
ufw status verbose
```

এখানে `2053` ও `2096` উদাহরণ। আপনার 3X-UI-তে Port আলাদা হলে প্রকৃত Port ব্যবহার করুন।

### VPN Inbound Port

Inbound তৈরি করার পর তার **সঠিক Port এবং Protocol** অনুযায়ী Rule যোগ করুন।

TCP 443-এর উদাহরণ:

```bash
ufw allow 443/tcp
```

UDP 443-এর উদাহরণ:

```bash
ufw allow 443/udp
```

WireGuard সত্যিই UDP 51820 ব্যবহার করলে:

```bash
ufw allow 51820/udp
```

`51820/UDP` সব VPN বা Xray Inbound-এর Port নয়। নিজের Inbound Configuration না দেখে এই Port খুলবেন না।

### Provider Firewall / Security Group

VPS Provider Dashboard-এ Firewall / Security Group / Network Rules খুলুন। UFW-তে যে Port ও Protocol খুলেছেন, Provider Firewall-এও প্রয়োজন অনুযায়ী Allow করুন:

- SSH: TCP `22022`
- Admin Panel: TCP `2053` (অথবা প্রকৃত Panel Port)
- Subscription: TCP `2096` (অথবা প্রকৃত Subscription Port)
- VPN: নিজের Inbound-এর প্রকৃত Port ও Protocol

শুধু UFW-তে Port Allow করলেই Provider Firewall-এর Block উঠে যায় না।

## ৭. Official 3X-UI Installation

Official Project: https://github.com/MHSanaei/3x-ui

Official Installer চালান:

```bash
bash <(curl -Ls https://raw.githubusercontent.com/mhsanaei/3x-ui/master/install.sh)
```

Installer-এর নির্দেশনা অনুসরণ করুন। Version অনুযায়ী Menu বা প্রশ্ন বদলাতে পারে।

- শক্তিশালী Admin Username ও Password দিন।
- Random এবং দীর্ঘ Web Base Path রাখুন।
- Panel Port চাইলে `2053` দিতে পারেন, যদি Portটি ব্যবহারযোগ্য থাকে।
- Installer যদি Random Port তৈরি করে, সেটি লিখে রাখুন।
- Domain বা SSL ব্যবহার না করলে সেগুলো Configure করবেন না।
- Installer শেষে দেখানো Panel URL, Username, Password ও Base Path নিরাপদে সংরক্ষণ করুন।

Service পরীক্ষা:

```bash
systemctl status x-ui --no-pager
```

অথবা:

```bash
x-ui status
```

## ৮. Admin Panel খোলা

Installer-এর দেওয়া প্রকৃত Port ও Web Base Path ব্যবহার করুন:

```text
http://YOUR_SERVER_IP:2053/YOUR_WEB_BASE_PATH/
```

Panel না খুললে পরীক্ষা করুন:

```bash
x-ui status
ss -lntp
ufw status verbose
journalctl -u x-ui -n 100 --no-pager
```

Provider Firewall-এ Panel Port Allow আছে কি না যাচাই করুন।

### SSH Tunnel দিয়ে নিরাপদ Panel Access

Public HTTP Panel-এ Login করলে Credentials এনক্রিপ্টেড থাকে না। SSH Tunnel ব্যবহার করতে নিজের কম্পিউটারে চালান:

```bash
ssh -p 22022 -L 2053:127.0.0.1:2053 root@YOUR_SERVER_IP
```

SSH Session চালু রাখুন। এরপর নিজের Browser-এ খুলুন:

```text
http://127.0.0.1:2053/YOUR_WEB_BASE_PATH/
```

Panel Port-কে Public Internet থেকে বন্ধ করার আগে Tunnel ঠিকমতো কাজ করছে নিশ্চিত করুন। Subscription Service আলাদা Port-এ থাকলে তার জন্যও HTTPS/TLS ব্যবহার করুন; Tunnel কেবল Admin Panel-এর উদাহরণ।

## ৯. Subscription Settings (IP Only)

3X-UI Panel-এ Login করে Subscription Settings খুলুন। Version অনুযায়ী Field-এর নাম বা অবস্থান ভিন্ন হতে পারে।

| Setting | Value |
|---|---|
| Public Address / Host | `YOUR_SERVER_IP` |
| Subscription Port | `2096` |
| Scheme | `http` (শুধু Plain HTTP Listener হলে) |
| Domain | খালি, যদি IP-only ব্যবহার করেন |
| Reverse Proxy URL | খালি, যদি ব্যবহার না করেন |
| TLS Certificate | HTTPS Setup না থাকলে প্রযোজ্য নয় |

Host Field-এ শুধু IP লিখুন—`http://` বা `https://` নয়, যদি Field-টি শুধু Host/IP চায়।

Save করার পর:

```bash
x-ui restart
ss -lntp | grep 2096
```

আপনার 3X-UI Version-এ Public Address Field না থাকলে Database-এর অজানা Key অনুমান করে পরিবর্তন করবেন না। Version-সংশ্লিষ্ট Official Documentation যাচাই করুন।

**HTTP/HTTPS:** Plain HTTP Listener-এর URL `http://` দিয়ে শুরু হবে। Plain HTTP Port-এ `https://` ব্যবহার করলে SSL Protocol Error হতে পারে। HTTPS চাইলে বৈধ TLS Certificate ও HTTPS Listener/Reverse Proxy সঠিকভাবে Configure করতে হবে।

## ১০. VPN Inbound তৈরি

3X-UI Panel থেকে **Inbounds → Add Inbound** নির্বাচন করুন। প্রয়োজন অনুযায়ী Protocol বেছে নিন:

- VLESS
- VMess
- Trojan
- Shadowsocks
- WireGuard
- Hysteria2

Inbound-এর Port, Transport, Security এবং অন্যান্য Option আপনার ব্যবহৃত Protocol ও Client App অনুযায়ী Configure করুন।

উদাহরণ:

| Inbound | Port | Protocol |
|---|---:|---|
| VLESS | 443 | TCP (উদাহরণ) |
| WireGuard | 51820 | UDP (উদাহরণ) |

এগুলো বাধ্যতামূলক মান নয়। আপনার Inbound-এ যে Port ও Protocol সেট করা আছে, Firewall-এ সেটিই খুলবেন।

Listening Port পরীক্ষা:

```bash
ss -lntup
```

### Subscription Port বনাম VPN Inbound Port

- **Subscription Port `2096`:** Client Configuration/Subscription আনার জন্য।
- **VPN Inbound Port:** VPN Tunnel Connection-এর জন্য, যেমন Inbound অনুযায়ী `443/TCP` বা `51820/UDP`।

VPN App-এর Endpoint-এ Subscription Port বসাবেন না। Client App-এ Subscription Import করলে App সাধারণত Configuration থেকে Inbound-এর তথ্য নেয়। WireGuard Profile হলে Profile-এর Endpoint ও Peer Settings অনুসরণ করুন।

## ১১. Client তৈরি ও Subscription Test

3X-UI থেকে Inbound-এর Client অংশে **Add Client** নির্বাচন করুন। প্রয়োজন অনুযায়ী Client Name, UUID বা Protocol-সংশ্লিষ্ট Credentials, Traffic Limit, Expiry Date এবং Enable Status সেট করে Save করুন।

Client-এর Subscription URL Panel থেকে Copy করুন। সাধারণ Format:

```text
http://YOUR_SERVER_IP:2096/YOUR_SUBSCRIPTION_PATH/CLIENT_TOKEN
```

প্রতিটি Client-এর Token আলাদা রাখুন। Token কাউকে প্রকাশ করবেন না।

VPS-এর ভিতর থেকে পরীক্ষা:

```bash
curl -i "http://127.0.0.1:2096/YOUR_SUBSCRIPTION_PATH/YOUR_CLIENT_TOKEN"
```

নিজের কম্পিউটার থেকে পরীক্ষা:

```bash
curl -i "http://YOUR_SERVER_IP:2096/YOUR_SUBSCRIPTION_PATH/YOUR_CLIENT_TOKEN"
```

সাধারণত সফল Response-এ `HTTP 200` এবং Subscription Data থাকবে। শুধু Status Code নয়, Response Body খালি কি না সেটিও দেখুন। বাস্তব Client Token প্রকাশ করবেন না।

## ১২. DNS Fix

VPN Connected দেখালেও Internet না চললে Client App-এর DNS Settings-এ পরীক্ষার জন্য নিচের DNS ব্যবহার করতে পারেন:

```text
1.1.1.1
1.0.0.1
```

অথবা:

```text
8.8.8.8
8.8.4.4
```

Client App-এ DNS ছাড়াও Routing, IPv4/IPv6 এবং DNS Routing Settings পরীক্ষা করুন।

### VPS-এর DNS পরীক্ষা

```bash
ping -c 3 1.1.1.1
getent hosts github.com
resolvectl status
ip route
```

- IP Ping চলে কিন্তু `getent hosts github.com` ব্যর্থ হলে DNS সমস্যা থাকতে পারে।
- IP Ping-ও ব্যর্থ হলে Gateway, Provider Network বা Routing সমস্যা হতে পারে।

### systemd-resolved DNS Fix (ঐচ্ছিক)

Ubuntu-তে `systemd-resolved` সক্রিয় থাকলে এবং Provider-এর নিজস্ব DNS প্রয়োজন না হলে নিচের Drop-in Configuration ব্যবহার করতে পারেন:

```bash
mkdir -p /etc/systemd/resolved.conf.d
cat > /etc/systemd/resolved.conf.d/99-custom-dns.conf <<'EOF'
[Resolve]
DNS=1.1.1.1 8.8.8.8
FallbackDNS=1.0.0.1 8.8.4.4
EOF

systemctl restart systemd-resolved
resolvectl status
getent hosts github.com
```

যদি `systemd-resolved` চালু না থাকে, Provider-এর Network/DNS Configuration আলাদা হতে পারে। আগে `systemctl status systemd-resolved` ও `ls -l /etc/resolv.conf` দিয়ে অবস্থা যাচাই করুন। DNS পরিবর্তনের পরও সমস্যা থাকলে Provider-এর DNS নির্দেশনা অনুসরণ করুন। অজানা অবস্থায় `/etc/resolv.conf` overwrite করবেন না।

## ১৩. Troubleshooting

### Panel Open হচ্ছে না

```bash
x-ui status
ss -lntp
ufw status verbose
journalctl -u x-ui -n 100 --no-pager
```

Panel Port, Web Base Path, UFW এবং Provider Firewall মিলিয়ে দেখুন।

### Subscription কাজ করছে না

```bash
ss -lntp | grep 2096
curl -I http://127.0.0.1:2096/
```

সঠিক Subscription Port, URL Path, Client Token এবং Scheme যাচাই করুন। HTTP Listener হলে URL-এ `http://` ব্যবহার করুন।

### VPN Connected কিন্তু Internet নেই

- Inbound Port ও Protocol সঠিক কি না দেখুন।
- UFW এবং Provider Firewall—দুই জায়গাতেই Rule আছে কি না দেখুন।
- Client-এর DNS, Routing, IPv4/IPv6 পরীক্ষা করুন।
- VPS-এর Default Route ও DNS পরীক্ষা করুন।

```bash
ip route
resolvectl status
ss -lntup
```

### Service Log

```bash
journalctl -u x-ui -n 200 --no-pager
systemctl status x-ui --no-pager
```

### Port ব্যবহার হচ্ছে কি না

```bash
ss -lntup
```

একই Port অন্য Service ব্যবহার করলে Inbound বা Panel-এর Port পরিবর্তন করুন।

## ১৪. Management Commands

3X-UI-এর Management Menu:

```bash
x-ui
```

সাধারণ Service Commands:

```bash
systemctl status x-ui
systemctl restart x-ui
systemctl stop x-ui
systemctl start x-ui
journalctl -u x-ui -f
```

সব Command সব Version-এ একইভাবে নাও থাকতে পারে। `x-ui` Menu-তে Version অনুযায়ী উপলব্ধ Option দেখুন।

## ১৫. Backup ও Security

### Backup

3X-UI-এর নিজস্ব Backup/Export Option থাকলে Panel বা `x-ui` Menu থেকে সেটি ব্যবহার করুন। অতিরিক্তভাবে পরিচিত Configuration Directory থাকলে Archive করতে পারেন:

```bash
mkdir -p /root/tzvpn-backups
tar -czf "/root/tzvpn-backups/x-ui-config-$(date +%F-%H%M%S).tar.gz" /etc/x-ui /usr/local/x-ui 2>/dev/null
ls -lh /root/tzvpn-backups
```

সব Version-এ একই File Path নাও থাকতে পারে। Archive তৈরি হয়েছে কি না পরীক্ষা করুন। Backup VPS-এর বাইরে নিরাপদ স্থানে রাখুন; এতে Database, Client Credentials বা Certificate Private Key থাকতে পারে।

### Security Checklist

- শক্তিশালী ও আলাদা Admin Password ব্যবহার করুন।
- Random Web Base Path ব্যবহার করুন।
- Admin Panel-এ HTTPS/TLS অথবা SSH Tunnel ব্যবহার করুন।
- অপ্রয়োজনীয় Port বন্ধ রাখুন।
- Provider Firewall এবং UFW দুটোই ঠিকভাবে Configure করুন।
- SSH Login ও Fail2ban Status পর্যবেক্ষণ করুন।
- ZIP-এর Script চালানোর আগে Source যাচাই করুন।
- Backup এবং Client Subscription Token গোপন রাখুন।
- নিয়মিত Ubuntu ও 3X-UI Update করুন।

Fail2ban পরীক্ষা:

```bash
systemctl status fail2ban --no-pager
fail2ban-client status
```

## ১৬. TZ VPN App

TZ VPN App-এ 3X-UI থেকে তৈরি করা Client Configuration বা Subscription URL ব্যবহার করুন—শুধু App-এর Version ও Protocol সমর্থিত হলে।

1. 3X-UI থেকে Client তৈরি করুন।
2. Client-এর Subscription URL Copy করুন।
3. TZ VPN App খুলুন।
4. Subscription/Add Configuration অংশে URL Paste বা Import করুন।
5. Configuration Update করে Connect করুন।
6. Connection না হলে Inbound Protocol, Port, DNS ও Client Compatibility যাচাই করুন।

**নিরাপত্তা:** Subscription URL-এ ব্যক্তিগত Client Token থাকতে পারে। Screenshot, Public Post বা অপরিচিত কাউকে URL দেবেন না।

## ১৭. Final Checklist

- [ ] Ubuntu 24.04 LTS VPS প্রস্তুত
- [ ] GitHub ZIP Download ও Extract সম্পন্ন
- [ ] SSH Login কাজ করছে
- [ ] SSH Port 22022 দিয়ে Login পরীক্ষা করা হয়েছে
- [ ] Provider Firewall ও UFW সঠিক
- [ ] Official 3X-UI Install সম্পন্ন
- [ ] Admin Panel নিরাপদে Access করা যাচ্ছে
- [ ] Subscription Port ও Scheme সঠিক
- [ ] VPN Inbound তৈরি
- [ ] Inbound-এর Port ও Protocol Firewall-এ Allow
- [ ] Client তৈরি ও Subscription Test সফল
- [ ] DNS ও Internet Test সফল
- [ ] Backup তৈরি ও নিরাপদ স্থানে রাখা হয়েছে
- [ ] TZ VPN App-এ Configuration Import পরীক্ষা করা হয়েছে

---

## Official Links

- **Eva Repository:** https://github.com/tzalmamun/Eva
- **TZVPN ZIP:** https://github.com/tzalmamun/Eva/raw/refs/heads/main/tzvpn.zip
- **3X-UI Official Project:** https://github.com/MHSanaei/3x-ui

**Maintained by TZ Mamun**
