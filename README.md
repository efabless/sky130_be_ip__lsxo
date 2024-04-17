# sky130_be_ip__lsxo
A low-speed crystal oscillator IP for the SKY130 PDK

Designed as part of the 2024 Efabless Chipalooza, this IP is a 32.768kHz crystal oscillator unit. 
It is intended for use in systems built in the SKY130A process that require the ability to disable the subcircuit during deep sleep operation.

## Instructions for Characterization using CACE

This project is using the newly developed tool CACE (https://github.com/efabless/cace) for circuit characterization. To install CACE from PyPI, use:

```
python3 -m pip install --upgrade cace
```

Clone this repository by navigating to the target install directory and using:

```
$ git clone https://github.com/b-etz/sky130_be_ip__lsxo
$ cd sky130_be_ip__lsxo/ 
```

In the new directory, you can inspect schematics and open manual testbenches in the `xschem/` directory, or you can initiate the verification suite from the repo root by using:

```
$ cace-gui
```

The listed specifications are provided in table form, with individual simulation buttons on the right-hand side.

Total simulation time is currently over 3 hours. I recommend running each simulation type individually. If possible, in your simulation home directory, update `.spiceinit` to include the line:

```
set num_threads=4
```

I recommend system RAM of at least 16 GB, and even then warn users of the tendency to run out of memory (at least in a WSL 2 environment). Due to simulation time limitations, all testbenches are performed during crystal startup, or simulating a crystal response with pure sinusoids. Some of these CACE testbenches that include a crystal model, such as duty cycle, can produce spurious results caused by variations in startup behavior. **To plot example behavior and tool with testbench settings, see the manual testbench in the `xschem/` directory.**

## Instructions for Layout Verification

This project used the osic-multitool library (https://github.com/iic-jku/osic-multitool) for most DRC, LVS, and parasitic extraction.

### For a Magic DRC check:

```
$ cd mag/iic-drc
$ {OSIC_MULTITOOL_PATH}/iic-drc.sh ../sky130_be_ip__lsxo.mag
```

### To run LVS:

```
$ cd mag/iic-lvs
$ {OSIC_MULTITOOL_PATH}/iic-lvs.sh -s ../../xschem/sky130_be_ip__lsxo.sch -l ../sky130_be_ip__lsxo.mag -c sky130_be_ip__lsxo
```
The Netgen output is readable as `sky130_be_ip__lsxo.lvs.out` using your favorite text editor.

### To run parasitic extraction:

I have not yet run parasitic extraction across resistance and capacitance corners. To perform an R-C extraction at the nominal values for both:
```
$ cd mag/iic-pex
$ {OSIC_MULTITOOL_PATH}/iic-pex.sh -m 3 ../sky130_be_ip__lsxo.mag
```
The spice output is readable as `sky130_be_ip__lsxo.pex.spice` using your favorite text editor.

## Specifications

### Provided Specifications

| Parameter                          | Minimum    | Typical | Maximum | Unit  |
| ---------------------------------- | ---------- | ------- | ------- | ----- |
| Analog Supply                      | 2.7        | 3.3     | 5.5     | V     |
| Operating Temperature              | -40        | 25      | 85      | deg C |
| AVDD Current Dissipation (Enabled) | 0.8        | 1       | 1.2     | uA    |
| Current Dissipation (Powered Down) | 0.5        | 1       | 2       | nA    |
| _dout_ Duty Cycle                  | 45         | 50      | 55      | %     |
| Startup Time (Power-on)            | 1          | 2       | 4       | ms    |
| Startup Time (Enable)              | 1          | 1       | 2       | ms    |
| Input Load Capacitance at Crystal  |            |         | 2       | pF    |
| Output Load Capacitance at _dout_  |            |         | 200     | fF    |
| Frequency Stability over Temp      | -200       | 0       | 0       | ppm   |
| Frequency Accuracy at 25 deg C     | -20        | 0       | 20      | ppm   |
| _dout_ Low Level (Vol)             |            | 0       | 0.1     | V     |
| _dout_ High Level (Vol)            | DVDD - 0.1 | DVDD    |         | V     |
| _dout_ Rise/Fall Time              | 4          | 5       | 6       | ns    |

### Achieved Specifications

This IP is in the design phase. The compliance table is in development. Typical simulated performance figures are listed below:

| Parameter                          | Typical | Worst | Unit  |
| ---------------------------------- | ------- | ----- | ----- |
| AVDD Current Dissipation (Enabled) | 0.8     | 0.9   | uA    |
| Current Dissipation (Powered Down) | 0.4     | 0.6   | nA    |
| _dout_ Duty Cycle                  | 50.4    | 62    | %     |
| Startup Time (Power-on)            | 1.3     | 5.9   | ms    |
| Startup Time (Enable)              | 1.7     | 5.9   | ms    |
| Frequency Stability over Temp      | n/a     | n/a   | ppm   |
| Frequency Accuracy at 25 deg C     | n/a     | n/a   | ppm   |
| _dout_ Low Level (Vol)             | 0       | 0     | V     |
| _dout_ High Level (Vol)            | DVDD    | DVDD  | V     |
| _dout_ Rise/Fall Time              | 4       | 2.6   | ns    |

### CACE Summary Capture (Schematic Capture)

Last updated April 01, 2024

![CACE Summary - Produced on 01 April 2024](https://github.com/b-etz/sky130_be_ip__lsxo/blob/main/images/cace_results_combined_20240401.png?raw=true)

Duty cycle tests are performed with a 10-nA current initial condition for the crystal motional inductance. This is done to accelerate startup (separate testbench) and capture a more realistic image of duty cycle across these corners.

Startup times appear strongly influenced by initial transient response from initial conditions, which vary slightly from sim to sim. The startup times could improve for 12.5-pF devices by loading the active device with more current and consuming greater startup power. However, this would reduce or eliminate compatibility with 6-pF and 4-pF devices entirely. Lower drive current is required to sustain oscillations with low-CL, low-ESR devices.

## Magic VLSI Layout Render

Last updated April 17, 2024

![GDS Rendered in Magic VLSI, produced 17 April 2024](https://github.com/b-etz/sky130_be_ip__lsxo/blob/main/images/magic_layout_20240417.png?raw=true)

## Theory of Operation

_Section is under development_

### Control Bits (`ena` and `standby`)

This device is designed with digital control bits at 1.8V LVCMOS logic levels, `ena` and `standby`.

`ena` is used to enable the crystal oscillator and amplitude regulation circuits (logic-level high) or power them down (logic-level low). `standby` is an active-low output clock enable, which enables the clock detection and output inverters (logic-level low) or disables them (logic-level high).

Please see the below table for operating states:

| (ena, standby) | State Name            | Typical Current Consumption | Clock Output?                       | Oscillator On? |
| -------------- | --------------------- | --------------------------- | ----------------------------------- | -------------- |
| (0, 0)         | External Clock Bypass | << 1 uA (varies)            | Yes (external oscillator on `xin`)  | No             |
| (0, 1)         | Power Down            | 1 nA                        | No                                  | No             |
| (1, 0)         | Regular Run           | 1 uA                        | Yes (on startup)                    | Yes            |
| (1, 1)         | `dout` Standby        | <1 uA                       | No                                  | Yes            |

The ripple-delay clock gating circuit provides a 4-cycle delay before outputting the clock signal generated by the clock buffer whenever `ena` has just been raised, or `standby` has just been lowered. This is to prevent glitches on `dout`. At the 32-kHz clock frequency, this imposes a 120 us delay to any `dout` edge when the oscillator is already running.

The `xin` and `xout` pins are independently monitored by the clock generator on the digital power domain. This means `dout` can be generated even when `ena` is low, so long as an external oscillator (e.g. TCXO, OCXO) is driving `xin`. In this mode, `xout` should be left floating. `xin` is 5-V tolerant, but testing has only accounted for 1.8V LVCMOS clock signals on `xin`. As this mode was not part of the IP specification, it was not tested rigorously.

### Crystal Oscillator Basics

Crystal oscillators use a transconductor (in this case, a common-source NMOS) to provide a non-linear feedback voltage to a VERY finely tuned filter. The result is a self-sustaining sinusoid that precisely tracks a target frequency.

The finely tuned filter is modeled quite literally as an L-C tank resonator with a tremendous Q-factor. For 32-kHz crystals, typical values of motional inductance range from 3,000 H to 8,000 H, with a motional capacitance of 3 to 7 fF. An equivalent series resistance of 30,000 to 90,000 Ohms attenuates crystal oscillations in the absence of an appropriate amplification and phase shift from the oscillator circuit. Crystal models also contain a parallel (shunt) capacitance originating from crystal case internals, which combines with stray capacitance and load capacitors on the application PCB to match the crystal's specified load capacitance (typically 6 pF to 12.5 pF).

Temperature and process variability dramatically affect the steady-state oscillation amplitude of a crystal oscillator, in the absence of feedback. This IP block uses voltage feedback from the crystal to reduce the current driving the active device. This is a configuration commonly referred to as an "amplitude regulator". The added benefits of amplitude regulation include power savings after startup and improved crystal reliability.

Startup time is commonly defined as the time taken from oscillator activation to the crystal reaching 90% of its steady-state oscillation amplitude. For a high-Q crystal oscillator, this may take thousands of cycles. This project uses the alternative definition of the time from oscillator activation to the first valid clock. Although the crystal does not oscillate within tolerance during startup, the early use of a clock could be useful for certain applications. **Designers using this IP block must be aware of the startup variability this design exposes, and gate the clock according to their system requirements.**

### Output Clock Generation

`xin` and `xout` are compared by a differential amplifier, which is fed a tail current bias from an external bias generator at 50 nA. This amplifies the `xin` signal level by approximately 20-30 times (until saturation at higher amplitudes). The output sinusoid is AC-coupled to a center-biased inverter chain that approximates a square waveform output. As described in the Control Bits section, the output clock is gated by a ripple delay counter, and amplified by a final inverter section to drive `dout` with the specified rise/fall times.

This clock output is not guaranteed by design to be a 50% duty cycle. This is the reason for the ripple delay glitch filter. Regardless, duty cycle deviations as extreme as 10% or 90% at the slow 32-kHz frequency are unlikely to cause setup/hold violations.

I would recommend investigating a Schmitt trigger or other latching mechanism with hysteresis to improve clock shape. This was not investigated in the first design pass due to the rapid startup specification.

### Recommended Crystal Specifications

This circuit can only be simulated so many ways, and development has focused on 12.5 pF load capacitance crystals. For that reason, higher load capacitances are recommended for frequency stability and startup assurance. Load capacitances of 6 pF have also been simulated successfully. At a glance, smaller load capacitances (e.g. 4 pF) may start up with a transconductor g_m of 0.3 uS to 3 uS. Larger load capacitances (e.g. 12.5 pF) require transconductances of 1 uS to 40 uS. This oscillator IP was designed with a transconductance of ~8 uS, which allows reasonably fast startup for 12.5 pF crystals, great startup for 6 pF crystals, and far exceeds the optimal g_m for 4 pF crystals. For this reason, 4 pF crystals may fail to start with this circuit.

The following specification table for appropriate crystals will keep your system capable of the proposed specifications listed earlier. For requirements given in ranges, devices with a range that fully includes the specified range are acceptable.

| Parameter                         | Requirement | Unit         |
| --------------------------------- | ----------- | ------------ |
| Nominal Frequency                 | 32,768      | Hz           |
| Load Capacitance                  | 6 to 12.5   | pF           |
| Calibration Tolerance (25 deg C)  | 15          | ppm absolute |
| Minimum Temperature Coefficient   | -0.040      | ppm*K^-2     |
| Turnover Temperature Range        | 20 to 30    | deg C        |
| Nominal Turnover Temperature      | 25          | deg C        |
| Operating Temperature Range       | -40 to 85   | deg C        |
| Storage Temperature Range         | -40 to 85   | deg C        |
| Equivalent Series Resistance      | <= 90,000   | Ohm          |
| Maximum Drive Level               | >= 0.5      | microWatt    |

The following table includes examples of available crystal part numbers and their post-silicon validation status:

| Manufacturer  | Part Number         | C_load  | Package         | Approx. $/10 | Digikey Link                           | Mouser Link            | Testing Status | Notes                              |
| ------------- | ------------------- | ------- | --------------- | ------------ | -------------------------------------- | ---------------------- | -------------- |----------------------------------- |
| ECS Inc.      | ECS-.327-12.5-34B-C | 12.5 pF | 3.2 x 1.5mm SMD | $4.50        | https://www.digikey.com/short/p27fn3tf | https://mou.sr/3xxjRjt | UNTESTED       | Low cost                           |
| IQD Freq Prod | LFXTAL062558        | 9 pF    | 2.0 x 1.2mm SMD | $7.10        | https://www.digikey.com/short/0zn0d8d4 | https://mou.sr/3Q7DbKu | UNTESTED       | Lower tempco, small size           |
| Abracon       | ABS06-32.768KHZ-6-1 | 6 pF    | 2.0 x 1.2mm SMD | $15.20       | https://www.digikey.com/short/080dd50r | https://mou.sr/3na9BJf | UNTESTED       | Low load capacitance (lower power) |

As with most PCB systems, reflow has an impact on circuit reliability. For a crystal oscillator circuit, aggressive reflow temperatures may cause crystals to deviate from their specified tolerances. Always follow the reflow profiles specified by the crystal manufacturer for your chosen part in its datasheet.

### Startup Characteristics

A big thank-you to IQD Frequency Products, Ltd. for providing the 12.5pF device characterization data used for these simulations.

Startup waveforms are interesting to observe for a crystal oscillator, because they expose the inner workings of the oscillator system. Below, I have an Ngspice plot of _xin_ (red) and _xout_ (blue) generated by the manual testbench with a rising edge on _ena_ and a falling edge on _standby_.

![XIN and XOUT Waveforms, 1.5sec sample, produced 01 April 2024](https://github.com/b-etz/sky130_be_ip__lsxo/blob/main/images/typical_startup_waveform.png?raw=true)

The bulk of the signal amplitude grow-up happens between 0.5 seconds and 1.1 seconds. Here, the _xin_ amplitude climbs from 1 mVpp to ~300 mVpp. The _xout_ amplitude inflects around 0.9 seconds in, and levels off alongside _xin_ around 1.2 seconds.

What causes the amplitude to plateau (deviate from the exponential growth)? Other than non-linearities in the common-source amplifier stage, it's the amplitude regulator built into the bias generator.

Below, I have another Ngspice plot of the current drawn from _avdd_ over time. By design, it is most power-hungry early during startup. After startup, the current consumption dwindles to a mere 300 nA because of a rising P-side bias voltage generated by the bias_gen subblock (noise around the ideal sigmoid is caused by AC feedthrough into the bias generator, as well as transient sim nonsense from the 1-us timestep).

![AVDD Waveform, 1.5sec sample, produced 01 April 2024](https://github.com/b-etz/sky130_be_ip__lsxo/blob/main/images/typical_startup_avdd_current.png?raw=true)

## Design References

Vittoz, E. Low-Power Crystal and MEMS Oscillators. Springer, 2010.

Lei, et al. Startup Time and Energy-Reduction Techniques for Crystal Oscillators in the IoT Era. IEEE Transactions on Circuits and Systems–II: Express Briefs, Vol. 18, 1.
IEEE, 2021.

Coustans, et al. A 32kHz crystal oscillator leveraging voltage scaling in an ultra-low power 40nA real-time clock. 31st IEEE International SOCC. IEEE, 2018.
