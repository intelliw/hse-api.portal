# /devices/dataset POST
---

The `/devices/dataset/{dataset}` path is for vendors and systems integrators to POST device data.

[http:/api.endpoints.sundaya.cloud.goog/devices/{dataset}](http:/api.endpoints.sundaya.cloud.goog/devices/dataset/epack)

- This route allows device **controllers** (e.g. Bus Bar Controller) and **gateways** (e.g. EHub Gateway) to accumulate and periodically send data from multiple monitored devices installed at a site.

### {dataset} parameter ###

The following types may be specified as the `{dataset}` parameter in all `/device` paths.

dataset | Description
--- | --- | --- 
`epack` | Data from pack management systems (PMS) for cabinets, including monitored epacks, cells, and mosfets.
`mppt` | Data from Maximum Power Point Tracking (MPPTcontrollers, including connected PV strings, batteries, and DC loads.
`inverter` | Data from Inverter charge controllers, including connected pv strings, batteries, and 


### 'epack' Body parameter

The following snippet shows the structure of a `epack` dataset:

```json
{
  "datasets": [
    { "cabinet": { "id": "CAB-01-001" }, 
      "data": [
        { "time": "20190209T150006.032-0700",
          "pack": { "id": "EPACK-01-001", "dock": "01", "volts": "55.1", "amps": "-1.601", "temp": ["35.0", "33.0", "34.0"] },
          "cell": { 
            "volts": ["3.92", "3.92", "3.92", "3.92", "3.92", "3.92", "3.92", "3.92", "3.92", "3.92", "3.92", "3.92", "3.92", "3.91"],
            "open": [1, 6] },
          "fet": { 
            "temp": ["34.1", "32.2", "33.5"], "open": [1, 2] }
        },
```

The dataset attributes are specified below. 

Attribute | Metric | Data | Optionality | Description
--- | --- | --- | --- | --- 
`cabinet.id` | - | string | mandatory | Id of the epack Cabinet. *(__Note__: each PMS controller (BBC) is able to handle 4 cabinets, with upto 12 packs in each cabinet, and 14 cells per pack)*.
`time` | - | datetime | mandatory | The time of the event which produced this data sample, in compressed `ISO 8601/RFC3339` (YYYYMMDDThhmmss±hhmm).
`pack.id` | - | string | mandatory | The pack identifier. *(__Note__: ideally this should be a logical identifier not the MCU hardware id) which gets flashed onto the pack control board when it is first commissioned, or returned to service after it is affixed to a different pack)*. 
`pack.slot` | - | integer | mandatory | The cabinet slot (1-12) in which this pack is installed. 
`pack.volts` | volts | float | mandatory | The voltage of this pack. *(__Note__: `pack.volts` may not be the sum of `cell.volts` during cell balancing, if a cell is overcharged and bypassed)*.
`pack.amps` | amps | float | mandatory | The current draw from this pack.  
`pack.soc` | % | float | required on change | The % state-of-charge (0-100) of this pack.
`pack.temp` | degC | float | required on change | The temperature of the cell-pack at its top, middle, and bottom. 
`cell.volts` | volts | float *(array)* | mandatory | An ordered set of Voltage readings for 14 cellblocks in this pack.
`fet.temp` | degC | float | required on change | The temperature of the input (CMOS) and output (DMOS) MOSFETS.
`fet.status` | (open/closed* | boolean | required on change | The status of the input (CMOS) and output (DMOS) MOSFETS (*1=open/0=closed*).
 
- Attributes marked as '*required on change*' may be ommitted if the value has not changed since the last successful post (a POST is successful if acknowledged by the server with a 201 response). All other attributes must be provided. 

### 'mppt' Body parameter

The following snippet shows the structure of a `mppt` dataset:

```json
{
  "datasets": [
    { "mppt": { "id": "IT6415AD-01-001" }, 
      "data": [
        { "time": "20190209T150006.032-0700",
          "pv": { "volts": ["48.000", "48.000"], "amps": ["6.0", "6.0"] },
          "batt": { "volts" : "55.1", "amps": "-1.601" }, 
          "load": { "volts": ["48.000", "48.000"], "amps": ["1.2", "1.2"] }
        },
```

The dataset attributes are specified below. 

Attribute | Metric | Data | Optionality | Description
--- | --- | --- | --- | --- 
`mppt.id` |  | string | mandatory | Id of the MPPT charge controller. *(__Note__: a site can have any number of MPPT controllers, and each controller can have multiple PV strings and Loads)*.
`time` |  | datetime | mandatory | The time of the event which produced this data sample, in compressed `ISO 8601/RFC3339` (YYYYMMDDThhmmss±hhmm).
`pv.volts` | volts | float *(array)* | mandatory | An ordered set of Voltage readings for PV strings connected to this MPPT. *(__Note__: presently upto 4 PV strings per controller are supported. In future each string will have its own dedicated controller)*.
`pv.amps` | amps | float *(array)* | mandatory | An ordered set of Current readings for PV strings, corresponding to values in `pv.volts`.  
`battery.volts` | volts | float | mandatory | The voltage of the connected Battery.
`load.volts` | volts | float *(array)* | mandatory | An ordered set of Voltage readings for connected Loads. *(__Note__: in a BTS site Load 1 is the VSAT system and Load 2 is the BTS)*.
`load.amps` | amps | float *(array)* | mandatory | An ordered set of Current readings for connected Loads, corresponding to values in `load.volts`.  

### 'inverter' Body parameter

The following snippet shows the structure of an `inverter` dataset:

```json
{
  "datasets": [
    { "inverter": { "id": "SPI-B2-01-001" }, 
      "data": [
        { "time": "20190209T150006.032-0700",
          "pv": { "volts": ["48.000", "48.000"], "amps": ["6.0", "6.0"] },
          "batt": { "volts" : "55.1", "amps": "-1.601" }, 
          "load": { "volts": ["48.000", "48.000"], "amps": ["1.2", "1.2"] }
        },
```

The dataset attributes are specified below. 

Attribute | Metric | Data | Optionality | Description
--- | --- | --- | --- | ---
`inverter.id` | - | string | mandatory | Id of the Inverter charge controller. *(__Note__: a site can have any number of Inverter controllers, and each controller can have multiple PV strings and Loads)*.
`time` | - | datetime | mandatory | The time of the event which produced this data sample, in compressed `ISO 8601/RFC3339` (YYYYMMDDThhmmss±hhmm).
`pv.volts` | volts | float *(array)* | mandatory | An ordered set of Voltage readings for PV strings connected to this MPPT. *(__Note__: presently upto 4 PV strings per controller are supported. In future each string will have its own dedicated controller)*.
`pv.amps` | amps | float *(array)* | mandatory | An ordered set of Current readings for PV strings, corresponding to values in `pv.volts`.  
`battery.volts` | volts | float | mandatory | The voltage of the connected Battery.
`load.volts` | volts | float *(array)* | mandatory | An ordered set of Voltage readings for connected Loads.
`load.amps` | amps | float *(array)* | mandatory | An ordered set of Current readings for connected Loads, corresponding to values in `load.volts`.  

# /device/dataset GET
---

The `/device/{device-id}/dataset/{dataset}` path allows field engineers to monitor an individual device during operation.
 
 [http:/api.endpoints.sundaya.cloud.goog/device/{**device-id**}/dataset/{**dataset**}/period/week/20150204/1](http:/api.endpoints.sundaya.cloud.goog/device/BBC-PR1202-999/dataset/BBC-MPPT/period/week/20150204/1)

- This path provide a dedicated endpoint to retrive data for an individual **device** and dataset. 

### Path parameters

The following path parameters are required in device GET requests. If a path parameter is omitted it will be substituted as described.    

Parameter | Description 
--- | --- 
`device` | The device identifier. 
`dataset` | The dataset type (see **{dataset} parameter** above). 

### Query parameters
There are no query parameters for the `/device/dataset` route.