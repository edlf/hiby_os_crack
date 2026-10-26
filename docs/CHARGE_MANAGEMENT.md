# Charge Management
**Managing the charge voltage and amperage of the HiBy OS players**

*Plus information about handling replacement cells of different voltages than stock*


## Device Differences
The R1 uses the AXP2101 charge controller to manage all of the device's charging needs **(TODO: i think the R3II 2025 uses the exact same charging setup as the R1)**.

The R3Pro II uses the MP2731 to manage it's battery (voltage, charging amperage, etc.). Despite this, the R3Pro II also includes an AXP2101. I do not know what the R3Pro II does with its AXP2101, since it seems that all battery needs are managed entirely by the MP2731.


## [MP2731](docs/third_party/components/MP2731GQC.pdf)

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

### Limiting Voltage
(TODO)

### Limiting Charge Amperage
(TODO)


## Safely Handling Replacement Cells

> [!NOTE]
> This document will not discuss how to perform the battery replacement itsself, as that is out of scope. Here, we will only be talking about how to update the device configurations so that the new battery can be handled safely.
> 
> (TODO: make battery replacement guide)

The HiBy OS devices (at least all that I have checked) use LiHV cells, which are quite uncommon. LiHV cells have a maximum voltage of around 4.4V, compared to the 4.2V maximum voltage of a standard lithium cell.

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
