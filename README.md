# Configuring Xavier/Orin NX/AGX Boards for PSDK

> [!IMPORTANT]  
> **Important:** The initial setup steps are done **only once per board**. <br>
> After that, follow the per-drone steps **every time the board boots up**.

---

## One-Time Setup (Board Configuration)

### 1. Disable `l4t-device-mode` auto start
```bash
sudo systemctl disable nv-l4t-usb-device-mode.service
```

### 2. Download Bulk Mode Program Folder

Download and extract the configuration package to your Desktop as `startup_bulk`:
```bash
wget https://terra-1-g.djicdn.com/71a7d383e71a4fb8887a310eb746b47f/psdk/e-port/usb-bulk-configuration-reference.zip \
&& unzip usb-bulk-configuration-reference.zip -d startup_bulk \
&& mv startup_bulk/ ~/Desktop/
```

### 3. Replace System Device Mode Script

Replace the default script with your custom one:
```bash
sudo cp ./nv-l4t-usb-device-mode-start.sh /opt/nvidia/l4t-usb-device-mode/nv-l4t-usb-device-mode-start.sh
chmod +x ./nv-l4t-usb-device-mode-start.sh
```

### 4. Reboot
```bash
sudo reboot
```

### 5. Load Kernel Modules on Boot

Append the required modules:
```bash
echo -e "configfs\nlibcomposite\nusb_f_fs\ntegra-xudc" | sudo tee -a /etc/modules
```
✅ Double-check that `/etc/modules` is correctly formatted.

### 6. Test the Setup

Run the startup script:
```bash
/opt/nvidia/l4t-usb-device-mode/nv-l4t-usb-device-mode-start.sh
```

### 7. Re-enable the Service (if working)
```bash
sudo systemctl enable /opt/nvidia/l4t-usb-device-mode/nv-l4t-usb-device-mode.service
```

At this point, **bulk mode and network mode should be properly configured.**

----
----

## Per-Drone Setup (Run at Every Boot)
### DJI M300

#### Hardware
- Onboard computer = **host**
- E-Port = **device**
- Connections:

| ![E-Port to device mode](images/device_mode.jpg) | ![Agx connections](images/agx_connections.jpg) | 
|:--:|:--:|
| *E-Port to device mode* | *AGX connections* |


> [!NOTE]  
> Make sure the USB Type-C cable is connected to the correct port (the one supporting both bulk and host mode).

#### Software

Enable host mode:
```bash
echo host | sudo tee /sys/class/usb_role/usb2-0-role-switch/role
```
👉 You should see `/dev/ttyACM0` when powering on the drone.

### DJI M350
#### Hardware
- Onboard computer = **device**
- E-Port = **host**
- Connections:

| ![E-Port to host mode](images/host_mode.jpg) | ![Agx connections](images/agx_connections.jpg) | 
|:--:|:--:|
| *E-Port to host mode* | *AGX connections* |

#### Software
Enable device mode:
```bash
echo device | sudo tee /sys/class/usb_role/usb2-0-role-switch/role
```

Lift `l4tbr0` connection:
```bash
sudo ifconfig usb0 192.168.1.1 netmask 255.255.255.0 up
```

👉 Verify with:
```bash
ifconfig
```
You should see an `l4tbr0` entry. If not, repeat steps above.

### DJI M30/M30T
#### Hardware
- Onboard computer = **device**
- E-Port = **host**
- Connections:

| ![E-Port to host mode](images/host_mode.jpg) | ![Nx connections](images/nx_connections.jpg) | 
|:--:|:--:|
| *E-Port to host mode* | *NX connections* |

#### Software
Enable device mode:
```bash
echo device | sudo tee /sys/class/usb_role/usb2-0-role-switch/role
```

Lift `l4tbr0` connection:
```bash
sudo ifconfig usb0 192.168.1.1 netmask 255.255.255.0 up
```

👉 Verify with:
```bash
ifconfig
```
You should see an `l4tbr0` entry. If not, repeat steps above.

-----

## Summary Table
| Drone    | Onboard Computer Role | E-Port Role | Hardware Connection Notes                                  | Software Steps                                                                                                                                       |
| -------- | --------------------- | ----------- | ---------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| **M300** | Host                  | Device      | Connect AGX as host via E-Port in device mode. | `echo host \| sudo tee /sys/class/usb_role/usb2-0-role-switch/role` <br> - Check `/dev/ttyACM0`.                                                        |
| **M350** | Device                | Host        | Connect AGX as device via E-Port in host mode. | `echo device \| sudo tee /sys/class/usb_role/usb2-0-role-switch/role` <br> `sudo ifconfig usb0 192.168.1.1 netmask 255.255.255.0 up` <br> - Check `l4tbr0`. |
| **M30**  | Device                | Host        | Connect NX as device via E-Port in host mode.  | `echo device \| sudo tee /sys/class/usb_role/usb2-0-role-switch/role` <br> `sudo ifconfig usb0 192.168.1.1 netmask 255.255.255.0 up` <br> - Check `l4tbr0`. |

> [!NOTE]  
> Use correct USB-C port on AGX (supports both bulk & host).
