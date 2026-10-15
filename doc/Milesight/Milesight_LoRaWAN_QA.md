# Milesight and LoRaWAN: Questions and Answers

## 1. What does Milesight do?

Milesight makes connected devices and systems for businesses and cities. Its products include AI-enabled security and traffic cameras, LoRaWAN sensors and gateways, IoT controllers, and industrial cellular routers. These products are used in areas such as smart buildings, parking, traffic monitoring, and environmental sensing.

## 2. What is LoRaWAN?

LoRaWAN is a wireless communication protocol and open standard for low-power devices that need to send small amounts of data over long distances. It is commonly used by battery-powered sensors.

## 3. Is LoRaWAN a company name?

No. LoRaWAN is a protocol, not a company. The LoRa Alliance maintains the LoRaWAN standard. Milesight is one of many companies that makes products that use it.

## 4. What is a LoRaWAN sensor?

A LoRaWAN sensor is a device that measures something, such as temperature, humidity, or water level, and sends its readings using LoRaWAN. Usually, it sends the data to a nearby gateway, which forwards it to a network server or monitoring application.

## 5. What is the difference between LoRa and LoRaWAN?

LoRa is the radio technology used to transmit data over long distances with low power. LoRaWAN is the communication protocol and network standard that describes how devices use that radio link to communicate.

## 6. How do the parts work together?

A sensor measures a condition and sends data over LoRaWAN. A gateway receives the radio message and passes it to a network server. An application can then display the reading or trigger an alert.

**Example:** A Milesight temperature sensor can report a room's temperature through a LoRaWAN gateway to a building-monitoring system.

## 7. What format is the sensor data sent in?

There is no single payload format used by every LoRaWAN sensor. Many Milesight sensors send a compact binary payload. In a device's documentation or network console, the raw payload is often shown as hexadecimal bytes. A decoder for that exact sensor model interprets the bytes and can convert them into readable fields, often represented as JSON.

For example, Milesight's EM300-TH decoder documents this payload:

```text
01 75 5C  03 67 34 01  04 68 65
```

It decodes to:

```json
{
	"battery": 92,
	"temperature": 30.8,
	"humidity": 50.5
}
```

In this model's payload, `01 75` identifies a battery reading and `5C` is 92%. `03 67` identifies temperature; the next two bytes, `34 01`, are read in little-endian order and scaled by 10, giving 30.8 °C. `04 68` identifies humidity; `65` is scaled by 2, giving 50.5% relative humidity. These identifiers and scaling rules are model-specific; do not assume another sensor uses the same layout.

## 8. How is the data transmitted?

1. The sensor measures a value and prepares an uplink message, usually at a configured reporting interval or when an event occurs.
2. It sends the message over LoRa radio to one or more gateways in range.
3. A gateway forwards the received message over its internet or cellular connection to a LoRaWAN network server.
4. The network server handles the LoRaWAN connection and security, then forwards the application payload to an integration or platform.
5. A model-specific payload decoder turns the raw bytes into named readings. The application can store, display, or act on those readings.

The application may also receive metadata such as the device ID, receive time, radio signal strength (RSSI), and signal-to-noise ratio (SNR). Those are generally message metadata, separate from the sensor's measurement bytes. The exact fields and delivery method depend on the gateway, network server, and application integration.

## 9. How do I find a sensor's data format and device ID?

1. **Identify the exact sensor model.** Check the product label or QR code, the purchase record, or the device details in Milesight ToolBox. A family name alone may not be enough; note the full model and firmware version.
2. **Find that model's payload decoder.** Use its Milesight user guide or search the [Milesight SensorDecoders repository](https://github.com/Milesight-IoT/SensorDecoders). Look under the matching product-series folder for a README or decoder file. Check whether the document distinguishes firmware versions or payload types.
3. **Find the raw uplink.** In your LoRaWAN network server or IoT platform, open the device's uplink/event log. Look for the raw `data` or `frm_payload` field, usually shown as hexadecimal or Base64, along with the application port (`fPort`). Apply the decoder for the exact model; if your platform has a payload formatter or codec setting, configure it there.
4. **Find the device ID.** For LoRaWAN, the identifier commonly used to register and track a sensor is its `DevEUI`. Read it from the device label or QR code, Milesight ToolBox, or the device record in your network server. Use `DevEUI` to associate uplinks with the sensor.

