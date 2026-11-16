\# SafeHands Insurance: Linux Jump Server Hardening (Task 1)



\*\*Intern:\*\* Ijaz Ullah

\*\*Client:\*\* SafeHands Insurance Brokers

\*\*Goal:\*\* Harden an Ubuntu host against common attack vectors and document every change.



\## Environment

\- Ubuntu Desktop () in VirtualBox

\- Hostname: SOC101-ubantu

\- Network: NAT with port forward (Windows 127.0.0.1:2222 to VM port 22)

\- Admin workstation: Windows with OpenSSH client



\## Steps performed



\### 1. Update the system

`sudo apt update \&\& sudo apt upgrade -y`

Patches known vulnerabilities before any other change.



\### 2. Baseline audit (before)

Saved open ports, enabled services, UFW state and SSH settings to `evidence/\*\_before.txt`.



\### 3. Standard user with sudo

`sudo adduser safeadmin` then `sudo usermod -aG sudo safeadmin`

Screenshot: `screenshots/id\_safeadmin.png`



\### 4. RSA key authentication

Generated a 4096-bit RSA key on the Windows workstation with `ssh-keygen -t rsa -b 4096`. Only the public key was placed in `/home/safeadmin/.ssh/authorized\_keys` (permissions 700 and 600). The private key stays on the workstation.

Screenshot: `screenshots/key\_login\_success.png`



\### 5. SSH hardening

Added `/etc/ssh/sshd\_config.d/00-hardening.conf` (copy in `evidence/`):

PermitRootLogin no, PasswordAuthentication no, PubkeyAuthentication yes, PermitEmptyPasswords no, MaxAuthTries 3, LoginGraceTime 30, AllowUsers safeadmin. X11Forwarding was set to no in `sshd\_config`.

Screenshots: `root\_login\_denied.png`, `password\_login\_denied.png`, `ssh\_after.png`



\### 6. UFW firewall

`ufw default deny incoming`, `ufw default allow outgoing`, `ufw allow 22/tcp`, `ufw enable`

Screenshot: `screenshots/ufw\_status.png`



\### 7. Remove unnecessary services

Disabled and masked avahi-daemon, cups, cups-browsed and ModemManager using `systemctl disable --now` and `systemctl mask`.

Screenshots: `ports\_after.png`, `diff\_ports.png`, `diff\_services.png`



\## Before / After



| Item | Before | After |

|---|---|---|

| Root SSH login | prohibit-password | no |

| Password authentication | yes | no |

| Allowed SSH users | all | safeadmin only |

| UFW | inactive | active, deny incoming, allow 22/tcp |

| Network listeners | 22, 53, 323, 631, 5353 | 22 (SSH); 53 and 323 loopback only |

| Enabled unit files | 112 | 105 |



\## Verification against the mentor checks

1\. \*\*UFW active, only SSH allowed:\*\* `ufw status verbose` (`screenshots/ufw\_status.png`)

2\. \*\*Password auth disabled, key login works:\*\* `sshd -T` output and the denied and successful login screenshots

3\. \*\*Non-root sudo user, root login disabled:\*\* `id safeadmin` and `permitrootlogin no`



\## Lessons learned

\- Pasting the wrong clipboard text into `authorized\_keys` broke key login. I found it by checking the file size (47 bytes instead of about 740) and fixed it by piping the key over SSH.

\- A shell glob is expanded before `sudo` runs, so I used `sudo sh -c '...'` to copy files from another user's home.

\- Files in `sshd\_config.d/` can override the main config, so I put the hardening settings there.

\- I kept a second SSH session open and took a snapshot before changing SSH, to avoid a lockout.



\## Evidence

All before/after outputs are in the `evidence/` folder.

