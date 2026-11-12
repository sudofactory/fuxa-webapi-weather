# FUXA WebAPI Weather Demo

A FUXA 1.3.5 project that reads live weather from the [Open-Meteo](https://open-meteo.com/) Web API (no API key needed) and shows it on an HMI view.

- Device **My Weather** (WebAPI, GET, JSON, polled every 10 s) — Taipei (25.03, 121.56)
- Tags: `temperature_2m`, `relative_humidity_2m`, `wind_speed_10m` (values + units)
- View **MainView** with three value displays

## How to use

1. Install and start FUXA (http://localhost:1881).
2. Open the editor → **Open Project** → choose `fuxa-webapi-weather.project.json`.
3. Change `latitude` / `longitude` in the device address to your own location.

By Yottacontrol Co. — https://www.yottacontrol.com.tw
