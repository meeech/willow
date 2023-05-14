﻿# Willow

Willow is an [ESP IDF](https://github.com/espressif/esp-idf) based project for the [ESP BOX](https://github.com/espressif/esp-box) hardware from Espressif. Our goal is to provide Amazon Echo/Google Home competitive performance, accuracy, and functionality with Home Assistant and other platforms - 100% open source and completely self-hosted by the user (if desired) using low cost ($50 USD) commercially available hardware.

<img src="https://github.com/espressif/esp-box/blob/master/docs/_static/esp32_s3_box.png" width="200px" />

Current features include:

- Wake Word Engine. Say "Hi ESP" or "Alexa" (user configurable) and start talking!
- Voice Activity Detection. When you stop talking it will stop recording and take action.
- Support for Home Assistant! Simply configure Willow with your Home Assistant server address and access token.
- Support for other platforms. As long as your configured endpoint can take an HTTP POST with an auth header you can do anything with the speech output!
- Good far-field performance. We've tested wake and speech recognition from roughly 25 feet away in challenging environments with good results.
- Good audio quality - Willow supports features such as automatic gain control, noise separation, etc.
- Support for challenging Wi-Fi environments. Willow can (optionally) use audio compression to reduce airtime on 2.4 GHz Wi-Fi in cases where it's very busy.
- Operation on Wi-Fi "IoT" networks with no external internet/network access.
- LCD and touchscreen. The ESP BOX has color LCD and capacitive mult-point touchscreen. We support them with an initial (limited) user interface.
- Ready to go! The ESP BOX can be taken out of the package, flashed with Willow, and placed in your home or commercial/office environment and not look like a science experiment. All in a package that measures less than 3" x 3".
- Very low power. In our measurements the ESP BOX uses a maximum of 200 mW.
- Completely on device speech command recognition and support for our (soon to be released) Whisper-powered inference server (Tovera hosted best-effort inference server provided). Configure up to 400 commands detected completely on device or self-host our (coming soon) inference server to transcribe any speech!

Not bad for hardware you can order today from Amazon, Adafruit, The Pi Hut, or other preferred vendor for (approximately) $50 USD. Add a USB-C power supply and go!

## Getting Started

Configuring and building Willow for the ESP BOX is a multi-step process. We're working on changing that but for now...

### Dependencies

We use [tio](https://github.com/tio/tio) as a serial monitor so you will need to install that.

Ubuntu:

```sudo apt-get install tio```

Arch Linux:

```yay -S tio```

Mac (with homebrew):

```brew install tio```

### Set serial port

To do anything involving the serial port you will need to set the ```PORT``` environment variable for all further invocations of ```utils.sh```. 

With recent versions of ```tio``` you can use ```tio -L``` to list available ports. On Linux you can check ```dmesg``` and look for the path of the recently connected ESP BOX. On Linux it's ```/dev/ACM*``` and on Mac it's ```/dev/usbmodem*```.

Examples:

Linux:

```export PORT=/dev/ttyACM0```

Mac:

```export PORT=/dev/cu.usbmodem2101```

### Container
We use Docker for the build container. To build the docker image:

```./utils.sh build-docker```

Once the container has finished building you will need to enter it for all following commands:

```./utils.sh docker```

### Install
Once inside the container install the environment:

```./utils.sh install```

### Config

Start the config process:

```./utils.sh config```

Navigate to "Willow Configuration" to fill in your Wi-Fi SSID, Wi-Fi password (supports WPA/WPA2/WPA3), and your Willow server URI (best-effort Tovera hosted example provided).

For Home Assistant you will also need to create a [long lived access token](https://developers.home-assistant.io/docs/auth_api/#:~:text=Long%2Dlived%20access%20tokens%20can,access%20token%20for%20current%20user.) and configure your server address. By default we use ```homeassistant.local``` which should use mDNS to resolve your local Home Assistant instance. Put your long lived access token in the text input area. We recommend testing both your Home Assistant server address and token before flashing.

There are also various other configuration options for speaker volume, display brightness, NTP, etc.

If you want to change the wake word from the default "Hi ESP" you can navigate from the main menu to ```ESP Speech Recognition --> Select wake words --->``` and select Alexa or whichever. NOTE: If changing the wake word *ALWAYS* use the ```wn9``` variants.

Once you've provided those press 'q'. When prompted to save, do that.

### Build

```./utils.sh build```

### Flash

NOTE: Due to container issues with serial devices you need to flash from the host, so run the flash command(s) in another terminal with ```PORT``` defined.

For out of the box/factory new ESP BOX hardware you will need to (one time) erase the factory flash before flashing Willow:

```./utils.sh erase-flash```

Once you have done that you can flash:

```./utils.sh flash```

It should build, flash, and connect you to the serial monitor.

### Let's talk!

If you have made it this far - congratulations! You will see serial monitor output ending like this:

```
I (10414) AFE_SR: afe interface for speech recognition

I (10424) AFE_SR: AFE version: SR_V220727

I (10424) AFE_SR: Initial auido front-end, total channel: 3, mic num: 2, ref num: 1

I (10434) AFE_SR: aec_init: 1, se_init: 1, vad_init: 1

I (10434) AFE_SR: wakenet_init: 1

MC Quantized wakenet9: wakeNet9_v1h24_hiesp_3_0.63_0.635, tigger:v3, mode:2, p:0, (May  5 2023 20:32:52)
I (10704) AFE_SR: wake num: 3, mode: 1, (May  5 2023 20:32:52)

I (13:26:42.433) AUDIO_THREAD: The feed_task task allocate stack on external memory
I (13:26:42.434) AUDIO_THREAD: The fetch_task task allocate stack on external memory
I (13:26:42.442) AUDIO_THREAD: The recorder_task task allocate stack on external memory
I (13:26:42.451) WILLOW: app_main() - start_rec() finished
I (13:26:42.457) AUDIO_THREAD: The at_read task allocate stack on external memory
I (13:26:42.466) WILLOW: esp_netif_get_nr_of_ifs: 1
I (13:26:42.471) WILLOW: Startup complete. Waiting for wake word.
```

Your ESP BOX will initialize. You should see some help text on the display to use your configured wake word. Try some built in Home Assistant [intents](https://www.home-assistant.io/integrations/conversation/) like:

- "(Your wake word) Turn on master bedroom lights"
- "(Your wake work) Turn off kitchen lights"

The available commands and specific names, etc will depend on your Home Assistant configuration.

You can also provide free-form text to get an idea of the accuracy and speed provided by our inference server implementation. The commands will fail unless you've defined them in Home Assistant but the display will show the speech recognition results.

### Things went sideways - reset
In the event your environment gets out of whack we have a helper to reset:

```./utils.sh destroy```

As the plentiful messages indicate it's a very destructive process but it will reset your environment. After it completes you can start from the top and try again.

## Exit serial monitor
To exit ```tio``` you need to press CTRL+t and then 'q'.

## Recover from a bad flash

ESP devices are very robust to flashing failures but it can happen! If you end up "bricking" your device you can erase the flash and re-flash:

```./utils.sh erase-flash```

```./utils.sh flash```

## Advanced Usage

```utils.sh``` will attempt to load environment variables from ```.env```. You can define your ```PORT``` here to avoid needing to define it over and over.

The ESP-IDF, ESP-ADF, ESP-SR, LVGL, etc libraries have a plethora of configuration options. DO NOT change anything outside of "Willow Configuration" (other than wake word) unless you know what you are doing.

If you want to quickly and easily flash multiple devices or distribute a combined firmware image you can use the ```dist``` arguments to ```utils.sh```:

```./utils.sh dist``` - builds the combined flash image (```willow-dist.bin```)

```./utils.sh flash-dist``` - flashes the combined flash image

This combined firmware image can be used with any ESP flashing tool like the web flasher [ESP Tool](https://espressif.github.io/esptool-js/) so you can send firmware images to your less technical friends!. Just make sure to use offset 0x0 with those tools as we include the bootloader.

## Development

Development usually involves a few steps:

1) Code - do your thang!
2) Build
3) Flash

Unless you change the wake word and/or are using local command recognition (Multinet) you can selectively flash the application partition alone. This avoids long flash times with the wakenet and multinet model partition, etc:

```./utils.sh build```

```./utils.sh flash-app```

## The Future (in no particular order)

### Performance improvements
Willow and air-infer-api/Multinet already provide "faster-than-Alexa" responsiveness for a voice user interface. However, there are multiple obvious optimizations that could be made:

- ADF pipeline handing (we're waiting on ESP-ADF 2.6 with ESP-IDF 5)
- Websockets for inference server
- Websockets for Home Assistant
- Likely many more

These enhancements alone should dramatically improve responsiveness.

### No CUDA
The air-infer-api inference server (open source release soon) will run CPU only but the performance is not comparable to heavily optimized implementations like [whisper.cpp](https://github.com/ggerganov/whisper.cpp). For an Alexa/Echo competitive voice interface we currently believe that our implementation with CUDA or local Multinet (for limited commands) is the best approach. However, we also understand that isn't practical for many users. Between on device Multinet and further development on CPU-only Whisper implementations, ROCm, etc we will get there.

### TTS Output
Given the capabilities of Whisper speech commands like "What is the weather in Sofia, Bulgaria?" are transcribed but need to match a command (HA intent) on the destination. AIA (air-infer-api) has a text to speech engine and Home Assistant has a variety of options as well. In the event the final response to a given command results in audio output we can play that via the speakers in the ESP BOX.

### Higher Quality Audio Output
The ESP BOX supports bluetooth. In applications where higher quality audio is desired (music streaming, etc) we could support pairing to bluetooth speaker devices. Who knows? Eventually we may even design our own device with better internal speakers...

### LCD and Touchscreen Improvements
The ESP BOX has a multi-point capacitive touchscreen and support for many GUI elements. We currently only provide basic features like touch screen to wake up, a little finger cursor thing, and a Cancel button to cancel/interrupt command streaming. There's a lot more work to do here!

### Dynamic Configuration
Docker, building, configuring, flashing, etc is a pain. There are several approaches we plan to take to avoid this and ease the barrier to entry for users to get started.

### Over the Air Updates
ESP IDF and ESP BOX has robust support for over the air firmware updates. Down the road we will support them.

### Multiple Devices
The good news is the far-field wake word recognition and speech recognition performance is very good. The bad news is if you have multiple devices in proximity they are all likely to wake and process speech simultaneously. Commands will still work but multiple confirmation/error beeps and hammering your destination command endpoint is less than ideal. We have a few ideas about dealing with this too :).

### Custom Wake Word
Espressif has a [wake word customization service](https://docs.espressif.com/projects/esp-sr/en/latest/esp32s3/wake_word_engine/ESP_Wake_Words_Customization.html) that enables us (or you!) to create custom wake words. We plan to create a "Hi Willow" or similar wake word.

### GPIO
The ESP BOX provides 16 GPIOs to the user. We plan to make these configurable by the user to enable all kinds of interesting maker/DIY functions.
