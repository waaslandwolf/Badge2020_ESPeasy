# Badge2020_ESPeasy

**Installing ESPeasy on the Fri3dcamp 2020 badge hardware**

This document descibes how to install and configure ESPeasy firmware on the Fri3dcamp 2020 badge.

What *works* and has been tested:
- TFT display
- NeoPixel LEDs
- Battery voltage monitoring
- BOOT button
- Buzzer
- LED on GPIO-25 shows wifi connection status

What *should work* but hasn't been tested:
- I²C communications
- ISP communications
- MHZ19 CO2 sensor
- Reading charge IC status on pins GPIO 34 and 35
- IR receiver (would probably work with custom_IR ESPeasy firmware but is untested)
- Temp & humidity (by populating an appropriate and supported IC)

What *doesn't work* (not supported in ESPeasy):
- LIS2DH12 accelerometer 
- Encryption chip

Work in progress:
- Controlling the display backlight from ESPeasy as opposed to using the provided switch/accelerometer configuration

## ESPeasy

The ESP Easy firmware can be used to turn the ESP module into an easy multifunction sensor device for Home Automation solutions like Home Assistant, OpenHAB, Domoticz or other software that speaks MQTT. Configuration of the ESP Easy is entirely web based, so once you've got the firmware loaded, you don't need any other tool besides a common web browser. For more info, see https://www.letscontrolit.com/wiki/index.php/ESPEasy

Here's the step for getting ESPeasy installed and configured on the Fri3dcamp 2020 badge.

## Getting ESPeasy

Download the latest release from Github: https://github.com/letscontrolit/ESPEasy/releases

Extract the ZIP file into it's own folder. Open that folder.

## Flashing the firmware

Refer to the ESPeasy documentation: https://www.letscontrolit.com/wiki/index.php/ESPEasy#Loading_firmware

Open the Espressif flash download tool from the folder **\Espressif_flash_download_tool_v3.8.5\flash_download_tool_3.8.5.exe**

Select `Developer Mode`

Select `ESP32 DownloadTool`


In the ESP32 Download tool, select the top row and click the ... to select a firmware file.

Browse to the `bin` folder and select the firmware file `ESP_Easy_mega_20220809_**display**_ESP32_4M316k.factory.bin`

Put a 0 in the load address (@ 0)

Set SPI speed to 40MHz

Set SPI mode to DIO

Set Flash Size to 16Mbit

Select your board's comm port

Click the `Start` button


Your firmware should now be flashed to your badge. Look in the terminal window associated with the ESP flash tool for any errors.

If flashing doesn"t work, hold the BOOT button on the badge, then press the RESET button, then release the BOOT button, then try to flash again. If you open a serial terminal to the com port of the badge, it should say "waiting for download" after you press the buttons in the correct sequence.

**Once the flash it complete, open a serial terminal to the com port of your badge and check the communications. You should see the ESP32 on the badge boot up and set up a wifi AP.**

## Connecting ESPeasy to your wifi network

If everything has gone right, you have a useable ESP Easy device now. As no parameters are set it will go to "AP mode" for configuration.

Use your computer, tablet or smartphone and search a WiFi network named `ESP_Easy_0`.

Connect to` ESP_Easy_0` using the password `configesp`

Open your internet browser and type `192.168.4.1` as internet address into the browser. The WiFi setup of the ESP Easy opens.

You can configure your own local WiFi network now. Select the SSID and enter your passphrase, click connect.

It will take 20 seconds until a result is shown. If you typed everything correctly it will show a message that it is connected to the network and it shows the IP address.

**Note the IP address!**

Connect your computer or whatever back to your usual network. Open a browser and type the ip address that you noted into the browser.

You should see the config pages of ESPEasy now. 

Refer to this tutorial: https://www.letscontrolit.com/wiki/index.php/Basics:_Connecting_and_flashing_the_ESP8266

## ESPeasy setup

Once ESPeasy is properly connected to your wifi you can configure it.


### Tab "Main Settings":

Set a unit Name (e.g. Fri3d-badge)

Set a Unit Number (e.g. 1)

Click `Submit`


### Tab "Hardware":

Wifi Status LED: `GPIO-25`

Reset Pin: `none`

I²C interface: `GPIO-SDA: GPIO-21`, `GPIO-SCL: GPIO-22` (untested but should work)

SPI interface: Init SPI: `VSPI: CLK=GPIO-18, MISO=GPIO-19, MOSI=GPIO-23`

Click `Submit`


### Tab "Devices:

**TFT Display**

Add a new device

Select `Display - ST77xx TFT`

Give it a name (e.g. Display)

Check enable

Set `GPIO -> CS` to `GPIO-5`

Set `GPIO -> DC` to `GPIO-33`

Set `GPIO -> RES` to `None`

Set `GPIO -> Backlight` to `GPIO-12` (not sure about this one)

Set `TFT display model` to `ST7789 240 x 240px`

Set `Rotation` to `+180°`

Set `Font Scaling` to 3 (or something else if you desire)

Set `Foreground colour` to white

Set `Background colour` to green

Put some text in Line 1 just as a test. If the display works, this text will appear on the display after submit or reboot.

Click `Submit`


Make sure the backlight of the TFT is on (little switch on the right hand side of the badge).

The display won't work unless SPI is configured, see Hardware tab above.


**NeoPixels**

Add a new device

Select `Output - NeoPixel (Basic)`

Check enable

Set `GPIO -> DIN` to `GPIO-2`

Set `Led Count` to `5`

Set `Strip Type` to `GRB`

Click `Submit`


**Battery voltage monitoring**

Add a new device

Select `Analog input - internal`

Check enable

Name it Battery

Set `Analog Pin` to `ADC1 ch7 / GPIO-35`

Set `Oversampling` to `Oversampling`

Check `Apply Factory Calibration`

Apply the formula `%value%*2/1000` to the Formula field in Values (this formula needs to be tuned and calibrated to match the torelances of the 100k/100k resistor divider on the board)

Click `Submit`

**BOOT switch**

Add a new device

Select `Switch input - Switch`

Check Enable

Set `GPIO` to `GPIO-0` (this selects the BOOT switch on the badge)

Set `Switch Type` to `Switch`

Set `Switch Button Type` to `Normal Switch`

Click `Submit`

### Tab "Tools":

Click the `Advanced button`

You can set NTP to `be.pool.ntp.org` here to get time through the internet

Enable `Rules`

Click `Submit`

### Tab "Rules"

(Rules needs to be enabled in Tools - Advanced to be able to see this tab)

Add the following rules:

    on Battery#Analog do
    logentry,Battery triggered, %eventvalue%
    st7789cmd,clear
    st7789,txp,1,2
    st7789,txt,VBat:%eventvalue%V
    endon

Click `Save`

Check the serial terminal for feedback. You should see the "on Battery#Analog do" get triggered (EVENT: Battery#Analog=4.23), and the st7789 display commands being sent. If all is well, you should see the voltage that is on GPIO-35 (converted into battery voltage) on the display.

## Odds and ends

The Buzzer on the badge is on pin `GPIO32` and can be controlled with the command `PWM,<GPIO>,<duty>,<duration>,<frequency>` (e.g. `PWM,32,100,100,3000`). It does not require a device to be set up.
    
The IR receiver is on pin `GPIO25 `but requires a different ESPeasy firmware (custom_IR?) to be flashed before it can be used.

 
Have fun, happy hacking!

    Raketman
  

Origin of this document: https://github.com/waaslandwolf/Badge2020_ESPeasy/blob/main/README.md
