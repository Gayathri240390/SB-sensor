# Milesight UG65 Gateway Guide

A plain-language guide to the UG65's role, how it receives AM300 sensor messages, and how readings travel from the gateway to a cloud or building system.

## 1. What is the UG65?

The Milesight UG65 is an indoor LoRaWAN gateway. It listens for radio messages from LoRaWAN sensors, then forwards received messages through an IP network to a LoRaWAN network server. The network server and application/cloud platform handle the next steps.

Think of the UG65 as a bridge between two networks:

- **Sensor side:** LoRaWAN radio over LoRa
- **Server side:** IP connectivity through the network connection configured for the gateway

The UG65 is not itself an air-quality sensor. Also, a gateway is not automatically the same thing as a network server or cloud dashboard; those are separate roles, even if a deployment combines some of them in one platform.

## 2. End-to-end communication

```text
1. AM300 measures indoor conditions
       |
       | LoRaWAN uplink over LoRa radio
       v
2. UG65 receives the radio message
       |
       | IP backhaul over the configured network
       v
3. LoRaWAN network server handles the device/session and routes the message
       |
       | cloud connector, API, webhook, or other configured integration
       v
4. Application decodes the AM300 payload and displays or uses the readings
```

An uplink may be heard by more than one gateway. The gateway forwards the received radio data; the network server decides how to process and route it. The gateway does not normally need to understand whether the payload means temperature, CO2, or another sensor value.

## 3. Which protocols are used?

| Part of the path | Protocol or connection | What it does |
| --- | --- | --- |
| AM300 to UG65 | LoRaWAN over LoRa radio | Carries low-power wireless sensor messages. The sensor and gateway need compatible regional radio settings. |
| UG65 to network server | IP network over the configured backhaul; gateway forwarding mode is configured for the chosen server | Carries received LoRaWAN packets from the site to the network server. The exact forwarding mode depends on UG65 software and the network-server deployment. |
| Network server to application | Platform-specific integration, optionally MQTT | Routes device data to a dashboard or application. MQTT is used only when a configured gateway, network server, or cloud integration publishes to a broker. |
| Application/BMS integration (optional) | For example, BACnet or Modbus where configured | Makes readings available to a building-management system or controller. This is an integration layer, not the AM300-to-UG65 radio protocol. |

Do not describe the whole path as “LoRaWAN to the cloud.” LoRaWAN is the sensor's radio/network protocol. From the gateway onward, IP connectivity and server-specific forwarding/integration are used. The exact cloud-side protocol is chosen by the platform and configuration, not by the AM300 alone.

### Where MQTT, APIs, and JSON fit

MQTT is an optional way to deliver uplink events to an application. The AM300 does not publish MQTT, and MQTT does not replace LoRaWAN on the AM300-to-UG65 radio link. An MQTT publisher sends messages to a broker on topics; an application subscribes to the topics it needs.

An **API** is a software interface. A REST API commonly lets an application send HTTP requests to a service and receive a response. For example, the Milesight Development Platform describes REST APIs and webhooks. That confirms API features for the platform; it does **not**, by itself, confirm that every UG65 exposes a generic REST API or directly sends sensor readings to any cloud. Confirm the exact feature in the UG65 manual for your hardware/firmware and in the selected platform's documentation.

Neither MQTT nor an API requires JSON. JSON is a message/data format, not a transport protocol. The AM300 sends a compact binary LoRaWAN application payload. After a model-specific decoder processes it, the network server or integration may publish decoded fields as JSON, for example:

```json
{
       "devEUI": "sensor-device-id",
       "temperature": 23.6,
       "humidity": 48.2
}
```

The actual field names, units, and envelope depend on the AM300 model and platform. If no decoder is applied, the MQTT or API message may contain the raw payload as hex or Base64 instead of readable measurements.

```text
AM300 --LoRaWAN/LoRa--> UG65 --IP forwarding--> LoRaWAN network server
                                                    |
                               configured MQTT publisher (if supported)
                                                    |
                                                    v
                                          MQTT broker --> app subscriber
```

The MQTT publisher may be the UG65 only if its exact firmware/manual documents that feature, the network server, or a cloud connector. The Milesight product pages alone do not establish which option is enabled on a particular UG65 firmware or installation. Check the exact UG65 user guide and the network-server/cloud documentation before designing the application around direct gateway-to-broker publishing.

For MQTT, configure the broker address and port, TLS/security settings, client credentials, topic, QoS, and message format. For a REST API, configure the endpoint URL, HTTP method, authentication, and request/response schema. In either case, verify whether the data is decoded JSON or encoded raw payload. Never put passwords, tokens, or keys in source code; use protected configuration such as environment variables or a secret store.

## 4. UG65 capabilities in Milesight's product information

Milesight describes the UG65 as a PoE-based indoor gateway using an NXP quad-core processor and Semtech SX1302 concentrator. The product page lists full-duplex 8-channel communication and LoRaWAN Class A/B/C device support. It also advertises capacity and coverage figures, but real results depend on the radio region, site layout, interference, antenna placement, data rates, and device reporting behavior.

### Maximum sensor count and practical capacity

Milesight advertises connectivity for **up to 2,000 devices**. Treat this as a stated maximum, not a promise that every deployment can support 2,000 sensors at any reporting rate. Practical capacity depends on how often sensors transmit, packet airtime and size, data rate, radio conditions, regional limits, confirmed messages/downlinks, and the capacity of the network server and cloud integration. For a large deployment, plan capacity using the expected traffic and the current UG65/network-server documentation.

### Radio band, coverage, and reporting frequency

