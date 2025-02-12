# pycoolmasternet-ng

A Python 3.9+ library for interacting with a [CoolMasterNet](https://coolautomation.com/products/coolmasternet/) HVAC bridge.
This is a fork of [pycoolmasternet-async](https://github.com/OnFreund/pycoolmasternet-async) with significant changes:

## Pluggable transports

Transports - methods that define communication with the CoolMasterNet device - are now in classes. TCP/IP and RS232 is implemented, REST is TODO, and maybe at some point CoolAutomation's cloud-based service can be reverse-engineered and implemented too (not sure if their cloud service just forwards commands or if it operates at a higher level).

## More "Pythonic"

Most values and data objects are mapped to Python classes - exceptions, enums, etc.

## Autodiscovery of device capabilities

Device capabilities are now inferred from "properties" saved in the CoolMasterNet unit. This means the CM device holds all configuration and the home automation controllers can just autodiscover them. See the CoolMasterNet manual on how to set these (you'll need to manually Telnet/serial in to set them first).

## Device capability database

There's a (work-in-progress, currently Mitsubishi Electric only) database of device capabilities such as allowed temperature ranges, etc - this would allow consumers of this library (such as home automation integrations) to autodiscover those and customize their UI accordingly.

## TODO

* synchronous (blocking) interface
* improve the device DB for non-Mitsubishi-Electric units
* REST transport
* support advanced CoolMasterNet functionality (I only have the CoolLinkHub with CoolPlugs so can't test that yet)

## Installation

You can install pycoolmasternet-async from [PyPI](https://pypi.org/project/pycoolmasternet-ng/):

    pip3 install pycoolmasternet-ng

Python 3.9 and above are supported. As this is primarily designed for use in Home Assistant, support for prior Python versions isn't a priority.

Extra protocols such as serial require additional dependencies:

    pip3 install pycoolmasternet-ng[serial]

## Basic usage

```python
from pycoolmasternet_ng import transports, models, structures, constants

# first create a transport - TCP/IP (CoolMasterNet calls this "aserver")
ts = transports.TCPTransport("192.168.0.123", port=12345)

# or Serial
ts = transports.SerialTransport('/dev/ttyS0', 9600)

# now create a gateway connected to your transport
# this will internally populate a cache of models.Device objects with their current state

gw = await models.Gateway.from_transport(transport=ts)

# a dictionary of structures.UID: models.Device
devices = gw.devices

# get device with UID L5.001
device = devices[structures.UID.from_string('L5.001')]

# get device info

device.power_state  # True
device.mode  # constants.Mode.COOL
device.fan_mode  # constants.FanMode.LOW
device.louver_position  # constants.LouverPositionState.NOT_SUPPORTED
device.target_temperature  # Decimal('21')
device.current_temperature  # Decimal('22')
device.temperature_unit  # 'C'
device.error_code  # None
device.demand  # True


# refresh that device - this will talk to the gateway to query the device's status and update the object
await device.refresh()

# now check the updated values

device.current_temperature  # Decimal('21')
device.demand  # False


# turn device off
await device.set_power_state(False)

# see the updated status
device.power_state  # False


# make multiple changes
await device.set_mode(constants.Mode.HEAT, refresh=False)
await device.set_temperature(23, refresh=False)
await device.set_power_state(refresh=False)

# note that device.power_state is still False as we haven't refreshed
device.power_state  # False

# now refresh explicitly
await device.refresh()

# now device.power_state is up to date
device.power_state  # True

# refresh all devices on the gateway
await gw.refresh()

# now gw.devices has been rebuilt with updated models.Device objects
# note that the objects are replaced and not updated in-place

devices[structures.UID.from_string('L5.001')] is device  ## False
```

## Credits

* [koreth's](https://github.com/koreth) [pycoolmasternet](https://github.com/koreth/pycoolmasternet)
* [OnFreund's](https://github.com/OnFreund) fork [pycoolmasternet-async](https://github.com/OnFreund/pycoolmasternet-async)
