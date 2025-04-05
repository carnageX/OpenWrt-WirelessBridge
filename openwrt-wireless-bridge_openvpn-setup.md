# OpenWRT Configuration Guide: Wireless Bridge with OpenVPN

This guide provides step-by-step instructions for setting up a **Wireless Bridge** and **OpenVPN** on an OpenWRT router. It also includes troubleshooting steps to resolve common issues.

---

## **1. Wireless Bridge Configuration**

### **Objective**
Set up the OpenWRT router as a wireless bridge to connect to a host router's SSID (`Skynet`) and broadcast its own SSID (`Skynet_W`). Devices on `Skynet_W` should not have access to devices on `Skynet`.

---

### **Steps**

#### **1.1. Connect to OpenWRT Router**
- Access the OpenWRT router's web interface (LuCI) or SSH into it.

#### **1.2. Configure Wireless Interface to Connect to `Skynet`**
1. Go to **Network > Wireless**.
2. Scan for the `Skynet` SSID and configure the wireless interface to connect to it as a **Client**.
3. Assign this interface to a new network (e.g., `wwan`).

#### **1.3. Create a New Subnet for `Skynet_W`**
1. Go to **Network > Interfaces**.
2. Add a new interface (e.g., `lan_skynet_w`) and assign it a different subnet than the `Skynet` network (e.g., `192.168.2.x`).
3. Disable DHCP on this interface if you want to manually assign IPs or use a separate DHCP server.
4. Set to Static Address
5. Assign to the `Bridge: br-lan` device

#### **1.4. Configure the `Skynet_W` SSID**
1. Go to **Network > Wireless**.
2. Add a new wireless network for the `Skynet_W` SSID.
3. Assign this SSID to the `lan_skynet_w` interface.
4. Set the mode to **Access Point (AP)**.

#### **1.5. Isolate the Networks**
1. Ensure there are no firewall rules or routes that allow traffic between the `wwan` (connected to `Skynet`) and `lan_skynet_w` (broadcasting `Skynet_W`) interfaces.
2. In **Network > Firewall**, create separate zones for `wwan` and `lan_skynet_w` and ensure they are not allowed to communicate with each other.

#### **1.6. Disable Forwarding Between Subnets**
1. In **Network > Firewall**, ensure that forwarding between the `wwan` and `lan_skynet_w` interfaces is disabled.

#### **1.7. Test the Setup**
- Connect a device to the `Skynet_W` SSID and verify it can access the internet (if desired) but cannot access devices on the `Skynet` network.

---

## **Steps to Switch the Host Wireless Network**
Switching the host wireless network (e.g., from `Skynet` to a different network) on your OpenWRT router involves reconfiguring the wireless client interface. Below are the steps to switch the host wireless network:

### **1. Access the OpenWRT Router**
- Log in to the OpenWRT router's web interface (LuCI) or connect via SSH.

---

### **2. Remove the Existing Wireless Client Configuration**
1. Go to **Network > Wireless**.
2. Locate the wireless interface configured as a **Client** (e.g., connected to `Skynet`).
3. Click **Delete** to remove the existing client configuration.

---

### **3. Scan for Available Networks**
1. In the **Network > Wireless** section, click **Scan**.
2. Wait for the scan to complete. A list of available wireless networks will appear.

---

### **4. Configure the New Host Network**
1. From the scan results, select the new host network you want to connect to.
2. Enter the following details:
   - **SSID:** The name of the new host network.
   - **Encryption:** Select the appropriate encryption type (e.g., WPA2-PSK).
   - **Key:** Enter the password for the new host network.
3. Set the **Mode** to **Client**.
4. Assign this interface to the `wwan` network (or the network you previously used for the host connection).
5. Click **Save**.

---

### **5. Verify the Connection**
1. Go to **Network > Interfaces**.
2. Check the `wwan` interface to ensure it has obtained an IP address from the new host network.
3. Test the connection by pinging an external IP (e.g., `8.8.8.8`):
   ```bash
   ping 8.8.8.8
   ```

---

### **6. Update Firewall Rules (if necessary)**
If the new host network uses a different subnet, update the firewall rules to ensure proper traffic routing:
1. Go to **Network > Firewall**.
2. Verify that the `wwan` zone is correctly configured to allow traffic to and from the new host network.

---

### **7. Restart Network Services**
Restart the network and wireless services to apply the changes:
```bash
/etc/init.d/network restart
/etc/init.d/firewall restart
```

---

## **Troubleshooting**

### **1. No Internet Access**
- Ensure the new host network provides internet access.
- Verify the wireless client configuration (SSID, encryption, and password).

### **2. IP Address Not Assigned**
- Check if the new host network’s DHCP server is functioning.
- Manually assign a static IP address to the `wwan` interface if necessary.

### **3. Firewall Blocking Traffic**
- Ensure the firewall allows traffic between the `wwan` interface and the LAN.

---

## **Example: Switching from `Skynet` to `NewNetwork`**
1. Delete the existing client configuration for `Skynet`.
2. Scan for available networks and select `NewNetwork`.
3. Enter the SSID (`NewNetwork`), encryption type, and password.
4. Set the mode to **Client** and assign the interface to `wwan`.
5. Save and apply the changes.
6. Verify the connection and update firewall rules if needed.

---

## **2. OpenVPN Configuration**

