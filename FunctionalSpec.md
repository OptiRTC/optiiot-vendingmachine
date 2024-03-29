# Opti Embedded Vending Machine Functional Specification #

## Purpose/Responsibilities
A C++11 application that mimics the functionality of a coffee vending machine. Provides the means of ordering one or more customizable cups of coffee, providing payment, receiving change, and dispensing the correct product.

## Assumptions and Constraints
- Compiles on [build.particle.io](https://build.particle.io) (you will need to create a free account to make this work). Note that you DO NOT need to buy a Particle device as part of this exercise - the goal is to write a firmware application against the Particle [Cloud](https://docs.particle.io/reference/api/) and [Firmware](https://docs.particle.io/reference/firmware/electron/) APIs as documented).
	- Note: we are not asking you to purchase a Particle device to actually run this code.
- Serial communications will be sent & received via [Serial](https://docs.particle.io/cards/firmware/serial/serial/) device using **Opti Vending Machine Message Protocol** defined below.
- Does not depend on any external web services.
- Does not require any persistent data storage.
- Only manages 1 coffee order at once. One order may consist of multiple coffees of different sizes. You can assume whatever physical apparatus is connected to the other end of this service prevents more than 1 user from using it at once.

## Useful Particle APIs

The following is a list of APIs we anticipate you will need to implement your solution.

- [digitalread()](https://docs.particle.io/cards/firmware/input-output/digitalread/) to read value of a button's pin
- [millis()](https://docs.particle.io/cards/firmware/time/millis/) to measure time
- [Serial](https://docs.particle.io/cards/firmware/serial/serial/)
  - [Serial.begin(9600)](https://docs.particle.io/cards/firmware/serial/begin/) to setup serial connection
  - [Serial.available()](https://docs.particle.io/cards/firmware/serial/available/)
    and [Serial.read()](https://docs.particle.io/cards/firmware/serial/read/) to read incoming messages
  - [Serial.write()](https://docs.particle.io/cards/firmware/serial/write/) to write messages

## Functional Requirements

- Order coffee in 3 sizes (small for $1.75, medium for $2.00, large for $2.25).
- Allow multiple coffee orders per payment transaction, up to 5 coffees.
- Allow adding money in standard monetary increments only (from $0.05 to $20).
- Allow user to specify the end of a payment transaction and "Dispense" coffee and change.
- “Dispense” coffee when an order is completed if adequate payment has been provided and display money remaining in payment transaction. The developer can assume that the physical implications of this dispense operation always succeed and do not need to be verified - in this context, "dispense" means complete the order.
- Insertion of money is communicated to device by serial message
- Device sends updates to user via events sent via Serial communications
- At start up, wait for message of the day template and then display message of the day on LCD

## Implementation Requirements

- Small Button is GPIO PIN A0
- Medium Button is GPIO PIN A1
- Large Button is GPI0 PIN A2
- Dispense/Cancel Button GPIO Pin A3
   - Dispense will be triggered by press & release
   - Cancel will be triggered by press & hold > 2 seconds
- Initialized state values (before messages arrive by Serial):
  - `msgDay = '' //empty ASCII string`
  - `order = {0x0, 0x0, 0x0} // zero small, zero medium, zero large coffees`
  - `curFunds = 0 //zero cents`
- Message of the day value is written to LCD by sending `setLCD` message.
  - The ASCII string to send with `setLCD` will be evaluated using last received `msgOfDay`
    template value and replacing each occurrence of `%KEY%` with the corresponding value for
    that `KEY`.
  - Each `%KEY%` will be matched and replaced left to right
  - If `KEY` is an unknown key, then it will not be replaced.
  - Known Keys and corresponding value to render (as base 10 string)
    - `smallCount`: the count of small coffees in order
    - `mediumCount`: the count of medium coffees in order
    - `largeCount`: the count of large coffees in order
    - `curFunds`: the accumulated money inserted (simply a base 10 string of cents accumulated)
  - Example: `Order: %smallCount%S %mediumCount%M %largeCount%L, Funds: %curFunds%`.
    * If `smallCount`, `mediumCount`, `largeCount` and `curFunds` are `1`, `2`, `3`
      and `625` respectively, then the expected output is `Order: 1S 2M 3L, Funds: 625`.
      Note: `curFunds` in this examples is less than funds required for order.
- When money is added,
   - first display human readable message of added value and accumulated `curFunds` on LCD using `setLCD`
   - then send message for `curFunds`
- When coffee is added
   - first display human readable message on LCD showing the 3 sizes of coffee and their counts using `setLCD`
   - then send message for `order`
- When dispense is triggered and there are insufficient funds:
   - first display human readable insufficient funds message on LCD
   - then send message for `insFunds`
- When dispense is triggered and there are sufficient funds:
   - first display human readable receipt message on LCD by sending `setLCD` message
   - then send messages for `receipt`, `order`, `refund` (in this order)
   - then reset state for `curFunds` and `order`
   - then send messages for `curFunds` and `order` (in this order)
   - then wait 3 seconds
   - then reset LCD with message of the day (using current `msgOfDay` template)
- When cancel is triggered:
   - first display human readable refund message on LCD by sending `setLCD` message
   - then send message for `cancel` and `refund` (in this order)
   - then reset state for `curFunds` and `order`
   - then send messages for `curFunds` and `order` (in this order)
   - then wait 3 seconds
   - then reset LCD with message of the day (using current `msgOfDay` template)
- `uint16_t` & `uint32_t` are little endian in messages sent and received
- Assume device hardware is little endian

## Opti Vending Machine Message Protocol

- Each serial message is 3 fields:
  - A `KEY` field of 8 bytes
    - If `Key` is less than 8 bytes, it will padded with `0x0`
  - A `VALUESIZE` field of 2 bytes (`uint16_t`)
  - A `VALUE` field of N bytes where N is the number of bytes defined by valueSize

### Supported Keys:

Messages received by device:
- `msgOfDay`: Specifies the message of the day template. Payload is ASCII string.
- `addValue`: Specifies value of money inserted. Payload is `uint32_t` representing value in cents

Messages sent by device:
- `setLCD`: Specifies LCD display message. Payload is ASCII string.
- `curFunds`: Specifies the accumulated money inserted. Payload is `uint32_t` representing value in cents
- `order`: Specifies number of small, medium and large coffees respectively. Payload is 3 `uint32_t` values in sequence of small, medium and large coffees.
- `insFunds`: Sent when dispense is triggered and there is insufficient funds for coffees in order. No payload. (`VALUESIZE is 0`)
- `receipt`: Sent when dispense is triggered and there is sufficient funds for coffees in order. Payload is human readable summary of order. Include coffee counts, subtotals, total, and refund.
- `cancel`: Sent when cancel is triggered. No payload. (`VALUESIZE is 0`)
- `refund`: Specifies refund after dispense or cancel is triggered. Payload is `uint32_t` specifiying value in cents.
