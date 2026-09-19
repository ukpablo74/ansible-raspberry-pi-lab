# Ansible + Raspberry Pi lab

A Raspberry Pi set up as a Linux target and managed with Ansible over SSH — the classic starting point for learning core Ansible: inventories, playbooks, modules and idempotency, without needing a network simulator.

## Why a Raspberry Pi

Almost every beginner Ansible course (including Jeff Geerling's) assumes you're managing a Linux host over SSH, which is exactly what a Pi is. It's the simplest and most realistic option for learning the fundamentals before moving on to anything more specialised, such as network device automation.

If you don't want to buy hardware, a Debian or Ubuntu VM (VirtualBox, or a Docker container with SSH enabled) on your own PC gives the same learning value at zero cost.

## Requirements

- A Raspberry Pi (this was built and tested on a Raspberry Pi 5)
- A microSD card, 8 GB minimum, 16 GB or more recommended
- A separate machine with Ansible installed (this used Ubuntu on WSL2)

## 1. Check your local Ansible and Python setup

Before targeting the Pi, confirm what's installed on the control machine:

```
ansible --version
```

This also shows the config file in use, typically:

```
config file = /etc/ansible/ansible.cfg
```

Check the Python version too, since Ansible depends on it:

```
python3 --version
```

## 2. Flash Raspberry Pi OS Lite to the SD card

Raspberry Pi OS Lite is headless — no desktop — which is exactly what you want for an Ansible target: less overhead, boots faster, and closer to a real server.

1. **Download the Raspberry Pi Imager** from [raspberrypi.com/software](https://www.raspberrypi.com/software/) and install it.
2. **Insert the SD card** using a card reader.
3. **Choose OS**: click "Choose Device" and select your Pi model, then "Choose OS" → Raspberry Pi OS (Other) → **Raspberry Pi OS Lite (64-bit)**.
4. **Choose Storage**: select your SD card. Double-check you've picked the right device, since this erases it.
5. **Pre-configure before writing.** Click the gear icon (or use the "Edit Settings" prompt) to open OS Customisation:
   - **General tab**: set a hostname (e.g. `ansible-target`), a username and password (avoid the old default `pi`/`raspberry`), Wi-Fi SSID and password if not using Ethernet, and your locale/timezone/keyboard layout.
   - **Services tab**: enable SSH. Choose "Use password authentication" to start simply, or "Allow public-key authentication only" if you already have an SSH key pair and want to paste the public key in.
   - **Options tab**: leave the defaults unless you have a reason to change them.
6. Click **Save**, then **Write**, and confirm. Writing and verifying takes a few minutes.

## 3. Boot and connect

Eject the card, insert it in the Pi, connect Ethernet (or rely on the Wi-Fi you configured), and power it on. No monitor or keyboard needed — give it a minute or two to boot and join the network.

Find it:

```
ping ansible-target.local
```

Or check your router's DHCP client list for the hostname or IP. Then connect once manually to confirm it works and accept the host key:

```
ssh <username>@ansible-target.local
```

## 4. Add it to an Ansible inventory

Create an inventory file on your control machine, for example `inventory.ini`:

```ini
[pi]
ansible-target.local

[pi:vars]
ansible_user=<username>
```

If you set up key-based SSH rather than a password, add:

```ini
ansible_ssh_private_key_file=~/.ssh/id_ed25519
```

## 5. Test the connection

```
ansible pi -i inventory.ini -m ping
```

A successful response looks like:

```
ansible-target.local | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

That's the "hello world" of Ansible, and confirms the Pi is ready for playbooks.

## Where this goes from here

With the ping working, the next step is writing playbooks: installing packages, managing files, and building out roles and handlers. This repo captures the environment setup; playbooks land here as they're built.

## Related

A separate lab running Nokia SR Linux routers as containers with Containerlab, also configured with Ansible, is in [network-automation-lab](https://github.com/ukpablo74/network-automation-lab).
