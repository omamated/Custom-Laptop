Ok so the motherboard will have a DC on it (this is ow most Mu's work) so we are going to need at least 12 volts. 
This will also make the battery easy-ish to swap.
Options:
We make our own (probably will do this)
Making our own:
we have to put a screw terminal on the board or JST connectors (idk too much abt that)
Ok so this is what we could do 
Battery- Tp4056 -> charging with USB-C & goes back to power the laptop with a screw terminal.


We might have to make our own regulator according to the goat Ben Makes everything (he also apparently made a video on battery regulators, might have to use that)


**Battery**  
Go with an **unprotected/bare 4S LiPo pack** — just 4 cells spot-welded in series with a 5-pin JST-XH balance tap exposed, no onboard protection PCB. You don't want a pack with its own built-in protection circuit fighting your BQ25798/BQ76905 — two protection systems on the same pack causes weird trip conflicts.

- Target ~3000–4000mAh (44–60Wh) for "light and small" — keeps you under the 100Wh threshold that avoids extra shipping/customs hassle
- Look at Grepow, or general RC/drone 4S pouch packs — pick a flat form factor to fit your 25mm chassis
- Confirm it exposes: main B+/B- leads (power) + the 5-pin balance connector (access to each cell tap — you'll need this for the BMS)

**Battery management (protection + balancing): add BQ76905**  
This is the piece your current plan is missing. BQ25798 charges the _pack_ as a whole (2 wires) but has zero visibility into individual cell voltages — over many cycles your 4 cells will drift apart without correction.

- 2S–5S battery monitor and protector, built for exactly this range and it directly supports host-controlled cell balancing
- Sits between your balance-tap connector and the cell stack: low-side protection FETs, per-cell voltage monitoring, over/undervoltage + overcurrent + overtemp protection, all over I2C
- 20-pin QFN, small footprint — fits your size constraints
- This IC is your real "battery management" block; BQ25798 stays as the charger/power-path piece feeding it from the front end

**Fuel gauge (OS-visible battery %): MAX17205**  
Already covered — I2C, up to 20V pack sensing, ModelGauge m5 SOC algorithm, gives you an actual percentage the LattePanda's OS can read (Windows/Linux battery icon) rather than raw voltage guessing.

Ok so we were thinking that baseus blade option
https://www.amazon.com/dp/B0DK8V9LSV?lv=shuf&channelId=500&plpRedirect=mhFallback
this can apparently work well with TCPM drivers on linux allowing the os to communicate with the battery.
If we use a power bank, this will allow us to take this internationally via flights and etc. It would also be much safer.

some research (by ai)
Yeah, that link is the **Baseus Blade 100W / 20,000mAh**. I checked the exact model you linked. [Baseus Blade 100W 20,000mAh official page](https://www.baseus.com/pages/download-center?utm_source=chatgpt.com)

### [Baseus Blade 100W 20000mAh](https://www.pzdeals.com/products/baseus-20000mah-100w-blade-laptop-portable-charger?utm_campaign=openai_catalog&utm_content=product_card&utm_medium=feed&variant=51517674258747&utm_source=chatgpt.com)

_$39.49_

### The good news

The Blade definitely has its **own internal battery-management electronics** and already knows its battery percentage—the built-in display reports remaining battery level. Its USB-C ports support USB-PD up to 100 W. ([Manuals Plus](https://manuals.plus/asin/B0DK8V9LSV?utm_source=chatgpt.com "Baseus Blade 100W 20000mAh Portable Charger User Manual"))

**But I cannot confirm from Baseus's documentation that it exposes that percentage over USB-PD.**

And that's the crucial part.

USB-PD has a mechanism for:

```text
Laptop → Get_Battery_Status
              ↓
Baseus → Battery_Status
              ↓
        battery information
```

That mechanism exists in USB-PD, but manufacturers don't necessarily implement it. There are reports specifically noting that many power banks classify themselves as a power supply rather than a battery and therefore don't return battery status. ([Stack Overflow](https://stackoverflow.com/questions/68275250/battery-stats-over-usb-pd?utm_source=chatgpt.com "Battery stats over USB-PD - Stack Overflow"))

### So don't design around it yet

For your laptop, I'd treat the possibilities like this:

**Case A — Blade supports Battery Status**

```text
Blade
  │ USB-C PD
  ▼
PD controller on your motherboard
  │
  ▼
Linux TCPM
  │
  ▼
Linux power_supply
  │
  ▼
🔋 73%
```

**Case B — Blade doesn't support it**

```text
Blade
  │ USB-C PD
  ▼
PD controller
  │
  ▼
Linux TCPM
  │
  └── "I know you're providing 20V @ 5A"
      
      ❌ No battery percentage
```

You can still use the Blade perfectly fine as the laptop's power source; you'd just have to get the percentage some other way.

### What I'd do in your situation

**Don't buy a PD analyzer yet.** Since you're already designing the carrier board, we can first investigate the exact Blade's PD implementation from Linux/USB-PD documentation and existing hardware reports.

And once you actually get the Blade, we can plug it into a normal Linux machine and see exactly what it advertises using Linux's USB-C/PD interfaces.

The fact that **the Blade itself already knows its percentage is promising**, but it does **not prove that it tells the laptop**.

If we can confirm that it responds to `Get_Battery_Status`, then **your idea of having the OS display the Baseus's actual battery % becomes very realistic**.