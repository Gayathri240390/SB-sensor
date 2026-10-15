# Milesight AM300 Series Sensor Guide

A practical guide to what the AM300 does, how it sends readings, and what is needed to get those readings into an application.

## 1. What is the AM300?

The Milesight AM300 is a family of indoor air-quality (IAQ) sensors. It measures environmental conditions in a room and reports readings wirelessly. Depending on the exact model, the device can monitor a different combination of parameters. Milesight describes the family as supporting up to nine environmental parameters and occupancy-related information.

The family page lists AM304L, AM305L, AM307, AM308/AM308L, and AM319. **Do not assume every model measures the same things.** Read the label on the unit and check that exact model's datasheet and user guide in the Milesight Document Center.

## 2. Simple picture of the system

```text
AM300 measures the room
        |
        | LoRaWAN radio (LoRa)
        v
UG65 gateway receives the radio message
        |
        | IP network, usually Ethernet/PoE backhaul
        v
LoRaWAN network server
        |
        | device-specific decoding and integration
        v
Cloud dashboard, building system, or application
```

The AM300 does not normally connect to Wi-Fi or send readings directly to the cloud. It sends a low-power LoRaWAN radio message to a gateway such as the UG65.

## 3. What protocols are involved?

| Link or job | What is used | In plain language |
| --- | --- | --- |
| AM300 to UG65 | LoRaWAN over LoRa radio | The sensor sends a short wireless message; no Wi-Fi password is entered into the sensor. |
| Sensor activation | Usually OTAA; ABP only if supported and configured | This is how the sensor is admitted to the LoRaWAN network. OTAA performs a join; ABP starts with preconfigured session details. |
| UG65 to network server | IP connection over the gateway's configured backhaul | The gateway forwards received LoRaWAN messages. The exact forwarding mode depends on the gateway software and network-server setup. |
| Network server to application | Platform-specific integration, such as MQTT, a built-in connector, API, or webhook | This carries the usable readings to a dashboard or another system. MQTT is optional and is not the sensor-to-gateway protocol. |
| Reading format | Model-specific binary payload, often shown as hex or Base64 | A decoder for the exact AM300 model turns the bytes into named values such as temperature or CO2. |

LoRaWAN is the radio-network protocol; it does not define one universal set of sensor readings or a universal JSON format. The model's decoder defines how its application payload is interpreted.

If the deployment uses MQTT, the AM300 still sends LoRaWAN radio messages to the UG65. A configured MQTT publisher later sends the received/processed event to an MQTT broker. Depending on the deployment, that publisher may be a supported gateway feature, the LoRaWAN network server, or a cloud integration. Confirm the publisher and message format in the matching device/platform documentation.

An **API** (application programming interface) is a documented way for software to request or send data. A REST API commonly uses HTTP requests and may use JSON for the request or response. This is separate from MQTT: MQTT publishes messages through a broker, while an API is usually called at an HTTP endpoint. Neither protocol requires JSON. The AM300's LoRaWAN payload is binary; JSON is available only if the network server or integration decodes it and formats it that way. Milesight documents REST APIs and webhooks for the Milesight Development Platform; check your chosen platform's documentation to confirm its API and decoded-data support.

## 4. What is sent in a reading?

The AM300 measures its configured parameters, then builds a compact application payload. The payload is usually binary to keep radio transmissions short and save power. In a network-server log, the raw bytes may appear as hexadecimal or Base64. After the correct model decoder is applied, the application may show readable values in JSON or a dashboard.

An uplink record can also contain information that is not part of the measured payload, for example:

- `DevEUI`: the sensor's LoRaWAN device identifier
- receive time, radio signal strength (RSSI), and signal-to-noise ratio (SNR)
- application port (`fPort`) and gateway metadata

For correct decoding, use the exact model's decoder and follow its firmware/version notes. Do not copy a decoder from another AM300 variant just because the model names are similar.

## 5. How does the sensor join the network?

The gateway and network server must already be configured and online. The sensor must use the same LoRaWAN region/frequency plan and LoRaWAN version expected by the network server.

### OTAA (recommended for most new deployments)

1. Read the AM300's exact model and `DevEUI` from its label/QR code or device tools.
2. Add the sensor as an end device in the LoRaWAN network server. Enter the matching device identifiers, LoRaWAN version, and regional frequency plan.
3. Configure the required join key or keys in the network server and in the sensor. The exact fields depend on the LoRaWAN version; older systems may call `JoinEUI` by the name `AppEUI`.
4. Configure the sensor using the method supported by that model. Milesight ToolBox/NFC may be available; check the model's guide rather than assuming.
5. Power on or restart the sensor. Look in the network-server event log for a join request, a successful join, and then an uplink.

### ABP (only if the model and server support it)

ABP uses preconfigured `DevAddr` and session keys instead of an over-the-air join. The sensor and server values must match exactly, and frame-counter settings must be compatible. ABP does not produce an OTAA join event. Follow the model and network-server manuals closely; OTAA is generally easier to manage securely at scale.

**Keep join keys and session keys private.** Do not publish them or include them in screenshots.

## 6. Basic deployment checklist

1. Confirm exact model, regional variant, and supported LoRaWAN version.
2. Place the sensor where the room reading is representative, away from direct sunlight, HVAC outlets, and other local sources that could distort a measurement. Follow the installation guide for height and spacing.
3. Configure the reporting interval and alarm/reporting options using the model's guide. This interval controls how often the sensor sends routine readings; the UG65 does not set or poll this schedule. Event/alarm reports may be sent in addition to routine reports. More frequent reporting can increase network traffic and may affect battery life.
4. Register the device and activate it with the network server (usually OTAA).
5. Confirm that the sensor joins and that the network server receives uplinks.
6. Apply the decoder for the exact model and verify that decoded values and units look reasonable.
7. Connect the application/cloud integration and confirm that readings arrive there too.

## 7. If readings do not appear

- **No join:** Check the DevEUI and join credentials, regional frequency plan, LoRaWAN version, gateway coverage, and gateway connection to the correct network server.
- **Joined but no readings:** Check the sensor reporting interval, radio coverage, and the uplink event log.
- **Raw bytes appear but values do not:** Select the decoder for the exact AM300 model and firmware, and check the application's `fPort`/payload settings.
- **Readings reach the network server but not the cloud app:** Check the network-server integration, routing, MQTT publisher/broker/topic (if used), API/webhook, or cloud connector.
- **Application expects JSON but receives raw data:** Check whether the network server or integration is applying the exact AM300 model decoder and whether its MQTT/API output is configured to publish decoded fields.
- **Values look implausible:** Confirm the model-specific decoder and units before changing sensor settings.

## 8. Official references

- [Milesight AM300 Series product page](https://www.milesight.com/iot/product/lorawan-sensor/am300)
- [Milesight Document Center](https://www.milesight.com/iot/resources/download-center/): find the exact model's user guide and datasheet
- [Milesight SensorDecoders](https://github.com/Milesight-IoT/SensorDecoders): model-specific payload decoding examples
- [Milesight UG65 product page](https://www.milesight.com/iot/product/lorawan-gateway/ug65): gateway used in the example path in this guide
- [LoRa Alliance](https://lora-alliance.org/): LoRaWAN standard information

**Model note:** The product pages describe the family. Exact sensors, units, radio region, power requirements, settings, and decoder behavior must be confirmed in the user guide for the specific unit.