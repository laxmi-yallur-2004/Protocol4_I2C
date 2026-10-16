# Protocol 4: I2C (SDA / SCL)

## Goal
An I2C master driver (bit-banged) with:
- Timeouts on every wait
- Bus recovery (SCL toggling / soft reset approach)
- NACK handling
- Demonstrated recovery from an unplugged device and a stuck bus scenario

---

## Concepts Covered (Must Learn)

1. **START / STOP / Repeated START**
   - START: SDA goes high to low while SCL is high
   - STOP: SDA goes low to high while SCL is high
   - Repeated START: a new START without a STOP in between
2. **ACK / NACK**
   - After 8 data bits, the receiver pulls SDA low on the 9th clock for ACK
   - SDA left high on the 9th clock means NACK
3. **Clock stretching awareness**
   - A slave can hold SCL low to slow the master down
   - The master must wait for SCL to go high, with a timeout
4. **Arbitration concept (even if single master)**
   - If two masters drive the bus at the same time, each one checks SDA while sending
   - A master that sends a 1 but reads a 0 has lost arbitration and must stop driving the bus
   - This project has a single master, so arbitration is never triggered; it is explained for understanding only
5. **Bus stuck conditions and recovery**
   - A slave can hold SDA low in the middle of a byte after a reset or glitch
   - Fix: toggle SCL up to 9 times until SDA is released, then send a STOP
6. **Timeouts for every wait**
   - No wait loop can run forever

---

## How the Code Covers Each Requirement

1. **START / STOP / Repeated START**
   - Implemented in `i2c_start()`, `i2c_stop()` and the repeated START demo in `loop()`
   - Log: `START OK`, `STOP OK`, `Repeated START demo`

2. **Clock stretching awareness**
   - Implemented in `waitSCLHigh()`
   - Log: `FAIL: Clock stretching timeout - SCL stuck low`

3. **Stuck bus (SDA/SCL low) and recovery**
   - `i2c_start()` checks `SDA_READ()` / `SCL_READ()` before starting
   - `busRecovery()` sends up to 9 clock pulses
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

## Hardware Setup
- Arduino board + USB cable only
- No I2C device is needed: "unplugged device" means nothing is on the bus
- Pull-ups on SDA and SCL (internal pull-ups, or external 4.7k for real hardware)

---

## Output Proof

### Log 1: Normal + Unplugged Device (PASS)

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
```

### Log 2: Stuck Bus Fault + Recovery (PASS)

```
--- Sending: 3
START OK
ACK OK
ACK OK
Repeated START demo
FAIL: Bus stuck SDA low / SCL low
-> Bus Recovery Started
PASS: Bus recovered
NACK - Device unplugged?
STOP OK
PASS: Cycle complete

--- Sending: 4
START OK
NACK - Device unplugged?
PASS: NACK handling - Unplugged device
STOP OK
```

### Fault Injection
1. Connect SDA to GND with a jumper wire while the program is running (stuck bus scenario)
2. The master detects the stuck bus and starts recovery
3. Remove the jumper wire
4. Recovery succeeds (`PASS: Bus recovered`) and normal transfers continue

Note: the `ACK OK` lines in Log 2 appear because SDA is held low by the jumper wire, so the master reads a low level on the 9th clock. This is a side effect of the fault injection, not a real device.

### Fault Still Present (Extra Case)
- If the jumper stays connected, recovery times out with `FAIL: Clock stretching timeout - SCL stuck low`
- The master does not hang: each attempt returns in about 50 ms and retries
- `PASS: Bus recovered` is never printed while the line is still held low

---

## Result

**PASS.** The master never hangs and recovers from every fault:
- Stuck bus (recovered after the fault is removed)
- Clock stretching timeout
- NACK from a missing device
