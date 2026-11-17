 Cockpit Installation and Custom Port Configuration on RHEL 9

---

## Objectives
- Install Cockpit Web Console
- Enable and manage the Cockpit service
- Configure firewall access
- Change the default Cockpit port
- Configure SELinux for the custom port
- Validate secure browser access
- Ensure Cockpit remains functional after reconfiguration

---

## Prerequisites
- RHEL 9 server or virtual machine
- Root or sudo privileges
- Active network connection
- Updated operating system
- SELinux enabled and enforcing
- Firewalld running

---

##  Implementation Summary

### Step 1: Update the System

```bash
sudo dnf update -y
sudo reboot
```

### Step 2: Install Cockpit

```bash
sudo dnf install cockpit -y
```

### Step 3: Start and Enable Cockpit

```bash
sudo systemctl enable --now cockpit.socket
sudo systemctl status cockpit.socket
```

### Step 4: Allow Cockpit Through the Firewall

```bash
sudo firewall-cmd --permanent --add-service=cockpit
sudo firewall-cmd --reload
```

### Step 5: Verify Default Access

Access Cockpit from a browser:

```text
https://SERVER_IP:9090
```

### Step 6: Change the Default Cockpit Port

Locate the directory:

```bash
find / -name cockpit.socket -type f
```

Create the configuration file:

```bash
sudo vi /etc/systemd/system/cockpit.socket.d
```

Add the new port number:

```ini
[Socket]
ListenStream=4545
```

### Step 7: Configure SELinux for the New Port

Install SELinux utilities if not already intalled:

```bash
sudo dnf install policycoreutils-python-utils -y
```

Label the new port:

```bash
sudo semanage port -a -t websm_port_t -p tcp 4545
```

Verify the configuration:

```bash
sudo semanage port -l | grep 4545
```

### Step 8: Reload Services and Update Firewall

```bash
sudo systemctl daemon-reload
sudo systemctl restart cockpit.socket
```

Allow the new port:

```bash
sudo firewall-cmd --permanent --add-port=4545/tcp
sudo firewall-cmd --reload
```

### Step 9: Verify the New Configuration

Confirm Cockpit is listening on the new port:

```bash
sudo ss -tulpn | grep 4545
```

Access Cockpit:

```text
https://SERVER_IP:4545
```

---

##  Validation Checklist

Verify the following:

- [ ] Cockpit package installed successfully
- [ ] `cockpit.socket` is active and running
- [ ] Firewall allows TCP 4545
- [ ] SELinux port mapping exists for `websm_port_t`
- [ ] Cockpit is listening on port 4545
- [ ] Browser successfully loads the Cockpit login page
- [ ] User authentication succeeds
- [ ] System dashboard is accessible
- [ ] SELinux remains in Enforcing mode
- [ ] No service errors reported in system logs

Validation commands:

```bash
systemctl status cockpit.socket
```

```bash
ss -tulpn | grep 4545
```

```bash
firewall-cmd --list-ports
```

```bash
getenforce
```

```bash
semanage port -l | grep 4545
```

---

## 🛠 Troubleshooting

### Cockpit Service Fails to Start

```bash
sudo systemctl status cockpit.socket
sudo journalctl -xe
```

### SELinux Blocking the Port

```bash
sudo ausearch -m AVC -ts recent
```

```bash
sudo semanage port -l | grep websm_port_t
```

### Custom Port Not Listening

```bash
sudo systemctl daemon-reload
sudo systemctl restart cockpit.socket
```

```bash
sudo ss -tulpn | grep 9443
```

### Firewall Blocking Connections

```bash
sudo firewall-cmd --list-all
```

```bash
sudo firewall-cmd --list-ports
```

### Browser Certificate Warning

Cockpit uses a self-signed certificate by default. Accept the warning for lab environments or replace it with a trusted SSL certificate.

---

## Skills Demonstrated

- Linux System Administration
- Red Hat Enterprise Linux (RHEL 9)
- Cockpit Web Console Administration
- Systemd Service Management
- SELinux Policy Management
- Firewalld Configuration
- Network Port Management
- Security Hardening
- System Troubleshooting and Log Analysis

---
```
