We should most likely make our motherboard, so it will be a tiny form factor. 

Ok so we are doing a LattePanda Mu, I'm pretty sure, just because of the better support
Resources to make a carrier board (MU):
https://github.com/LattePandaTeam/LattePanda-Mu
https://docs.lattepanda.com/content/mu_edition/design_guide_foreword/
https://www.hackster.io/piecol/moka-tiny-open-source-carrier-board-for-lattepanda-mu-7c6760
Example board (can download files): https://github.com/LattePandaTeam/LattePanda-Mu/tree/main/Electricals/Examples/%5BDFR1141%5DFull%20EVA%20Carrier%20for%20LattePanda%20Mu
www.youtube.com/watch?v=2VDRO9jjZzo

Here's a whole course on how to make a carrier board
https://www.youtube.com/watch?v=GE25eGYICBA
Notes:
High speed pcb: physics of fast signals
Hdmi- causes high - frequency issues

heres a guy who made his own: https://oshwlab.com/yoge2013/mini-carrier-demo-for-lattepanda-mu
I think I will use EasyEDA as it is simpler for this project, can run on my PC, and is easier to learn. NVM, we are using Kicad for better support

I think we gotta add our own powertrain within the mobo or a screw terminal
I think I might have to switch to the main eval board and take away the ports there 
I think we remove the PCIE, unless we make a for a GPU to work in here, which we aint doing
Changes
-removed the PCIE
-removed the SMART UPS expansion board 
-removed usb-c as we will be using a battery.
-added screw term
-deleted the 2 Usb 2.0's we're given to rewire those into the usb2517
-added usb2517 with 4 usb-c receptavcles and 2 USB-a ports
-the USB2517 needs a crystal clock, so heres the one we chose
https://www.lcsc.com/product-detail/C2905360.html?s_z=n_q_24MHz%2520crystal%25203225&spm=wm.fly.bg.0.xh&lcsc_vid=TlgMUFcDEwNcUwcFEQQIBVVXRVJYVlNeRVcLXwVXQ1YxVlNeRVVeX1ZeRFFeUzsOAxUeFF5JWBYZEEoKFBINSQcJGk4%3D

Finished the Schematic less go, moving onto the PCB first I need to figure out the size which will be rather difficult.

WAIT VERY IMPORTANT MUST DO::: ADD BATTERY MONITORING 

https://www.mouser.com/ProductDetail/Analog-Devices-Maxim-Integrated/MAX17048G%2B?qs=D7PJwyCwLArjnts0jPerYw%3D%3D
- **Linux**: mainline kernel already has a driver for the MAX17048 family (`max17040.c`, covers 17048/17049/etc). If it enumerates properly it'll show up under `/sys/class/power_supply/` and appear natively in GNOME/KDE battery indicators. The catch: Mu uses ACPI (x86), not devicetree, so the kernel needs to know an i2c device exists at that address — either via an ACPI SSDT entry describing it, or manually instantiating it with `i2c_new_device` at boot (a short udev/systemd script). Doable without touching BIOS.
- **Windows**: no built-in driver for this chip, and writing a real battery class driver (WDF) so it shows in the taskbar is a much bigger lift than the hardware itself.
for MAX17048