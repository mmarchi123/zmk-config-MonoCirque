# MonoCirque - Standalone Cirque Trackpad

A ZMK firmware configuration for a standalone Cirque trackpad using the Seeed Studio XIAO BLE controller.

## Overview

This project creates a wireless Bluetooth trackpad using:
- **Seeed Studio XIAO BLE** (nRF52840) - Wireless controller
- **Cirque Pinnacle trackpad** - Touch surface with SPI interface
- **ZMK firmware** - Modern keyboard/input device firmware

The device appears as a standard Bluetooth mouse to your computer, providing cursor movement and optional tap-to-click functionality.

## Hardware Requirements

### Components
- 1x Seeed Studio XIAO BLE (seeeduino_xiao_ble)
- 1x Cirque Pinnacle trackpad with SPI interface
- Connecting wires (24-30 AWG recommended)
- Optional: PCB or breadboard for mounting

### Pin Connections

| Cirque Pin | Function | XIAO BLE Pin | Physical Pin |
|------------|----------|--------------|--------------|
| VCC        | Power    | 3V3          | Pin 11       |
| GND        | Ground   | GND          | Pin 12       |
| SCK        | Clock    | D8/SCK       | Pin 8        |
| MOSI       | Data Out | D10/MOSI     | Pin 10       |
| MISO       | Data In  | D9/MISO      | Pin 9        |
| CS         | Chip Sel | D7           | Pin 7        |
| DR         | Data Rdy | D6           | Pin 6        |

### Wiring Diagram
```
XIAO BLE                    Cirque Trackpad
Pin 11 (3V3)  ────────────→ VCC
Pin 12 (GND)  ────────────→ GND
Pin 8  (SCK)  ────────────→ SCK
Pin 10 (MOSI) ────────────→ MOSI
Pin 9  (MISO) ────────────→ MISO
Pin 7  (D7)   ────────────→ CS
Pin 6  (D6)   ────────────→ DR
```

## Firmware Features

- **Bluetooth HID mouse** - Standard mouse compatibility
- **Power management** - Sleep/wake functionality
- **Configurable sensitivity** - 4x sensitivity by default
- **SPI communication** - Hardware SPI for reliable data transfer
- **No keyboard matrix** - Pure trackpad operation

## Building the Firmware

### Prerequisites
- ZMK development environment
- West build tool
- Access to this repository

### Build Process
1. Fork or clone this repository
2. The GitHub Actions will automatically build the firmware
3. Download the generated `.uf2` file from the Actions artifacts

### Manual Build
```bash
west build -s zmk/app -b seeeduino_xiao_ble -- -DSHIELD=monocirque
```

## Flashing Instructions

1. **Enter bootloader mode:**
   - Double-tap the RESET button on the XIAO BLE
   - The board will appear as a USB mass storage device

2. **Flash firmware:**
   - Copy the generated `.uf2` file to the mounted bootloader drive
   - The board will automatically reboot with new firmware

## Usage

### Pairing
1. Power on the device (connect battery or USB)
2. The trackpad will appear as "MonoCirque Trackpad" in Bluetooth settings
3. Pair like any other Bluetooth mouse

### Operation
- **Cursor movement:** Touch and move finger on trackpad surface
- **Clicking:** Currently disabled (no-taps mode)
- **Power saving:** Automatically sleeps after 5 minutes of inactivity

### Enabling Tap-to-Click
To enable tapping as left-click, modify `monocirque.overlay`:
```diff
trackpad: trackpad@0 {
    compatible = "cirque,pinnacle";
    reg = <0>;
    spi-max-frequency = <1000000>;
    status = "okay";
    dr-gpios = <&xiao_d 6 (GPIO_ACTIVE_HIGH)>;
    sensitivity = "4x";
    sleep;
-   no-taps;
+   // no-taps;  // Commented out to enable tapping
};
```

## Configuration Options

### Trackpad Settings (`monocirque.overlay`)
- **Sensitivity:** `"1x"`, `"2x"`, `"4x"`, `"8x"`
- **Sleep mode:** `sleep;` or remove for always-on
- **Tapping:** `no-taps;` to disable, remove to enable

### Power Management (`MonoCirque.conf`)
```conf
# Idle timeout before sleep (5 minutes)
CONFIG_ZMK_IDLE_TIMEOUT=300000

# Deep sleep timeout (15 minutes)
CONFIG_ZMK_IDLE_SLEEP_TIMEOUT=900000
```

## Troubleshooting

### Device Not Detected
- Check all connections, especially power (VCC/GND)
- Verify SPI pins are correctly connected (pins 6-12 on XIAO BLE)
- Ensure firmware flashed successfully

### Poor Tracking or Cursor Issues
- Adjust cursor speed by changing scaler values in overlay
- Check for interference on SPI lines
- Verify trackpad surface is clean
- Confirm Y-axis inversion setting matches preference

### Clicking Not Working
- Verify taps are enabled (no `no-taps;` in trackpad config)
- Check if trackpad surface responds to light taps
- Ensure hardware sensitivity is appropriate (`"4x"`)

### Scroll Mode Issues
- Double-tap timing may need adjustment (currently 200ms)
- Practice double-tapping in same spot on trackpad
- Any single tap should exit scroll mode back to cursor mode
- Mode has no visual indicator - test by attempting to scroll vs move cursor

### Connection or Pairing Issues
- Clear Bluetooth pairing and re-pair device
- Check battery level if using battery power
- Verify `CONFIG_ZMK_MOUSE=y` is in configuration
- Restart host device's Bluetooth if needed

### Build Failures
- Ensure Cirque input module is properly included in `west.yml`
- Verify all devicetree syntax is correct in overlay files
- Check that input processor includes are present
- Confirm no duplicate configuration files exist

## Hardware Notes

### Power Consumption
- Active tracking: ~5-15mA
- Sleep mode: <1mA
- Deep sleep: <100µA

### SPI Communication
- Frequency: 1MHz (adjustable)
- Mode: Standard SPI (CPOL=0, CPHA=0)
- Wire length: Keep under 10cm for reliability

## Repository Structure

```
├── build.yaml              # Build configuration
├── config/
│   ├── west.yml            # West manifest with Cirque module
│   ├── MonoCirque.conf     # Firmware configuration
│   ├── MonoCirque.keymap   # Minimal keymap (1 key)
│   └── boards/
│       ├── seeeduino_xiao_ble.conf
│       └── shields/
│           └── MonoCirque/
│               ├── Kconfig.defconfig
│               ├── Kconfig.shield
│               ├── monocirque.conf
│               └── monocirque.overlay
└── README.md
```

## Credits

- **ZMK Firmware:** [zmkfirmware.dev](https://zmk.dev)
- **Cirque Input Module:** [petejohanson/cirque-input-module](https://github.com/petejohanson/cirque-input-module)
- **Original MonoCirque:** [diecuriousdnd/zmk-config-MonoCirque](https://github.com/diecuriousdnd/zmk-config-MonoCirque)

## License

This configuration is provided under the MIT License. See ZMK's license for firmware components.

## Support

For issues related to:
- **ZMK firmware:** [ZMK Discord](https://zmk.dev/community/discord/invite)
- **Hardware connections:** Check the troubleshooting section above
- **This configuration:** Open an issue in this repository