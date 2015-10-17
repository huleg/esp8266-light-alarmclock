# ESP8266 Light Alarm Clock
Being woken up by a loud alarm sound, trying to find the dismiss button on your phone is propably one of the most disruptive ways to start your day. A much more convenient alternative is to slowly dim up the room light to simulate the sunrise. This project is an implementation of this alternative alarm clock.

## Hardware
![Schematic](/hardware/schematic.png?raw=true)
![Hardware in a box](/hardware/box.png?raw=true)
![PCB inside box](/hardware/pcb.png?raw=true)

The alarm clock is based on the ESP-01 WiFi module with the ESP8266 WiFi chip. GPIO2 of the ESP-01 is simply connected to some transistors that control an LED Strip (could be any other load). The capacitor makes the transition between different light intensities (achieved using PWM) smooth. The 3.3V for the ESP-01 can be provided by an LF33CV or LF33CDT. The default 512KB flash chip is enough for storing all the software and alarm times.

Includes a KiCad schematic at `hardware/schematic.sch`.

## Software
![Set brightness page](/screenshots/setbrightness.png?raw=true)
![Set alarm times page](/screenshots/alarmtimes.png?raw=true)

### Configuration
* You can either set a static IP Address or tell the clock to use DHCP. In order to use a dynamic address, comment out `#define USE_STATIC` in `src/user_config.h`. Otherwise, adapt these values in `src/user_config.h` to your network:

```c
#define USE_STATIC
#define IP_ADDR	"192.168.0.50"
#define IP_GATEWAY "192.168.0.2"
#define IP_SUBNET "255.255.0.0"
```

* To set your timezone or switch to another time API (other than [http://www.timeapi.org](http://www.timeapi.org)), edit `#define TIMEZONE "cet"` and `#define TIMEAPI_URL ...` in `src/user_config.h`
* If you want to change the light dimming behaviour when an alarm occurs, edit the following defines in `src/user_config.h`: `ALARM_HOLD_DURATION` (time that the light stays on at full brightness after dimming up), `ALARM_DIM_DURATION` (time that dimming up takes), `ALARM_DIM_INTERVAL` (interval in which the clock checks whether increasing the light intensity is necessary when dimming up), `ALARM_DIM_START` (intensity at which dimming starts, then linear interpolation until `ALARM_DIM_STOP` is used), `ALARM_DIM_STOP` (intensity at which dimming ends, then jumps to `ALARM_DIM_FINAL` intensity and holds it), `ALARM_DIM_FINAL` (intensity that is held for `ALARM_HOLD_DURATION` seconds)

### Compilation
In order to compile esp8266-light-alarmclock, you need a fully set up [https://github.com/pfalcon/esp-open-sdk](esp-open-sdk), tested with Espressif's IOT SDK version 1.4.

You have to set your WiFi SSID and passphrase at compile time. In order to do that, execute
```bash
make flash WIFI_PASS="YOUR_PASSPHRASE" WIFI_SSID="YOUR_SSID"
```
You can then proceed to flash the image to your ESP8266 with `make flash`.

### Description
* The software acts as an HTTP server. You can access the alarm clock's site at its IP, Port 80. There you can manually set the light intensity or add and remove (up to 21) alarm times. The website is optimized for mobile browsers.
* The software also contains an HTTP client that connects up to [http://www.timeapi.org](http://www.timeapi.org) to retrieve the current time in your timezone. But even if your network connection happens to fail sometimes, the internal clock will keep going. This could potentially be replaced with NTP.
* Alarm times that are set on the webpage are saved to RAM and Flash so that even after a power failure the clock will remember when to wake you up.
* Shortly after the alarm times, the light will slowly dim up. If, however, you manually set the light's brightness manually, this will override the dimming process.

## Attribution
* Compile using pfalcon's [esp-open-sdk](https://github.com/pfalcon/esp-open-sdk)
* Makes use of Spritetm's wonderful [libesphttpd](http://git.spritesserver.nl/libesphttpd.git/) / [GitHub mirror](https://github.com/Spritetm/libesphttpd)
* Includes (a slightly modified version of) caerbannog's [esphttpclient](https://github.com/Caerbannog/esphttpclient)
* Uses [http://www.timeapi.org](http://www.timeapi.org) as an easy way to synchronize network time. Can be modified to use similar services instead.