- **Radio frequency band:** The supported band depends on the UG65 regional hardware variant and local regulations. Common LoRaWAN plans include EU868 (roughly 863–870 MHz) and US915 (roughly 902–928 MHz); these are examples, not interchangeable settings. Check the UG65 label/manual and use the same regional plan for the AM300 and network server.
- **Coverage:** Milesight advertises up to **2 km in an urban area**. Real indoor coverage can be much shorter or longer depending on walls, floors, antenna placement, interference, and the sensor's location. Test coverage at the actual installation points.
- **How often data arrives:** The UG65 listens on its configured LoRaWAN channels; it does not poll the AM300 on a fixed schedule. The sensor sends uplinks according to its configured reporting interval and may send extra event/alarm messages. Change the report interval in the sensor configuration, not in the gateway. More frequent reporting uses more radio airtime and can affect battery life and network capacity.

The product page mentions integrated Wi-Fi and cellular antennas as well as an external LoRa antenna. Do not assume every connectivity option is present or enabled on every hardware SKU. Check the unit's label and its matching user guide. Use the documented power method and approved antenna for the exact version.

## 5. What must be configured?

The following pieces must agree for readings to reach the application:

- **Radio region/frequency plan:** Must match the sensors and the deployment's legal regional band.
- **Gateway backhaul:** The UG65 needs a working IP connection to the selected network server. Use the available connection on the exact device/configuration, such as wired Ethernet/PoE or another supported option.
- **Gateway forwarding target:** The UG65 must be configured to forward packets to the correct LoRaWAN network server using a supported mode for that software version.
- **Network-server device record:** Add each AM300 with its `DevEUI`, LoRaWAN version, activation method (usually OTAA), and matching join credentials.
- **Application integration:** Configure the network server or platform to pass the uplinks to the cloud dashboard, API, webhook, or building system.
- **Payload decoder:** Use a decoder for the exact AM300 model and firmware to turn raw bytes into human-readable readings.

The gateway's connection to the network server is separate from a sensor's OTAA/ABP activation. A sensor does not join the gateway itself; it joins the LoRaWAN network managed by the network server.

## 6. Setup sequence

Exact interface labels and available connection options depend on UG65 hardware and firmware. Use the current UG65 user guide from Milesight's Document Center for device-specific instructions.

1. **Plan coverage:** Choose an indoor location with power and a reliable network connection. Keep the LoRa antenna clear of metal and obstructions, and confirm that the AM300s are within usable radio coverage.
2. **Install and power the gateway:** Connect the LoRa antenna and use the power method supported by the exact UG65 model. Milesight describes the UG65 as PoE-based; confirm the required PoE standard and accessories in its manual.
3. **Connect the backhaul:** Connect the gateway to the site's IP network using an available, supported uplink. Confirm it can reach the configured network server; firewall rules may need to permit the documented traffic.
4. **Configure the UG65:** Use the documented local or remote management interface to set regional radio parameters and the connection/forwarding details for the selected network server. Change default credentials and apply approved firmware/security settings.
5. **Configure the network server:** Add the AM300 device record with the exact DevEUI, LoRaWAN version, regional plan, and OTAA or ABP details. Keep matching keys secret.
6. **Verify the path:** Check the gateway status first, then the network-server event log for sensor join/uplink events. Finally, check the application for decoded readings.

## 7. Where does decoding happen?

The AM300 sends a compact model-specific binary payload. The gateway receives and forwards the radio message; in the normal LoRaWAN architecture, the application/network-server integration applies the decoder and exposes named readings. The exact place where decoding runs depends on the chosen platform. The UG65's product page also describes Milesight integrations and BMS options, including BACnet and Modbus; these require their own documented configuration.

## 8. Troubleshooting by layer

- **Gateway has no server connection:** Check power, Ethernet/network settings, IP address, DNS/time settings if required, firewall access, and the configured network-server target.
- **Gateway is online but no sensor uplinks appear:** Check antenna connection, regional frequency plan, sensor coverage, sensor activation, and whether the gateway is connected to the same network-server deployment.
- **Join attempts fail:** Check the network-server device record, DevEUI/JoinEUI, key values, LoRaWAN version, and region. A gateway only relays the join exchange; the network server accepts or rejects it.
- **Raw uplinks appear but no readable data:** Configure the correct model/firmware decoder in the platform.
- **Cloud dashboard is empty while uplinks arrive:** Check routing from the network server to the cloud/application, integration credentials, API/webhook configuration, and application device mapping.
- **Intermittent messages:** Review radio coverage and interference, gateway placement, antennas, reporting rates, and server/backhaul availability. Do not treat advertised maximum range or capacity as a guarantee for every building.

## 9. Security and reliability basics

- Change default gateway credentials and restrict management access to trusted administrators.
- Keep LoRaWAN join/session keys, cloud tokens, and integration credentials secret.
- Keep the gateway firmware and cloud/server integrations maintained according to Milesight and platform guidance.
- Use a stable, secured IP backhaul and suitable power backup where continuous monitoring is required.
- Remember that LoRaWAN radio coverage and Internet availability are separate: a sensor may reach the gateway while the gateway's cloud connection is down.

## 10. Official references

- [Milesight UG65 product page](https://www.milesight.com/iot/product/lorawan-gateway/ug65)
- [Milesight AM300 Series product page](https://www.milesight.com/iot/product/lorawan-sensor/am300)
- [Milesight Document Center](https://www.milesight.com/iot/resources/download-center/): get the UG65 user guide for the exact hardware/firmware and each AM300 model's guide
- [Milesight Development Platform](https://www.milesight.com/product/development-platform): platform APIs, webhooks, and device deployment information
- [LoRa Alliance](https://lora-alliance.org/): LoRaWAN standard information

**Important:** The product pages explain product capabilities, not every deployment-specific setting. Confirm regional frequency, hardware options, supported forwarding modes, and firewall requirements in the current UG65 user guide and the chosen network server's documentation.