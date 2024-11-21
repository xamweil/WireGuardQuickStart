# Wireguard Set-Up

## Installation

Installation guid is for Linux.

Install [WireGuard](https://www.wireguard.com/install/) and necessary toosl:
```bash
sudo apt install wireguard wireguard-tools -y
```

## Set-Up VPN Server
If not automatically created, create folder to store keys and configuration file, with appropriate permission.
```bash
mkdir /etc/wireguard
sudo chmod 700 /etc/wireguard
```
### Generate keys
Generate the public key for the server. You might need root access for that: ```sudo -i```.
```bash
sudo wg genkey | sudo tee /etc/wireguard/server_private.key
```

From the private key generate the public key:
```bash 
sudo cat /etc/wireguard/server_private.key | wg pubkey | sudo tee /etc/wireguard/server_public.key
```

### Configurate the server
Open the configuration file:
```bash
sudo nano /etc/wireguard/wg0.conf
```

Add the following content to the file:
```bash
[Interface]
Adress 10.0.0.1/24
ListenPort = 51820      # standard for WireGuard
PrivateKey = <Add server_private.key>

#Enable IP forwarding and add NAT
PostUP = sysctl -w net.ipv4.ip_forward=1; iptables -t nar -A POSTROUTING -s 10.0.0./24 -o <your network interface> -j MASQUERADE
PostDown = sysctl -w net.ipv4.ip_forward=0; -t nat -D POSTROUTING -s 10.0.0.0/24 -0 <your network interface> -j MASQUERADE
```
- **<your network interface\>:** Replace with the name of the network adapter (something like wlan0 or eth0). Can be found by typing ```ip addr``` in the terminal. 

### Enable IP Forwarding
To make communication with other devices on the network possible IP forwarding mus be enables. To do this permanently add the following line to ```/etc/sysctl.conf```:
```bash
net.ipv4.ip_forward = 1
```
### Note:
To start the client one can run:
```bash
sudo wg-quick up wg0
```

To shut the client down:
```bash
sudo wg-quick down wg0
```

To start WireGuard on boot:
```bash
sudo systemctl enable wg-quick@wg0
```

## Enable Port Forwarding on Router
Log in to your router by typing the router IP in any browser.
You can find the router IP by typing '''ip route''' in the terminal.
Log in and search for something like **Port Forwarding**, **Virtual Server** or **Appplications** under **Advanced** or **Network** settings

### Note:
- Use IPv4
- Make sure something like 'Activate in savings' is enabled, so that it will be active when the router reboots.
- Internet Port Number: 51820
-Internet IP Adress: leave blank or 0.0.0.0
- Protocol: UDP
- Device: Choose the device that runs WireGuard.
- Destination IP adress: IP adress of the device running WireGuard
Destination Port number: 51820
- **Don't forget to make your systems IP static**
## Add New Device
Activate root access. 

```bash
sudo -i
```
Go into your wireguard root. (e.g. ```cd etc/wireguard```)

### Generate the key pair for the new device.

For the private key:
```bash
wg keygen| tee device_private.key
```

From the private key generate the public key:

```bash
cat device_private.key | wg pubkey | tee device_public.key 
```

Open ```sudo nano etc/wireguard/wg0.conf```

### Add a new ```Device```.
Every peer/device needs a unique IP of the form ```10.0.0.x/32```.

**Add PEER to wg0.conf file.**

```bash
[PEER]
Publickey = <Add devices public Key>
AllowedIPs = 10.0.0.2/32
```

Restart wg by running:

```bash
sudo wg-quick down wg0 
sudo wg-quick up wg0
```

### Create a configuration file to export to the device.

Exit root access, type: ``èxit``

Choose a location and create a file:
```bash
sudo nano device.conf
```
Add the following content to the file:
```bash
[Interface]
PrivateKey = <Add private key of device>
Address = 10.0.0.x/32
DNS = 8.8.8.8

[PEER]
PublicKey = <Add public key of the VPN>
Endpoint = <Add Global IP of your router>
AllowedIPs = 0.0.0.0/0
```

- **<PrivateKey\>:** Is the generated key for the device.
- **<Endpoint\>:** The global IP can be found by typing something like      "What is my IP" into any browser. The correct IP is under IPv4.
- **<PublicKey\>:** Is stored in under ```etc/wireguard/server_public.key```

### Download [WireGuard](https://www.wireguard.com/install/) programm/app for your Os.
Import the device.conf file. To test if the connection was successfull run:
```bash
sudo wg 
```
On the VPN host. This shows the PEER's and the last handshake.
