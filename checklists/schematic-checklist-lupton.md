# Schematic review checklist

## General

* [ ] CAD ERC 100% clean. If some errors are invalid due to toolchain quirks, each exception must be inspected and signed
off as invalid.
* [ ] Verify pin numbers of all schematic symbols against datasheet or external interface specification document (if not yet board proven).
  * [ ] Confirm FMC pins are done correctly
  * [ ] Confirm EMBER pins are oriented correctly
  * [x] Confirm EMBER pins are mapped correctly to package
  * [ ] Confirm EMBER pins are mapped correctly to socket
  * [ ] Confirm all 256 EMBER pins are routed
* [ ] Schematic symbol matches chosen component package
* [ ] Thermal pads are connected to correct power rail (may not always be ground)
* [?] Debug interfaces are not power gated in sleep mode (What is sleep mode?)

## Passive components
* [ ] Power/voltage/tolerance ratings specified as required
  * [ ] Regulators work up to 500 mA
  * [ ] Switches are rated for up to 500 mA
  * [ ] Capacitors are rated for >3.3V
  * [ ] Ferrite beads are rated for >3.3V
* [ ] Ceramic capacitors appropriately de-rated for C/V curve
  * [ ] Using the setup from TCAM board
* [ ] Polarized components specified in schematic if using electrolytic caps etc.

## Power supply

### System power input

* [?] Fusing and/or reverse voltage protection at system power inlet (we don't have this; fusing may be desirable, but the EMBER current pull is small;  Unless gnd gets flipped to 3.3V somehow, I don't see MOSFET oxide perforation becoming an issue)
* [?] Check total input capacitance and add inrush limiter if needed  (Smallest RC with shunt = 2 Ohm * ~32 uF --> 64 usec)

### Regulators

* [ ] Under/overvoltage protection configured correctly if used
* [ ] Verify estimated power usage per rail against regulator rating (vddio_dac rail will use current heavily; confirm regulator power)
* [x] Current-sense resistors on power rails after regulator output caps, not in switching loop
  * [x] Done on power measurement daughter board...
* [---] Remote sense used on low voltage or high current rails
* [ ] Linear regulators and voltage reference ICs are stable with selected output cap ESR
* [ ] Confirm power rail sequencing against device datasheets

### Decoupling
* [?] Decoupling present for all ICs (local decoupling next to EMBER chip on power/gnd pins?)
* [---] Decoupling meets/exceeds vendor recommendations if specified
* [ ] Bulk decoupling present at PSU

### General
* [ ] All power inputs fed by correct voltage
* [ ] Check high-power discrete semiconductors and passives to confirm they can handle expected load
* [?] Analog rails filtered/isolated from digital circuitry as needed (are the gnds/supplies isolated for FSM and non-FSM power rails?)
  * [x] Separate planes should help

## Signals

### Digital

* [ ] Signals are correct logic level for input pin
  * [ ] All signals will be at FPGA setting
* [ ] Pullups on all open-drain outputs
* [ ] Pulldowns on all PECL outputs
* [?] Termination on all high-speed signals (Series termination?)
* [---] AC coupling caps on gigabit transceivers
* [?] TX/RX paired correctly for UART, SPI, MGT, etc
* [ ] Differential pair polarity / pairing correct
* [ ] Active high/low enable signal polarity correct
* [ ] I/O banking rules met on FPGAs etc
* [ ] When using auto-sensing level shifters, ensure the intended receiver doesn't have a pullup/down  (very crucial;  you don't want to fight that resistance)

### Analog

* [?] RC time constant for attenuators sane given ADC sampling frequency (What is this?  Clarify.)
* [x] Verify frequency response of RF components across entire operating range. Don't assume a "1-100 MHz" amplifier has the
same gain across the whole range.  (Verified INA138 behavior using TI's Bode plot)
* [x] Verify polarity of op-amp feedback (INA 138 designed according to TI's suggested layout & +/- pin convention was followed)

### Clocks

* [?] All oscillators meet required jitter / frequency tolerance. Be extra cautious with MEMS oscillators as these tend to have higher jitter.  (no on-board oscillator)
* [?] Correct load caps provided for discrete crystals
* [?] Crystals only used if IC has an integrated crystal driver
* [ ] Banking / clock capable input rules met for clocks going to FPGAs
    * [ ] Xilinx FPGAs: single ended clocks use _P half of differential pairs
    * [ ] If possible, create dummy design with all clocks and other key signals and verify it P&R's properly

### Strap/init pins
* [?] Pullup/pulldowns on all signals that need defined state at boot  (necessary?  do we reset the chip prior to initializing how we want?)
* [ ] Strap pins connected to correct rail for desired state
  * [x] DIP switches are strapped to io_pwr
* [---] JTAG/ICSP connector provided for all programmable devices
* [?] Config/boot flash provided for all FPGAs or MPUs without internal flash  (Dev board?)
* [ ] Reference resistors correct value and reference rail

### External interface protection

* [?] Power outputs (USB etc) current limited
  * [ ] Regulators do this
* [?] ESD protection on data lines going off board
  * [x] Exist in EMBER and FPGA

### Debugging / reworkability

* [ ] Use 0-ohm resistors vs direct hard-wiring for strap pins when possible
* [ ] Provide multiple ground clips/points for scope probes (layout check)
  * [ ] Available on power header and analog header
* [ ] Dedicated ground in close proximity to analog test points (layout check; is there an analog ground plain?)
* [ ] Test points on all power rails (layout check)
  * [ ] Accessible through power headers
* [ ] Test points on interesting signals which may need probing for bringup/debug
  * [ ] Heartbeat
  * [ ] All EMBER signals can be probed on bottom through socket

## Thermal

* [ ] Power estimates for all large / high power ICs
  * [ ] EMBER should never exceed 100 mA
* [ ] Thermal calculations for all large / high power ICs  (Would need knowledge of thermal conductivity of CPGA package)
* [ ] Specify heatsinks as needed
