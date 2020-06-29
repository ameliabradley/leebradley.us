+++
title = "Mini ARM servers using old phones and a pi at 91¢ a year (PART 1 - Wires)"
date = 2020-06-28
category = "Prog"

[taxonomies]
tags = ["rust", "ssg", "other"]
+++

Old phones have tons of good hardware in them, ripe for a hacked-together webserver at [91¢ a pop](https://www.zdnet.com/article/heres-how-much-it-costs-to-charge-a-smartphone-for-a-year/)! Groovy. Much less than the $57 / year you'd be shelling out for a barebones AWS nano server (which in many ways isn't nearly as powerful).

That's enough to convince me! Let's get cracking.
<!-- more -->

You're going to need:
* Raspberry Pi
* Dusty old ARM phones (NO ROOT NECESSARY 🎉)
* OPTIONAL: Powered USB hub

## Architecture overview

{% mermaid() %}
graph LR
  I(("🌍")) -->|Ethernet|RP
  RP("Raspberry 🥧") -->|USB|SP("Samsung Galaxy S5 📱")
  RP -->|USB|N6("Nexus 6 📱")
  RP -->|USB|Foo("Next junk phone...")

class I transparent;
{% end %}

**It might seem strange, but this is the only viable setup.** I explain at the bottom why you can't use wifi or ethernet for old unrooted phones.

## Android phone prep

We need remote access to a terminal. If possible, install [Termux](https://f-droid.org/en/packages/com.termux/) and start `sshd`.

If not, install [SimpleSSHD](https://f-droid.org/en/packages/org.galexander.sshd/) and start it up.

Uninstall and disable, and stop all other non-essential programs (mail, calendar, etc).

**NOTE:** Technically your phones can function as servers at this point. But really, SSH into these phones now. Notice a lag? Your phones are old and slow, but not THAT slow. This is why we need the pi.

## Pi setup

Connect the pi to your wifi or ethernet. It needs to bridge the internet and these phones. We'll do that with ADB.

On the pi:

```
sudo apt-get update
sudo apt-get install -y android-tools-adb android-tools-fastboot
```

## Frankenstein it all together

For each phone:
* Enable [developer mode](https://developer.android.com/studio/debug/dev-options)
* Enable USB debugging
* Plug into the pi's open USB port
* You will get a popup on the screen asking you to add the pi as a trusted source. Add it.

**NOTE:** You might need to supplement the pi's power to the phones with a [powered USB hub](https://www.amazon.com/s?k=Powered+Usb+Hub).

Now, on the pi, get the phone ids assigned by adb:
```
$ adb devices -l
List of devices attached
9660a7c2               device usb:1-1.2.5 product:jfltevzw model:SCH_I545 device:jfltevzw transport_id:6
ZX1G227F87             device usb:1-1.4 product:shamu model:Nexus_6 device:shamu transport_id:1
```

Forward the necessary ports for each device with this command.
```
adb -s [PHONE ID] forward tcp:[PI PORT] tcp:[PHONE PORT]
```

Forward sshd with either:
* 8022 (Termux default)
* 2222 (OpenSSHD default)

You can view what ports you have forwarded with this friendly command:
```
$ adb forward --list
ZX1G227F87 tcp:8022 tcp:8022
9660a7c2 tcp:2222 tcp:2222
```

You should now be able to ssh into your phones using your pi as a jump proxy. There should be a dramatic jump in speed.

## Security concerns

Elephant in the room here. We're talking about old unmaintained phones. These guys aren't getting any security updates. If it suits your purposes, maybe you should *disable the phone wifi entirely and only expose the minimum necessary ports via the pi*. You can't be too paranoid. Honestly it might not be a bad idea to stick them in a faraday cage. Don't give the phones any way "reach out" to the rest of your network. They could rebel! 🦾

Also, don't root the phones... it's unnecessary, but it's also a potential security problem. Not kidding.

## Conclusions

Now you have phones with a fast network connection. That should be enough to tinker around a bit. I'd suggest compiling some Go or Rust code for ARM7 and pushing it over to see how it runs.

That's it for *PART 1 - Wires*. Things I hope to cover in the next chapters:
* Lightweight reverse proxy using [Sōzu](https://github.com/sozu-proxy/sozu)
* Blazing fast logging using [Vector](https://vector.dev/)

## Questions and answers

### Why not use Wifi?
<img src="/blog/arm-phone-army/stop-wifi.png" style="width: 100%">

**Android Wifi is purposefully slow.** Testing revealed intermittent delays of up to seconds per request. The wifi chip in most phones is setup to turn off periodically for power-saving measures. **This is hardcoded the kernel.**

### Why not use USB ethernet?
<img src="/blog/arm-phone-army/power-usb.svg" style="width: 100%">

**Power or USB host controller? Pick one.** My old phones have microusb ports with USB 3.0, which does not support charging while connected to an external device (using something like an ethernet -> USB dongle). 

**NOTE:** This is NOT an issue for USB C. Looking forward to dedicated ethernet ports for my next generation of frankenstein ARM servers 😁