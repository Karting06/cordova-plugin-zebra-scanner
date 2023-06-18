# Zebra Barcode Scanner Plugin [Android]
This plugin allows you to communicate with Zebra scanners over bluetooth 2.0 (possibly USB as well) on Android.

It uses the Zebra SDK version 2.6.16.0.

## Requirements
* Cordova 5.0.0 or higher
* Android 11 or higher - target Android API 33

## Limitations
* iOS is not supported. It looks like I won't get more time allocated to develop iOS part.
  However this plugin can be used instead: https://github.com/shanghai-tang/cordova-plugin-zebra-scanner
* Only one scanner can be connected. The Zebra SDK does not support multiple connections.
* The Zebra SDKHandler is automatically initialized when the plugin starts.


## Install
`cordova plugin add https://github.com/PreciousBiscuit/cordova-plugin-zebra-scanner.git`

## Overview
These bluetooth barcode scanners are supported (in `SSI BT Classic (Discoverable)` mode):
```
• CS4070
• LI4278
• DS6878
• RFD8500
• DS3678
• LI3678
• DS8178 (tested)
```
These USB barcode scanners should be supported as well. The Presentation Cradle should be in `Symbol Native API (SNAPI) with Imaging Interface` mode (see your barcode scanner manual).
```
• PL3307
• DS457
• DS4308
• LS2208 (if it has the silver Symbol logo and manufactured after 31 December 2014, it should work)
• DS6878 and Presentation Cradle
• MP6210 (CSS + Scale) + EAS (Sensormatic)
• DS8178 and Presentation Cradle (tested)
```

In theory, all versions compatible with SDK version 2.6.16.0 should work. [See Zebra Docs](https://techdocs.zebra.com/dcs/scanners/sdk-android/about/).

## Methods
* [zebraScanner.startScan](#startscan)
* [zebraScanner.stopScan](#stopscan)
* [zebraScanner.getAvailableDevices](#getavailabledevices)
* [zebraScanner.getActiveDevices](#getactivedevices)
* [zebraScanner.getPairingBarcode](#getpairingbarcode)
* [zebraScanner.connect](#connect)
* [zebraScanner.disconnect](#disconnect)
* [zebraScanner.subscribe](#subscribe)
* [zebraScanner.unsubscribe](#unsubscribe)

### startScan
Starts scanning for bluetooth and USB connected devices. Scanning is expensive so call stopScan() as soon as
possible. The method returns all discovered and paired devices.

```javascript
zebraScanner.startScan(successCallback, errorCallback)
```

##### Success
```
{
  "status": <string>
  ["device"]: {
    "id": <number>
    ["name"]: <string>
    ["model"]: <string>
    ["serialNumber"]: <string>
  }
}
```
* status - One of these: ["scanStart", "scanStop", "deviceFound", "deviceLost"]
* device - Available for statuses "deviceFound" and "deviceLost"
  * name - Available for status "deviceFound"
  * model - Available for status "deviceFound"
  * serialNumber - Available for status "deviceFound"

##### Errors
* "Scanning is already in a progress." -- startScan() was already called but stopScan() was not


### stopScan
Stops scanning for bluetooth devices.

```javascript
zebraScanner.stopScan(successCallback, errorCallback)
```

##### Success
"ok"

##### Errors
* "Scanning was not in a progress." -- startScan() was not called


### getAvailableDevices
Returns all bluetooth (possibly USB connected) devices found by Zebra SDK including paired devices.
Zebra SDK searches for devices non-stop and stores all devices it had found. This method returns all stored devices.

```javascript
zebraScanner.getAvailableDevices(successCallback, errorCallback)
```

##### Success
An array of devices
```
[
  "device": {
    "id": <number>
    "name": <string>
    "model": <string>
    "serialNumber": <string>
    "connectionType": <string>
  },
  ...
]
```

##### Errors
None


### getActiveDevices
Returns all connected devices. Zebra SDK supports only one connection so this method returns maximum one device.

```javascript
zebraScanner.getActiveDevices(successCallback, errorCallback)
```

##### Success
An array of devices
```
[
  "device": {
    "id": <number>
    "name": <string>
    "model": <string>
    "serialNumber": <string>
    "connectionType": <string>
  }
]
```

##### Errors
None


### getPairingBarcode
Returns barcode for pairing devices. The barcode is JPG encoded in base64.

```javascript
zebraScanner.getPairingBarcode(successCallback, errorCallback)
```

##### Success
A string with base64 encoded JPG

##### Errors
None


### connect
Connects to a device. The device needs to be found by Zebra SDK before it is possible to connect.
Call getAvailableDevices() to check if the device was already found or use startScan() to search for it.
It is possible to connect only to a single device. If a connection to a device is interrupted
 errorCallback will be called with a message "Disconnected".

```javascript
zebraScanner.connect(successCallback, errorCallback, params)
```

##### Params
```
{
  "deviceId": <number>
}
```
* deviceId - ID of a device retrieved from methods startScan() or getAvailableDevices()

##### Success
```
{
  "status": "connected"
  "device": {
    "id": <number>
  }
}
```

##### Errors
* "Missing parameters" -- Params were not provided
* "Invalid parameter - deviceId" -- Parameter 'deviceId' was not a valid number
* "Scanner is not available." -- Device was not found by Zebra SDK
* "Already connected to a scanner." -- Connection to a scanner is already established
* "Unable to connect to a scanner." -- Connection to a scanner was not successful
* "Disconnected" -- Device was disconnected; either by calling disconnect() or by itself


### disconnect
Disconnects from a device.

```javascript
zebraScanner.disconnect(successCallback, errorCallback, params)
```

##### Params
```
{
  "deviceId": <number>
}
```
* deviceId - ID of a device retrieved from methods startScan() or getAvailableDevices()

##### Success
"ok"

##### Errors
* "Missing parameters" -- Params were not provided
* "Invalid parameter - deviceId" -- Parameter 'deviceId' was not a valid number
* "Scanner is not available." -- Device was not found by Zebra SDK
* "Never connected to a scanner." -- Connection to a scanner wasn't established
* "Unable to disconnect from a scanner." -- Disconnect from a scanner was not successful


### subscribe
Subscribes for a barcode events.

```javascript
zebraScanner.subscribe(successCallback, errorCallback)
```

##### Success
```
{
  "deviceId": <number>
  "barcode": {
    "type": <string>,
    "data": <string>
  }
}
```
* barcode
  * type - Type of a barcode that was read; It could be for example 'QR Code', 'EAN 128', or 'Code 128'.
  * data - Data that was read from a barcode.

##### Errors
* "No connected scanner" -- No device is connected


### unsubscribe
Unsubscribes from a barcode events. Events are unsubscribed if a device is disconnected.

```javascript
zebraScanner.unsubscribe(successCallback, errorCallback)
```

##### Success
"ok"

##### Errors
* "No active subscription" -- subscribe() was not called
