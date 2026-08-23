# hosts-tools: Linux Content Filtering & Self-Control Toolkit

This toolkit helps you set up a strong, hard-to-bypass content filter on your Linux computer. It is designed for self-control, ensuring that once you lock your system down, you cannot easily revert the changes on impulse.

## Overview
This setup utilizes three main components:
1. **Cloudflare Family DNS (1.1.1.3):** Automatically blocks adult content and malware at the network level.
2. **Dynamic Hosts Updater (`dynamic-hosts`):** A daily automated script that pulls blocklists (like HaGeZi) to block specific websites (social media, gambling) and unwanted search engines.
3. **24-Hour Timelock (`timelock`):** A script that securely encrypts your system's master (root) password and forces a 24-hour waiting period to decrypt it, preventing impulsive bypasses.

---

## Prerequisites
**⚠️ READ BEFORE STARTING:** This process intentionally locks down your administrative permissions (`sudo`). Make sure you are comfortable with basic terminal commands and understand that you will lose instant administrator access to this machine.

Open a terminal window and identify your exact Linux username by running:
```bash
whoami
```

Keep this username handy, you will need it in the final steps.

---

## Step 1: Configure Cloudflare DNS

First, we will set your system to use Cloudflare's Family DNS to handle base-level filtering.

1. Open your system DNS config file:
```bash
sudo nano /etc/systemd/resolved.conf
```

2. Find or add the `DNS=` line under `[Resolve]` and set it to Cloudflare's family servers:
```ini
DNS=1.1.1.3 1.0.0.3
```

3. Save and exit (`Ctrl+O`, `Enter`, `Ctrl+X`).

4. Restart the DNS service:
```bash
sudo systemctl restart systemd-resolved
```

---

## Step 2: Setup Dynamic Hosts Updater

Instead of manually updating your hosts file, we will set up a daily script to pull community blocklists.

1. **Find your Blocklists:**
* Go to the [HaGeZi DNS Blocklists](https://github.com/hagezi/dns-blocklists) repository and find the raw text URLs for your desired categories (Gambling, Social Media, NSFW, DoH/VPN Bypass).
* Find a Search Engines list, copy it locally, and remove the one engine you plan to use (e.g., Google, as its SafeSearch works seamlessly with Cloudflare). Host this modified list somewhere accessible (like a GitHub Gist).

2. **Configure the Script:** Update the `BLOCKLISTS` array inside the `dynamic-hosts` file with the URLs you gathered in step 1.

3. **Copy the Script:**
```bash
sudo cp ./dynamic-hosts /etc/cron.daily/
```

4. **Secure the Script:**
```bash
sudo chown root:root /etc/cron.daily/dynamic-hosts
sudo chmod 755 /etc/cron.daily/dynamic-hosts
```

5. **Run it once** to apply the initial blocklist:
```bash
sudo /etc/cron.daily/dynamic-hosts
```

---

## Step 3: Setup the 24-Hour Timelock

This script will act as the gatekeeper for your root password.

1. **Copy the Script:**
```bash
sudo cp ./timelock /usr/local/bin/timelock
```

2. **Secure the Script:**
```bash
sudo chown root:root /usr/local/bin/timelock
sudo chmod 755 /usr/local/bin/timelock
```

---

## Step 4: Lock the Root Password

Instead of hiding a physical piece of paper, we will use the Timelock script to digitally bury your root password.

1. Go to a [Random Password Generator](https://www.random.org/passwords/) and generate a 27-character random password. **Copy it to your clipboard.**

2. Change your system's root password:
```bash
sudo passwd root
```

3. **Encrypt the password** using the script:
```bash
sudo timelock encrypt "YOUR_27_CHARACTER_PASSWORD"
```

4. **Save the Output:** The script will output a base64 encrypted string (e.g., `U2FsdGVkX1...`). Save this encrypted string in a standard text file on your desktop or documents folder.

5. **Clear your clipboard**, terminal history, and lose the original plain-text password. The only way to get it back now is to decrypt that string.

---

## Step 5: Restrict Your Admin (Sudo) Privileges

Finally, we remove your ability to bypass the system, leaving you only with basic update/power permissions and the ability to trigger the timelock.

1. Open the privilege configuration file safely:
```bash
sudo visudo
```

2. Paste the following block at the very bottom (Replace `YOUR_USERNAME` with your actual Linux username):
```text
# Allow software updates
YOUR_USERNAME ALL=(ALL:ALL) /usr/bin/apt, /usr/bin/flatpak, /usr/bin/snap

# Allow basic power operations
YOUR_USERNAME ALL=(ALL:ALL) /sbin/reboot, /sbin/poweroff, /sbin/shutdown

# Allow access to the Timelock gatekeeper
YOUR_USERNAME ALL=(ALL:ALL) /usr/local/bin/timelock
```

3. Save and exit the editor.

4. Remove yourself from the sudo group (this revokes your general admin rights):
```bash
sudo deluser YOUR_USERNAME sudo
```

5. Restart your computer for the changes to take effect:
```bash
sudo reboot
```

---

## How to Get Root Access

If you genuinely need administrative access to your computer, you must wait 24 hours.

1. Open your terminal and run the delayed decryption on your saved string:
```bash
sudo timelock decrypt "yourEncryptedStringHere..."
```

2. **Leave the terminal open.** It will hang for exactly 24 hours. If it closes or the computer restarts, the timer resets.

3. After 24 hours, the script will output your 27-character root password.

4. Switch to the root user:
```bash
su -
```