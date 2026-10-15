# AM300 Sensor and UG65 Gateway

A simple presentation about how indoor readings travel from a Milesight sensor to an application.

---

## Slide 1: From Room to Application

**Milesight AM300 + UG65 + LoRaWAN**

- AM300 measures indoor conditions.
- UG65 receives the sensor's wireless message.
- A network server and cloud integration deliver readings to an application.

**Speaker notes:** This presentation follows one reading from the room sensor to the screen where a user sees it.

---

## Slide 2: The Four Main Parts

1. **AM300 sensor:** Measures selected indoor air-quality conditions.
2. **UG65 gateway:** Receives LoRaWAN radio messages.
3. **LoRaWAN network server:** Manages device sessions and routes messages.
4. **Cloud/app:** Decodes, stores, displays, or uses the readings.

```text
Room -> AM300 -> UG65 -> Network server -> Cloud/app
```

**Speaker notes:** A gateway is a bridge. It is not automatically the cloud dashboard or the network server.

---

## Slide 3: Which Protocol Goes Where?

| Connection | Protocol or method |
| --- | --- |
| AM300 to UG65 | LoRaWAN over LoRa radio; sensor payload is compact binary |
| UG65 to network server | IP connection using the configured forwarding mode |
| Server/integration to application | Platform-specific; MQTT, REST API, webhook, or connector may be used |
| Optional building-system connection | BACnet or Modbus when configured |

**Speaker notes:** MQTT sends messages through a broker; an API is a software interface, often called over HTTP. Either may be used downstream if the selected platform supports it. Neither one is the AM300-to-UG65 radio link.

---

## Slide 4: What the AM300 Does

- Measures the parameters supported by its exact model.
- Sends compact LoRaWAN uplinks at configured intervals or for events.
- Uses OTAA in many deployments; ABP is used only when supported and configured.
- Sends model-specific binary payloads, not necessarily JSON.

**Important:** AM304L, AM305L, AM307, AM308/AM308L, and AM319 do not necessarily measure the same things. Check the unit label and exact user guide.

**Speaker notes:** First identify the full model. The decoder and units depend on that model and sometimes on firmware.

---

## Slide 5: What the UG65 Does

- Listens for LoRaWAN messages from sensors in radio range.
- Forwards received messages through its IP backhaul to a network server.
- Can hear an uplink that is also received by another gateway.
- Needs correct regional radio settings, reliable power, and a working network connection.
- Milesight advertises up to 2,000 devices, 8-channel communication, and up to 2 km urban coverage; actual results depend on site and traffic.
- The UG65 listens on configured radio channels; the AM300's report interval determines routine uplink timing.

**Speaker notes:** The gateway relays messages; the network server decides whether a sensor can join and where its data should go.

**Speaker notes:** The advertised device count and range are maximum product figures, not a guarantee for every building. Regional band depends on hardware variant and location. Confirm both in the UG65 manual. The sensor sends on its own configured schedule; alarms may create extra messages.

---

## Slide 6: How MQTT, APIs, and JSON Can Fit

```text
AM300 --LoRaWAN--> UG65 --IP--> Network server
                                      |
                       +--------------+--------------+
                       |                             |
                 MQTT publisher                 REST API/webhook
                       |                             |
                       v                             v
                  MQTT broker                   Cloud service
                       |                             |
                 App subscribes                 App/API client
```

- MQTT and REST APIs are optional downstream integrations; the exact supported path depends on the UG65 firmware and network-server/cloud platform.
- JSON is a data format, not a protocol. The AM300 sends compact binary sensor data over LoRaWAN.
- After the model-specific decoder runs, the platform can provide decoded JSON; otherwise the message may contain raw hex or Base64 data.
- Milesight documents REST APIs and webhooks for its Development Platform. Confirm any direct UG65 API/MQTT feature in the exact gateway manual.

**Speaker notes:** Ask where decoding happens and which component publishes to the broker or API. That determines whether the application receives readable JSON or raw sensor bytes.

---

## Slide 7: What Must Be Configured?

- Confirm model, region/frequency plan, and LoRaWAN version.
- Connect the UG65 to power and the IP network; configure its forwarding target.
- Register the AM300 in the network server and match its activation credentials.
- Configure the model-specific payload decoder.
- If using MQTT, set broker address, TLS/authentication, topic, QoS, and message format.
- Keep join keys, broker passwords, and tokens private.

**Speaker notes:** Test one layer at a time: gateway online, sensor joined, uplink received, payload decoded, then application delivery.

---

## Slide 8: Troubleshoot in Order

1. **No sensor join:** Check credentials, region, LoRaWAN version, and coverage.
2. **No uplink at server:** Check sensor reporting, antenna, gateway status, and forwarding configuration.
3. **Raw bytes but no readings:** Check the exact model decoder and firmware notes.
4. **Server has readings but app does not:** Check MQTT/API/webhook routing, broker topic and credentials, and application subscription.

**Speaker notes:** Knowing the last working step helps locate the issue without changing every setting at once.

---

## References

- [Milesight AM300 Series](https://www.milesight.com/iot/product/lorawan-sensor/am300)
- [Milesight UG65](https://www.milesight.com/iot/product/lorawan-gateway/ug65)
- [Milesight Document Center](https://www.milesight.com/iot/resources/download-center/)
- [Milesight SensorDecoders](https://github.com/Milesight-IoT/SensorDecoders)
- [LoRa Alliance](https://lora-alliance.org/)

**Note:** Product variants, firmware features, MQTT publisher location, regional radio settings, and message formats must be confirmed in the matching manuals and platform configuration.