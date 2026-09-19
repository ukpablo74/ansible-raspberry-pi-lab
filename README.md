# Ansible + Raspberry Pi lab

A Raspberry Pi set up as a Linux target and managed with Ansible over SSH — the classic starting point for learning core Ansible: inventories, playbooks, modules and idempotency, without needing a network simulator.

## Why a Raspberry Pi

Almost every beginner Ansible course (including Jeff Geerling's) assumes you're managing a Linux host over SSH, which is exactly what a Pi is. It's the simplest and most realistic option for learning the fundamentals before moving on to anything more specialised, such as network device automation.

If you don't want to buy hardware, a Debian or Ubuntu VM (VirtualBox, or a Docker container with SSH enabled) on your own PC gives the same learning value at zero cost.

## Requirements

- A Raspberry Pi (this was built and tested on a Raspberry Pi 5)
- A microSD card, 8 GB minimum, 16 GB or more recommended (I used 32 GB)
- A separate machine with Ansible installed (this used Ubuntu on WSL2)

## 1. Check your local Ansible and Python setup

Before targeting the Pi, confirm what's installed on the control machine in VS Code:

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

## 2. Flash Raspberry Pi OS Lite to the SD card, with Wi-Fi pre-configured

Raspberry Pi OS Lite is headless — no desktop — which is exactly what you want for an Ansible target: less overhead, boots faster, and closer to a real server. Setting up Wi-Fi at this stage means the Pi joins your network on its own the first time it boots, with no monitor, keyboard, or Ethernet cable ever needed.

### Step by step

1. **Download the Raspberry Pi Imager.** Go to [raspberrypi.com/software](https://www.raspberrypi.com/software/), download the installer for your operating system (Windows, macOS or Linux), and install it like any normal application.

2. **Insert the SD card into your computer.** Use a card reader if your laptop doesn't have a built-in slot. An 8 GB card is the bare minimum; 16 GB or more is more comfortable.

3. **Open Raspberry Pi Imager** and click **"Choose Device"**. Select your model — **Raspberry Pi 5**.

4. **Click "Choose OS"**, then:
   - Select **"Raspberry Pi OS (other)"**
   - Select **"Raspberry Pi OS Lite (64-bit)"**

   Lite means no desktop is installed, which is what you want for a device you'll only ever reach over SSH.

5. **Click "Choose Storage"** and select your SD card. Double check you've picked the right device in this step, since everything on it will be erased.

6. **Click the gear icon** in the bottom-right corner (on some versions you'll instead see an "Edit Settings" button, or a prompt asking "Would you like to apply OS customisation settings?" — choose **Edit Settings**). This opens the OS Customisation screen, where the Wi-Fi setup happens.

7. **On the "General" tab:**
   - **Set hostname:** tick this box and enter something memorable, e.g. `ansible-target`. This is how you'll find the Pi on your network later (`ansible-target.local`).
   - **Set username and password:** tick this box. Choose your own username and a strong password — avoid the old default `pi` / `raspberry`, since that combination is well known and insecure on a device connected to your network.
   - **Configure wireless LAN:** tick this box. Now fill in:
     - **SSID:** your Wi-Fi network's name, typed exactly as it appears (case-sensitive).
     - **Password:** your Wi-Fi password.
     - **Wireless LAN country:** select the two-letter code for the country you're in (e.g. `GB` for the United Kingdom). This matters — Wi-Fi radio regulations differ by country, and getting this wrong can stop the Wi-Fi chip from working properly.
   - **Set locale settings:** tick this box, and pick your time zone (e.g. `Europe/London`) and keyboard layout (e.g. `gb`).

8. **Click the "Services" tab:**
   - Tick **"Enable SSH"**.
   - Choose **"Use password authentication"** to keep things simple while you're learning. (Once you're comfortable, "Allow public-key authentication only" is more secure — you'd paste in the public half of an SSH key pair here instead.)

9. **Click the "Options" tab** and leave everything at its default unless you have a specific reason to change it.

10. **Click "Save"**, then **"Write"** (sometimes labelled "Yes" to confirm). The Imager will write the OS image to the card and then verify it — this takes a few minutes. Don't remove the card until it says it's finished.

### Boot it up

11. Once writing finishes, safely eject the SD card, insert it into the Raspberry Pi 5, and connect the power supply. No monitor, keyboard, or network cable is needed — the Wi-Fi details you entered are already baked into the card.

12. Give it 1–2 minutes for the first boot, during which it'll join your Wi-Fi network automatically.

### If you need to add or fix Wi-Fi after the Pi is already running

If you skipped Wi-Fi during imaging, or need to switch networks later, you can do it from the Pi itself once you have any way of reaching it (SSH over Ethernet, for example):

```
sudo raspi-config
```

Then navigate to **"1 System Options"** → **"S1 Wireless LAN"**, and enter your country, SSID and password when prompted. Select **"Finish"** to exit and apply the change.

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
