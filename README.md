````markdown
# Raspberry Pi 5G with Quectel RM520N-GL PCIe HAT+

Enable 5G internet on Raspberry Pi 5 using the Quectel RM520N-GL PCIe to 5G HAT+. This README walks you through hardware requirements, setup steps, PCIe enablement, DIP-switch control, kernel update, and internet dial-up.

---

## 📦 Hardware Requirements

- **Raspberry Pi 5**  
  [Buy on Raspberry Pi Store](https://www.raspberrypi.com/products/raspberry-pi-5/)  
- **USB Keyboard & Mouse**  
  [Buy on Amazon India](https://www.amazon.in/s?k=usb+keyboard+mouse)  
- **HDMI Monitor**  
  [Buy on Amazon India](https://www.amazon.in/s?k=hdmi+monitor)  
- **Boot-ready SD Card (16 GB or more)**  
  [Buy on Amazon India](https://www.amazon.in/s?k=raspberry+pi+boot+sd+card)  
- **Quectel RM520N-GL PCIe to 5G HAT+**  
  [Buy from Waveshare](https://www.waveshare.com/pcie-to-5g-hat-plus.htm)  

---

## 📋 Table of Contents

1. [Install Raspberry Pi OS](#install-raspberry-pi-os)  
2. [Enable PCIe Support](#enable-pcie-support)  
3. [DIP Switch Control](#dip-switch-control)  
4. [Reset / Power On-Off Module with Code](#reset--power-on-off-module-with-code)  
5. [Update Kernel](#update-kernel)  
6. [Verify Kernel & PCIe Detection](#verify-kernel--pcie-detection)  
7. [Dial-up to the Internet](#dial-up-to-the-internet)  
8. [Power Monitoring](#power-monitoring)  
9. [Dial-up Test](#dial-up-test)  

---

## 1. Install Raspberry Pi OS

1. Download **Raspberry Pi Imager** from the [official site](https://www.raspberrypi.com/software/).  
2. Flash **Raspberry Pi OS (64-bit)** onto your SD card.  
3. Insert the SD card, connect keyboard, mouse, monitor, and power on the Pi.  
4. Complete first-boot setup (locale, Wi-Fi, password).

---

## 2. Enable PCIe Support

1. Open `config.txt`:

    ```bash
    sudo nano /boot/firmware/config.txt
    ```

2. Add at the end:

    ```ini
    dtparam=pciex1
    dtoverlay=pciex1-debug
    ```

3. Save & reboot:

    ```bash
    sudo reboot
    ```

---

## 3. DIP Switch Control

The RM520N-GL HAT has two DIP switches on GPIO5 (RST) and GPIO6 (PWR).  
- **Default**: RST = ON, PWR = OFF  
- Control reset/power via GPIO when Pi restarts.

![PCIe HAT+ DIP Switches](images-results/PCIe-TO-5G-HAT+-DIP.png)

---

## 4. Reset / Power On-Off Module with Code

Use **gpiozero** in Python to toggle RST/PWR:

```python
import time
from gpiozero import LED

# led = LED(5)  # Control PWR
led = LED(6)    # Control RST

led.on()        # HIGH → reset/power on
time.sleep(0.5)
led.off()       # LOW → normal
````

---

## 5. Update Kernel

1. Run Waveshare’s one-key installer (5–10 min):

   ```bash
   wget -O - https://files.waveshare.com/wiki/PCIe-TO-5G-HAT%2B/install.sh | sudo bash
   ```

2. After auto-reboot, verify:

   ```bash
   uname --all
   ```

---

## 6. Verify Kernel & PCIe Detection

1. Check PCIe device:

   ```bash
   lspci
   ```

   Expected:

   ```
   02:00.0 Network controller: Qualcomm Device 0306 (rev 01)
   ```

2. Or via dmesg:

   ```bash
   dmesg | grep -i pcie
   ```

---

## 7. Dial-up to the Internet

Use Waveshare’s dial-up tool:

```bash
sudo waveshare-CM
```

![Results of Running waveshare-CM](images-resultswaveshare-cm.png)

---

## 8. Power Monitoring

The onboard INA219 chip monitors 5 V input:

1. Download demo:

   ```bash
   wget https://files.waveshare.com/wiki/PCIe-TO-5G-HAT%2B/PCIe_TO_M.2_HAT%2B.zip
   unzip -o PCIe_TO_M.2_HAT+.zip -d ./PCIe_TO_M.2_HAT+
   cd PCIe_TO_M.2_HAT+
   sudo python INA219.py
   ```

2. Default I²C address: `0x40` (changeable via back resistor).

---

## 9. Dial-up Test

After PCIe mode is up and kernel is loaded, run:

```bash
sudo waveshare-CM
```

You should see PPP negotiation and be online.

---

> **You’re Done!**
> Your Raspberry Pi 5 is now connected via 5G through the Quectel RM520N-GL PCIe HAT+.


Speed Test :
![Speed Test Results](images-results/speedtest.png)
---


Assigned IP : 

![Speed Test Results](images-results/config.png)
````markdown
## 🔄 Fallback: Switch to USB Mode via Windows & PuTTY

If you encounter any PCIe-related errors on the Raspberry Pi, you can switch the RM520N-GL back to USB mode using a Windows PC and PuTTY:

1. **Connect the Module to Windows**  
   – Use the USB-C port on the RM520N-GL board and a USB-C cable.

2. **Open PuTTY**  
   – Connection Type: **Serial**  
   – Serial line: the “AT Port” COM number (e.g., COM3)  
   – Speed: **115200**  
   – Click **Open**  

3. **Switch to USB Mode**  
   ```text
   AT+QCFG="data_interface",0,0
   AT+QCFG="usbnet",2
````

Both should return `OK`.

4. **Verify Signal & Registration**

   ```text
   AT                ← Check modem responds (OK)
   AT+CSQ           ← Check signal strength (+CSQ: <rssi>,99; rssi > 10)
   AT+CPIN?         ← SIM status (READY)
   AT+COPS?         ← Network registration status
   AT+COPS=0        ← Auto-register if not already
   AT+CGDCONT=1,"IP","jionet"  ← Set APN
   ```

   – Ensure `+CSQ:` value is ≥ 10 (and not 99).
   – In Windows **Settings → Network & Internet → Cellular**, add both “Internet” and “Normal” APNs if required.

For more AT Commands Go through Documents Folder and also check this link https://www.waveshare.com/wiki/RM520N-GL_5G_HAT+


5. **Reboot & Test**
   – Unplug the USB cable, flip the DIP switches back for PCIe mode, and reconnect to the Raspberry Pi.
   – On the Pi, re-enter PCIe mode and APN via AT commands (using a USB serial adapter or Waveshare’s USB-C cable and `minicom`/`screen`).
   – Finally, run:

   ```bash
   sudo waveshare-CM
   ```

   – Confirm PPP negotiation and Internet connectivity.

```
```
