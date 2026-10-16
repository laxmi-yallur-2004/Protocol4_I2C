# Protocol 4: I2C 

## Goal
An I2C master driver (bit-banged) with:
- Timeouts on every wait
- Bus recovery (SCL toggling / soft reset)
- NACK handling
- Demonstrated recovery from an unplugged device and a stuck bus
- Validation with logic analyzer (LA) captures, checking clock/data alignment

---

## How the Code Covers Each Requirement

1. **START / STOP / Repeated START**
   - Implemented in `i2c_start()`, `i2c_stop()`, and the repeated START demo in `loop()`
   - Log: `START OK`, `STOP OK`, `Repeated START demo`

2. **Clock stretching awareness**
   - Implemented in `waitSCLHigh()`
   - Log: `FAIL: Clock stretching timeout - SCL stuck low`

3. **Stuck bus (SDA/SCL low) and recovery**
   - `i2c_start()` checks `SDA_READ()` / `SCL_READ()`
   - `busRecovery()` sends 9 clock pulses
   - Log: `FAIL: Bus stuck SDA low / SCL low` -> `PASS: Bus recovered`

4. **Timeouts on every wait**
   - `TIMEOUT_MS = 50` for SCL
   - `SDA_LOW_TIMEOUT = 5000`
   - No `delay()`, only `millis()` and `delayMicroseconds(5)`

5. **NACK handling**
   - `i2c_writeByte()` checks ACK on the 9th clock
   - Log: `NACK - Device unplugged?`

6. **Recovery from unplugged device**
   - NACK is detected, STOP is issued, master continues
   - Log: `PASS: NACK handling - Unplugged device`

---

## Output Proof (No Device Connected)

```
Protocol 4 I2C Master Ready - No delay()
-> Bus Recovery Started
PASS: Bus recovered

--- Sending: 1
START OK
NACK - Device unplugged?
PASS: NACK handling - Unplugged device
STOP OK

--- Sending: 2
START OK
NACK - Device unplugged?
PASS: NACK handling - Unplugged device
STOP OK





1. START / STOP conditions (SDA transition while SCL is high)
2. The 9 recovery clock pulses on SCL
3. The NACK on the 9th clock (SDA high)
