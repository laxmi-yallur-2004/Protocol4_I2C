# I2C Master Driver -(Protocol 4)

I2C master driver with:

- **Timeouts everywhere:** The data read waits at most 100 ms. If 2 bytes are not received, it is reported as `I2C: FAIL TIMEOUT`.
- **Bus recovery (SCL toggling / soft reset approach):** `I2C_BusRecovery()` stops the bus, toggles SCL 9 times, then restarts the bus.
- **NACK handling:** When the PSU does not acknowledge (NACK), the LCD shows `ERR 3` for voltage and current, and `I2C: FAIL NACK`.

The final result is **PASS** only when the I2C communication works and the readings are in range.

## How it works

1. Turns the PSU ON using pin `PA2` and waits 5 seconds.
2. Every 1 second, reads voltage (`0x8B`) and current (`0x8C`) from the PSU at I2C address `0x03`.
3. If a read fails (NACK, timeout, bus error), the bus is recovered automatically.
4. Shows the readings and the PASS/FAIL result on the LCD.

## Connections

| Signal | Pin |
|--------|-----|
| I2C SDA | `PB11` |
| I2C SCL | `PB10` |
| PSU ON | `PA2` |
| LCD backlight | `PA11` |
| LCD (RS, RW, E) | `PC8`, `PC9`, `PC10` |
| LCD data (D0-D7) | `PC0` - `PC7` |

## Pass limits

- Voltage: 22.0 V to 28.0 V
- Current: 0.05 A or more
- I2C: voltage and current must both be read OK

## LCD output

**PASS (device connected)**

```
Voltage: 24.73 V
Current: 0.27 A
I2C: PASS ACK OK
TEST: PASS
```

**FAIL (device not responding / unplugged)**

```
Voltage: ERR 3
Current: ERR 3
I2C: FAIL NACK
TEST: FAIL COMM
```

## Libraries

`Wire`, `LiquidCrystal`
