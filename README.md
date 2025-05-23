# DIY Mobile Bonding Router with Raspberry Pi and OpenMPTCProuter

## Introduction

This project outlines the steps to create a Do-It-Yourself (DIY) mobile internet bonding router using a Raspberry Pi and multiple LTE USB modems. The goal is to aggregate the bandwidth of several mobile connections to achieve higher speeds and better reliability in locations with poor individual carrier performance. This setup serves as a self-hosted alternative to commercial services like Speedify, by leveraging your own home server to bundle the connections.

The solution is based on [OpenMPTCProuter](https://www.openmptcprouter.com/), a specialized OpenWrt-based distribution designed to aggregate multiple internet connections using MultiPath TCP (MPTCP) or other tunneling technologies. It provides a robust platform for both the mobile router (client) and the aggregation server.

## System Architecture

The system consists of two main components:

1.  **Mobile Router (Client):**
    *   A Raspberry Pi (RPi 4 or 5 recommended) equipped with multiple (e.g., 4) USB LTE modems.
    *   Runs the OpenMPTCProuter firmware.
    *   Connects to the internet via the available LTE networks.
    *   Establishes an encrypted tunnel (using technologies like Shadowsocks or Glorytun, managed by OpenMPTCProuter) to the home server.
    *   Provides Wi-Fi and/or Ethernet connectivity for client devices (laptops, phones, etc.).
    *   Devices connect to the Raspberry Pi (via Wi-Fi or Ethernet) to access the bonded internet connection.

2.  **Aggregation Server (Home Server/VPS):**
    *   A server located at your home (or a cloud VPS) with a stable internet connection and a public IP address.
    *   Runs a compatible Linux distribution (Debian 10/11/12 recommended).
    *   The OpenMPTCProuter VPS script is installed on this server, which configures it to:
        *   Receive encrypted traffic from the Raspberry Pi.
        *   Aggregate the data from the multiple tunnels.
        *   Route the aggregated traffic to the internet.
        *   Send return traffic back to the Raspberry Pi through the tunnels.

All internet traffic from devices connected to the Raspberry Pi will be routed through this encrypted, bonded tunnel to the home server, and then out to the internet, effectively combining the bandwidth of the multiple LTE connections.

## Prerequisites

### Hardware

**1. Mobile Router (Client - Raspberry Pi Setup):**

*   **Raspberry Pi:**
    *   **Model:** Raspberry Pi 4 Model B or Raspberry Pi 5 recommended for better performance, especially with multiple LTE modems and routing. Older models like RPi 3B+ might work but could be a bottleneck.
    *   **SD Card:** A good quality microSD card, at least 16GB (32GB recommended), Class 10 or A1 rated for speed.
    *   **Power Supply:** Official Raspberry Pi power supply for your RPi model, or a high-quality USB-C power supply capable of delivering sufficient power (e.g., 3A for RPi 4, 5A for RPi 5). This is crucial when multiple USB modems are attached.
    *   **LTE USB Modems:** Typically 4 for this setup.
        *   Ensure these are compatible with Linux and ideally with OpenWrt/ModemManager. Common brands like Huawei, ZTE, or Quectel often have good support. USB dongles that present as Ethernet interfaces (CDC Ethernet, NCM, MBIM) are often easier to configure.
        *   Check their LTE band compatibility with your chosen mobile carriers.
        *   You will need SIM cards with active data plans for each modem.
        *   Consider models with external antenna ports if you anticipate being in very weak signal areas.
    *   **Powered USB Hub (Recommended):** To provide adequate power to multiple LTE modems. Some modems can draw significant power, and the Raspberry Pi's USB ports might not be sufficient for all of them simultaneously.
    *   **Ethernet Cable:** For initial setup and connection to your LAN if using the Pi as a Wi-Fi access point.
    *   **Optional:**
        *   Raspberry Pi Case: To protect the device.
        *   Cooling (Heatsinks/Fan): Especially if running in a warm environment or under heavy load.

**2. Aggregation Server (Home Server/VPS):**

*   **Machine:** A dedicated physical machine at home, or a Virtual Private Server (VPS) in the cloud.
    *   **CPU:** x86_64 architecture.
    *   **RAM:** Minimum 1GB RAM (2GB or more recommended for smoother operation).
    *   **Storage:** At least 10-20GB of disk space.
*   **Operating System:** A minimal installation of Debian 10 (Buster), Debian 11 (Bullseye), or Debian 12 (Bookworm) is recommended. The OpenMPTCProuter VPS script also supports compatible Ubuntu LTS versions (e.g., 20.04, 22.04).
*   **Internet Connection:**
    *   A stable and reasonably fast broadband internet connection.
    *   A **static public IP address** is ideal. If you have a dynamic IP, you will need to set up Dynamic DNS (DDNS) service (e.g., No-IP, DuckDNS) on your home server or router, and use the DDNS hostname in the OpenMPTCProuter client configuration on the Pi.
*   **Port Forwarding Capability:** You must have administrative access to your home internet router (the one providing internet to your server) to configure port forwarding. This is essential to allow the Raspberry Pi to connect to your home server from the internet.

### Software

*   **OpenMPTCProuter Raspberry Pi Image:** Download the appropriate precompiled image for your Raspberry Pi model from the [OpenMPTCProuter website](https://www.openmptcprouter.com/download). Look for the "factory" image for initial installation.
*   **OpenMPTCProuter VPS Script:** This script will be downloaded directly to your home server during its setup process. The command is provided in the server setup section.
*   **SD Card Flashing Software:**
    *   [Raspberry Pi Imager](https://www.raspberrypi.com/software/) (Recommended, available for Windows, macOS, Ubuntu)
    *   [BalenaEtcher](https://www.balena.io/etcher/) (Windows, macOS, Linux)
    *   `dd` command-line utility (Linux, macOS)
*   **SSH Client:**
    *   PuTTY (Windows)
    *   Terminal (built-in on macOS and Linux)
    *   Windows Subsystem for Linux (WSL) or PowerShell's SSH client on newer Windows versions.

## Raspberry Pi Setup (Client)

This section details the steps to configure your Raspberry Pi as the mobile bonding router using OpenMPTCProuter.

1.  **Download OpenMPTCProuter Image:**
    *   Go to the [OpenMPTCProuter Downloads page](https://www.openmptcprouter.com/download).
    *   Download the "factory" image (`.img.gz` or similar) specifically for your Raspberry Pi model (e.g., RPi 4/5).

2.  **Flash the Image to SD Card:**
    *   Use your chosen SD card flashing software (Raspberry Pi Imager, BalenaEtcher, etc.).
    *   Select the downloaded OpenMPTCProuter image file.
    *   Select your microSD card as the target.
    *   Write the image to the card. This will erase all existing data on the card.
    *   Example using Raspberry Pi Imager:
        *   Choose OS -> Use custom -> Select your downloaded `.img.gz` file.
        *   Choose Storage -> Select your SD card.
        *   Click Write.

3.  **Boot the Raspberry Pi:**
    *   Safely eject the microSD card from your computer and insert it into the Raspberry Pi.
    *   Connect your LTE modems to the Raspberry Pi (preferably via a powered USB hub, if you have one).
    *   Connect an Ethernet cable from the Raspberry Pi's Ethernet port to your computer or your local network switch. (This is for initial configuration; later you might configure Wi-Fi).
    *   Power on the Raspberry Pi. Allow a few minutes for the first boot to complete.

4.  **Initial Connection & Accessing Web Interface:**
    *   The Raspberry Pi (OpenMPTCProuter) will create a new network. By default, its IP address is `192.168.100.1`.
    *   If you connected the Pi directly to your computer via Ethernet, ensure your computer's network settings for that Ethernet adapter are configured to obtain an IP address automatically (DHCP). Your computer should receive an IP in the `192.168.100.x` range.
    *   If you connected the Pi to your network switch, your main router might assign an IP from its own DHCP range, or you might need to temporarily set a static IP on your computer in the `192.168.100.x` range (e.g., `192.168.100.10`, subnet mask `255.255.255.0`) to access the Pi if your main network is on a different subnet.
    *   Open a web browser on your computer and navigate to `http://192.168.100.1`.
    *   You should see the OpenWrt LuCI login page. The default username is `root` and there is **no password** by default. It is highly recommended to set a root password immediately via `System -> Administration`.

5.  **Basic OpenWrt/OpenMPTCProuter Configuration:**
    *   **Set Root Password:** Navigate to `System -> Administration`. Enter a strong password for the `root` user and save.
    *   **Network Interfaces (LTE Modems):**
        *   This is the most crucial part for your setup. Your LTE USB modems should appear as network interfaces.
        *   You can use commands like `lsusb` to list connected USB devices and `dmesg` to check kernel messages for modem detection information if you encounter issues.
        *   Go to `Network -> Interfaces`. You should see your LAN interface. Your LTE modems might appear as `WAN`, `WAN1`, `WAN2`, etc., or they might need to be configured.
        *   If modems are not automatically configured, you may need to:
            *   Install modem-related packages if missing: `System -> Software`. Search for packages like `modemmanager`, `luci-proto-modemmanager`, `luci-proto-ncm`, `luci-proto-mbim`, or specific drivers for your modems if known.
            *   Add new interfaces: Click "Add new interface..."
                *   Name it (e.g., `LTE1`, `LTE2`).
                *   Protocol: This depends on your modem. Common choices are `ModemManager`, `QMI Cellular`, `NCM`, `MBIM`. You might need to try a few or consult your modem's documentation.
                *   Firewall Zone: Assign these WAN interfaces to the `wan` firewall zone.
                *   Enter APN details if required by your mobile carrier (usually under the interface settings).
        *   Refer to the OpenWrt and OpenMPTCProuter documentation for more detailed guidance on modem setup, as this can vary widely.
    *   **Wi-Fi Setup (Optional):** If you want the Raspberry Pi to act as a Wi-Fi access point for your devices:
        *   Go to `Network -> Wireless`.
        *   You should see your Pi's built-in Wi-Fi radio(s). Click "Edit".
        *   Under "Interface Configuration", in the "Network" dropdown, make sure it's assigned to `lan`.
        *   Under "Wireless Security", configure your Wi-Fi network name (SSID) and password (WPA2/WPA3).
        *   Click "Save" and then "Enable" for the Wi-Fi interface.

6.  **Configure OpenMPTCProuter Client Settings:**
    *   Once your WAN interfaces (LTE modems) are recognized and have internet connectivity (you can test this individually if OpenMPTCProuter allows), you need to configure the connection to your home server.
    *   Navigate to `Services -> OpenMPTCProuter`.
    *   This is where you will enter the details obtained from your home server setup (from the `/root/openmptcprouter_config.txt` file on the server). This typically includes:
        *   Server IP address or DDNS hostname.
        *   The specific port numbers for the chosen tunnel technology (e.g., Shadowsocks port, Glorytun port).
        *   The pre-shared keys or passwords for the tunnel.
    *   Enable the connection to the server.
    *   Assign your configured LTE WAN interfaces to be used by OpenMPTCProuter.
    *   Save and apply the settings. The system will attempt to establish the connection to your server.

For detailed configurations and troubleshooting, always refer to the official [OpenMPTCProuter Wiki](https://github.com/Ysurac/openmptcprouter/wiki) and the [OpenWrt documentation](https://openwrt.org/docs/start).

## Home Server (VPS) Setup

This section covers the configuration of your home server, which will act as the aggregation point for your mobile connections. While the term "VPS" is used in OpenMPTCProuter documentation, this can be your own server at home.

1.  **Operating System & Initial Preparation:**
    *   Ensure your server is running a minimal installation of **Debian 10 (Buster), Debian 11 (Bullseye), or Debian 12 (Bookworm) x86_64**. Compatible Ubuntu LTS versions (e.g., 20.04, 22.04) are also supported. The script will attempt to upgrade your OS to Debian 12 if a compatible older version is detected.
    *   Log in to your server via SSH.
    *   Update your server's package list and upgrade existing packages:
        ```bash
        sudo apt-get update && sudo apt-get upgrade -y
        ```
    *   If you intend to use IPv6, ensure it is configured and working on your server *before* running the installation script.

2.  **Run the OpenMPTCProuter VPS Installation Script:**
    *   Execute the following command as **root** (e.g., log in as root or use `sudo -i`):
        ```bash
        wget -O - https://www.openmptcprouter.com/server/debian-x86_64.sh | KERNEL="6.1+" sh
        ```
        *   **Note on KERNEL version:** `KERNEL="6.1+"` will install a recent kernel (e.g., 6.1.X). Always check the [OpenMPTCProuter VPS install documentation](https://github.com/Ysurac/openmptcprouter/wiki/Install-or-update-the-VPS) for the latest recommended kernel version that matches your router's OpenMPTCProuter image version, as this can change. Specific kernel versions (e.g., `KERNEL="6.6"`, `KERNEL="5.15"`) might be required for optimal compatibility and performance with certain router image releases.
        *   If `wget` fails due to certificate issues, you might need to install `ca-certificates`:
            ```bash
            sudo apt-get install -y ca-certificates
            ```
            Then try the `wget` command again.
    *   The script will install and configure the MPTCP kernel, various tunneling software (Shadowsocks, Glorytun, V2Ray, XRay), a firewall (Shorewall), and other necessary services. It will also change the default SSH port.

3.  **Note Critical Configuration Details:**
    *   During the installation, the script will display important information, including generated keys and port numbers.
    *   After the script completes, these details are saved in `/root/openmptcprouter_config.txt` on the server. **This file is very important.**
    *   Key information you will need for configuring the Raspberry Pi client includes:
        *   Shadowsocks port and password/key.
        *   Glorytun port and key (if you choose to use it).
        *   V2Ray/XRay details (if used).
        *   The new SSH port for server access (default: `65222`).
    *   Make a secure copy of this file or its contents.

4.  **Reboot the Server:**
    *   After the script finishes, you **MUST** reboot the server for the new kernel and changes to take effect:
        ```bash
        sudo reboot
        ```

5.  **Crucial: Configure Port Forwarding on Your Home Internet Router:**
    *   This is a critical step for the Raspberry Pi (out on the internet using LTE) to be able to connect to your home server.
    *   Log in to your main home internet router's administration interface (the one that provides internet to your home server).
    *   Find the "Port Forwarding," "NAT Forwarding," or similar section.
    *   You need to forward the ports used by OpenMPTCProuter to the **internal IP address** of your home server (the one it has on your local network).
    *   **Ports to forward (TCP & UDP for all unless specified):**
        *   **SSH:** The new SSH port (default `65222` TCP).
        *   **OpenMPTCProuter services:** The script uses a range of ports. For simplicity, you can forward the entire range `65000-65535` for both TCP and UDP.
        *   Alternatively, forward specific ports mentioned in `/root/openmptcprouter_config.txt` or the OpenMPTCProuter documentation, such as:
            *   Shadowsocks port (e.g., `65101` TCP & UDP).
            *   Glorytun UDP port (e.g., `65001` UDP).
            *   OpenVPN port (e.g., `65301` TCP or UDP, depending on configuration).
            *   WireGuard port (e.g., `65311`, `65312` UDP).
            *   V2Ray/XRay ports.
        *   **ICMP (Ping):** Ensure your router is not blocking ICMP (ping) requests to your server's public IP, as OpenMPTCProuter uses it for connectivity checks.
    *   Consult your router's manual for specific instructions on how to set up port forwarding.
    *   **If your home server has a dynamic public IP:** Remember to have DDNS configured so your Raspberry Pi can find it using a hostname.

Once these steps are complete, your home server should be ready to accept connections from your OpenMPTCProuter on the Raspberry Pi.

## Integration and Basic OpenMPTCProuter Configuration (on Raspberry Pi)

After setting up both the Raspberry Pi and the Home Server, you need to configure the OpenMPTCProuter instance on the Pi to connect to the server and manage your LTE connections.

1.  **Access OpenMPTCProuter Web UI:**
    *   Ensure your computer is connected to the Raspberry Pi's LAN network (either via Ethernet or Wi-Fi if you configured it).
    *   Open a web browser and navigate to `http://192.168.100.1`.
    *   Log in with username `root` and the password you set earlier.

2.  **Configure Server Connection:**
    *   Navigate to `Services -> OpenMPTCProuter`.
    *   This page is the main control panel for your OpenMPTCProuter client.
    *   You will need the information from the `/root/openmptcprouter_config.txt` file that was generated on your **Home Server**.
    *   **Enable OpenMPTCProuter:** Ensure the "Enable OpenMPTCProuter" checkbox is ticked at the top.
    *   **Server Configuration:**
        *   You'll likely see a default server entry or an option to add/edit one.
        *   **Server Address:** Enter your Home Server's public IP address or its DDNS hostname.
        *   **Connection Type/VPN Protocol:** Select the protocol you intend to use (e.g., Shadowsocks, Glorytun UDP). This should match what was configured by the server script. Shadowsocks is often a good default.
        *   **Port:** Enter the corresponding port for the selected protocol (e.g., `65101` for Shadowsocks).
        *   **Key/Password:** Enter the generated key or password for the selected protocol.
        *   Other settings might be available depending on the protocol (e.g., encryption cipher for Shadowsocks). Usually, the defaults provided by the server script are suitable.
    *   Click "Save & Apply".

3.  **Manage WAN Interfaces (LTE Modems):**
    *   Still under `Services -> OpenMPTCProuter`, look for a section related to interfaces or WAN management.
    *   Your configured LTE modem interfaces (e.g., `LTE1`, `LTE2`, `wwan0`, `eth1`, etc., depending on how they were detected and named in `Network -> Interfaces`) should be listed here.
    *   **Enable Interfaces:** Ensure that each LTE WAN interface you want to use in the bond is enabled.
    *   **Priority/Tier:** You can often set priorities or tiers for interfaces. Interfaces with lower numbers (e.g., Tier 1) are typically preferred. You might set all your LTE modems to the same tier to actively use them all.
    *   **Multipath Strategy:** OpenMPTCProuter allows different MPTCP schedulers or bonding strategies. The default is often fine to start with. You can explore options like "Default," "Redundant," or "ECF" (Equal Cost Fanout) if available and you understand their implications.
    *   Click "Save & Apply" after making changes.

4.  **Check Status and Connectivity:**
    *   **OpenMPTCProuter Status Page:** Navigate to `Status -> OpenMPTCProuter`.
        *   This page provides a dashboard showing the status of the connection to the server, the individual WAN interfaces, their current usage, and the overall bonded connection speed.
        *   Check if the "Master Interface" (the tunnel to your server) is connected.
        *   Verify that your individual LTE WANs are showing as "UP" and have IP addresses.
    *   **System Log:** `Status -> System Log` can provide detailed messages if you are troubleshooting connection issues.
    *   **Interfaces Overview:** `Network -> Interfaces` will show you the status of each individual modem interface (IP address, RX/TX traffic).

5.  **Testing the Bonded Connection:**
    *   Connect a device (laptop, phone) to the Raspberry Pi's LAN network (Ethernet or Wi-Fi).
    *   Open a web browser on that device and try accessing various websites.
    *   Perform an internet speed test (e.g., `speedtest.net`, `fast.com`). The speed should ideally be higher than any single LTE connection, though overhead from the tunneling process means it won't be a perfect sum of all connections.
    *   Check your public IP address (e.g., by searching "what is my IP"). It should be the public IP address of your **Home Server**, not the IP of any individual LTE modem.

This provides a basic setup. The OpenMPTCProuter interface offers many more advanced settings for tweaking performance, failover behavior, and more, which you can explore in the official documentation.

## Testing Your Setup

Once everything is configured, perform these checks:

1.  **Verify Public IP Address:**
    *   Connect a device to your Raspberry Pi's network (Wi-Fi or LAN).
    *   On this device, open a web browser and search for "what is my IP address".
    *   The displayed IP address should be the public IP address of your **Home Server**, not the IP of any of your individual LTE modems. This confirms traffic is being routed through the server.

2.  **Speed Tests:**
    *   Run speed tests using services like [Speedtest.net](https://www.speedtest.net) or [Fast.com](https://fast.com).
    *   The aggregated speed should ideally be higher than any single LTE connection. However, expect some overhead (typically 10-20%) due to encryption and tunneling.
    *   Test with multiple simultaneous downloads to see how the load is distributed.

3.  **Failover Testing (Optional but Recommended):**
    *   While running a continuous ping or a download, try disconnecting one of the LTE modems from the Raspberry Pi.
    *   The connection should remain active, possibly with a brief interruption or speed reduction, as traffic fails over to the remaining active connections.
    *   Reconnect the modem and observe if it rejoins the bond.

4.  **Check OpenMPTCProuter Status Page:**
    *   In the Raspberry Pi's OpenMPTCProuter web UI (`Status -> OpenMPTCProuter`), monitor the traffic graphs for each WAN interface and the master tunnel. You should see traffic flowing over multiple WANs when the bonded connection is heavily used.

## Troubleshooting Common Issues

*   **LTE Modem Not Detected/Not Connecting:**
    *   **Power:** Ensure the modem has enough power. Use a powered USB hub if unsure.
    *   **Compatibility:** Verify the modem is compatible with OpenWrt/Linux. Search OpenWrt forums for your modem model.
    *   **SIM Card:** Check if the SIM card is inserted correctly, active, and has a data plan. Test the SIM in a phone.
    *   **APN Settings:** Ensure the correct APN for your carrier is configured in the interface settings (`Network -> Interfaces -> Edit LTE_Interface`).
    *   **Firmware/Driver:** The modem might require specific firmware or drivers. Check `System -> Software` in OpenWrt to see if additional packages like `modemmanager`, `luci-proto-modemmanager`, or protocol-specific packages (`luci-proto-ncm`, `luci-proto-mbim`, `luci-proto-qmi`) are needed and installed.
    *   **USB Port/Cable:** Try a different USB port or cable.
    *   **System Log:** Check `Status -> System Log` for error messages related to the modem.

*   **Cannot Connect to Home Server:**
    *   **Server IP/Hostname:** Double-check the server IP address or DDNS hostname in the Pi's OpenMPTCProuter settings.
    *   **Port Forwarding:** This is the most common culprit. Verify that all necessary ports (especially the tunnel port like Shadowsocks `65101` and SSH `65222`) are correctly forwarded on your **home internet router** to the internal IP of your home server. Test port forwarding using an online port checker tool from a device outside your home network.
    *   **Firewall on Server:** Ensure the firewall on the home server (Shorewall, installed by the script) is not blocking incoming connections on the required ports. The script should configure this correctly, but it's worth checking (`sudo shorewall status`).
    *   **Firewall on Home Router:** Ensure your main home router's firewall isn't blocking the forwarded ports.
    *   **Server Script Errors:** Review the output from the OpenMPTCProuter VPS script execution for any errors.
    *   **Keys/Passwords:** Ensure the keys/passwords match exactly between the server configuration (`/root/openmptcprouter_config.txt`) and the Pi's client settings.
    *   **Server Reachability:** Ping your server's public IP/hostname from an external network to ensure it's online.

*   **Slow Speeds/Poor Performance:**
    *   **Signal Strength:** Check the signal strength of your LTE modems. Position them for the best reception (e.g., near windows, using external antennas if supported).
    *   **LTE Band Congestion:** Some LTE bands might be more congested than others. Some modems allow band locking.
    *   **Server's Internet Connection:** The bonded speed will be limited by your home server's upload and download speed.
    *   **Server Load:** Check CPU and memory usage on both the Raspberry Pi and the home server. The RPi, especially older models, can be a bottleneck.
    *   **MTU Issues:** Incorrect MTU settings can sometimes cause performance problems. OpenMPTCProuter usually handles this, but advanced users might investigate.
    *   **Tunnel Overhead:** Some performance overhead is normal.

*   **Instability:**
    *   **Power Supply:** Insufficient power to the Raspberry Pi or modems can cause instability.
    *   **Overheating:** Ensure the Raspberry Pi has adequate cooling.
    *   **Software Bugs:** Keep OpenMPTCProuter updated (both Pi and server script) as new versions may contain bug fixes. However, remember the advice to backup configs before updates.

## Important Notes

*   **Security:**
    *   By opening ports on your home router and running a server, you are increasing your network's exposure. Ensure your home server is kept updated with security patches.
    *   Use strong passwords for your Raspberry Pi, your home server, and any services.
    *   Regularly check for updates for OpenMPTCProuter (both the Pi image and the VPS script from their official sources) and apply them after backing up your configurations, as updates may include security patches.
    *   Consider additional security measures for your home server if you are comfortable (e.g., Fail2Ban, IDS/IPS).
*   **Data Usage:**
    *   Bonding multiple LTE connections means you will be using data from multiple plans simultaneously. Monitor your data usage to avoid overage charges.
*   **Dynamic DNS (DDNS):**
    *   If your home server's public IP address is dynamic (changes periodically), you **must** use a DDNS service. Configure the DDNS client on your home server or on your main internet router. Use the DDNS hostname in the Raspberry Pi's OpenMPTCProuter server settings.
*   **LTE Modem Antennas:**
    *   For better signal quality, consider using external antennas with your LTE modems if they support them.
*   **Official Documentation:**
    *   This README provides a setup guide based on available information. Always refer to the official [OpenMPTCProuter Wiki](https://github.com/Ysurac/openmptcprouter/wiki) and the [OpenWrt Documentation](https://openwrt.org/docs/start) for the most current, detailed, and advanced configurations. The project's GitHub issues page can also be a valuable resource for troubleshooting.
*   **Experimentation:**
    *   Different combinations of LTE networks, modem settings, and OpenMPTCProuter configurations might yield varying results. Some experimentation might be needed to find the optimal setup for your specific environment.