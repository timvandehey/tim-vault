This guide will walk you through setting up your Raspberry Pi to run Obsidian in kiosk mode using an AppImage, ensuring persistence, automatic restarts, basic power outage handling, and continued SSH accessibility.

**Important Pre-requisites:**

  * **Raspberry Pi OS with Desktop:** This guide assumes you have Raspberry Pi OS (formerly Raspbian) with a graphical desktop environment installed. Kiosk mode for GUI applications requires a display server.
  * **Obsidian AppImage:** Download the appropriate Obsidian AppImage for your Raspberry Pi's architecture (likely ARM64 for newer Pis, or ARMv7 for older models like Pi 3B+). You can find it on the official Obsidian download page.
  * **Basic Linux Command Line Knowledge:** This guide involves editing configuration files and running commands in the terminal.

-----

## 1\. Initial Setup and Preparation

### 1.1 Update and Upgrade Your System

It's always a good idea to start with an up-to-date system.

```bash
sudo apt update
sudo apt full-upgrade -y
sudo reboot
```

### 1.2 Download and Make Obsidian AppImage Executable

Choose a location for your AppImage, for example, `/opt/Obsidian/`.

```bash
sudo mkdir -p /opt/Obsidian
cd /opt/Obsidian/
wget [Obsidian AppImage Download URL] # Replace with the actual URL
chmod +x Obsidian*.AppImage # Replace with your downloaded AppImage name
```

**Note on AppImage persistence:** AppImages usually store configurations in `~/.config` or `~/.local/share`. For better portability and to ensure settings persist even if the AppImage itself is updated or moved, you can run it once with `  --appimage-portable-home ` and `--appimage-portable-config` flags. This will create `.home` and `.config` directories next to the AppImage.

```bash
cd /opt/Obsidian/
./Obsidian*.AppImage --appimage-portable-home --appimage-portable-config
# Let it launch, then close it. This creates the portable directories.
```

### 1.3 Disable Screen Blanking and Power Saving

For a true kiosk, you don't want the screen to blank or turn off.

**Using `xset` for current session (temporary):**

```bash
xset -dpms # disable DPMS (Energy Star) features
xset s off # disable screen saver
xset s noblank # don't blank the video device
```

**For persistent screen blanking disable:**

Edit or create `/etc/xdg/lxsession/LXDE-pi/autostart` (or `~/.config/lxsession/LXDE-pi/autostart` for user-specific settings).
Add these lines at the end:

```
@xset -dpms
@xset s off
@xset s noblank
@xset s noexpose
@xset r off
```

**Disable Monitor Sleep via `raspi-config`:**

```bash
sudo raspi-config
```

Navigate to **Display Options** -\> **Screen Blanking** and disable it.

### 1.4 Install `unclutter` (Optional, but recommended for Kiosk Mode)

`unclutter` hides the mouse cursor when it's idle.

```bash
sudo apt install unclutter -y
```

-----

## 2\. Configure Kiosk Mode with a Systemd Service

We'll use a systemd service to launch Obsidian automatically at boot, keep it running, and restart it if it crashes.

### 2.1 Create the Kiosk Service File

```bash
sudo nano /etc/systemd/system/obsidian-kiosk.service
```

Paste the following content, adjusting paths and usernames as necessary:

```ini
[Unit]
Description=Obsidian Kiosk Mode
After=network-online.target graphical.target
Wants=network-online.target

[Service]
Type=simple
User=pi # Replace 'pi' with your Raspberry Pi username
Environment=DISPLAY=:0
ExecStart=/usr/bin/xinit /opt/Obsidian/Obsidian*.AppImage --no-sandbox --kiosk --enable-features=UseOzonePlatform --ozone-platform=wayland -- "%_vt"
Restart=on-failure
RestartSec=10
KillMode=process
IgnoreSIGPIPE=false
StandardOutput=journal
StandardError=journal
WorkingDirectory=/opt/Obsidian/

[Install]
WantedBy=graphical.target
```

**Explanation of the `ExecStart` line:**

  * `/usr/bin/xinit`: This command starts the X Window System. It's crucial for running graphical applications in kiosk mode without a full desktop environment.
  * `/opt/Obsidian/Obsidian*.AppImage`: The path to your Obsidian AppImage.
  * `--no-sandbox`: Some AppImages, especially Electron-based ones like Obsidian, might require this flag to run on certain systems, especially in a kiosk setup.
  * `--kiosk`: This is a Chromium/Electron flag that makes the application run in full-screen, dedicated kiosk mode.
  * `--enable-features=UseOzonePlatform --ozone-platform=wayland`: If you are using Wayland (default on newer Raspberry Pi OS versions), these flags might be necessary for Obsidian to run correctly. If you encounter issues, try removing them.
  * `-- "%_vt"`: This is part of `xinit`'s syntax to pass arguments to the X server.

**Important Notes:**

  * **`User=pi`**: Ensure this is the correct username you want Obsidian to run as. This user needs to be able to access the display.
  * **`Environment=DISPLAY=:0`**: This tells the X server where to display the application.
  * **`Restart=on-failure`**: This ensures that if Obsidian crashes, systemd will attempt to restart it.
  * **`RestartSec=10`**: Waits 10 seconds before attempting a restart.
  * **`WorkingDirectory=/opt/Obsidian/`**: Sets the working directory for the AppImage.

### 2.2 Enable and Start the Service