### **Objective**
Configure the OpenWRT router to connect to a remote OpenVPN server and route all traffic through the VPN tunnel.

---

### **Steps**

#### **2.1. Install OpenVPN**
1. Install OpenVPN via SSH or the LuCI web interface:
```bash
   opkg update
   opkg install openvpn-openssl
   opkg install luci-app-openvpn
```
#### **2.2. Upload the OpenVPN Configuration File**
Obtain the .ovpn configuration file from your VPN provider.

Upload the .ovpn file to /etc/openvpn/ or create a new file:

```bash
vi /etc/openvpn/client.conf
```
Paste the contents of the .ovpn file and save it.

#### **2.3. Configure Authentication (if required)**
Create an authentication file:

```bash
vi /etc/openvpn/auth.txt
```
Add your username and password:
username
password

Update the .ovpn file to reference this file:

```bash
auth-user-pass /etc/openvpn/auth.txt
```
#### **2.4. Enable and Start OpenVPN**
Enable the OpenVPN service:

```bash
/etc/init.d/openvpn enable
```

Start the OpenVPN service:

```bash
/etc/init.d/openvpn start
```

#### **2.5. Verify the Connection**
Check the OpenVPN logs:

```bash
logread | grep openvpn
```

Verify the VPN interface (tun0) is up:

```bash
ifconfig tun0
```

#### **2.6. Configure Routing**
Add a route for the VPN subnet (if provided by the server):

```bash
ip route add <vpn-subnet> dev tun0
```
Ensure the default route points to tun0:

```bash
ip route show
```

#### **2.7 Setup Custom DNS Servers**
1. Network -> Interfaces -> `lan`
1. Advanced tab, Use Custom DNS Server
1. `8.8.8.8` for Google, `1.1.1.1` for CloudFlare
1. Click Save

#### **2.8 Firewall Setup**
1. Click on Network in the top bar and then on Interfaces to open the interfaces configuration page.
1. Click on button Add new Interface...
1. Fill the form with the following values: Name = OpenVPN, Protocol = Unmanaged, Interface = tun0. Then click on Create Interface.
1. Edit the interface.
1. In panel General Settings: unselect the checkbox Bring up on boot.
1. In panel Firewall Settings: Assign firewall-zone to wan.
1. Click on Save and Apply the new configuration.
1. Reboot the router.

#### **2.8 Enable Killswitch**
The “Network Killswitch” functionality, forces all traffic to go through the VPN. It's a fancy name for what is actually just a firewall rule.
This is best for privacy and security as it will ensure that no traffic can reach the Internet bypassing the VPN you have set up.
This also means that if the VPN connection is terminated, you lose access to the Internet, since no traffic is allowed outside of your VPN.

If you are setting up a Killswitch, it's strongly recommended to set the OpenVPN client to start and connect automatically on boot with the “Enable” checkbox, so that if the router is rebooted you don't lose Internet access (as without a VPN connected you will not be able to access the Internet anymore).

1. Remove the tun interface from `wan` zone in case you have followed the previous step 2.8.
Go to **Network** -→ **Firewall**, click on the Edit button of the `lan` zone.

1. Click on the **Allow forward to destination zones**: menu and deselect the `WAN` zone, then click on Save.

1. Click on Add button under the Zones list to add a new zone, named `OpenVPN`

1. Select Masquerading, MSS Clamping and select the LAN interface in the Allow forward from source zones menu

1. Select the `OpenVPN` interface(s) in the **Covered Networks** menu and then click on Save.

1. Click **Save and Apply**



### **3. Troubleshooting**
#### **3.1. DNS Resolution Issues**
Symptoms: VPN connects but DNS queries fail.

Solution:

Update /etc/resolv.conf with the VPN-provided DNS servers:

```bash
nameserver 10.0.0.241
nameserver 10.0.0.243
```
Use a custom script to update DNS settings:

```bash
#!/bin/sh
case "$script_type" in
    up)
        echo "nameserver 10.0.0.241" > /etc/resolv.conf
        echo "nameserver 10.0.0.243" >> /etc/resolv.conf
        ;;
    down)
        echo "nameserver 8.8.8.8" > /etc/resolv.conf
        ;;
esac
exit 0
```

#### **3.2. Route Conflicts**
Symptoms: Two default routes exist, causing intermittent connectivity.

Solution:

Add the following to your .ovpn file:

```bash
route-nopull
route 0.0.0.0 0.0.0.0 vpn_gateway
route-delay 5
```

Verify the routes:

```bash
ip route show
```

#### **3.3. Firewall Issues**
Symptoms: Traffic is blocked after the VPN connection is established.

Solution:
Add a firewall rule to allow traffic through the VPN interface:

```bash
config rule
    option name 'Allow-VPN-Traffic'
    option src 'vpn'
    option dest '*'
    option target 'ACCEPT'
```
Restart the firewall:

```bash
/etc/init.d/firewall restart
```
#### **3.4. TLS Handshake Errors**
Symptoms: VPN connection fails with TLS handshake errors.

Solution:
Ensure the VPN server is reachable:

```bash
ping <vpn-server-ip>
```
Verify the .ovpn file is correctly configured and does not conflict with the server’s pushed options.

### **4. Final Notes**
Ensure the router has a stable internet connection before configuring the VPN.

Use static IP addresses for the VPN server to avoid DNS resolution issues.

Disable IPv6 if it is not required to avoid conflicts.
