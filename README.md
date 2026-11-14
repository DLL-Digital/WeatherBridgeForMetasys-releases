# <img src="images/icon.svg" width="40" height="40" align="center" alt=""> Weather Bridge for Metasys

**Live outdoor weather and forecasts as Metasys points.** Weather Bridge for Metasys reads current conditions and the
forecast from [OpenWeather](https://openweathermap.org/) and writes them to points in a Johnson Controls Metasys 16
building automation system through the Metasys REST API. Sequences, trends, graphics and alarms can then use outdoor
temperature, humidity, wind, enthalpy, the forecast high and low and more, like any other point.

It runs as a Windows service on a server that can reach both the internet and the Metasys site server, and it is set
up and monitored from a secure web interface. A product of **DLL Digital**.

This repository publishes the installers, with their license agreement and third-party notices. It contains no source
code.

**[Download the latest version](https://github.com/DLL-Digital/WeatherBridgeForMetasys-releases/releases/latest)**

**Not affiliated with Johnson Controls.** Weather Bridge for Metasys is an independent product. It is not affiliated
with, sponsored by, endorsed by or promoted by Johnson Controls International plc or its subsidiaries (JCI). Metasys is
a registered trademark of Johnson Controls; all other trademarks belong to their owners. This software is not a JCI
product, is not supported by JCI, and has no ties to JCI. It interacts with Metasys only through its published API.

## Features

- **Weather data.** Current conditions and a 5-day forecast with the OpenWeather free tier, or hourly and 8-day
  forecasts and weather alerts with a One Call 3.0 subscription. Wet bulb, enthalpy and dew point are calculated, and
  the site's elevation improves wet bulb and enthalpy when no station pressure is reported.
- **Backup provider.** If OpenWeather fails twice in a row, current conditions and the forecast come from
  [MET Norway](https://www.met.no/en) (no key needed) until OpenWeather answers again. On by default.
- **National Weather Service (optional, US sites).** Current conditions from the nearest weather station and active
  alerts from the [National Weather Service](https://www.weather.gov/).
- **Point setup in minutes.** Browse the Metasys site, create a standard set of weather points (with optional trend
  extensions) in one step, or map any weather value to a new or existing point.
- **Controlled writes.** Unit conversion, scale and offset, deadband, minimum and refresh intervals, read-back
  verification, and a hold or fallback value when weather data is stale.
- **Weather condition as a multistate value.** The current condition can be written to a 13-state point (clear,
  cloudy, fog, rain, snow, thunderstorm, severe and so on).
- **Monitoring.** Heartbeat and online status points to alarm on in Metasys, a `/health` address for IT monitoring
  tools, and entries in the Windows Event Log.
- **Dashboard and history.** Weather source, Metasys connection, the status of every mapping, live values and history
  charts on one page, plus *Sync now*.
- **Built-in help.** Every task and setting is explained in the web interface, and the help works offline.

## Requirements

| Item | Details |
|---|---|
| Server | 64-bit Windows Server 2016 / Windows 10 or later; Windows Server 2022 or later, or Windows 11, recommended. It can be the ADX/OAS itself. No .NET, IIS or SQL Server needed. |
| Network | HTTPS to `api.openweathermap.org` and `api.met.no` (and `api.weather.gov` if National Weather Service data is used), directly or through a proxy, and HTTPS to the Metasys site server. |
| Metasys | Metasys 16 with the REST API. The **MONCMD-API** licence is needed for reads, writes and commands. |
| Metasys account | A dedicated account allowed to use the API and to read and write objects, and to create objects if the bridge creates the points. |
| OpenWeather | An API key (the free tier is enough). A One Call 3.0 subscription is optional. New keys can take up to 2 hours to work. |
| Contact | An email address or web page for the site operator, which MET Norway and the National Weather Service require. |

The bridge brings its own copy of .NET and its own web server. It does not change the .NET runtimes, IIS, SQL Server
or anything else Metasys uses.

## Install

1. Download `MetasysWeatherBridge-x.y.z.msi` from the
   [latest release](https://github.com/DLL-Digital/WeatherBridgeForMetasys-releases/releases/latest) and run it as an
   administrator on the server. Keep the web interface port at **8443** (never 443 or 80, which IIS and Metasys use).
   Optionally enter a listen address on the BMS network to limit where the interface can be reached from.
2. Open `https://localhost:8443` on the server and create the administrator account. There are no default passwords.
3. In **Settings → OpenWeather**, enter the API key, the site location and the contact for MET Norway and the National
   Weather Service.
4. In **Settings → Metasys connection**, connect to the site server, then create and map points.

Setup takes about 15 minutes. **Help** in the web interface walks through each step, including silent installation
(`msiexec /i MetasysWeatherBridge-x.y.z.msi /qn HTTPSPORT=8443 OPENFIREWALL=1`), remote first-time setup and the
Metasys alarms to configure.

**Check a download** (prints True when it matches). Each release has a `.sha256` file next to the installer. In
PowerShell:

```powershell
(Get-FileHash .\MetasysWeatherBridge-x.y.z.msi -Algorithm SHA256).Hash -eq (Get-Content .\MetasysWeatherBridge-x.y.z.msi.sha256).Split(' ')[0]
```

## Upgrade

Run the newer MSI over the installed one. The install folder, port, listen address, firewall choice, settings, users,
mappings and history are kept, and a copy of the database is saved before an upgrade changes it. Each release's notes
list what changed and any steps for that version.

**Hear about new versions:** turn on **Settings → General & backup → Updates → Check for new versions** (off by
default). Once a day the bridge reads the latest release from this repository and shows a notice on the dashboard with
links to the release notes and the installer. It never downloads or installs anything, and sends nothing but the
request. Available in 1.7.6 and later. **A site on 1.8.0 must install 1.8.1 or later by hand once**, because the
repository moved from github.com/LarLTU to DLL-Digital; later versions find updates on their own again.

## Security

- **HTTPS only.** The web interface uses a self-signed certificate until an organisation certificate is imported, and
  warns 30 days before an imported certificate expires.
- **Sign-in with roles.** *Viewer* (dashboard and status), *Technician* (also points, mappings and the Activity log)
  and *Admin* (everything). Accounts lock for 15 minutes after five failed sign-ins.
- **First-time setup** only on the server itself, or remotely with a one-time setup code.
- **Encrypted secrets.** The OpenWeather key, Metasys and proxy passwords and the web certificate are encrypted with
  Windows DPAPI. The data folder is readable only by SYSTEM, Administrators and the service.
- **Safe Metasys connection.** HTTPS only, with the site server's certificate pinned by default. After a rejected
  login the bridge waits 15 minutes before trying again, so it can't lock out the Metasys account.
- **Activity log.** Every change records who made it and when.

## License and notices

Weather Bridge for Metasys is proprietary software, © 2026 DLL Technologies LLC (DLL Digital), all rights reserved.
Installing, copying or using it is subject to the [license agreement](LICENSE), which the installer (1.7.7 and later)
also shows. Only the released installers are licensed for use; nothing here grants a right to redistribute them. The
open-source components the program includes, and the terms of the weather data sources it uses (OpenWeather,
MET Norway, US National Weather Service), are in [THIRD-PARTY-NOTICES.md](THIRD-PARTY-NOTICES.md).

Weather data and forecasts can be wrong, late or missing. Make sure the building's sequences behave safely when a
weather point is stale or unavailable.
