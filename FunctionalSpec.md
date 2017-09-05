# Opti Embedded Vending Machine Functional Specification #

## Purpose/Responsibilities
A C++11 application that mimics the functionality of a coffee vending machine. Provides the means of ordering one or more customizable cups of coffee, providing payment, receiving change, and dispensing the correct product.

## Assumptions and Constraints
- Compiles on build.particle.io (you will need to create a free account to make this work). Note that you DO NOT need to buy a Particle device as part of this exercise - the goal is to write a firmware application against the Particle [Cloud](https://docs.particle.io/reference/api/) and [Firmware](https://docs.particle.io/reference/firmware/electron/) APIs as documented).
- Receives input via messages sent to the device via the Particle event messaging system (the device must subscribe to these events as part of its initialization with Particle.subscribe()).
- Returns output via messages sent to the Particle cloud via Particle.publish(). Note that  all messages sent over [Particle.publish()](https://docs.particle.io/reference/firmware/electron/#particle-publish-) and [Particle.subscribe()](https://docs.particle.io/reference/firmware/electron/#particle-subscribe-) can be up to 255 bytes in size!
- Does not depend on any external web services other than Particle.io for input/output as described above.
- Runs in Automatic device mode - you do not need to be responsible for managing device connectivity. You can assume that while your firmware is running, the device has a connection to the cloud.
- All communications with the cloud succeed.
- Does not require any persistent data storage - any partially-complete transactions during a device reboot (e.g. power cycle) should be discarded.
- Only manages 1 coffee order at once. You can assume whatever physical apparatus is connected to the other end of this service prevents more than 1 user from using it at once.

## Functional Requirements
- Order coffee in 3 sizes (small for $1.75, medium for $2.00, large for $2.25).
- Allow multiple coffee orders per payment transaction, up to 5 coffees.
- Allow adding money in standard monetary increments only (from $0.05 to $20).
- Allow user to specify the end of a payment transaction and "Dispense" coffee and change.
	- “Dispense” coffee when an order is completed if adequate payment has been provided and display money remaining in payment transaction. The developer can assume that the physical implications of this dispense operation always succeed and do not need to be verified - in this context, "dispense" means complete the order.
- Update the user via events sent to Particle.publish() regarding the current state of the order. Give warning if the user attempts to "Dispense" without adding adequate payment.
- Document the different HTTP calls that drive this system.

## Bonus points
- Developer should provide a minimum set of passing unit tests.
