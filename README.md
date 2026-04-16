# How to Install Kali Linux on Raspberry Pi 4 and Access It with SSH and Remote Desktop from Windows Machine
---

This step-by-step guide shows how to install Kali Linux on a Raspberry Pi 4 and access it from Windows using SSH and Remote Desktop. It includes flashing the Kali ARM image, identifying the Raspberry Pi IP address on the network, logging in with PuTTY, changing the default username and password, setting a static IP, and enabling XRDP for desktop access.

---

**Hardware used**
- **Raspberry Pi 4 (8GB)**
- **SanDisk Ultra microSD 32GB**

It covers:
- Flashing the Kali ARM image
- Getting the Raspberry Pi IP address via **DHCP**
- Finding the Pi on the network using **Nmap/Zenmap**
- First login using **PuTTY**
- Changing the default credentials
- Setting a static IP
- Enabling **Remote Desktop Connection**
- Fixing the lock screen name if it still shows `kali`

## What this guide uses
- **Kali ARM image**: https://www.kali.org/get-kali/#kali-arm
- **Raspberry Pi Imager**: https://www.raspberrypi.com/software/
- **balenaEtcher**: https://etcher.balena.io/
- **PuTTY**: https://putty.org/index.html
- **Nmap / Zenmap**: https://nmap.org/download.html
- **Windows Remote Desktop Connection** (`mstsc`)

---

## 1) Flash Kali ARM to the microSD card

1. Download the correct **Kali ARM image** for **Raspberry Pi 4** from:
   - https://www.kali.org/get-kali/#kali-arm
2. Flash it to the microSD card using either:
   - **Raspberry Pi Imager**, or
   - **balenaEtcher**
3. After flashing, insert the microSD card into the Raspberry Pi.

---

## 2) Boot the Raspberry Pi and get an IP via DHCP

1. Connect **Ethernet** from the Raspberry Pi to the same network as your Windows PC.
2. Connect power to the Raspberry Pi.
3. Wait for the Pi to boot fully.
4. The Pi should automatically obtain an IP address via **DHCP**.

---

## 3) Find the Raspberry Pi IP address from Windows using Nmap

Install **Nmap** for Windows if needed:
- https://nmap.org/download.html

Open **Zenmap** and scan your local subnet. Example:

```text
nmap -sn 10.x.x.0/24
```

Look for a result that shows a host identified as **Raspberry Pi**.

Example style of IP to expect:

```text
10.x.x.25
```

This is just a randomly assigned DHCP IP example. In the scan output, the Raspberry Pi entry may appear with a label such as:

```text
raspberry pi Trading
```

That IP is the initial Raspberry Pi IP address you will use for SSH.

---

## 4) SSH into the Pi using PuTTY

Open **PuTTY**.

In **PuTTY Configuration**, enter `kali@10.x.x.25` under **Host Name (or IP address)**, leave **Port** as `22`, and keep **Connection type** set to `SSH`.

Default Kali ARM credentials:

```text
Username: kali
Password: kali
```

---

## 5) Change the default credentials properly

Do **not** try to rename the logged-in `kali` user directly.

Create the new user first, then remove `kali`.

### 5.1 Create the new user

Copy-paste:

```bash
sudo adduser <preferred_name>
sudo usermod -aG sudo <preferred_name>
```

Set `<preferred_password>` when prompted.

### 5.2 Log out and SSH back in with the new account

Reconnect in PuTTY using your new account:

```text
Username: <preferred_name>
Password: <preferred_password>
```

### 5.3 Remove the default `kali` account

After confirming the new account works, copy-paste:

```bash
sudo deluser --remove-home kali
```

---

## 6) Set a static IP (example: `10.x.x.67`)

Edit the DHCP client config:

```bash
sudo nano /etc/dhcpcd.conf
```

Add this at the bottom:

```conf
interface eth0
static ip_address=10.x.x.67/24
static routers=10.x.x.1
static domain_name_servers=10.x.x.1 8.8.8.8
```

Save and reboot:

```bash
sudo reboot
```

> Update the IP, gateway, and DNS values to match your own network.

---

## 7) Install the desktop and Remote Desktop service from terminal

SSH back in using your new username, then copy-paste:

```bash
sudo apt update
sudo apt install -y kali-desktop-xfce xrdp
sudo adduser xrdp ssl-cert
printf 'xfce4-session\n' > ~/.xsession
chmod +x ~/.xsession
sudo systemctl enable xrdp
sudo systemctl restart xrdp
```

---

## 8) Connect to the GUI from Windows

1. On Windows, press **Win + R**
2. Run:

```text
mstsc
```

3. In **Computer**, enter the Raspberry Pi IP address, for example:

```text
10.x.x.67
```

4. Log in with:

```text
Username: <preferred_name>
Password: <preferred_password>
```

---

## 9) Update Kali after logging in via Remote Desktop

Open the terminal in the desktop session and run:

```bash
sudo apt update && sudo apt full-upgrade -y
sudo reboot
```

---

## 10) Fix the lock screen name if it still shows `kali`

If the desktop lock screen still shows `kali` even after creating your new user, update the account display name and hostname.

### 10.1 Update the user full name

```bash
sudo chfn -f "<preferred_name>" <preferred_name>
```

### 10.2 Change the hostname

Example:

```bash
sudo hostnamectl set-hostname <preferred_name>-pi
```

### 10.3 Update `/etc/hosts`

Open:

```bash
sudo nano /etc/hosts
```

Change the line:

```text
127.0.1.1    kali-raspberrypi
```

to:

```text
127.0.1.1    <preferred_name>-pi
```

Then reboot:

```bash
sudo reboot
```

---

## 11) Quick command block

Use this only **after** you have already SSHed in as `kali`:

```bash
sudo adduser <preferred_name>
sudo usermod -aG sudo <preferred_name>
```

Then log out, log back in as `<preferred_name>`, and run:

```bash
sudo deluser --remove-home kali
sudo apt update
sudo apt install -y kali-desktop-xfce xrdp
sudo adduser xrdp ssl-cert
printf 'xfce4-session\n' > ~/.xsession
chmod +x ~/.xsession
sudo systemctl enable xrdp
sudo systemctl restart xrdp
```

If you want a static IP, also configure:

```bash
sudo nano /etc/dhcpcd.conf
```

and add:

```conf
interface eth0
static ip_address=10.x.x.67/24
static routers=10.x.x.1
static domain_name_servers=10.x.x.1 8.8.8.8
```

Then reboot:

```bash
sudo reboot
```

---

## Notes

- This guide uses **manual Kali ARM image flashing**.
- It does **not** use **Raspberry Pi Connect**.
- For first boot, connect the Pi by **Ethernet** so it gets an IP automatically via **DHCP**.
- Initial access is via **PuTTY (SSH)**.
- GUI access is via **Windows Remote Desktop Connection**.
- Use **Zenmap/Nmap on Windows** to identify the Pi before the first SSH login.
