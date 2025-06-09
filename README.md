````markdown
<p align="center">
  <img src="https://raw.githubusercontent.com/<your-username>/raspberry-pi-5g-quectel-module/main/images-results/banner.gif" alt="Raspberry Pi 5G" width="800"/>
</p>

# 🚀 Raspberry Pi 5G with Quectel RM520N-GL PCIe HAT+

> **Enable blazing-fast 5G internet on your Raspberry Pi 5!**  
> Use the Quectel RM520N-GL PCIe-to-5G HAT+ to get truly mobile connectivity for IoT, remote monitoring, or backup networking.

---

<p align="center">
  <a href="#hardware-requirements">Hardware</a> •
  <a href="#installation">Install OS</a> •
  <a href="#pcie-support">PCIe</a> •
  <a href="#dip-switch-control">DIP Switch</a> •
  <a href="#kernel-update">Kernel</a> •
  <a href="#dialup">Dial-up</a> •
  <a href="#fallback-usb-mode">Fallback</a>
</p>

---

## 📦 Hardware Requirements

| Item                             | Description & Purchase Link                                                 |
|----------------------------------|-------------------------------------------------------------------------------|
| 🎛️ Raspberry Pi 5                | [Buy on Raspberry Pi Store](https://www.raspberrypi.com/products/raspberry-pi-5/) |
| ⌨️ USB Keyboard & Mouse          | [Buy on Amazon India](https://www.amazon.in/s?k=usb+keyboard+mouse)             |
| 📺 HDMI Monitor                  | [Buy on Amazon India](https://www.amazon.in/s?k=hdmi+monitor)                  |
| 💽 Boot-ready SD Card (≥16 GB)    | [Buy on Amazon India](https://www.amazon.in/s?k=raspberry+pi+boot+sd+card)     |
| 📡 Quectel RM520N-GL PCIe HAT+   | [Buy from Waveshare](https://www.waveshare.com/pcie-to-5g-hat-plus.htm)        |

---

## 🔧 1. Install Raspberry Pi OS

1. Download **Raspberry Pi Imager**:  
   <https://www.raspberrypi.com/software/>  
2. Flash **Raspberry Pi OS (64-bit)** onto your SD card.  
3. Insert SD card, connect peripherals, and power on.  
4. Complete setup (locale, Wi-Fi, password).

<details>
<summary>📺 Animated Preview</summary>
<p align="center">
  <img src="https://raw.githubusercontent.com/<your-username>/raspberry-pi-5g-quectel-module/main/images-results/os-install.gif" alt="OS Install Animation" width="600"/>
</p>
</details>

---

## ⚙️ 2. Enable PCIe Support

```bash
sudo nano /boot/firmware/config.txt
# Add at EOF:
dtparam=pciex1
dtoverlay=pciex1-debug
sudo reboot
````

<details>
<summary>🖼️ View PCIe Enablement</summary>
<p align="center">
  <img src="images-results/pcie-enable.png" alt="PCIe Config" width="600"/>
</p>
</details>

---

## 🎚️ 3. DIP Switch Control

* **RST** (GPIO 6): ON
* **PWR** (GPIO 5): OFF

<p align="center">
  <img src="images-results/PCIe-TO-5G-HAT+-DIP.png" alt="DIP Switches" width="500"/>
</p>

<details>
<summary>🔄 Collapse/Expand Details</summary>
Adjust switches to control module reset/power via GPIO on Pi reboot.
</details>

---

## 🔄 4. Reset / Power ON-OFF with Code

```python
from gpiozero import LED
import time

led = LED(6)    # RST
led.on()        # HIGH → reset/power on
time.sleep(0.5)
led.off()       # LOW → normal
```

---

## ⚡ 5. Update Kernel

```bash
wget -O - https://files.waveshare.com/wiki/PCIe-TO-5G-HAT%2B/install.sh | sudo bash
# Wait ~5–10 min for auto-reboot
uname --all
```

---

## ✅ 6. Verify PCIe & Kernel

```bash
lspci
# Expect: Qualcomm Device 0306 (rev 01)
dmesg | grep -i pcie
```

---

## 📞 7. Dial-up to the Internet

```bash
sudo waveshare-CM
```

<p align="center">
  <img src="images-results/dialup-demo.gif" alt="Dial-up Animation" width="600"/>
</p>

---

## 🔋 8. Power Monitoring (INA219)

```bash
wget https://files.waveshare.com/wiki/PCIe-TO-5G-HAT%2B/PCIe_TO_M.2_HAT%2B.zip
unzip -o PCIe_TO_M.2_HAT+.zip -d ./demo
cd demo
sudo python INA219.py
```

---

## 📡 9. Dial-up Test

```bash
sudo waveshare-CM
# Watch PPP negotiation → Online!
```

---

## 🔄 Fallback: USB Mode via Windows & PuTTY

1. **Switch to USB Mode**

   ```text
   AT+QCFG="data_interface",0,0
   AT+QCFG="usbnet",2
   ```
2. **AT Commands**

   ```text
   AT
   AT+CSQ        # Signal ≥10
   AT+CPIN?      # SIM READY
   AT+COPS?      # Reg status
   AT+CGDCONT=1,"IP","jionet"
   ```
3. **Revert to PCIe**
   Flip DIP, reconnect to Pi, then `sudo waveshare-CM`.

---

## 🎉 You’re Done!

Your Raspberry Pi 5 now enjoys high-speed 5G through Quectel RM520N-GL.
Enjoy seamless mobile connectivity!

<p align="center">
  <img src="https://raw.githubusercontent.com/<your-username>/raspberry-pi-5g-quectel-module/main/images-results/success.gif" alt="Success" width="400"/>
</p>
```
