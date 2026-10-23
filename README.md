# weewx-airQ
WeeWX service for Corant airQ devices over local HTTP, including particulate
matter, gas readings, indoor climate and device indices. Available readings
depend on the sensors fitted to each device.

This repository mirrors [roe-dl/weewx-airQ](https://github.com/roe-dl/weewx-airQ).
The installation below uses the separately repaired catalog archive
`0.9b3.post20261022`. A GitHub source-branch download does not include those
packaging repairs.

The catalog targets WeeWX 5.5–5.x and Python 3.10–3.14. The runtime still reports
0.9b3; the catalog suffix identifies the managed installation layout.

PM<sub>1.0</sub>, PM<sub>2.5</sub>, PM<sub>10.0</sub>, TVOC,
CO, CO<sub>2</sub>, O<sub>2</sub>, O<sub>3</sub>, NO<sub>2</sub>,
H<sub>2</sub>S, SO<sub>2</sub>, noise, health index, performance index,
temperature, humidity, dewpoint, air pressure

## Prerequisites

An airQ device reachable from the WeeWX host, its device password, and the
Python packages `pycryptodomex` and `six` in WeeWX's environment:

```sh
python -m pip install pycryptodomex six
```

`Cryptodome` comes from `pycryptodomex`; Python's base64, HTTP, JSON and threading
modules are part of the standard library.


## Installation instructions:

Install the published archive with WeeWX 5's extension installer:

```sh
weectl extension install https://github.com/weewx-packages/catalog/releases/download/airq/v0.9b3.post20261022/airq-0.9b3.post20261022.tar.gz
```

Use the `weectl` from your WeeWX environment. Add
`--config /path/to/weewx.conf` when the station configuration is not found
automatically. Package installations may require `sudo`; activate the WeeWX
virtual environment for pip installations. Restart your WeeWX service after
configuration, for example `sudo systemctl restart weewx` on systemd.
The installer does not restart it for you.

The catalog installer adds `user.airQ_corant.AirqUnits` to `prep_services` and
`user.airQ_corant.AirqService` to `data_services`. It installs
`airQ_corant.py` and `airq_conf.py` in the station's `bin/user`, without a
separate top-level `airq_conf` executable. Keep the existing station driver.

3) edit configuration in weewx.conf

   ```
   [airQ]

       query_interval = 5.0 # this is the default, if option is missing
       volume_mass_method = 1 # 0 - temp/pressure independent factor

       [[first_device]]
           host = replace_me_by_host_address_or_IP
           password = replace_me
           #prefix = replace_me # optional
           #altitude = value, unit # optional, default station altitude
           #query_interval = value # optional, if different from general setting

       [[second_device]]
           ...
   ...
   [Engine]
       [[Services]]
           prep_services = ... ,user.airQ_corant.AirqUnits
           data_services = ... ,user.airQ_corant.AirqService
           ...
   ```

   The section names can be any name. It need not be something like `[[first_device]]`. We recommend
   to use some reference to the location of the device like `[[bedroom]]`, `[[livingroom]]`, or the like.

5) restart weewx

   ```
   sudo /etc/init.d/weewx stop
   sudo /etc/init.d/weewx start
   ```

## Usage:

Using this extension you get the readings out of the airQ measuring
device and can

* save them to the database
* display them using tags in skins
* display them in graphs

### Prefix

Most of the the observation types provided by the airQ device are
predefined within WeeWX. If no special configuration besides host
address and password is provided the readings are stored to
those observation types.

If you have more than one airQ device, you need different names of the
observation types for each of them. That's the prefix for.
Imagine there are 3 airQ devices, one outside, one in the bedroom and
one in the living room. Your could configure the outside device without
prefix to put `co`, `co2`, `no2`, `noise`, `o3`, `pm1_0`, `pm2_5`,
`pm10_0`, and `so2` into the predefined columns of the WeeWX database.
For the bedroom device you could set `prefix = bedroom`. So you get
`bedroom_co`, `bedroom_co2` etc. for that device. And for the living
room device you could set `prefix = livingroom`. So you get
`livingroom_co`, `livingroom_co2`, etc. for that.

