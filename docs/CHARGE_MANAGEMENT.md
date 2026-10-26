# Charge Management
**Managing the charge voltage and amperage of the HiBy OS players**

*Plus information about handling replacement cells of different voltages than stock*


## Device Differences
The R1 uses the AXP2101 charge controller to manage all of the device's charging needs **(TODO: i think the R3II 2025 uses the exact same charging setup as the R1)**.

The R3Pro II uses the MP2731 to manage it's battery (voltage, charging amperage, etc.). Despite this, the R3Pro II also includes an AXP2101. I do not know what the R3Pro II does with its AXP2101, since it seems that all battery needs are managed entirely by the MP2731.


## [MP2731](docs/third_party/components/MP2731GQC.pdf)

### Runtime Charge Limits

An effective way of limiting charge, that can provide more flexibility is changing the charge parameters at runtime (be it through a kernel module or through a running program). This method can allow for exact percentage-based limits by enabling and disabling charging based on the read values rather than having the MP2731 handle the limit on its own.

**(TODO: write how to do this. i think it's got something to do with step charging)**

**(TODO: write what makes this method safe, talking about how potential bricks are avoided)**

### Limiting Voltage

Register `0x07` in the MP2731 is for battery voltage regulation settings. Bits 7 through 1 define the battery regulation voltage (i.e. how high it will charge up to). It has a 3.4V offset and a 3.4V to 4.67V range, with a step size of 10mV. Bit 0 sets the battery recharge threshold, which is not relevant to this, so we will ignore that bit.

| Bit | Meaning |
|-----|---------|
| 7   | 640mV   |
| 6   | 320mV   |
| 5   | 160mV   |
| 4   | 80mV    |
| 3   | 40mV    |
| 2   | 20mV    |
| 1   | 10mV    |

*Example: register `0x07` being `0b1010000X` means a voltage limit of 4.2V (the X is just since we aren't using the 0 bit)*

### Limiting Charge Amperage

#### Fast Charge Amperage

Register `0x05` in the MP2731 is for the fast charge current. Bits 6 through 0 define the fast charge current (i.e. the maximum number of amps it will charge with in fast-charge). It has a 320mA offset and a 320mA to 4520mA range, with a step size of 40mA. Bit 7 sets the pre-charge to fast charge threshold which is not relevant to this, so we will ignore that bit.

| Bit | Meaning |
|-----|---------|
| 6   | 2560mA  |
| 5   | 1280mA  |
| 4   | 640mA   |
| 3   | 320mA   |
| 2   | 160mA   |
| 1   | 80mA    |
| 0   | 40mA    |

*Example: register `0x05` being `0bX0100110` means a fast charge amperage limit of 1840mA (the X is just since we aren't using the 7 bit)*

#### Pre-Charge Amperage

Register `0x06` in the MP2731 is for the pre-charge and termination current. Bits 7 through 4 define the pre-charge current (i.e. the maximum number of amps it will charge with in pre-charge). It has a 150mA offset and a 150mA to 750mA range, with a step size of 40mA. Bits 3 through 0 define the termination current, which is not relevant to this, so we will ignore those bits.

| Bit | Meaning |
|-----|---------|
| 7   | 320mA   |
| 6   | 160mA   |
| 5   | 80mA    |
| 4   | 40mA    |

*Example: register `0x06` being `0b0010XXXX` means a pre-charge amperage limit of 230mA (the Xs are just since we aren't using the 3 through 0 bits)*

## [AXP2101](docs/third_party/components/X-power-AXP2101_SWcharge_V1.0.pdf)

### Runtime Charge Limits

**(TODO: more research needed into how to prevent device bricking)**
Enabling and disabling charging at runtime (be it through a kernel module or through a running program) can provide more flexibility. This method can allow for exact percentage-based limits by enabling and disabling charging based on the read values rather than having the AXP2101 handle the limit on its own.

**However** this can potentially lead to a bricked device. **(TODO: write potential bricks)**

### Limiting Voltage

Register `0x64` in the AXP2101 is for battery voltage regulation settings. Bits 2 through 0 define the battery regulation voltage (i.e. how high it will charge up to). Each bit combination has a different meaning, not in the additive manner like it works for the MP2731. Bits 7 through 3 are not defined in the manual, so we will ignore those.

| Value | Meaning  |
|-------|----------|
| 0b000 | reserved |
| 0b001 | 4.00V    |
| 0b010 | 4.10V    |
| 0b011 | 4.20V    |
| 0b100 | 4.35V    |
| 0b101 | 4.40V    |
| 0b110 | reserved |
| 0b111 | reserved |

*Example: register `0x64` being `0bXXXXX011` means a voltage limit of 4.2V (the X is just since we aren't using the 7 through 3 bits)*

### Limiting Charge Amperage

#### Pre-Charge Amperage

Register `0x61` in the AXP2101 is for the pre-charge current. Bits 3 through 0 define the pre-charge current (i.e. the maximum number of amps it will charge with in pre-charge). Each bit combination has a different meaning, not in the additive manner like it works for the MP2731. Bits 7 through 4 are not defined in the manual, so we will ignore those.

| Value  | Meaning  |
|--------|----------|
| 0b0000 | 0mA      |
| 0b0001 | 25mA     |
| 0b0010 | 50mA     |
| 0b0011 | 75mA     |
| 0b0100 | 100mA    |
| 0b0101 | 125mA    |
| 0b0110 | 150mA    |
| 0b0111 | 175mA    |
| 0b1000 | 200mA    |
| ...    | reserved |


*Example: register `0x61` being `0bXXXXX0100` means a pre-charge amperage limit of 100mA (the X is just since we aren't using the 7 through 4 bits)*


#### Constant Current Charge Amperage

Register `0x62` in the AXP2101 is for the constant current charge current. Bits 4 through 0 define the constant current charge current (i.e. the maximum number of amps it will charge with in constant current charge). Each bit combination has a different meaning, not in the additive manner like it works for the MP2731. Bits 7 through 5 are not defined in the manual, so we will ignore those.

*Note: values `0b00001` through `0b00011` are not defined in the manual. It is unsure whether these would be 25mA, 50mA, and 75mA respectively, or whether they're reserved, or what.*

| Value   | Meaning  |
|---------|----------|
| 0b00000 | 0mA      |
| 0b00001 | ???      |
| 0b00010 | ???      |
| 0b00011 | ???      |
| 0b00100 | 100mA    |
| 0b00101 | 125mA    |
| 0b00110 | 150mA    |
| 0b00111 | 175mA    |
| 0b01000 | 200mA    |
| 0b01001 | 300mA    |
| 0b01010 | 400mA    |
| 0b01011 | 500mA    |
| 0b01100 | 600mA    |
| 0b01101 | 700mA    |
| 0b01110 | 800mA    |
| 0b01111 | 900mA    |
| 0b10000 | 1000mA   |
| 0b10001 | 1100mA   |
| 0b10010 | 1200mA   |
| 0b10011 | 1300mA   |
| 0b10100 | 1400mA   |
| 0b10101 | 1500mA   |
| ...     | reserved |


*Example: register `0x62` being `0bXXXX01101` means a constant current charge amperage limit of 700mA (the X is just since we aren't using the 7 through 5 bits)*


#### Charge Termination Amperage

Register `0x63` in the AXP2101 is for the charge termination current. Bits 3 through 0 define the charge termination current (i.e. the maximum number of amps it will charge with in charge termination). Each bit combination has a different meaning, not in the additive manner like it works for the MP2731. Bits 7 through 5 are not defined in the manual, so we will ignore those. Bit 4 sets whether charging termination current is enabled (1 is enabled, 0 is disabled).

| Value  | Meaning  |
|--------|----------|
| 0b0000 | 0mA      |
| 0b0001 | 25mA     |
| 0b0010 | 50mA     |
| 0b0011 | 75mA     |
| 0b0100 | 100mA    |
| 0b0101 | 125mA    |
| 0b0110 | 150mA    |
| 0b0111 | 175mA    |
| 0b1000 | 200mA    |
| ...    | reserved |


*Example: register `0x63` being `0bXXXXX0010` means a charge termination amperage limit of 50mA (the X is just since we aren't using the 7 bit)*


## Safely Handling Replacement Cells

> [!NOTE]
> This document will not discuss how to perform the battery replacement itsself, as that is out of scope. Here, we will only be talking about how to update the device configurations so that the new battery can be handled safely.
> 
> **(TODO: make battery replacement guide)**

The HiBy OS devices (at least all that I have checked) use LiHV cells, which are quite uncommon elsewhere. LiHV cells have a maximum voltage of around 4.4V, compared to the 4.2V maximum voltage of a standard lithium cell.

> [!NOTE]
> **Battery Language Clarification**
> 
> Battery voltage is typically discussed using "nominal" voltage. That value is somewhere between the minimum and maximum voltage of the cell. Here, we are talking mosetly about maximum voltage instead, which is the voltage at which the cell is at 100% capacity.
> 
> Typical lithium cells have a "nominal" voltage of 3.7V and a maximum voltage of 4.2V.
> LiHV cells usually have a "nominal" voltage of around 3.8V and a maximum voltage of around 4.4V.

LiHV cells tend to fail after much fewer charge cycles than comparable standard lithium cells, so they are often one of the first things to fail in one of the HiBy OS devices. Getting a compatible LiHV cell to replace any failed ones is quite hard, due to how uncommon LiHV cells are, so it's most feasable to replace them with standard lithium cells.

### What Settings Need to Change

**The Settings that MUST Change:**
- Update maximum charge voltage (a.k.a. charge termination voltage)
    - e.g. from 4.4V to 4.2V
- Update recharge threshold voltage
    - e.g. from 4.3V to 4.1V

**The Settings that are Important (but not literally required):**
- Updating the CW2015 (fuel gauge) configuration
    - visible in the stock firmware as `fuel_gauge` in `/module_driver/cw2015.sh` is a proprietary array of 64 bytes that define battery details
    - this will make the calculated battery percentage actually be representative of the new cell
- Update the kernel low battery shutdown threshold
    - (TODO: is this actually applicable?)
- Update thermal charge profiles
    - (TODO: what exactly needs to change?)

### Applying These Changes

**Maximum Charge Voltage**:

Both the AXP2101 and the MP2731 have the maximum charge voltage (a.k.a charge termination voltage) as settings. These can be set directly using what's discussed above in this document.


**Recharge Threshold Voltage**:

(TODO: how is this configurable on the AXP2101?)
The MP2731 defines its recharge threshold voltage based on a configurable offset from the charge termination voltage (rather than defining its own voltage), therefore it should be safe to leave it unchanged from the previous setting.


**CW2015 Configuration**:

(TODO: more details on how to get the array + examples and such)
Find suitable values for the 64 byte array, then update `/module_drive/cw2015.sh` with the new information.
