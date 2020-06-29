+++
title = "Host on ARM using old phones at 91¢ a year"
date = 2020-06-28
category = "Prog"

[taxonomies]
tags = ["rust", "ssg", "other"]
+++

Old phones have tons of good hardware in them, ripe for a hacked-together webserver. At first I thought there's no way this is efficient... I mean, the phone's got a big screen, lots of unnecessary unremovable apps, etc. But then I came across this ZDNet article: [Here's how much it costs to charge a smartphone for a year](https://www.zdnet.com/article/heres-how-much-it-costs-to-charge-a-smartphone-for-a-year/) (Spoiler: 91¢)

That's enough to convince me! Here's my setup:
<!-- more -->

{% mermaid() %}
graph LR
  I(("🌍")) --> RP
  RP("Raspberry 🥧") --> SP("Samsung Galaxy S5 📱")
  RP --> N6("Nexus 6 📱")

class I transparent;
{% end %}

## Problems and solutions

### Phone wifi is purposefully slow 
<img src="/blog/stop-wifi.png" style="width: 100%">

**Problem:** Testing revealed intermittent delays of seconds per request. The wifi chip in most phones is setup to turn off periodically for power-saving measures. This is hardcoded the kernel.

**Solution:** Wired internet connection

### USB Charging or Host Controller? Pick one

**Problem:** So, ethernet? My old phones have microusb ports with USB 3.0, which does not support charging while connected to an external device (using something like an ethernet -> USB dongle).

**Solution:** Connect the phones to a spare powered USB hub, as a subdevice is connected to the pi. Use ADB on the pi to forward ports. This is a little more work, but works.

First, install adb:

```
sudo apt-get update
sudo apt-get install -y android-tools-adb android-tools-fastboot
```

Next, make sure all the phones have developer mode enabled. Add the pi as a trusted source.

Get the ID of the device:
```
$ adb devices -l
List of devices attached
9660a7c2               device usb:1-1.2.5 product:jfltevzw model:SCH_I545 device:jfltevzw transport_id:6
ZX1G227F87             device usb:1-1.4 product:shamu model:Nexus_6 device:shamu transport_id:1
```

Then forward the port for each device.
```
adb -s [PHONE ID] forward tcp:[PI PORT] tcp:[PHONE PORT]
```

You'll probably want to forward things like:
* sshd - Install on the phones with either [SimpleSSHD](https://f-droid.org/en/packages/org.galexander.sshd/) or preferably [Termux](https://f-droid.org/en/packages/com.termux/)
* Phone ports you want exposed

You can view what ports you have forwarded with this friendly command:
```
$ adb forward --list
ZX1G227F87 tcp:8022 tcp:8022
9660a7c2 tcp:2222 tcp:2222
ZX1G227F87 tcp:8090 tcp:8090
```

## Security concerns

## Comparison with AWS

For static hosting, AWS charges $1-2 / month (or 50¢ / month at the free tier) [[1]](https://aws.amazon.com/getting-started/hands-on/host-static-website/). You don't get a dedicated quad-core 2.5 GHz processor like my discontinued [Samsung Galaxy S5](https://www.gsmarena.com/samsung_galaxy_s5-6033.php).

## Logging

https://vector.dev/guides/getting-started/structuring

## Reverse proxy

https://github.com/sozu-proxy/sozu

Points:
* Link to my own forked code for ARM7
* Let's Encrypt?