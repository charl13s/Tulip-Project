# ICS 4111 — Semester Project, Deliverable 1

**Course:** ICS 4111 · Embedded Systems & IoT (Apr–Jul 2026)
**Client / Use case:** Flora Farms, Naivasha — remote greenhouse monitoring
**Assigned flower:** **Tulips** (*Tulipa* spp.)
**Date:** 2026-06-07

---
**Group Members:**
* 158657	Kuria Shawn James
* 165948	Murage Charles
* 166096	Munyiri Timothy
* 160761	Muriuki Nelly Nkatha
* 168669	Riang'a Ravine Kerubo
* 167022	Billy John Igiraneza
* 168661	Diang'a Erika Abuchie

## 1. Environmental requirements for healthy Tulip growth

Tulips are temperate, cool-season geophytes. They originate from the steppes of Central Asia, so a Kenyan greenhouse has to actively *cool and dry* the canopy to keep tulips happy. The table below is the team's working reference — every sensor we choose later has to be able to measure within these ranges with usable resolution.

| # | Parameter | Optimal range for Tulips | Notes |
|---|-----------|---------------------------|-------------------|
| a | **Air temperature** | **15 – 18 °C** during forcing/bloom; 5 – 9 °C during cool rooting; absolute ceiling ≈ 20 °C | Above 20 °C the buds blast and stems weaken. The greenhouse heater covers cold nights; vents handle hot afternoons. |
| b | **Relative humidity** | **63 – 73 % RH** | Below 60 % the petals desiccate; above 75 % *Botrytis* outbreaks become common. |
| c | **Soil type** | **Well-drained sandy loam**, light, friable, organic-matter | Bulbs rot in heavy clays. Sandy loam has a low field-capacity and drains quickly after irrigation. |
| d | **Soil moisture content** |**22 % – 28 %** VWC during active growth and bloom. Allow surface to dry ~2 cm between irrigations. | Tulips are drought-tolerant once rooted. Waterlogged substrate rots bulbs within days. |
| e | **Soil pH** | **6.0 – 7.0** Slightly acidic to neutral | Outside this band, Fe / Mn uptake suffers and bulbs yellow. |
| f | **Sunlight (direct)** | **6 – 8 hours / day** of bright direct light during the growth phase | In Naivasha the issue is excess afternoon sun,  shade cloth at midday keeps leaf temperature down. |

The Wi-Fi monitoring node we develop must therefore measure: **T, RH, soil VWC, soil pH, light intensity, and LPG concentration**, the last because the heater fuel is a safety hazard in addition to being an environmental input.

---

## 2. Hardware components

The list below covers everything we need to build, test and prototype the device.

### 2.1 Sensing & I/O

| # | Component | Qty | Purpose |
|---|-----------|-----|---------|
| 1 | ESP32-S DevKit (30-pin, ESP-WROOM-32) | 2 | Main MCU + Wi-Fi/BLE radio |
| 2 | DHT22 / AM2302 | 1 | Air temperature & relative humidity |
| 3 | MQ-5 gas sensor module | 1 | LPG / methane / butane / propane (heater-leak safety) |
| 4 | Capacitive soil moisture sensor v1.2 (analog) | 1 | Soil VWC — capacitive avoids the corrosion of resistive probes |
| 5 | Gravity analog spear-tip soil pH probe (DFRobot SEN0249) | 1 | Soil pH 0 – 14 |
| 6 | BH1750FVI ambient-light sensor (I²C) | 1 | Lux / sunlight intensity |
| 7 | 1.3" 128×64 OLED display, SH1106, I²C | 1 | On-device readout for field maintenance |
| 8 | 5 V 1-channel low-level-trigger relay module (SRD-05VDC-SL-C) | 1 | Switches the MQ-5 sensing branch (and can drive an alert beacon / vent fan) |
| 9 | Passive buzzer (5 V) | 1 | Local audible alarm on gas detect |

### 2.2 Power-supply chain

| # | Component | Qty | Purpose |
|---|-----------|-----|---------|
| 10 | LM2596 step-down buck module | 1 | Drops greenhouse battery down to 5 V |

### 2.3 Passive components

