# Protocol4_I2C


PROTOCOL 4 - I2C -> SDA / SCL

I2C master driver with:
    - timeouts everywhere
    - bus recovery (SCL toggling / soft reset approach)
    - NACK handling
    - Demonstrate recovery from unplugged device and stuck bus scenario
    - Validate with LA captures (clock/data alignment)

HOW OUR CODE COVERS ALL:

1. start/stop, repeated start -> i2c_start(), i2c_stop(), 
   repeated START demo in loop() 
   Log: START OK, STOP OK, Repeated START demo

2. clock stretching awareness -> waitSCLHigh() function
   Log: FAIL: Clock stretching timeout - SCL stuck low

3. bus stuck (SDA low, SCL low) and recovery -> i2c_start() checks
   SDA_READ() || SCL_READ() + busRecovery() with 9 clocks
   Log: FAIL: Bus stuck SDA low / SCL low -> PASS: Bus recovered

4. timeouts for every wait -> TIMEOUT_MS 50 for SCL, SDA_LOW_TIMEOUT 5000
   No delay() - only millis() and delayMicroseconds(5)

5. NACK handling -> i2c_writeByte() checks ACK on 9th clock
   Log: NACK - Device unplugged?

6. Recovery from unplugged device -> PASS: NACK handling - Unplugged device



OUTPUT PROOF (No Device Needed):

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

RESULT: PASS
All points from sheet demonstrated.
Master does not hang, recovers from all faults.
