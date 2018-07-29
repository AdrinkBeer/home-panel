# Home Panel

[![GitHub Release](https://img.shields.io/github/release/timmo001/home-panel.svg)](https://github.com/timmo001/home-panel/releases)
[![License](https://img.shields.io/github/license/timmo001/home-panel.svg)](LICENSE.md)
[![pipeline status](https://gitlab.com/timmo/home-panel/badges/master/pipeline.svg)](https://gitlab.com/timmo/home-panel/commits/master)
[![Waffle.io - Columns and their card count](https://badge.waffle.io/timmo001/home-panel.svg?columns=To%20Do,On%20Hold,In%20Progress,Done)](https://waffle.io/timmo001/home-panel)

[![Docker Version][version-shield]][microbadger]
[![Docker Layers][layers-shield]][microbadger]
[![Docker Pulls][pulls-shield]][dockerhub]
[![Anchore Image Overview][anchore-shield]][anchore]

A touch-compatible web-app for controlling the home.

![Light Theme Screenshot][light-theme]

![Dark Theme Screenshot][dark-theme]

![More Info Screenshot][more-info]

## Features

- Supports and can be used as alternate frontend for [Home Assistant](https://www.home-assistant.io/)
- Supports MJPEG and related image-based camera/image feeds
- Add weather and weather icons using Home Assistant's
 [Dark Sky](https://www.home-assistant.io/components/weather.darksky/) component
- Made for touch screens with a sideways scrolling Material
 Design interface. (Compatible with deskops also)

## Setup

### Docker Compose

- Install [Docker](https://www.docker.com/community-edition) and
 [Docker Compose](https://docs.docker.com/compose/install/)
- Create a directory for your compose file. For example, `home-panel`
- Create a `config.json` using the template below
- Create a `docker-compose.yml` file:

  > This example maps the `ssl` directory in the home directory.
  > Inside this directory are two files `fullchain.pem` and `privkey.pem`
  > which are files generated by [Let's Encrypt](https://letsencrypt.org/).
  >
  > Change the environment variables to your own Home Assistant details

  ```yaml
  ---
  version: '3'

  services:
    home-panel:
      image: timmo001/home-panel
      environment:
        REACT_APP_HASS_HOST: 'hassioserver.com'
        REACT_APP_HASS_PASSWORD: 'supersecurepassword'
        REACT_APP_HASS_SSL: 'true'
        REACT_APP_API_URL: https://localhost:3234
      ports:
        - 8234:443
      volumes:
        - ~/ssl:/ssl
        - $(pwd)/config.json:/usr/src/app/config.json
    home-panel-api:
      image: timmo001/home-panel-api
      environment:
        CERTIFICATES_DIR: /ssl
      ports:
        - 3234:3234
      volumes:
        - ~/ssl:/ssl
  ```

---

### Docker

- Install [Docker](https://www.docker.com/community-edition)
- Create a `config.json` using the template below
- Run image

  ```bash
  docker run -d \
  -e REACT_APP_HASS_HOST='hassioserver.com' \
  -e REACT_APP_HASS_PASSWORD='supersecurepassword' \
  -e REACT_APP_HASS_SSL='true' \
  -p 8234:443 \
  -v ~/ssl:/ssl \
  -v $(pwd)/config.json:/usr/src/app/config.json \
  timmo001/home-panel
  ```

  ```bash
  docker run -d \
  -e CERTIFICATES_DIR='/ssl' \
  -p 3234:3234 \
  -v ~/ssl:/ssl \
  timmo001/home-panel-api
  ```

---

### Node JS

#### API Setup

- First clone the
- [Home Assistant API](https://github.com/timmo001/home-panel-api) repository
- Checkout the version you want via releases
- Install packages

  ```bash
  yarn install
  ```

- Run

  ```bash
  node index.js
  ```

#### Webapp

- Clone this repository
- Checkout the version you want via releases
- Copy `.env` to `.env.local` and update it to use your own Home Assistant details
- Rename `config.template.json` to `config.json`

  ```bash
  cp config.template.json src/config.json
  ```

- Update `src/config.json` to your configuration. (See below for sample and options)
- Install packages

  ```bash
  yarn install
  ```

- Copy `config.json` into `node_modules/config.json`

  ```bash
  cp config.json node_modules/
  ```

- Build a production version

  ```bash
  yarn build
  ```

##### Production - Secure

- Install nginx
- Edit your server config `/etc/nginx/conf.d/default.conf`

  ```conf
  server {
    listen 443 ssl http2 default_server;
    listen [::]:443 ssl http2 default_server;

    root /usr/share/nginx/html;
    index index.html;
    server_name 172.0.0.1;

    ssl_certificate /ssl/fullchain.pem;
    ssl_certificate_key /ssl/privkey.pem;

    location / {
      try_files $uri /index.html;
    }
  }
  ```

- Copy the files in the build folder into nginx's html directory at
 `/usr/share/nginx/html`
- Start your server

##### Development

> **This option is not secure. Do not open to the outside world!**

- Run the app

  ```yarn start```

- The app should open in your default browser under `http://localhost:3000`

## Starter Template

```json

{
  "theme": {},
  "header": {
    "left_outdoor_weather": {
      "dark_sky_icon": "sensor.dark_sky_icon",
      "condition": "sensor.pws_weather",
      "data": [
        {
          "entity_id": "sensor.pws_temp_c",
          "unit_of_measurement": "°C"
        },
        {
          "entity_id": "sensor.pws_relative_humidity",
          "unit_of_measurement": "%"
        }
      ]
    },
    "right_indoor": [
      {
        "label": "Living Room",
        "data": [
          {
            "entity_id": "sensor.dht22_01_temperature",
            "unit_of_measurement": "°C"
          },
          {
            "entity_id": "sensor.dht22_01_humidity",
            "unit_of_measurement": "%"
          }
        ]
      }
    ]
  },
  "items": [
    {
      "name": "Living Room",
      "cards": [
        {
          "entity_id": "light.tv_light"
        }
      ]
    },
    {
      "name": "Dining Room",
      "cards": [
        {
          "entity_id": "light.table_light"
        }
      ]
    }
  ]
}

```

[light-theme]: https://raw.githubusercontent.com/timmo001/home-panel/master/docs/resources/light-theme.png
[dark-theme]: https://raw.githubusercontent.com/timmo001/home-panel/master/docs/resources/dark-theme.png
[more-info]: https://raw.githubusercontent.com/timmo001/home-panel/master/docs/resources/more-info.png
[anchore-shield]: https://anchore.io/service/badges/image/9577aceb95056f417958e6bb7536cc0394b5add554df0c63780875f3669f5c2e
[anchore]: https://anchore.io/image/dockerhub/timmo001%2Fhome-panel%3Alatest
[dockerhub]: https://hub.docker.com/r/timmo001/home-panel
[layers-shield]: https://images.microbadger.com/badges/image/timmo001/home-panel.svg
[microbadger]: https://microbadger.com/images/timmo001/home-panel
[pulls-shield]: https://img.shields.io/docker/pulls/timmo001/home-panel.svg
[version-shield]: https://images.microbadger.com/badges/version/timmo001/home-panel.svg