| # | Component | Qty | Purpose |
|---|-----------|-----|---------|
| 11 | 10 kΩ resistor | 5 | DHT22 data pull-up; voltage-divider for MQ-5 raw mode |
| 12 | 4.7 kΩ resistor | 4 | External I²C pull-ups (SDA / SCL) |
| 13 | 1 kΩ resistor | 2 | GPIO-to-opto current limit for relay drive |
| 14 | 470 Ω resistor | 4 | UART series protection in Schematic B |
| 15 | 100 nF ceramic capacitor (0603 / through-hole) | 8 | Vcc decoupling at each IC |
| 16 | 10 µF electrolytic capacitor (16 V) | 4 | Bulk decoupling on 5 V rail |
| 17 | 1N4148 small-signal diode | 1 | Flyback on relay coil |
| 18 | LED (3 mm, red & green) | 2 | Power / status indicators |
| 19 | 330 Ω resistor | 2 | Current-limiting resistors for the status LEDs |

### 2.4 Prototyping tools

| # | Component | Qty | Purpose |
|---|-----------|-----|---------|
| 20 | 830-point breadboard (full-size) | 1 | Bench prototyping — reused as each architecture is built |
| 21 | Male–male jumper wires (20 cm) | 20 | Breadboard interconnect |
| 22 | Male–female jumper wires (20 cm) | 20 | Module-to-breadboard interconnect (sensor modules use male headers) |
| 23 | Female–female jumper wires (20 cm) | 10 | Board-to-board links (Schematics B and C) |
| 24 | USB-A → micro-USB cable | 2 | ESP32-S programming/power (both ESP32s run together in B and C) |


---

## 3. Datasheets


| # | Component | Resource | Link |
|---|-----------|----------|------|
| a | 1.3" White IIC 128×64 OLED LCD | SH1106 controller datasheet (SparkFun mirror) | <https://cdn.sparkfun.com/assets/2/6/8/9/7/1.3inch-SH1106-OLED_Datasheet.pdf> |
| b | ESP32S DevKIT — 30-pin board pinout reference | ESPBoards — DOIT ESP32 DevKit V1 | <https://www.espboards.dev/esp32/esp32doit-devkit-v1/> |
| c | DHT22 / AM2302 Temperature & Humidity sensor | Aosong AM2302 product manual | <https://cdn-shop.adafruit.com/datasheets/Digital+humidity+and+temperature+sensor+AM2302.pdf> |
| d | MQ-5 LPG / natural gas / coal gas sensor | Hanwei MQ-5 technical data | <https://files.seeedstudio.com/wiki/Grove-Gas_Sensor-MQ5/res/MQ-5.pdf> |
| e | 5 V 1-Channel Low-Level Trigger Relay Module | Songle SRD-05VDC-SL-C relay datasheet | <https://www.circuitbasics.com/wp-content/uploads/2015/11/SRD-05VDC-SL-C-Datasheet.pdf> |

---

## 4. Schematic designs



### 4.1 Schematic A — single ESP32-S handling MQ-5 + DHT22 + OLED



![Schematic A]("/images/4A.png")


### 4.2 Schematic B — two ESP32-S boards, MQ-5 ⇄ DHT22, direct UART link


![Schematic B](/images/4B.png)



### 4.3 Schematic C — DHT22 node gates power to the MQ-5 node through a relay

![Schematic C](/images/4C.png)



## 5. Evidence of group work

![Evidence](/images/Evidence.jpeg)

## 7. References (citations used in this document)

- Baballe, M. A. et al. (2021). *Automatic Gas Leakage Monitoring System using MQ-5 Sensor.* Review of Computer Engineering Research, 8(2). (Figure 7 used as the detail-level template.)
- DryGair — *What are the Ideal Conditions for Greenhouse Tulips?* <https://drygair.com/blog/greenhouse-tulips/>
- Agriculture Institute — *Growing Tulips: Varieties, Propagation, and Climate Requirements.* <https://agriculture.institute/floriculture-and-landscaping/growing-tulips-varieties-propagation-climate/>
- Johnny's Seeds — *Tulip Key Growing Information.* <https://www.johnnyseeds.com/growers-library/flowers/tulips/tulip-key-growing-information.html>
- Soltech — *The Ultimate Guide to Tulip Plant Care.* <https://soltech.com/blogs/blog/the-ultimate-guide-to-tulip-plant-care-how-to-ensure-a-vibrant-spring-bloom>
- Cornell NRCCA — *Soil Hydrology, Field Capacity & Permanent Wilting Point.* <https://nrcca.cals.cornell.edu/soil/CA2/CA0212.1-3.php>
- ConnectedCrops — *A Guide to Soil Moisture.* <https://connectedcrops.ca/the-ultimate-guide-to-soil-moisture/>
