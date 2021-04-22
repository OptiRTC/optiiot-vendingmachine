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

## Functional Requirements

- Order coffee in 3 sizes (small for $1.75, medium for $2.00, large for $2.25).
- Allow multiple coffee orders per payment transaction, up to 5 coffees.
- Allow adding money in standard monetary increments only (from $0.05 to $20).
- Allow user to specify the end of a payment transaction and "Dispense" coffee and change.
- “Dispense” coffee when an order is completed if adequate payment has been provided and display money remaining in payment transaction. The developer can assume that the physical implications of this dispense operation always succeed and do not need to be verified - in this context, "dispense" means complete the order.
- Insertion of money is communicated to device by serial message
- Device sends updates to user via events sent via Serial communications
- At start up display message of the day on LCD

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
- Message of the day value is written to LCD by sending `setLCD` message.  The ASCII string
  to send with `setLCD` will be evaluated using last received `msgOfDay` template value and
  replacing each occurrence of `%KEY%` with the corresponding value for that `KEY` where `KEY`
  is one of the supported keys excluding `msgOfDay` and `setLCD`.  If `KEY` is an unknown key
  or `msgOfDay` or `setLCD`, then it will not be replaced.
- When money is added,
   - display message of added value and accumulated `curFunds` on LCD using `setLCD`
   - send message for `curFunds`
- When coffee is added
   - display message on LCD showing the 3 sizes of coffee and their counts using `setLCD`
   - send message for `order`
- When dispense is triggered and there are insufficient funds:
   - display human readable insufficient funds message on LCD
   - send message for `insFunds`
- When dispense is triggered and there are sufficient funds:
   - display `receipt` messaged on LCD
   - send messages for `receipt`, `order`, `refund`,
   - send messages for `curFunds` and `order` (reset their values)
   - wait 3 seconds and reset LCD to start up display message (message of the day)
- When cancel is triggered:
   - display `refund` message on LCD
   - send message for `cancel` and `refund`
   - send messages for `curFunds` and `order` (reset their values)
   - wait 3 seconds and reset LCD to start up display message (message of the day)

## Opti Vending Machine Message Protocol

- Each serial message is 3 fields:
  - A `KEY` field of 8 bytes
  - A `VALUESIZE` field of 2 bytes
  - A `VALUE` field of N bytes where N is the number of bytes defined by valueSize

### Supported Keys:

Messages received by device:
- `msgOfDay`: Specifies the message of the day template. Payload is ASCII string.
- `addValue`: Specifies value of money inserted. Payload is Int32 representing value in cents

Messages sent by device:
- `setLCD`: Specifies LCD display message. Payload is ASCII string.
- `curFunds`: Specifies the accumulated money inserted. Payload is Int32 representing value in cents
- `order`: Specifies number of small, medium and large coffees respectively. Payload is 3 Int32 values in sequence of small, medium and large coffees.
- `insFunds`: Sent when dispense is triggered and there is insufficient funds for coffees in order.
- `receipt`: Sent when dispense is triggered and there is sufficient funds for coffees in order. Payload is human readable summary of order. Include coffee counts, subtotals, total, and refund.
- `cancel`: Sent when cancel is triggered. No payload
- `refund`: Specifies refund after dispense or cancel is triggered. Payload is Int32 specifiying value in cents.
