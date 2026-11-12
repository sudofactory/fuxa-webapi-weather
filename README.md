# FUXA WebAPI Examples

**English** | [繁體中文](README.zh-TW.md)

Three ready-to-import [FUXA](https://github.com/frangoteam/FUXA) projects that pull live data from public Web APIs and put it on an HMI screen. No hardware, no API key, no account.

They build on each other, so start with the first one.

| # | File | What you get | Scripts |
|---|------|--------------|---------|
| 1 | [`fuxa-webapi-weather.project.json`](fuxa-webapi-weather.project.json) | Live temperature, humidity and wind for your city | None |
| 2 | [`fuxa-webapi-weather-alarm.project.json`](fuxa-webapi-weather-alarm.project.json) | Example 1 plus data logging, a 24-hour trend chart and an alarm | None |
| 3 | [`fuxa-webapi-adopt-pet.project.json`](fuxa-webapi-adopt-pet.project.json) | A "pet of the day" adoption card with photo, built from a list-type API | One view without, one view with |

Tested on FUXA 1.3.5.

## How to import

1. Install and start FUXA, then open the editor at `http://localhost:1881/editor`.
2. Click the menu at the top left and choose **Open Project**.
3. Pick one of the `.project.json` files.

> **Open Project replaces the project you have open.** If you have work of your own, use **Save Project as…** first to keep a copy.

> **The screen is blank at first. Wait about a minute.** FUXA only reads the Web API once every polling interval (60 seconds in the weather examples), so right after you import a project or start FUXA, the values are empty until the first reading arrives. Nothing is broken; give it a minute.

The FUXA server makes the API calls, so the machine running FUXA needs internet access.

---

## 1. Weather — `fuxa-webapi-weather.project.json`

The starting point: one Web API connection, three numbers on screen.

- **Data source:** [Open-Meteo](https://open-meteo.com/) current weather. Free, no API key.
- **Device `My Weather`:** type WebAPI, method GET, format JSON, polled every 60 seconds. The address is set to Taipei (`latitude=25.03&longitude=121.56`).
- **Tags:** `temperature_2m`, `relative_humidity_2m` and `wind_speed_10m`. Each one exists twice: the value (`current:…`, number) and its unit (`current_units:…`, text).
- **View `MainView`:** three value displays, each showing a number followed by its unit.
- **`My Weather Connection Status`:** a tag on the FUXA Server device that tells you whether the API connection is up.

**Make it yours:** open **Connections**, edit `My Weather`, and change `latitude` and `longitude` in the address to your own city.

**First minute:** the three values show nothing until the first 60-second poll completes.

## 2. Weather with history and an alarm — `fuxa-webapi-weather-alarm.project.json`

The same weather screen, now doing two things a plain web page does not: it remembers the past and it warns you.

Everything from example 1, plus:

- **Data logging:** the humidity tag has DAQ switched on, so FUXA stores a reading every 60 seconds.
- **Trend chart `Humidity history`:** sits beside the numbers and covers the last 24 hours. It reloads the stored readings when you open the page, so the line is still there after a refresh or restart.
- **Alarm `Muggy air`:** a High alarm on humidity. When the reading is between 59 and 100 %, FUXA raises the message "Muggy air outside" and the alarm bell in the header shows a count. When humidity drops back below 59 %, the alarm ends but stays in the list until you acknowledge it.

Good to know:

- The numbers are blank for the first minute, as in example 1.
- The chart starts empty right after import. It fills in as readings arrive, one per minute.
- Taipei is usually above 59 %, so you should see the alarm soon after import. To change the limit, open **Alarms** in the editor and edit `Muggy air`.

## 3. Pet of the day — `fuxa-webapi-adopt-pet.project.json`

A SCADA tool showing something that is clearly not a factory: animals in Taiwan's public shelters that are waiting for a home. The on-screen text is in Traditional Chinese.

- **Data source:** the Taiwan Ministry of Agriculture open dataset "動物認領養" (animals open for adoption) at `data.moa.gov.tw`. No API key.
- **What is different from the weather API:** the response is a *list* of records, not a single object. The address ends in `$top=1&$skip=N`, which asks for one record, the Nth. Tag addresses therefore start with `[0]:`, for example `[0]:animal_kind`.

The project has two views. The start view is the scripted one.

Both views also start out empty. The no-script view fills in within about 10 seconds; the daily card can take up to a minute while the script finds an animal with a photo.

### View "WebAPI 極簡綁定" — no scripts

The simplest way to show list data: connect, point tags at fields, bind tags to the screen.

- **Device `ADOPT_BASIC`:** WebAPI, GET, JSON, polled every 10 seconds, fixed at `$skip=0`.
- **Tags `PB_*`:** eight fields — species, breed, sex, size, age, colour, shelter name and photo URL.
- **Screen:** each field is shown next to its JSON field name, so you can see which field feeds which element. Values appear exactly as the API sends them, such as `M`, `SMALL`, `CHILD`.
- **Photo:** an SVG widget with a `_ps_url` variable bound to the `album_file` tag. The widget sets that URL as the image source.

To see a different animal, edit `ADOPT_BASIC` in **Connections** and change the number after `$skip=`. Some records have no photo, in which case the photo area stays empty.

### View "每日認養毛孩" — with scripts

The same data turned into a finished card. Two scripts do the work.

**`petDaily`** (server script, runs every 3 seconds)

- Picks one animal at random per day (Taiwan time) by rewriting `$skip` in the `ADOPT_PET` connection address and triggering a read.
- Skips records that have no photo, are not yet open for adoption, or were shown in the last 30 picks, and tries again.
- Translates the API's codes into readable words: `M`/`F` to 公/母, `SMALL`/`MEDIUM`/`BIG` to 小型/中型/大型, `CHILD`/`ADULT` to 幼年/成年, `T`/`F` to 已完成/尚未.
- Works out how many days the animal has been waiting, counted from its open-for-adoption date.
- Writes the finished text to 21 internal tags (`PAD_*`) on the FUXA Server device. The screen binds to those.
- Remembers today's pick and the history in `_petdaily/state.json` under the folder FUXA runs from, so a restart keeps the same animal.

**`petUI`** (client script, runs when the page opens)

- Loads the photo first, then fades the whole card in, so you never see new text beside the old photo.
- Shrinks long breed names, shelter names and addresses to fit their boxes.
- Adds the **透視 JSON** (X-ray) button: it overlays each element with the raw JSON field, its original value and the tag it is mapped to. Useful for explaining how the card is built.
- Makes **官方認養資訊** open the official adoption notices at `pet.gov.tw` in a new tab.
- Does nothing inside the editor, so opening the view to edit it will not save run-time changes into the project.

**On the screen**

- **換一隻** (Next one) sets the `PAD_CMD` tag to 1. `petDaily` sees it and picks another animal. The button reads 正在找… while it searches.
- Device `ADOPT_PET` has 17 tags (`PA_*`) for the raw fields. It is polled hourly because the script triggers a read whenever it needs one.

**Data licence:** the adoption data is published by the Ministry of Agriculture under Taiwan's [Open Government Data License](https://data.gov.tw/license). Details can be out of date; always confirm with the shelter.

---

By Yottacontrol Co. — https://www.yottacontrol.com.tw · [YouTube](https://www.youtube.com/@Yottacontrol巨控自動化)