### Adding additional columns to the database

This extension comes with a special configuration tool available as `user.airq_conf`.
The tool is based on the WeeWX `wee_database` tool and provides means
to add the necessary columns according to the prefix setting to the
database or drop them from it. See below for details.

### Special cases

If you run several instances of WeeWX and transfer readings from one
to the other by MQTT, parallel database access, or something, then
you also need to install
weewx-airQ at the target system and include `user.airQ_corant.AirqUnits`
in line `prep_services`, but omit `user.airQ_corant.AirqService`
in line `data_services`.
That is because in initialization of the weewx-airQ
extension several additional units and conversations are
added to the lists in `units.py` They are neccassary to convert
between ppm and ppb as well as µg/m<sup>3</sup>, mg/m<sup>3</sup>,
and g/m<sup>3</sup>, respectively.

### Display values (CheetahGenerator)

The observation types described below can be used for tags as described
in the [WeeWX Customization
guide](http://weewx.com/docs/customizing.htm#Tags).

Example: `$current.TVOC`

### Diagrams (ImageGenerator)

To create diagrams you need to include additional sections into the
[ImageGenerator] section of skin.conf. This is an example:

```
        [[[dayPM]]]
            [[[[pm1_0]]]]
            [[[[pm2_5]]]]
            [[[[pm10_0]]]]
```

The result is as follows:

<img src="dayPM.png" />


## Observation types:

Dependent on hardware configuration the following observation types
are provided. The names are given as if no prefix is specified:

* **airqDeviceID**: (provided if included in `[StdWXCalculate]` only)
  device ID of the device
* **airqStatus**: (provided if included in `[StdWXCalculate]` only)
  sensor error messages or "OK" if none
* **airqBattery**: battery status (actually not used)
* **co**: CO concentration
* **co2**: CO<sub>2</sub> concentration
* **dCO2dt**: CO<sub>2</sub> changing rate
* **dHdt**: absolute humidity changing rate
* **airqDewpoint**: dewpoint
* **airqDoorEvent**: (experimental) door opened or closed
* **h2s**: H<sub>2</sub>S concentration
* **airqHealthIdx**: health index (special index according to a newly
  developed algorithm from the manufacturer)
* **airqHumidity**: relative humidity
* **airqHumAbs**: absolute humidity
* **airqMeasuretime**: duration of the last measuring cycle
* **no2**: NO<sub>2</sub> concentration
* **o3**: O<sub>3</sub> concentration
* **o2**: O<sub>2</sub> concentration
* **airqPerfIdx**: performance index (special index according to a newly
  developed algorithm from the manufacturer)
* **pm1_0**, **pm2_5**, **pm10_0**: particulate matter
* **cnt0_3**, **cnt0_5**, **cnt1_0**, **cnt2_5**, **cnt5_0**, **cnt10_0**:
  amount of particles
  of the appropriate size
* **TypPS**: typical particle size
* **airqPressure**: air pressure
* **airqAltimeter** altimeter value (air pressure corrected by altitude,
  software calculated, not received from the device)
* **airqBarometer**: barometer value (air pressure corrected by altitude
  and outside temperature, software calculated, not received from the
  device)
* **so2**: SO<sub>2</sub> concentration
* **airqNoise**: sound level
* **airqTemp**: temperature
* **TVOC**: volatile organic compounds concentration
* **airqUptime**: uptime of the device

If a prefix is provided "airq" is replaced by the prefix. If the
name does not start by "airq" the prefix is prepended to the name.

## Configuration tool `user.airq_conf`

Use the Python environment running WeeWX. Replace `/path/to/station/bin` with
the parent of your installed `user` directory:

```sh
PYTHONPATH=/path/to/station/bin python -m user.airq_conf --help
```

The reference below uses that invocation. These legacy helper functions depend
on WeeWX database utility APIs; check `--help` before using them on your version.


### General options

* `--device=DEVICE`: airQ device to set/get configuration for
* `--config=CONFIG_FILE`: use configuration file CONFIG_FILE. Default
  according to the way of WeeWX installation
* `--binding=BINDING_NAME`: Use binding BINDING_NAME. Default is `wx_binding`.

### General commands

* `PYTHONPATH=/path/to/station/bin python -m user.airq_conf --help`:
  display usage instructions
* `PYTHONPATH=/path/to/station/bin python -m user.airq_conf --device=DEVICE --print-config`:
  read the device configuration and display

### Add or drop columns to/from the database

The `wee_database` utility allows adding columns to the database one by one
only. As the airQ device provides a lot of additional observation types,
this is quite uncomfortable. The `airq_conf` tool provides the possibility
to add all the additional columns in one step. It regards the `prefix`
configuration option. By using the original WeeWX
functions for that, it should be safe. Nevertheless, backups are alsways a
good idea. And stop WeeWX for that.

**CAUTION:** Stop WeeWX and make a backup of the database before using these
commands.

* `PYTHONPATH=/path/to/station/bin python -m user.airq_conf --device=DEVICE --add-columns`:
  add the necessary columns to the database
* `PYTHONPATH=/path/to/station/bin python -m user.airq_conf --device=DEVICE --drop-columns`:
  drop the columns from the database

### Create a skin

The configuration utility comes with an option to create a simple skin
based on the Seasons skin and the settings in `weewx.conf`. It creates
a separate page for each device configured showing the current readings
and diagram images. Diagram are only possible for observation types
that are saved to the database.

* `PYTHONPATH=/path/to/station/bin python -m user.airq_conf --create-skin`

The command does **not** change `weewx.conf`. If you want to use this
skin you need to add the following section into the `StdReport` section
of `weewx.conf`.

```
    [[airqReport]]
        enable = true
        skin = airQ
        lang = de
        unit_system = METRIC
        HTML_ROOT = /var/www/html/weewx/airQ
```

The `lang` option must have the same value as it has for the Seasons
skin.

For observation types that are not saved to the database, no diagram
can be created. In these cases the diagram is empty.

**Note**: The language and unit system options are available from
WeeWX version 4.6 alpha on, only. Starting with WeeWX version
4.6 beta, the localization changed vom bracket to function style.
That is automatically honored.

Example:

<img src="weewx-airQ-Screenshot.png" />

### Set configuration parameters in the airQ device

Those commands change configuration settings within the airQ device.

* `PYTHONPATH=/path/to/station/bin python -m user.airq_conf --device=DEVICE --set-location=station`:
  set the location of the airQ device to the station location
* `PYTHONPATH=/path/to/station/bin python -m user.airq_conf --device=DEVICE --set-location=LATITUDE,LOGITUDE`:
  set the location of the airQ device to latitude and longitude provided
* `PYTHONPATH=/path/to/station/bin python -m user.airq_conf --device=DEVICE --set-roomsize=HEIGHT,AREA`:
  set the room size data
* `PYTHONPATH=/path/to/station/bin python -m user.airq_conf [--device=DEVICE] --set-ntp=NTP_SERVER`:
  set the NTP server
* `PYTHONPATH=/path/to/station/bin python -m user.airq_conf [--device=DEVICE] --set-ntp=de`:
  set the NTP server to the official german server of PTB.

## Links:

* [airQ homepage](https://www.air-q.com) - [airQ forum](https://forum.air-q.com)
* [airQ data sheet (german)](https://uploads-ssl.webflow.com/5bd9feee2fb42232fe1d0196/5f898b110a9e9fea8049fa29_air-Q_Specs_de_aktuell_2020-06-25.pdf)
* [WeeWX homepage](http://weewx.com) - [WeeWX Wiki](https://github.com/weewx/weewx/wiki)
* [Wöllsdorf weather conditions](https://www.woellsdorf-wetter.de)

## Upgrade, removal and license

Back up `weewx.conf`, custom templates and databases before installing a newer
fixed archive. To remove the extension, use `weectl extension uninstall airQ`
and restart WeeWX. The extension does not automatically add every device
observation to your archive schema. Prefixes need matching database columns
and template fields. Device access and sensor accuracy require your hardware.

Johanna Roedenbeck's extension retains its [GPLv3 license](LICENSE).
