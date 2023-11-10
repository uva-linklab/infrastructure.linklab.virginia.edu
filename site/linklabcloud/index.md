Link Lab Cloud
==============

The Link Lab cloud is a cloud-hosted streaming data backend that supports dozens
of sensor deployments and applications that use streaming sensor data.


Underlying Technologies
-----------------------

The Link Lab Cloud is built on (InfluxDB
v1](https://docs.influxdata.com/influxdb/v1/), a timeseries database for
streaming data.

[MQTT](https://mqtt.org/) provides connections between data sources and the
database.

All metadata are stored in a Postgres database and appended as tags for each
data item streamed to the Link Lab Cloud.

Data Format
-----------

### Data Stored in the Database

All datapoints in the LLC are stored in this general format:

```
Measurement: <measurement name>
  value: <measurement value>

Tags:
  <tag name>: <tag value>
  ...

Timestamp: <nanosecond timestamp>
```

This format follows the InfluxDBv1 format, except we use only a single field in
the measurement, which is always called "value". This makes it easier to query
for a specific type of data, no matter which sensor type or source it originated
from (since all temperature data will have the same measurement name and same
field name).

### Data Sent to LLC

When sending data to the LLC, the data are generally in the following JSON-esque
format:

```
{
    "<measurement1>": value1,
    "<measurement2>": value2,
    ...
    "_meta": {
        "device_id": "<unique device id>",
        "receiver": "<source of the data>",
        "model_number": "<model #>",
        ...
    }

}
```

`device_id` is required.

**Important!** It is critical that all fields in the `_meta` object change
**rarely** for each device_id. The `_meta` fields should be fixed properties of
the device (e.g. the model number of the sensor) and NOT a sensor reading (e.g.
output of a GPS). It is ok to change the `_meta` fields _occasionally_ (e.g.
every three months), or on rare occasions (like the device is moved to a new
location). Any value which is dynamic must be stored in the normal measurement
key-value pairs.


### Standard Data Keys

To make it easier to query for data across sensors, we maintain a list of known
measurement names that sensors should use if possible. They are listed in the
table:

| InfluxDB Measurement Name | Description                                                                |
|---------------------------|----------------------------------------------------------------------------|
| altitude_m                | Altitude in meters.                                                        |
| battery_%                 | Battery percentage.                                                        |
| co2_ppm                   | Carbon dioxide (CO2) concentration in parts-per-million.                   |
| Contact                   | Magnet contact sensor or reed switch. 1 means switch is closed, 0 is open. |
| energy_kWh                | Energy in kilowatt hours.                                                  |
| frequency_hz              | Frequency in hertz. Often used for AC electricity.                         |
| Humidity_%                | Humidity in percent.                                                       |
| Illumination_lx           | Light illumination in lux.                                                 |
| PIR Status                | Motion detected by a PIR sensor. 255 means motion, 0 means no motion.      |
| pm10.0_μg/m3              | Concentration of PM10.0 particles in micrograms per cubic meter.           |
| pm2.5_μg/m3               | Concentration of PM2.0 particles in micrograms per cubic meter.            |
| power_w                   | Power in watts.                                                            |
| pressure_hPa              | Pressure in hectoPascals (hPa).                                            |
| rssi_dbm                  | RSSI (received signal strength indicator) in decibel-milliwatts (dBm).     |
| spl_a                     | Sound pressure level averaged.                                             |
| Temperature_°C            | Temperature in degrees Celsius.                                            |
| voc_ppb                   | Volatile organic compounds (VOC) in parts-per-billion.                     |
| voltage_v                 | Voltage in volts.                                                          |

Of course, this is far from comprehensive. Other datastreams may use other
measurement names.

In general, we have found it useful to use the format `<data type>_<unit>`. This
balances complexity (it is fairly simple) with readability (queriers can
generally tell what the data are.)


Metadata
--------

Metadata for the LLC is stored in a table with the following structure:

| Column            | Type                   | Description                                                                                                                                                    |
|-------------------|------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------|
| sensorid          | character varying(128) | Same as `device_id`.                                                                                                                                           |
| type              | text                   | General name of the sensor type.                                                                                                                               |
| location_general  | text                   | Broad indication of the device's location and deployment scope. This is often a city, state, university, etc.                                                  |
| location_area     | text                   | Device's location at the scope of a physical system or where its data would be used with other devices in the same are. For example, the name of the building. |
| location_specific | text                   | A location identifier which is reasonably clear where the device is. For example a room number.                                                                |
| description       | text                   | Additional information about what the device is monitoring or where it is placed.                                                                              |
| details           | text                   | More information which might be useful. Sometimes a geohash or coordinate.                                                                                     |

On each matching `device_id == sensorid`, the other fields are appended as tags
to the incoming data point.