```bash
sudo systemctl daemon-reload
sudo systemctl enable obsidian-kiosk.service
sudo systemctl start obsidian-kiosk.service
```

### 2.3 Verify the Service Status

```bash
sudo systemctl status obsidian-kiosk.service
```

You should see "active (running)". If not, check `journalctl -u obsidian-kiosk.service` for error messages.

-----

## 3\. Configure Autologin for Kiosk Mode

For the kiosk to boot directly into Obsidian, you need to configure the Raspberry Pi to autologin a user.

```bash
sudo raspi-config
```

Navigate to **System Options** \> **Boot / Auto Login**.
Select **Desktop Autologin** (or **Console Autologin** if you only want the X server to start your app and not a full desktop). Choosing "Desktop Autologin" logs the specified user into the graphical desktop, and our systemd service will then take over.

-----

## 4\. Ensure Persistence and Power Outage Handling

### 4.1 SD Card Corruption Mitigation

Power outages can corrupt SD cards, especially if writes are in progress. While a UPS is the best hardware solution (see 4.2), here are software mitigations:

  * **Minimize Writes to SD Card:**

      * **Disable Swap:** If you're not heavily using the Pi, disabling swap can reduce SD card writes.
        ```bash
        sudo systemctl disable dphys-swapfile
        sudo systemctl stop dphys-swapfile
        ```
      * **Log to RAM (tmpfs):** Redirect logs to RAM to prevent constant writes. This means logs will be lost on reboot.
        ```bash
        sudo nano /etc/systemd/journald.conf
        ```
        Change or add:
        ```
        Storage=volatile
        ```
        Then restart the journald service:
        ```bash
        sudo systemctl restart systemd-journald
        ```
      * **Obsidian Sync:** If you're using Obsidian Sync, this will handle data integrity across devices.

  * **Read-Only Filesystem (Advanced):** For extreme reliability, you can configure a read-only root filesystem, but this makes system updates and configuration changes more complex. Only recommended if you truly need a static, highly robust setup.

### 4.2 Hardware Solutions for Power Outages (Recommended)

The most effective way to handle power outages is with hardware:

  * **Uninterruptible Power Supply (UPS) for Raspberry Pi:** Many small UPS modules are available specifically for Raspberry Pi (e.g., PiSugar, UPS HATs). These provide a battery backup, allowing the Pi to gracefully shut down or continue operating during short power interruptions. Some even have GPIO pins that can signal a power loss, triggering a clean shutdown script.

-----

## 5\. Maintain SSH Accessibility

SSH access is critical for managing your kiosk without a keyboard and mouse.

### 5.1 Enable SSH

SSH is usually enabled by default on Raspberry Pi OS. If not:

```bash
sudo raspi-config
```

Navigate to **Interface Options** \> **SSH** and enable it.

### 5.2 Configure SSH for Reliability

  * **Static IP Address (Recommended):** Assigning a static IP address to your Raspberry Pi makes it easier to connect via SSH without constantly checking for its IP.

      * Edit `/etc/dhcpcd.conf`:
        ```bash
        sudo nano /etc/dhcpcd.conf
        ```
        Add the following lines at the end, adjusting `interface`, `static ip_address`, `static routers`, and `static domain_name_servers` to match your network:
        ```
        interface eth0 # Or wlan0 for Wi-Fi

        static ip_address=192.168.1.100/24
        static routers=192.168.1.1
        static domain_name_servers=192.168.1.1 8.8.8.8
        ```
        Reboot for changes to take effect: `sudo reboot`.

  * **KeepAlive:** To prevent SSH sessions from timing out, you can configure `ClientAliveInterval` on the server-side (`/etc/ssh/sshd_config`) or `ServerAliveInterval` on your client.

    On your Raspberry Pi (server-side):

    ```bash
    sudo nano /etc/ssh/sshd_config
    ```

    Add or modify:

    ```
    ClientAliveInterval 300 # Send a null packet every 300 seconds (5 minutes)
    ClientAliveCountMax 2   # Disconnect after 2 unresponsive intervals
    ```

    Restart SSH service:

    ```bash
    sudo systemctl restart ssh
    ```

-----

## 6\. Testing and Troubleshooting

  * **Reboot:** After making all changes, reboot your Raspberry Pi:

    ```bash
    sudo reboot
    ```

    Obsidian should launch automatically in kiosk mode.

  * **Check Logs:** If Obsidian doesn't launch or crashes, check the systemd journal for your service:

    ```bash
    journalctl -u obsidian-kiosk.service -f
    ```

    The `-f` flag will follow the logs in real-time.

  * **SSH Access:** Ensure you can still SSH into your Raspberry Pi while Obsidian is running. If you chose "Console Autologin" and then `xinit` for the kiosk, SSH should still work normally. If you chose a full desktop autologin and Obsidian overlays everything, SSH will still be functional in the background.

  * **Temporary Disable Kiosk:** If you need to access the desktop for troubleshooting:

    ```bash
    sudo systemctl stop obsidian-kiosk.service
    ```

    Then, if Obsidian was the only thing running on that display, you might need to restart the display manager or graphical session:

    ```bash
    sudo systemctl restart lightdm # Or whatever display manager you use (gdm3, sddm, etc.)
    ```

    To re-enable the kiosk:

    ```bash
    sudo systemctl start obsidian-kiosk.service
    ```

This detailed guide should help you successfully configure your Raspberry Pi to run Obsidian in a robust and persistent kiosk mode.
