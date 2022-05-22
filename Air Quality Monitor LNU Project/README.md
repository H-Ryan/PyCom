# Building an indoor Air Quality Monitor 


###### Authors: `Elias Lindfors (el224tn) & Hamed Talebi (ht222jp)`
###### tags: `IoT` `LoRa` `MQTT` `SSL` `TIG Stack`

This tutorial presents the steps required to make your indoor air quality monitor. It shows how to gather sensor information like; temperature, humidity, and CO2 from your environment and store them on the cloud, and finally using visualization analytics web application to monitor updates and changes.

With the basic understanding of microcontroller, python programming, Linux administration, and superficial knowledge about[ **LoRa**](https://en.wikipedia.org/wiki/LoRa) communication protocol you should be able to finish this project in approximately ten to fifteen hours.


## Objective
According to [**WHO**](https://www.who.int/health-topics/air-pollution) air pollution kills about seven million people worldwide every year. One of the places we spend most of our time each day is our bedroom while we are sleeping. Therefore, indoor air quality is significant when it comes to sleep and, in the long term, has a direct impact on cardiovascular diseases.

Finding a proper pattern for our ventilation system helps us to breathe healthier air, and to find that pattern we need to collect surroundings data. [**Volatile Organic Compounds (VOC)**](https://en.wikipedia.org/wiki/Volatile_organic_compound) are organic chemicals that have a high vapor pressure at room temperature which has a direct effect on the existence of [**Equivalent Carbon Dioxide (eCO2)**](https://theclimatecenter.org/resources/glossary/) in an indoor environment and both have a negative impact on sleep quality.

In this project, we plan to consider the effect of temperature and humidity levels (both indoor and outdoor) on VOCs and eCO2. We’re particularly considering if a drastic change in outdoor temperature and humidity has a direct impact on VOC emission.


## Material
The following materials used in this project:


| Hardware                  | Price and link          |
| ------------------------- |:----------------------- |
| ![](https://alepycom.gitbooks.io/pycom-docs/content/img/lopy4.png =130x)LoPy4 Basic bundle (Two sets for LoRa Gateway and Node)        | [849 SEK](https://www.electrokit.com/produkt/lnu-1dt305-tillampad-iot-lopy4-basic-bundle/ "ElectroKit")   |
| ![](https://www.mysensors.org/uploads/57c3ec0c4d04abe84cd93e0f/image/dht11.png =100x)DHT11 moisture sensor     | [49 SEK](https://www.electrokit.com/produkt/digital-temperatur-och-fuktsensor-dht11/ "ElectroKit")     |
| ![](https://cdn.shopify.com/s/files/1/0410/7888/2465/products/eff1b1c386c3409c8cc2d4028f4de147674a39b49867770537045b44d6e1ba2d.png?v=1600968764 =100x)DHT22 moisture sensor     | [99 SEK](https://www.electrokit.com/produkt/temp-fuktsensor-rht03/ "ElectroKit")     |
| ![](https://www.joy-it.net/files/files/Produkte/SEN-CCS811V1/SEN-CCS811V1-01g.png =100x)CCS811 Air Quality sensor | [140 SEK](https://www.amazon.se/KEYESTUDIO-luftkvalitetssensor-kolmonoxid-luftkvalitet-gasgivare/dp/B08GP947QS/ref=sr_1_1?dchild=1&keywords=ccs811&qid=1626980467&sr=8-1 "Amazon SE")     |
| Cloud Server Renting (Ubuntu 18 installed on Digital Ocean)| [112 SEK/month](https://digitalocean.com "DigitalOcean") |
| Cloud Server Renting (CentOS 7 installed on Hetzner)| [40 SEK/month](https://www.hetzner.com/cloud "Hetzner") |

* The two LoPy4 act as node and gateway respectively.
* Both the DHT11 and DHT22 are Air Moisture and Temperature Sensors.
* The CCS811 sensor detects eCO2 Volatile Organic Compounds in the air.
* The Gateway-device will connect over MQTT to one of the two Server Hosts. (only one server host is needed)


## Computer Setup
### Flashing LoPy4 Firmware
It is recommended by PyCom to update and use the latest firmware as it may have some bug patches for your new board. You can follow the detailed PyCom [**documentation (here)**](https://docs.pycom.io/updatefirmware/device/) to download the firmware application and upgrade board in your preferred operating system.
 
### Installing IDE and Uploading Code  
We used Visual Studio Code with Pymakr to easily connect the Lopy4 device over USB to our Windows operating system.

**We need to follow these steps to set up a development environment for our LoPy4:**

* Download [**VSCode**](https://code.visualstudio.com/download), as well as [**NodeJS**](https://nodejs.org/en/).

* Connect the Pycom device to its Expansion Board by having the rgbLED towards the USB connector, align the pins, and press firmly. Connect the included micro-USB cable to the Expansion Board and Computer.
* In VSCode, install the Pymakr extension by going to the Extensions tab and searching for ‘Pymakr’
![](https://i.imgur.com/mJEvvuw.png)

* Once installed you should have these extra options on the blue bottom bar, as well as a new REPL terminal. From here you can upload/download your project files, and run the currently open file.
![](https://i.imgur.com/ScBbohM.png)

* After we finish coding we can upload the code to the board from here, or the top right corner. ![](https://i.imgur.com/2SljOpq.jpg) 

* **Note:** You can see all commands by clicking “All commands” at the bottom, or pressing ctrl+shift+P and entering `Pymakr`
![](https://i.imgur.com/XLpADvq.png)
 
<br/>

**An alternative route** to uploading files is to transfer them over FTP by making sure your Pycom-device is connected to the local Wifi. Run the following commands over on the REPL-terminal:
```python=1
Import machine
Import pycom
From network import WLAN
pycom.wifi_mode_on_boot(WLAN.STA)
pycom.wifi_on_boot(True)
pycom.wifi_ssid_sta(‘<wifi-ssid>’)
pycom.wifi_pwd_sta(‘<wifi-password>’)
machine.reset()
```
Then connect to it with an FTP client application like [**WinSCP**](https://winscp.net/eng/download.php) and copy files to your “flash” directory.

## Putting everything together
The hardware has been connected together on a Breadboard with jumper-wire like this:
![](https://i.imgur.com/ORLQyE1.png)

<br/>

The system domain diagram of different parts looks like this:
![](https://i.imgur.com/oY8UTwY.png)

### Electrical calculations
Three AAA batteries have around 1000mWh 1,2V and gives a total of 3000mWh 3,6V. This gives 833mAh of current, though we might be undershooting these values by a bit.

[**The Lopy4 specsheet**](https://cdn.sparkfun.com/assets/e/b/2/9/0/lopy4-specsheet.pdf) approximates the idle current consumption at 35,4mA.
Transmitting over LoRa draws 108mA, while WiFi draws 99mA.
**NOTE:** The transmission time is difficult to approximate as it should only last for a second per reading, so we’ll bump up the consumption during use to 50mA.

* The CCS811 sensor draws ~29mA when in use.
* The DHT11 draws at most 2.5mA when reading, with an idle draw at 150uA.
* The DHT22 draws 1.5mA when reading, and 50uA idle.

The lopy will read once every 5 minutes, with the reading taking about 15 seconds.
This means during a typical hour the device is powered on for 450 seconds and sleeps 3150 seconds.
((sensors) + device) * total on-time = hourly power needed to use sensors.
`((29 + 2.5 + 1.5)mA + 50mA) * 450 seconds = 83mA * 0.125h = 10,375mAh`

We add the current still drawn while the device is not measuring, to get the total power-draw.
`35.4mA * 3150 seconds = 35.4mA * 0.875h =  30.975mAh`
Totaling to 41.35mAh. With the 833mAh battery-pack the device can be plugged in for a total of **20 hours**.

The device draws much less current when instead put into a Deepsleep. 18.5uA compared to 35.4mA.
`0.0185mA * 3150 seconds = 0.0185mA * 0.875h = ~0.02mAh`
Totaling to 10.395mAh. The device can thereby be plugged in for a total of **80 hours**.

Increasing the time between readings will lessen the power consumption, though it comes with the disadvantage of fewer data-points. With readings every 30 minutes, the device is powered on for 30 seconds per hour and deepsleeps for 3570 seconds.
`((29 + 2.5 + 1.5)mA + 50mA) * 30 seconds = 83mA * 0.008333...h =  0.69mAh`
Adding the current drawn during Deepsleep gives a total of 0.71mAh. Giving a total of **1173 hours** of use-time (48 days).

We have to do 10 consecutive readings, taking one second each, to get a good estimate with the CCS811 sensor. If we had a better CO2 sensor we could lower it to just a few seconds, saving even more battery time.


## Platform
We first tested [**Adafruit**](https://io.adafruit.com/) and [**Ubidots**](https://ubidots.com/) to see how other platforms handle device integration and Dashboards. It gave us some valuable ideas about what each platform has to offer. For example, Adafruit uses MQTT as the communication protocol and utilizes different time-based and value-based alarm triggering. Ubidots uses both MQTT and REST API for uploading data, and more integration options to connect it to different platforms like [**The Things Stack (TTS)**](https://www.thethingsindustries.com/).
 
We finally chose [**TIG Stack**](#Presenting-the-data) to have full control of our data and not need a platform subscription to utilize its full extent. Besides, we can keep our data as long as we need it to have better data analysis in the future, and also for scaling up we do not have any restrictions.

## The Code
As it showed in [**Putting everything together**](#Putting-everything-together), we used two sets of **LoPy4**. One acts as a node and the other one as a gateway. We have uploaded the complete code for both gateway and node to our GitHub [**here**](https://github.com/Ryan-HT/air-quality-lnu-project). All code files are commented thoroughly to be self-explainable and we present the code functionality in the following two sections briefly.
<br/>
### Node
An air quality sensor (**CCS811**) and two temperature and humidity sensors (**DHT11** & **DHT22**) are connected to the LoPy4 to form a node. The node is responsible to read the value of the sensors and send it to the gateway. Our nodes try to read sensors’ value ten times and get the reading average every five minutes (for testing, better to read every 15-20 minutes) then use low power **LoRa** communication protocol to send it to the gateway. It tries to send the values three times, and each time listens for gateway acknowledgment. If it receives the “ack” package it goes to sleep for five minutes, otherwise it goes to sleep for two minutes and tries to send data again. We used two libraries for our sensors [“CCS811.py”](https://github.com/Ryan-HT/air-quality-lnu-project/blob/main/Node_Files/lib/CCS811.py) and [“dht.py”](https://github.com/Ryan-HT/air-quality-lnu-project/blob/main/Node_Files/lib/dht.py) and give the link to the source in our code.
Our node use `struct` to pack sensors data to send over LoRa in the following format:
```python=-
# deviceId+lastMessageRandomId+indoorTemperature+indoorHumidity+outdoorTemperature+outdoorHumidity+eCO2+eTVOC
# lastMessageRandomId control message duplication at gateway side
_LORA_PKG_FORMAT = "!BBffffff"
struct.pack(_LORA_PKG_FORMAT, _DEVICE_ID, msg_id, indoorTemp, indoorHumid, outdoorTemp, outdoorHumid, eco2, etvoc)
```
The file structure for our node is the following: 
```
The_Node_Files
|-lib
|  |-CCS811.py
|  |-dht.py
|-main.py		
```
<br/>

### Gateway
The second LoPy4 acts as a gateway. It is always on and listens to incoming messages over LoRa. When it receives the message; it checks if the packet size is according to what it expects then tries to unpack the message and if it could do it successfully (which means the format and size would match) then it checks if the message is not a duplication. If it gets a duplicate of the same message then it ignores it, otherwise use the **MQTT** protocol to send it to our MQTT server.

We use different techniques to make the gateway robust, like; a watchdog timer to reset the board if it is stuck or the internet connection to the board disconnects. It also uses garbage collection in form of memory management to handle overflow when clients send lots of requests. The file structure for our gateway is as below:
```
The_Gateway_Files
|-lib
|  |-mqtt.py
|-boot.py		
|-main.py		
|-config.py
```
<br/>

## Transmitting the data / connectivity
Transmission of data illustrated in [**Putting everything together**](#Putting-everything-together) and is done with LoRa communication protocol from the sensors to a gateway, and MQTT message formatting protocol from the gateway to the server. All formatting is documented in the GitHub and LoRa message structure between sensors and gateway shown under [**The Code**](#The-Code).

LoRa is a long-range low power network modulation technology that is often used in the IoT space. We decided to use it as it will save us battery power on the sensor node and allows us to connect multiple sensors very easily using just one Gateway device. The sensor sends data to the gateway once every five minutes (It is for gathering presentation data and could be raised to every twenty minutes) and sleeps in-between to save power.

MQTT, Message Queuing Telemetry Transport, is an IoT protocol often used to send small amounts of data very efficiently, but due to needing an Internet connection, it’s more power-hungry than LoRa. We thereby use it for the gateway device only, which will get power through USB instead of a battery.

The Server runs Eclipse Mosquitto to receive the data sent over MQTT. Mosquitto acts as an MQTT-broker and allows devices and services to either publish data or subscribe to topics.  The transmission over MQTT is encrypted through SSL with the help of Client and Server certificates.



### Generating all the certificates
To generate working certificates and keys you need to sign them, either yourself or by a Certificate Authority. We will create our own Certificate Authority that will sign the Server and Client certificates and keys. Thereby only we can create new Certificates.

Make sure to install the tools needed with `sudo apt-get install gnutls-bin` (**On Ubuntu**) or `sudo yum install gnutls-utils` (**On CentOS**)
Now we generate our Certificate Authority with these two commands:
:::warning
Ubuntu and CentOS
```shell=1
sudo certtool --generate-privkey --outfile CA-key.pem
sudo chmod 400 CA-key.pem
sudo certtool --generate-self-signed --load-privkey CA-key.pem --outfile CA.pem
```
:::

You will need to fill out a form after running the second command. Some of the options can be left blank, but it is important you choose a Common name and an expiration date far into the future, and select YES on these three options:
- Does the certificate belong to an authority
- Will the certificate be used to sign other certificates
- Will the certificate be used to sign CRLs

#### Generating a server certificate signed by out new authority
:::warning
Ubuntu and CentOS
```shell=4
sudo certtool --generate-privkey --outfile server-key.pem --bits 2048
sudo certtool --generate-request --load-privkey server-key.pem --outfile server-request.pem
sudo certtool --generate-certificate --load-request server-request.pem --outfile server-cert.pem --load-ca-certificate CA.pem --load-ca-privkey CA-key.pem
```
:::

Here it is important you pick a date far into the future, the correct dnsName, and select YES for
- Is this a TLS web client certificate
- Is this a TLS web server certificate

The dnsName is the same as the hostname for the Application it will be used for. I.E. if you're going to use it for `mqtt.example.com`, you enter `mqtt.example.com` and not `example.com`. If the dnsName is not correct you'll need to generate a new certificate.

#### Generating a Client Certificate signed by our new authority

:::warning
Ubuntu and CentOS
```shell=7
sudo certtool --generate-privkey --outfile client-key.pem --bits 2048
sudo certtool --generate-request --load-privkey client-key.pem --outfile client-request.pem
sudo certtool --generate-certificate --load-request client-request.pem --outfile client-cert.pem --load-ca-certificate CA.pem --load-ca-privkey CA-key.pem
```
:::

Here you also enter an expiration date far into the future and answer YES to these two questions
- Is this a TLS web client certificate
- Is this a TLS web server certificate

The dnsName is not as important here.

#### Where the certificates go
The Server Applications need `CA.pem`, `server-cert.pem` and `server-key.pem`.
The Client Applications need `CA.pem`, `client-cert.pem` and `client-key.pem`.
The `CA.key`, `server-key.pem` and `client-key.pem` are private-keys and need to be kept secret.

### Installing the MQTT Broker
A reference-guide can be found [**here (Ubuntu)**](https://www.digitalocean.com/community/tutorials/how-to-install-and-secure-the-mosquitto-mqtt-messaging-broker-on-ubuntu-18-04) and [**here (CentOS)**](https://www.digitalocean.com/community/tutorials/how-to-install-and-secure-the-mosquitto-mqtt-messaging-broker-on-centos-7). Here's the steps as well:

#### Installing Mosquitto
:::info
Ubuntu
```shell=1
sudo apt install mosquitto mosquitto-clients
```
:::
:::success
CentOS
```shell=1
sudo yum -y install epel-release
sudo yum -y install mosquitto
```
:::

(optional) We generate a usename and enter the password after with
:::info
Ubuntu
```shell=2
sudo mosquitto_passwd -c /etc/mosquitto/passwd <username>
```
:::
:::success
CentOS
```shell=3
sudo mosquitto_passwd  -c /etc/mosquitto/passwd admin
sudo mosquitto_passwd  /etc/mosquitto/passwd <username>  #pass: <password>
```
:::

Then we edit the `default.conf` file with
:::info
Ubuntu
```shell=3
sudo nano /etc/mosquitto/conf.d/default.conf
```
:::
:::success
CentOS
```shell=5
sudo nano /etc/mosquitto/mosquitto.conf
```
:::
and enter the following to end of config file:
```
allow_anonymous false
password_file /etc/mosquitto/passwd
listener 1883 localhost

listener 8883
cafile /etc/mosquitto/cert/CA.pem
certfile /etc/mosquitto/cert/server-cert.pem
keyfile /etc/mosquitto/cert/server-key.pem
require_certificate true
```
Once saved, we restart Mosquitto for the changes to take effect
:::info
Ubuntu
```shell=4
sudo systemctl restart mosquitto
```
:::
:::success
CentOS
```shell=6
sudo service mosquitto start
sudo systemctl enable mosquitto
```
:::


## Presenting the data
We utilize TIG Stack to collect, store, and visualize the data. TIG is short for Telegraf, InfluxDB, and Grafana. Telegraf is used to capture the MQTT data, InfluxDB stores the data, and Grafana visualizes the data. The data comes in every five minutes (for testing and better to be every twenty minutes to save energy on the sensor node) so our Dashboard and queries update accordingly.

![](https://myvmworld.fr/wp-content/uploads/2017/12/telegraf.png =100x)
Telegraf can read from one or multiple inputs, in our case MQTT, and send the results back to outputs, like InfluxDB. It is a great “middleman” application for gathering data and sending it to where it needs to go in case you have multiple input sources and output destinations.

![](https://dbdb.io/media/logos/InfluxDB.png =100x)
InfluxDB is more or less an SQL database but with Time-Series data in mind, where it always saves the current time when inserting data. We chose InfluxDB as it’s quick and easy to set up and fits IoT purposes very well, especially in a Docker container.

![](https://stitch-microverse.s3.amazonaws.com/uploads/domains/grafana-logo.png =100x)
Grafana is the visualization part of the TIG Stack. It has multiple plugins for visualizing your data the way you want it. For example Time Series, charts, tables, and graphs. Triggers/Alerts can be set up where if a goal is met then something triggers. It can be something like alerting on the Dashboard when an error is sent from the Pycom-device, or if a value from one of the sensors passes a set threshold then a Webhook message is sent.
### Installing TIG stack on Ubuntu or CentOS
Install Docker with [these instructions for Ubuntu](https://docs.docker.com/engine/install/ubuntu/) or [these instructions for CentOS](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-centos-7).
Install Docker-compose with [these instructions for Ubuntu](https://docs.docker.com/compose/install/ ) or [these instructions for CentOS](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-compose-on-centos-7.amp).
Download the Docker_Files folder through our [Github repository](https://github.com/Ryan-HT/air-quality-lnu-project), and configure the two files inside to fit your setup.

Once done, run `docker-compose up` inside the Docker_Files folder and it will set up automatically. Thereafter you can follow [**this tutorial**](https://hackmd.io/@lnu-iot/tig-stack) to connect Grafana to your InfluxDB and create a Dashboard and adding panels to your dashboard. 
 
### Automation and Data Trigger in Grafana
After you create your dashboard and panels you can set an alarm to trigger on some of your panels like Time-Series bar charts. We did not use any trigger as this project is more about gathering data but you can do the following steps to set up a data trigger in your Grafana and send an alarm message to your **Discord’s** channel:

* To create a channel in Discord and active Webhook, you should follow section [**”Creating a server and activating webhook on a channel in Discord”**](https://hackmd.io/@lnu-iot/Hys6ha6Tu) from this tutorial.
* Then, open Grafana, and from the left panel click on the bell sign then choose "Notification channel" and click on "Add channel".
![](https://i.imgur.com/debFNRR.jpg) 

* Choose a name, select Discord, and copy your Webhook link. You can test by clicking test and if you received a test message push the “save” button.
![](https://i.imgur.com/AyokBFb.jpg)  

* Open your desired panel like our humidity time series bar chart monitor in the just under the chart click on “Alert” then “Create Alert”.
![](https://i.imgur.com/JkAfZFS.jpg) 

* In the opened panel choose the trigger threshold then choose the webhook alarm from “send to”, write a message, and save both panel and Dashboard.
![](https://i.imgur.com/ZN7k0oz.jpg) 

* When it triggers the alarm it will send a formatted message to your Discord channel.
![](https://i.imgur.com/6aI7GkE.jpg)

<br/>

## Final Design
Here are images of our finalized hardware:
The Gateway		|	The Gateway
:-------------------------:	|	:-------------------------:
![](https://i.imgur.com/LG3CqSl.jpg)	|	![](https://i.imgur.com/xmGV4Oq.jpg)
 
Sensor Node		|	Sensor Node
:-------------------------:	|	:-------------------------:
![](https://i.imgur.com/gDfWNy1.jpg)  	|	![](https://i.imgur.com/vZVBwTv.jpg) 
 
Outdoor Sensor		|	Outdoor Sensor
:-------------------------:	|	:-------------------------:
![](https://i.imgur.com/pMdQW7C.jpg)  	|	![](https://i.imgur.com/KBgwIrm.jpg)

Sensor Node		|	Sensor Node
:-------------------------:	|	:-------------------------:
![](https://i.imgur.com/lcEXFck.png =1000x) | ![](https://i.imgur.com/SDASRxf.png =1000x)
 
And some images of Dashboard examples:
![](https://i.imgur.com/HWnLag6.png)

![](https://i.imgur.com/Iwle1Vl.jpg)

<br/>

## Lesson Learned & Further Optimization

We reached most of our initial goals in this project. We have managed to make a robust node and gateway which did not crash in ten days’ run. We successfully installed the whole environment on our private servers and encrypted all communication with SSL. Although we still need to gather more data and run a suitable machine-learning algorithm, to find out if our hypothesis about the change in outdoor humidity and temperature on indoor air quality is correct. The following two sections are our proposal for further optimization: 

###  Sensors
* **DHT11** is not a good choice for outdoor humidity and temperature because it is not as accurate and stable as DHT22 even for indoor reading and we should find a more suitable sensor. 
* **CCS811** is the same with accuracy and reliability and it only approximates the CO2 levels based on VOC, temperature, and moisture readings.
* **Using More Sensors** would give us more data, like if we know the habit of opening/closing ventilation system or bedroom window, which make some difference on sensors’ data. We can also use some air particle (dust) sensor or the ambient light which affects sleep quality. 
 
### Analyzing Data
The next step would be using a suitable data analyzer like some machine learning algorithm to help us conclude the reasoning of the phenomenon.

<style>

.markdown-body code{
    font-size: 1em !important;
}
.markdown-body .a{
    font-size: 5em !important;
}
.markdown-body pre {
    background-color: #333;
    border: 1px solid #333 !important;
    color: #dfdfdf;
    font-weight: 600;
}
.token.operator, .token.entity,
.token.url, .language-css .token.string,
.style .token.string {
    background: #000;
}
.markdown-body table {
    display: table;
    padding: 1em;
    width: 100%;
}
.markdown-body table th,
.markdown-body table td,
.markdown-body table tr {
    border: none !important;
}
.markdown-body table tr {
    background-color: transparent !important;
    border-bottom: 1px solid rgba(0, 0, 0, 0.2) !important;
}
</style>
