
<h1 align="center">
  <br>
  <a href="http://www.amitmerchant.com/electron-markdownify"><img src="https://usenocturne.com/images/logo.png" alt="Nocturne" width="200"></a>
  <br>
  nocturne-image
  <br>
</h1>

<h4 align="center">A pre-built Debian 12 image for the <a href="https://carthing.spotify.com/" target="_blank">Spotify Car Thing</a>.</h4>

<p align="center">
  <a href="#how-to-use">How To Use</a> •
  <a href="#download">Download</a> •
  <a href="#troubleshooting">Troubleshooting</a> •
  <a href="#donate">Donate</a> •
  <a href="#credits">Credits</a> •
  <a href="#related">Related</a> •
  <a href="#license">License</a>
</p>

<br>
<img src="https://raw.githubusercontent.com/brandonsaldan/nocturne-image/refs/heads/main/pictures/nocturne-1.png" alt="screenshot">

## How To Use

To run Nocturne on your Car Thing, you will need a host device tethered to it, such as a Raspberry Pi, Windows PC, or Linux PC, along with a Spotify Premium account. To flash your Car Thing, you can use the web-based flashing tool, [Terbium](https://terbium.app/), a tool designed for quick and easy setup with a compatible browser. If Terbium is not working, then you can use [superbird-tool](https://github.com/thinglabsoss/superbird-tool) to flash the image to your Car Thing. 

### Flashing the Spotify Car Thing

To flash your Car Thing with Nocturne, download and unzip the latest image from [Releases](https://github.com/usenocturne/nocturne-image/releases/latest). Next, connect the Car Thing to your computer in USB Mode (hold preset buttons 1 and 4 while connecting), and follow either of the flashing methods.

<details>
<summary>Using Terbium (Recommended)</summary>

Open [Terbium](https://terbium.app/) in a WebUSB compatible browser (ex. Google Chrome, Chromium, etc).

Follow the prompts in Terbium and select the folder path `/path/to/nocturne-image/image` when prompted to select an image folder.

</details>

<details>
<summary>Using superbird-tool</summary>

  Download [superbird-tool](https://github.com/thinglabsoss/superbird-tool) and follow the setup process detailed [here](https://github.com/thinglabsoss/superbird-tool?tab=readme-ov-file#supported-platforms).

  ```bash
  # Go into the superbird-tool repository
  $ cd /path/to/superbird-tool-main

  # Find device
  $ python superbird_tool.py --find_device

  # Flash Nocturne image, without resetting the data partition 
  $ python superbird_tool.py --dont_reset --restore_device /path/to/nocturne-image/image 
  ```

  `python` on Windows could also use `py`, macOS will use `/opt/homebrew/bin/python3` and, Linux will use `python3` instead. 
</details>

### Setting up a host device

<details>
<summary>Raspberry Pi (Recommended)</summary>

NOTE: This setup requires the following extra hardware:
```
- Pi Zero W or Pi Zero 2 W
- Micro USB to USB C Cable (to connect the Car Thing to the Pi)
- Micro USB to USB A/USB C Cable (to power the Pi)  
- SD Card >= 8GB
- 5V 2A power supply
```

Download and open [Raspberry Pi Imager](https://www.raspberrypi.com/software/), select Raspberry Pi OS (64-bit) Lite, select "Edit Settings", check "Set hostname", check "Set username and password" (set a password), check "Configure wireless LAN", (enter your network's SSID and password), check "Set local settings". Open the Services tab, enable SSH, and use password authentication. Write the configured OS to your microSD card and insert it into your Raspberry Pi.

After the OS is successfully flashed to the SD card, copy the setup script to your Pi connect your car thing and run the commands as follows:

```bash
# SSH into Raspberry Pi
$ ssh pi@raspberrypi.local

# Download setup_host_rpi.sh to Raspberry Pi
$ wget https://raw.githubusercontent.com/usenocturne/nocturne-image/refs/heads/main/setup-scripts/setup_host_rpi.sh

# Make setup_host_rpi.sh executable
$ chmod +x /home/pi/setup_host_rpi.sh

# Execute setup_host_rpi.sh
$ sudo ./setup_host_rpi.sh

# Reboot Raspberry Pi
$ sudo reboot
```

If you would like to use Nocturne in a car, you can run the following commands to allow your phone's hotspot to provide internet: 

```bash
# SSH into Raspberry Pi
$ ssh pi@raspberrypi.local

# Download setup_hotspot_rpi.py to Raspberry Pi
$ wget https://raw.githubusercontent.com/usenocturne/nocturne-image/refs/heads/main/setup-scripts/setup_hotspot_rpi.py

# Execute setup_hotspot_rpi.py
$ sudo python3 ./setup_hotspot_rpi.py
```
</details>

<details>
<summary>Linux (WIP)</summary>

NOTE: A rewrite of the script is in progress to address edge cases and allow for greater device security. If you still would like to persue this route, you will need to know how to configure your local firewall for NAT and forwarding of network traffic for your specific distro. Unfortunately, this varies greatly between OS (ex. Ubuntu uses UFW, Fedora uses firewalld, and Debian uses nftables).

</details>

<details>
<summary>Windows (UNSUPPORTED)</summary>

NOTE: This setup method is not recommended as there are frequent issues with the AMD USB chipset with earlier Ryzen models and recognizing the Car Thing. Proceed with caution knowing that it may not work on your PC.

Enter the following commands in PowerShell as Administrator to allow internet access to your Car Thing:

```bash
#Identify correct network adapter is present:
$ctNic = (Get-NetAdapter -InterfaceDescription "*NDIS*")

#Set IP address of network interface:
$ctNic | Set-NetIPAddress -IPAddress 192.168.7.1 -PrefixLength 24

#Allow sharing of network connection to Car Thing
New-NetNat -Name "CarThing" -InternalIPInterfaceAddressPrefix 192.168.7.0/24

```

Your mileage may vary greatly here. You may need to configure the Windows Firewall to allow this traffic depending on your environment. 

</details>

### Setting up Nocturne UI

Follow the steps as described [here](https://github.com/usenocturne/nocturne-ui?tab=readme-ov-file#spotify-developer-setup), scan the QR code, enter the Client ID and Client Secret, and log into Spotify to finish setting up Nocturne.

## Troubleshooting

If you are having issues flashing Nocturne to your Car Thing, check out the guides below. 


<details>
<summary>superbird-tool: USBTimeoutError</summary>

If you are encountering this error while flashing your Car Thing, try using the option `--slow_burn` or `--slower_burn` in the command used to flash. 

This will look like the following:
```bash
$ python ./superbird_tool.py --dont_reset --slow_burn --restore_device /path/to/nocturne/image
``` 

If this still does not resolve the error, then you will have to edit line 164 (the one that says `MULTIPLIER = 8`) in `superbird_device.py` 

If your flashing is failing at `executing bulkcmd: "amlmmc part 1"`, then try running the following command manually. This may take a few tries to succeed.

```bash
$ python ./superbird_tool.py --bulkcmd "amlmmc part 1"
``` 

 `python` in the above commands depends on what OS you are running. 

For Windows, it will be `python`. 

For macOS, it will be `/opt/homebrew/bin/python3`. 

For Linux, it will be `python3`

</details>

<details>
<summary>superbird-tool: "bulkcmd timed out or failed!" after system_b </summary>

If you are encountering this error while flashing your Car Thing, you must replace the `superbird_partitions.py` file in the `superbird-tool` folder with the one provided in this repo. 

This error occurs since some devices have a smaller data partition, causing the error when attempting to flash the data partition.
</details>

<details>
<summary>superbird-tool: AssertionError </summary>

If you are encountering this error while flashing your Car Thing, you must install the `libusbk` driver in Zadig. You can do this with the steps found [here](https://github.com/thinglabsoss/superbird-tool?tab=readme-ov-file#windows), and replacing `libusb-win32` with `libusbk` instead.
</details>

If your issue is not listed here, or if you need help, join our Discord [here!](https://discord.gg/GTP9AawHPt)

## Download

You can download the latest flashable version of Nocturne for Windows, macOS and Linux [here](https://github.com/usenocturne/nocturne-image/releases/latest).

## Donate

Nocturne is a massive endeavor, and the team have spent everyday over the last few months making it a reality out of our passion for creating something that people like you love to use.

All donations are split between the four members of the Nocturne team, and go towards the development of future features. We are so grateful for your support!

[Buy Me a Coffee](https://buymeacoffee.com/brandonsaldan) |
[Ko-Fi](https://ko-fi.com/brandonsaldan)

## Credits

This software was made possible only through the following individuals and open source programs:

- [Benjamin McGill](https://www.linkedin.com/in/benjamin-mcgill/), for giving me a Car Thing to develop with
- [shadow](https://github.com/68p), for testing, troubleshooting, and crazy good repo maintainence
- [Dominic Frye](https://x.com/itsnebulalol), for debugging, testing, and marketing
- [bbaovanc](https://x.com/bbaovanc), for OS development, debugging, and testing
- [bishopdynamics](https://github.com/bishopdynamics), for creating the original [superbird-tool](https://github.com/bishopdynamics/superbird-tool), [superbird-debian-kiosk](https://github.com/bishopdynamics/superbird-debian-kiosk), and modifying [aml-imgpack](https://github.com/bishopdynamics/aml-imgpack)
- [Thing Labs' fork of superbird-tool](https://github.com/thinglabsoss/superbird-tool), for their contributions on the original superbird-tool

## Related

[nocturne-ui](https://github.com/usenocturne/nocturne-ui) - The web application that this Debian image runs

## License

This project is licensed under the **MIT** license.

---

> [brandons.place](https://brandons.place/) &nbsp;&middot;&nbsp;
> GitHub [@brandonsaldan](https://github.com/brandonsaldan) &nbsp;&middot;&nbsp;
> Twitter [@brandonsaldan](https://twitter.com/brandonsaldan)