Do not confuse `DevEUI` with `DevAddr`: a `DevAddr` is a network address assigned for a LoRaWAN session and can change. A manufacturer's serial number is also a different identifier. Never share the device's application key or other join credentials publicly.

The exact payload decoder cannot be selected reliably without the full sensor model and, in some cases, its firmware version. Raw payload bytes are measurements encoded for that model; the device ID and radio metadata are usually separate fields in the uplink record.

## 10. How does a sensor connect to a gateway?

The sensor sends short radio messages using LoRa. A LoRaWAN gateway within radio range receives them and forwards them over its internet or cellular connection to the LoRaWAN network server. The sensor does not normally connect to the gateway with a cable, Wi-Fi, or Bluetooth, and it is not usually paired to one specific gateway. More than one gateway may hear the same uplink.

For the connection to work:

1. The sensor and gateway must use compatible LoRaWAN radio settings for the region, such as the correct frequency plan.
2. The gateway must be online and connected to a LoRaWAN network server.
3. The sensor must be registered with that network server. With **OTAA**, the sensor joins using its device identifiers and securely configured join credentials. **ABP** is another activation method that uses preconfigured session details.
4. After joining, the sensor sends uplinks according to its reporting schedule or when an event occurs. The gateway relays those messages; the network server manages the LoRaWAN connection and routes the data onward.

In most deployments, you configure the device and its credentials in the sensor and network server. You do not enter the gateway's Wi-Fi name into the sensor.

## 11. What are the setup steps for OTAA and ABP?

The exact menu names depend on the sensor model, firmware, and network server. Check that the sensor supports the activation method before configuring it. In both cases, the sensor and network server must use the same LoRaWAN version and regional frequency plan, and a gateway must be connected to that network server and within range.

### OTAA (Over-the-Air Activation)

OTAA is commonly recommended for new deployments. The sensor requests to join the network; the network verifies it and establishes session credentials.

1. In the network server, add an end device with the correct model/LoRaWAN version and regional frequency plan.
2. Enter or register the sensor's `DevEUI` and `JoinEUI` (called `AppEUI` in older LoRaWAN versions).
3. Set the required root key or keys in the network server. The names and number of keys depend on the LoRaWAN version; common names include `AppKey` and, for LoRaWAN 1.1, `NwkKey`.
4. Enter the matching identifiers and keys in the sensor, often using Milesight ToolBox over NFC.
5. Power on or restart the sensor. In the network server's event log, check for a join request followed by a successful join accept, then check for uplink messages.

### ABP (Activation by Personalization)

ABP skips the over-the-air join. The network address and session keys are preconfigured in both the sensor and network server.

1. Confirm that the sensor and network server support ABP for the chosen LoRaWAN version.
2. In the network server, add the device as ABP and configure its `DevAddr` and required session keys. For LoRaWAN 1.0.x, these commonly include `NwkSKey` and `AppSKey`; LoRaWAN 1.1 uses additional network session keys.
3. Enter exactly matching ABP settings in the sensor, using its supported configuration method.
4. Check the frame-counter settings in both places, then power on or restart the sensor and look for uplinks in the server event log. ABP does not produce an OTAA join exchange.

OTAA is generally preferable because it establishes fresh session credentials when joining. Treat all keys as secrets: never publish them or include them in screenshots shared publicly. The device manual and network-server documentation are authoritative for the exact fields and steps.

## Sources

- Milesight: https://www.milesight.com/
- Milesight EM300-TH payload decoder: https://github.com/Milesight-IoT/SensorDecoders/tree/main/em-series/em300-th
- LoRa Alliance: https://lora-alliance.org/,https://www.thethingsnetwork.org/docs/lorawan/what-is-lorawan/
