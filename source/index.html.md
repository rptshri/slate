---
title: Infi-IoT Hardware Documentation

language_tabs: # must be one of https://git.io/vQNgJ
  - shell: cURL
  - cpp: ESP  
  - python: Python

toc_footers:
  - Copyright &copy<a href='https://infiiot.com/'> Infi-IoT</a> 2020. 
  - All Rights Reserved.

includes:
  - errors

search: true
---

# Introduction

Welcome to the Infi-IoT hardware documentation. This documentation will help you learn how to integrate your hardware devices using our supported communications protocols ie. HTTPS, and MQTT.

We use REST API for better performance and scalability. The purpose of this section is to help you understand what happens in the backstage when communicating with Infi-IoT, so you can replicate this in your firmware. 

Please note that in all the example codes below will not use our libraries, so that you understand what happens in the backend and it is easier for you to replicate on your side. 

## Methods to send/upload data to Infi-IoT Platform
**Prerequisites:**
* If you don't have an Infi-Iot subscription, create one for free before you begin.
* A device that meets the following requirements:
    * Your device must be able to communicate by using HTTP or MQTT protocols.
    * The device messages must conform to the Infi-IoT Platform message payload requirements.

<aside class="notice">For more Information on Infi-IoT Platform payload requirements see csv requirements!</aside>

[.csv requirements here.](http://infiiot.com/csv-format)

Infi-IoT enables you to ingest high volumes of telemetry
from your IoT devices into the cloud for storage or analysis. Following are the
ways you can add data to the cloud:

* Using Our Libraries or APIs:
    1. If you have a device able to communicate by using HTTP or MQTT protocols, you can use our libraries which are designed for different hardwares and tools.
    2. You can use our strong API service and perform all tasks by including our API into your apps or programs

* Using .csv file:
    If you have been collecting and storing data for a certain time and wish to analyze your data using our services then you can import logged data onto the Infi-IoT Platform. Using .csv file in the format specified by us to import data to your account using the import data section. You can Import data for a single variable as well as for the whole device.

## How Infi-IoT Works? 

Every time a device updates a sensor value in a variable, firstly user is authenticated using Auth-Token Key, and if the user is recognized a data-point is created. Infi-IoT stores data that come from your devices inside variables and these data points have corresponding timestamps.

Each dot contains Value and timestamp.

### Values

A numerical value. Infi-IoT accepts up to 16 floating-point length numbers.

    {"value" : 34.87654974}


<aside class="notice">Super Tip: It is mandatory to insert sensor data into your data payload.</aside>

### Timestamps

 A timestamp is a digital record of the time of occurrence of a particular event. 
 We use ISO 8859 format for timestamps

    {"timestamp" : 2020-05-07T09:25:24+0000}

The above timestamp corresponds to Thursday, September 20, 2018, at 2:30:24 PM.

<aside class="success">Super Tip: If not specified, then our servers will assign timestamps upon data payload reception.</aside>

### Data Illustration:

## Device authentication

Infi-IoT uses Auth (Authentication) Token key to authorize your device to ingest data inside your Infi-IoT account.

Authentication token keys are UUID 4 format keys that are generated when you register your Infi-IoT account. You can find your Auth token Key

<aside class="warning">Super Tip: Keep the authentication Token keys physically safe. This will avoid a malicious device masquerade as a registered device.</aside>

## Protocols

Infi-IoT supports two protocols for device connection and communication: MQTT and HTTP. Devices communicate with Infi-IoT Platform across a "bridge" — either the MQTT bridge or the HTTP bridge.

* MQTT is a standard publish/subscribe protocol that is frequently used and supported by embedded devices, and is also common in machine-to-machine interactions.

* HTTP is a "connectionless" protocol: with the HTTP bridge, devices do not maintain a connection to the Infi-IoT Platform. Instead, they send requests and receive responses. Infi-IoT Core supports HTTP 1.1 only (not 2.0).

The following table compares how the two protocols work on Infi-IoT Platform:

|MQTT|HTTP|
|---|---|
|MQTT bridge   |HTTP bridge   |
|Device connection is maintained   |Connectionless (request/response)   |
|Full-duplex TCP connection   |Half-duplex TCP connection   |
|A token is sent in the password field of the CONNECT message   |A token is sent in the Authorization header of the HTTP request   |
|Telemetry events are pushed to Cloud Pub/Sub   |Telemetry events are pushed to Pub/Sub   |
|The device connection status is reported   |No device connection status reported   |
|Device configurations are propagated via subscriptions   |Device configurations must be explicitly requested (via polling)   |
|Most recent configuration (whether newer or not) is always received by devices on a subscription   |Devices can specify that only newer configurations should be received   |
|Device configurations are acknowledged (ACKed) when using QoS 1   |No explicit ACK for device configurations   |
|Last device heartbeat time is retained   |No device heartbeat data  |

You might also want to consider the following general features of each protocol:

* MQTT
    *   Lower bandwidth usage
    *   Lower latency
    *   Higher throughput
    *   Supports raw binary data

* HTTP
    *   Lighter weight (easy to get started; simple curl commands)
    *   Fewer firewall issues
    *   Binary data must be base64-encoded, which requires more network and CPU resources

<aside class="notice">Super Tip: If you're not sure which protocol is best for your use cases, start with HTTP to get familiar with Cloud IoT Core, and then switch to MQTT if needed.</aside>

# HTTP

As seen earlier, HTTP is Connectionless (request/response) and Half-duplex TCP connection-based communication protocol

This section explains how devices can use the HTTP bridge to communicate with the Infi-IoT Platform. For general information about HTTP and MQTT, see [Protocols](##Protocols)

Be sure to refer to the [API documentation- Software](https://www.infiiot.com/documentation-software) for full details about each method described in this section.

HTTP is a request/response protocol, this means that every request that you make is answered by the server. This response includes a number (response code) and a body. For example, when you make a request to retrieve information from a webpage "(e.g. google.com)", you build a GET request ( `GET http://www.google.com` ). If the request is correct, the server will typically return a 200 response code, along with the information to render a webpage for you.

Likewise, when your device sends a request to Infi-Iot requesting last value for a variable, the request is processed and Infi-IoT sends back a response that contains a status code and a body section containing the data (value and timestamp) related to your device variable.

## HTTP Methods
You can think of this as the verb that tells the server what action to perform on a resource. The two most common HTTP request methods you'll see are GET and POST. When you think about retrieving information, think GET, and when you think about putting the information to a server, think POST.

We prefer POST methods for all tasks for security reasons because even when you want to retrieve the last value of your sensor data you will need to send your Token Key via URL(if GET Method is used) which is quite insecure. So keep in mind even when you want to retrieve data you use the POST method so that your Token is transferred securely to our servers.

## HTTP Request
An HTTP request also needs the parameters below:

* **Host:** Specifies the server you will be making HTTP requests to. For example, `www.infiiot.com`

* **Path:** This is typically the remaining portion of the URL that specifies the resource you want to consume, be it a variable or a device. For example, 
if an API endpoint is: https://infiiot.com/apis/v1.0/get-Devices then the path would be `/apis/v1.0/get-Devices`

    For more information about the URL refer [here](https://launchschool.com/books/http/read/what_is_a_url#introduction).

* **Headers:** HTTP headers allow the client and the server to send additional information during the request/response HTTP cycle. Headers are colon-separated name-value pairs that are sent in plain text. 

  Headers define the operating parameters of the HTTP request such as authentication, User-Agent, Content-Type, Content-Length, etc.  

* **Body/payload:** In the case of POST and PATCH requests, this is the data sent by your device to the server. GET requests typically do not have a body because they are meant to request data, not to send data. But as we prefer POST we will always have Body section.

  Bodies can be broadly divided into two categories:
  Single-resource bodies, and Multiple-resource bodies. We require Single-resource bodies consisting of one single file, defined by the two headers: Content-Type and Content-Length.

  For more information about body refer [here](https://developer.mozilla.org/en-US/docs/Web/HTTP/Messages).

We use JSON format (JavaScript Object Notation) Content-type. JSON is a typical HTTP data type, it is a collection of name/value pairs. In various programming languages, this is treated as an object, record, struct, dictionary, hash table, keyed list, or associative array. It is also human-readable and language independent. An example of a JSON data type that InfiIoT accepts can be referenced below:

    {"variable": "temperature", "data": {"value":10, "timestamp":1534881387000}}

A typical HTTP request to Infi-IoT should be set as below:

    POST {PATH} HTTP/1.1<CR><LN>
    Host: {HOST}<CR><LN>
    User-Agent: {USER_AGENT}<CR><LN>
    X-Auth-Token: {TOKEN}<CR><LN>
    Content-Type: application/json<CR><LN>
    Content-Length: {PAYLOAD_LENGTH}<CR><LN><CR><LN>
    {PAYLOAD}
    <CR><LN>

Where:

*	**{PATH}:** Path to the resource to consume. For example, `/apis/v1.0/get-Variable` to get a variable information.
* **{HOST}:** Host URL. Example: `infiiot.com`
*	**{USER_AGENT}:** An optional string used to identify the type of client, be it by application type, operating system, software vendor, or software version of the requesting user agent. Examples: `ESP8266/1.0, Particle/1.2`
*	**{TOKEN}:** Unique key that authorizes your device to ingest data inside your Infi-IoT account.
*	**{PAYLOAD_LENGTH}:** The number of characters in your payload. Example: The payload `{"variable": "temperature", "value": 20}` will have a content-length of 38.
*	**{PAYLOAD}:** Data to send. Example: `{"variable": "temperature", "value": 20}`

# Infi-IoT API URLs

API access can be over HTTP or secure HTTP (HTTPs).

<aside class="notice">Note: We strongly advise to use HTTPs to make sure your data travels encrypted, avoiding the exposure of your API token and/or sensor data.</aside>

## Our URLs:

|Using|Url|Port|
|---|---|---|
|HTTP|https://infiiot.com|80|
|HTTPs|https://infiiot.com|443|
|IP |127.0.0.1|80/443|

## Why use HTTPs?

As requests can contain the Token keys, which uniquely identifies you to the server, so if someone else copied this Token keys, they could craft a request to the server and pose as your client, and thereby automatically be logged in without even having access to your username or password.

This is where Secure HTTP, or HTTPS, helps. A resource that's accessed by HTTPS will start with `https://` instead of `http://`, and usually be displayed with a lock icon in most browsers:
With HTTPS every request/response is encrypted before being transported on the network. This means if a malicious hacker sniffed out the HTTP traffic, the information would be encrypted and useless.

# Send data

Using our APIs you can send data to the platform in two ways:
* Send data to a device
* Send data to a variable

If you are looking for more advanced API capabilities, such as editing devices, deleting devices and, many more, please refer to [Infi-IoT Documentation-hardware](http://infiiot.com/Documentation-hardware).

## Send data to a device

**Request structure:**

    POST /api/v1.6/update-Device HTTP/1.1<CR><LN>
    Host: {Host}<CR><LN>
    User-Agent: {USER_AGENT}<CR><LN>
    X-Auth-Token: {TOKEN}<CR><LN>
    Content-Type: application/json<CR><LN>
    Content-Length: {PAYLOAD_LENGTH}<CR><LN><CR><LN>
    {PAYLOAD}
    <CR><LN>

**Expected Response:**

    HTTP/1.1 200 OK<CR><LN>
    Server: nginx<CR><LN>
    Date: Tue, 04 Sep 2018 22:35:06 GMT<CR><LN>
    Content-Type: application/json<CR><LN>
    Transfer-Encoding: chunked<CR><LN>
    Vary: Cookie<CR><LN>
    Allow: GET, POST, HEAD, OPTIONS<CR><LN><CR><LN>
    {PAYLOAD_LENGTH_IN_HEXADECIMAL}<CR><LN>
    {"{VARIABLE_LABEL}": [{"status_code": 201}]}<CR><LN>
    0<CR><LN>

You can keep the device-name as desired. If the device does not exist in your Infi-IoT Account, it will be automatically created. Device labels can contain any alphanumerical characters with or without blank spaces.

To set the values of the variable, just create a JSON data payload as below

```
POST https://infiiot.com/api/v1.0/update-Device
```

> Send a Single data-point 

```shell
curl -X POST -H "X-Auth-Token: BBFF-Rfcgaxns6HlVb155WA0RhSY85xNDmB" -H "Content-Type: application/json" -d '{"temperature": 27}' https://infiiot.com/api/v1.0/devices/update-device
```

```cpp
/*************************************************************************************************
 * This Example sends harcoded data to Ubidots and serves as example for users that have devices
 * based on ESP8266 chips
 *
 * This example is given AS IT IS without any warranty
 *
 * Made by Jose García., adapted from the original WiFiClient ESP8266 example
 *************************************************************************************************/

/********************************
 * Libraries included
 *******************************/

#include <ESP8266WiFi.h>

/********************************
 * Constants and objects
 *******************************/
char const * SSID_NAME = "..."; // Put here your SSID name
char const * SSID_PASS = "...."; // Put here your password

char* TOKEN = ""; // Put here your TOKEN

char* DEVICE_LABEL = "truck"; // Your Device label

/* Put here your variable's labels*/
char const * VARIABLE_LABEL_1 = "speed";

/* HTTP Settings */
char const * HTTPSERVER = "infiiot.com";
const int HTTPPORT = 80;
char const * USER_AGENT = "ESP8266";
char const * VERSION = "1.0";

WiFiClient clientinfi;

/********************************
 * Auxiliar Functions
 *******************************/

void SendToInfiiot(char* payload) {

  int contentLength = strlen(payload);

  /* Connecting the client */
  clientinfi.connect(HTTPSERVER, HTTPPORT);

  if (clientinfi.connected()) {
    /* Builds the request POST - Please reference this link to know all the request's structures https://ubidots.com/docs/api/ */
    clientinfi.print(F("POST /api/v1.0/update-device"));
    clientinfi.print(DEVICE_LABEL);
    clientinfi.print(F(" HTTP/1.1\r\n"));
    clientinfi.print(F("Host: "));
    clientinfi.print(HTTPSERVER);
    clientinfi.print(F("\r\n"));
    clientinfi.print(F("User-Agent: "));
    clientinfi.print(USER_AGENT);
    clientinfi.print(F("/"));
    clientinfi.print(VERSION);
    clientinfi.print(F("\r\n"));
    clientinfi.print(F("X-Auth-Token: "));
    clientinfi.print(TOKEN);
    clientinfi.print(F("\r\n"));
    clientinfi.print(F("Connection: close\r\n"));
    clientinfi.print(F("Content-Type: application/json\r\n"));
    clientinfi.print(F("Content-Length: "));
    clientinfi.print(contentLength);
    clientinfi.print(F("\r\n\r\n"));
    clientinfi.print(payload);
    clientinfi.print(F("\r\n"));

    Serial.println(F("Making request to Ubidots:\n"));
    Serial.print("POST /api/v1.0/update-device");
    Serial.print(DEVICE_LABEL);
    Serial.print(" HTTP/1.1\r\n");
    Serial.print("Host: ");
    Serial.print(HTTPSERVER);
    Serial.print("\r\n");
    Serial.print("User-Agent: ");
    Serial.print(USER_AGENT);
    Serial.print("/");
    Serial.print(VERSION);
    Serial.print("\r\n");
    Serial.print("X-Auth-Token: ");
    Serial.print(TOKEN);
    Serial.print("\r\n");
    Serial.print("Connection: close\r\n");
    Serial.print("Content-Type: application/json\r\n");
    Serial.print("Content-Length: ");
    Serial.print(contentLength);
    Serial.print("\r\n\r\n");
    Serial.print(payload);
    Serial.print("\r\n");
  } else {
    Serial.println("Connection Failed ubidots - Try Again");
  }

  /* Reach timeout when the server is unavailable */
  int timeout = 0;
  while (!clientinfi.available() && timeout < 5000) {
    timeout++;
    delay(1);
    if (timeout >= 5000) {
      Serial.println(F("Error, max timeout reached"));
      break;
    }
  }

  /* Reads the response from the server */
  Serial.println(F("\nUbidots' Server response:\n"));
  while (clientinfi.available()) {
    char c = clientinfi.read();
    //Serial.print(c); // Uncomment this line to visualize the response on the Serial Monitor
  }

  /* Disconnecting the client */
  clientinfi.stop();
}

/********************************
 * Main Functions
 *******************************/

void setup() {
  Serial.begin(115200);
  /* Connects to AP */
  WiFi.begin(SSID_NAME, SSID_PASS);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
    }

  WiFi.setAutoReconnect(true);
  Serial.println(F("WiFi connected"));
  Serial.println(F("IP address: "));
  Serial.println(WiFi.localIP());

  if (clientinfi.connect(HTTPSERVER, HTTPPORT)) {
    Serial.println("connected to Ubidots cloud");
  } else {
    Serial.println("could not connect to Ubidots cloud");
  }

}

void loop() {

  // Space to store values to send
  char payload[200];
  char str_val_1[30];
  char str_lat[30];
  char str_lng[30];

  /*---- Simulates the values of the sensors -----*/
  float sensor_value_1 = random(0, 1000)*1.0;

  /*---- Transforms the values of the sensors to char type -----*/

  /* 4 is mininum width, 2 is precision; float value is copied onto str_val*/
  dtostrf(sensor_value_1, 4, 2, str_val_1);

  // Important: Avoid to send a very long char as it is very memory space costly, send small char arrays
  sprintf(payload, "{\"");
  sprintf(payload, "%s%s\":{\"value\":%s}", payload, VARIABLE_LABEL_1, str_val_1);
  sprintf(payload, "%s}", payload);

  /* Calls the Ubidots Function POST */
  SendToInfiiot(payload);

  delay(5000);
}
```

```python
'''
This Example sends harcoded data to Ubidots using the request HTTP
library.

Please install the library using pip install requests

Made by Jose García @https://github.com/jotathebest/
'''

import requests
import random
import time

'''
global variables
'''

ENDPOINT = "infiiot.com"
DEVICE_LABEL = "weather-station"
VARIABLE_LABEL = "temperature"
TOKEN = ""
DELAY = 1  # Delay in seconds


def post_var(payload, url=ENDPOINT, device=DEVICE_LABEL, token=TOKEN):
    try:
        url = "http://{}/api/v1.0/update-device/{}".format(url, device)
        headers = {"X-Auth-Token": token, "Content-Type": "application/json"}

        attempts = 0
        status_code = 400

        while status_code >= 400 and attempts < 5:
            print("[INFO] Sending data, attempt number: {}".format(attempts))
            req = requests.post(url=url, headers=headers,
                                json=payload)
            status_code = req.status_code
            attempts += 1
            time.sleep(1)

        print("[INFO] Results:")
        print(req.text)
    except Exception as e:
        print("[ERROR] Error posting, details: {}".format(e))

def main():
    # Simulates sensor values
    sensor_value = random.random() * 100

    # Builds Payload and topíc
    payload = {VARIABLE_LABEL: sensor_value}

    # Sends data
    post_var(payload)


if __name__ == "__main__":
    while True:
        main()
        time.sleep(DELAY)
```

> Send data-point with coordinates

```shell
curl -X POST -H "X-Auth-Token: BBFF-Rfcgaxns6HlVb155WA0RhSY85xNDmB" -H "Content-Type: application/json" -d '{"position": {"value" : 1, "timestamp": 1514808000000, "context":{"lat":-6.2, "lng":75.4}}}' https://infiiot.com/api/v1.0/devices/update-device
```
```cpp
/*************************************************************************************************
 * This Example sends harcoded data to Ubidots and serves as example for users that have devices
 * based on ESP8266 chips
 *
 * This example is given AS IT IS without any warranty
 *
 * Made by Jose García., adapted from the original WiFiClient ESP8266 example
 *************************************************************************************************/

/********************************
 * Libraries included
 *******************************/

#include <ESP8266WiFi.h>

/********************************
 * Constants and objects
 *******************************/
char const * SSID_NAME = "..."; // Put here your SSID name
char const * SSID_PASS = "...."; // Put here your password

char* TOKEN = ""; // Put here your TOKEN

char* DEVICE_LABEL = "truck"; // Your Device label

/* Put here your variable's labels*/
char const * VARIABLE_LABEL_1 = "position";

/* HTTP Settings */
char const * HTTPSERVER = "infiiot.com";
const int HTTPPORT = 80;
char const * USER_AGENT = "ESP8266";
char const * VERSION = "1.0";

WiFiClient clientinfi;

/********************************
 * Auxiliar Functions
 *******************************/

void SendToInfiiot(char* payload) {

  int contentLength = strlen(payload);

  /* Connecting the client */
  clientinfi.connect(HTTPSERVER, HTTPPORT);

  if (clientinfi.connected()) {
    /* Builds the request POST - Please reference this link to know all the request's structures https://ubidots.com/docs/api/ */
    clientinfi.print(F("POST /api/v1.0/update-device"));
    clientinfi.print(DEVICE_LABEL);
    clientinfi.print(F(" HTTP/1.1\r\n"));
    clientinfi.print(F("Host: "));
    clientinfi.print(HTTPSERVER);
    clientinfi.print(F("\r\n"));
    clientinfi.print(F("User-Agent: "));
    clientinfi.print(USER_AGENT);
    clientinfi.print(F("/"));
    clientinfi.print(VERSION);
    clientinfi.print(F("\r\n"));
    clientinfi.print(F("X-Auth-Token: "));
    clientinfi.print(TOKEN);
    clientinfi.print(F("\r\n"));
    clientinfi.print(F("Connection: close\r\n"));
    clientinfi.print(F("Content-Type: application/json\r\n"));
    clientinfi.print(F("Content-Length: "));
    clientinfi.print(contentLength);
    clientinfi.print(F("\r\n\r\n"));
    clientinfi.print(payload);
    clientinfi.print(F("\r\n"));

    Serial.println(F("Making request to Ubidots:\n"));
    Serial.print("POST /api/v1.0/update-device");
    Serial.print(DEVICE_LABEL);
    Serial.print(" HTTP/1.1\r\n");
    Serial.print("Host: ");
    Serial.print(HTTPSERVER);
    Serial.print("\r\n");
    Serial.print("User-Agent: ");
    Serial.print(USER_AGENT);
    Serial.print("/");
    Serial.print(VERSION);
    Serial.print("\r\n");
    Serial.print("X-Auth-Token: ");
    Serial.print(TOKEN);
    Serial.print("\r\n");
    Serial.print("Connection: close\r\n");
    Serial.print("Content-Type: application/json\r\n");
    Serial.print("Content-Length: ");
    Serial.print(contentLength);
    Serial.print("\r\n\r\n");
    Serial.print(payload);
    Serial.print("\r\n");
  } else {
    Serial.println("Connection Failed ubidots - Try Again");
  }

  /* Reach timeout when the server is unavailable */
  int timeout = 0;
  while (!clientinfi.available() && timeout < 5000) {
    timeout++;
    delay(1);
    if (timeout >= 5000) {
      Serial.println(F("Error, max timeout reached"));
      break;
    }
  }

  /* Reads the response from the server */
  Serial.println(F("\nUbidots' Server response:\n"));
  while (clientinfi.available()) {
    char c = clientinfi.read();
    Serial.print(c); // Uncomment this line to visualize the response on the Serial Monitor
  }

  /* Disconnecting the client */
  clientinfi.stop();
}

/********************************
 * Main Functions
 *******************************/

void setup() {
  Serial.begin(115200);
  /* Connects to AP */
  WiFi.begin(SSID_NAME, SSID_PASS);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
    }

  WiFi.setAutoReconnect(true);
  Serial.println(F("WiFi connected"));
  Serial.println(F("IP address: "));
  Serial.println(WiFi.localIP());

  if (clientinfi.connect(HTTPSERVER, HTTPPORT)) {
    Serial.println("connected to Ubidots cloud");
  } else {
    Serial.println("could not connect to Ubidots cloud");
  }

}

void loop() {

  // Space to store values to send
  char payload[200];
  char str_val_1[30];
  char str_lat[30];
  char str_lng[30];

  /*---- Simulates the values of the sensors -----*/
  float sensor_value_1 = random(0, 1000)*1.0;
  float lat = -6.256;
  float lng = 75.236;

  /*---- Transforms the values of the sensors to char type -----*/

  /* 4 is mininum width, 2 is precision; float value is copied onto str_val*/
  dtostrf(sensor_value_1, 4, 2, str_val_1);
  dtostrf(lat, 4, 2, str_lat);
  dtostrf(lng, 4, 2, str_lng);

  // Important: Avoid to send a very long char as it is very memory space costly, send small char arrays
  sprintf(payload, "{\"");
  sprintf(payload, "%s%s\":{\"value\":%s", payload, VARIABLE_LABEL_1, str_val_1);
  sprintf(payload, "%s,\"context\":{\"lat\":%s, \"lng\":%s}}", payload, str_lat, str_lng);
  sprintf(payload, "%s}", payload);

  /* Calls the Ubidots Function POST */
  SendToInfiiot(payload);

  delay(5000);

}

```
```python
'''
This Example sends harcoded data to Ubidots using the request HTTP
library.

Please install the library using pip install requests

Made by Jose García @https://github.com/jotathebest/
'''

import requests
import random
import time

'''
global variables
'''

ENDPOINT = "infiiot.com"
DEVICE_LABEL = "truck"
VARIABLE_LABEL = "position"
TOKEN = "BBFF-YTP65d9ngV1useDm97rfZQ7j4QtijV"
DELAY = 1  # Delay in seconds


def post_var(payload, url=ENDPOINT, device=DEVICE_LABEL, token=TOKEN):
    try:
        url = "http://{}/api/v1.0/update-device/{}".format(url, device)
        headers = {"X-Auth-Token": token, "Content-Type": "application/json"}

        attempts = 0
        status_code = 400

        while status_code >= 400 and attempts < 5:
            print("[INFO] Sending data, attempt number: {}".format(attempts))
            req = requests.post(url=url, headers=headers,
                                json=payload)
            status_code = req.status_code
            attempts += 1
            time.sleep(1)

        print("[INFO] Results:")
        print(req.text)
    except Exception as e:
        print("[ERROR] Error posting, details: {}".format(e))


def main():
    # Simulates sensor values
    sensor_value = random.random() * 100
    lat = 6.101
    lng = -71.293

    # Builds Payload and topíc
    payload = {VARIABLE_LABEL: {"value": sensor_value,
                                "context": {"lat": lat, "lng": lng}}
               }

    # Sends data
    post_var(payload)


if __name__ == "__main__":
    while True:
        main()
        time.sleep(DELAY)
```

> Send multiple data-points 

```shell
curl -X POST -H "X-Auth-Token: BBFF-Rfcgaxns6HlVb155WA0RhSY85xNDmB" -H "Content-Type: application/json" -d '{"temperature": {"value": 27, "timestamp": 1514808000000}, "humidity": 55, "pressure": {"value": 78, "context":{name" : "John"}}}' 
https://infiiot.com/api/v1.0/devices/update-device
```
```cpp
/*************************************************************************************************
 * This Example sends harcoded data to Ubidots and serves as example for users that have devices
 * based on Arduino Ethernet shield
 *
 * This example is given AS IT IS without any warranty
 *
 * Made by Jose García., adapted from the original Arduino Ethernet example
 *************************************************************************************************/

/********************************
 * Libraries included
 *******************************/

#include <ESP8266WiFi.h>

/********************************
 * Constants and objects
 *******************************/
char const * SSID_NAME = "..."; // Put here your SSID name
char const * SSID_PASS = "...."; // Put here your password

char* TOKEN = ""; // Put here your TOKEN

char* DEVICE_LABEL = "weather-station"; // Your Device label

/* Put here your variable's labels*/
char const * VARIABLE_LABEL_1 = "temperature";
char const * VARIABLE_LABEL_2 = "humidity";

/* HTTP Settings */
char const * HTTPSERVER = "infiiot.com";
const int HTTPPORT = 80;
char const * USER_AGENT = "ESP8266";
char const * VERSION = "1.0";

WiFiClient clientinfi;

/********************************
 * Auxiliar Functions
 *******************************/

void SendToInfiiot(char* payload) {

  int contentLength = strlen(payload);

  /* Connecting the client */
  clientinfi.connect(HTTPSERVER, HTTPPORT);

  if (clientinfi.connected()) {
    /* Builds the request POST - Please reference this link to know all the request's structures https://ubidots.com/docs/api/ */
    clientinfi.print(F("POST /api/v1.0/update-device"));
    clientinfi.print(DEVICE_LABEL);
    clientinfi.print(F(" HTTP/1.1\r\n"));
    clientinfi.print(F("Host: "));
    clientinfi.print(HTTPSERVER);
    clientinfi.print(F("\r\n"));
    clientinfi.print(F("User-Agent: "));
    clientinfi.print(USER_AGENT);
    clientinfi.print(F("/"));
    clientinfi.print(VERSION);
    clientinfi.print(F("\r\n"));
    clientinfi.print(F("X-Auth-Token: "));
    clientinfi.print(TOKEN);
    clientinfi.print(F("\r\n"));
    clientinfi.print(F("Connection: close\r\n"));
    clientinfi.print(F("Content-Type: application/json\r\n"));
    clientinfi.print(F("Content-Length: "));
    clientinfi.print(contentLength);
    clientinfi.print(F("\r\n\r\n"));
    clientinfi.print(payload);
    clientinfi.print(F("\r\n"));

    Serial.println(F("Making request to Ubidots:\n"));
    Serial.print("POST /api/v1.0/update-device");
    Serial.print(DEVICE_LABEL);
    Serial.print(" HTTP/1.1\r\n");
    Serial.print("Host: ");
    Serial.print(HTTPSERVER);
    Serial.print("\r\n");
    Serial.print("User-Agent: ");
    Serial.print(USER_AGENT);
    Serial.print("/");
    Serial.print(VERSION);
    Serial.print("\r\n");
    Serial.print("X-Auth-Token: ");
    Serial.print(TOKEN);
    Serial.print("\r\n");
    Serial.print("Connection: close\r\n");
    Serial.print("Content-Type: application/json\r\n");
    Serial.print("Content-Length: ");
    Serial.print(contentLength);
    Serial.print("\r\n\r\n");
    Serial.print(payload);
    Serial.print("\r\n");
  } else {
    Serial.println("Connection Failed ubidots - Try Again");
  }

  /* Reach timeout when the server is unavailable */
  int timeout = 0;
  while (!clientinfi.available() && timeout < 5000) {
    timeout++;
    delay(1);
    if (timeout >= 5000) {
      Serial.println(F("Error, max timeout reached"));
      break;
    }
  }

  /* Reads the response from the server */
  Serial.println(F("\nUbidots' Server response:\n"));
  while (clientinfi.available()) {
    char c = clientinfi.read();
    Serial.print(c); // Uncomment this line to visualize the response on the Serial Monitor
  }

  /* Disconnecting the client */
  clientinfi.stop();
}

/********************************
 * Main Functions
 *******************************/

void setup() {
  Serial.begin(115200);
  /* Connects to AP */
  WiFi.begin(SSID_NAME, SSID_PASS);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
    }

  WiFi.setAutoReconnect(true);
  Serial.println(F("WiFi connected"));
  Serial.println(F("IP address: "));
  Serial.println(WiFi.localIP());

  if (clientinfi.connect(HTTPSERVER, HTTPPORT)) {
    Serial.println("connected to Ubidots cloud");
  } else {
    Serial.println("could not connect to Ubidots cloud");
  }

}

void loop() {

  // Space to store values to send
  char payload[200];
  char str_val_1[30];
  char str_val_2[30];

  /*---- Simulates the values of the sensors -----*/
  float sensor_value_1 = random(0, 1000)*1.0;
  float sensor_value_2 = random(0, 1000)*1.0;

  /*---- Transforms the values of the sensors to char type -----*/

  /* 4 is mininum width, 2 is precision; float value is copied onto str_val*/
  dtostrf(sensor_value_1, 4, 2, str_val_1);
  dtostrf(sensor_value_2, 4, 2, str_val_2);

  // Important: Avoid to send a very long char as it is very memory space costly, send small char arrays
  sprintf(payload, "{\"");
  sprintf(payload, "%s%s\":%s", payload, VARIABLE_LABEL_1, str_val_1);
  sprintf(payload, "%s,\"%s\":%s", payload, VARIABLE_LABEL_2, str_val_2);
  sprintf(payload, "%s}", payload);

  /* Calls the Ubidots Function POST */
  SendToInfiiot(payload);

  delay(5000);

}
```
```python
'''
This Example sends harcoded data to Ubidots using the request HTTP
library.

Please install the library using pip install requests

Made by Jose García @https://github.com/jotathebest/
'''

import requests
import random
import time

'''
global variables
'''

ENDPOINT = "infiiot.com"
DEVICE_LABEL = "weather-station"
VARIABLE_LABEL_1 = "temperature"
VARIABLE_LABEL_2 = "humidity"
TOKEN = "..."
DELAY = 1  # Delay in seconds


def post_var(payload, url=ENDPOINT, device=DEVICE_LABEL, token=TOKEN):
    try:
        url = "http://{}/api/v1.0/update-device/{}".format(url, device)
        headers = {"X-Auth-Token": token, "Content-Type": "application/json"}

        attempts = 0
        status_code = 400

        while status_code >= 400 and attempts < 5:
            print("[INFO] Sending data, attempt number: {}".format(attempts))
            req = requests.post(url=url, headers=headers,
                                json=payload)
            status_code = req.status_code
            attempts += 1
            time.sleep(1)

        print("[INFO] Results:")
        print(req.text)
    except Exception as e:
        print("[ERROR] Error posting, details: {}".format(e))


def main():
    # Simulates sensor values
    sensor_value_1 = random.random() * 100
    sensor_value_2 = random.random() * 100

    # Builds Payload and topíc
    payload = {VARIABLE_LABEL_1: sensor_value_1,
               VARIABLE_LABEL_2: sensor_value_2
              }

    # Sends data
    post_var(payload)


if __name__ == "__main__":
    while True:
        main()
        time.sleep(DELAY)
```


## Send data to a variable

```
POST https://infiiot.com/api/v1.0/update-variable
```
> Send a Single data-point 

```shell

```
```cpp

```
```python

```

> Send data-point with coordinates

```shell

```
```cpp

```
```python

```

> Send multiple data-points 

```shell

```
```cpp

```
```python

```


# Retrieve  data
Using our APIs you can retrive data from the platform in two ways:
* Retrieve  device data 
* Retrieve  variable data 

If you are looking for more advanced API capabilities, such as retrieving a specific number of values from device, retrieving data with exponential smoothing or data with pagination and, many more, please refer to [Infi-IoT Documentation-hardware](http://infiiot.com/Documentation-hardware).

# Response Codes
Response codes help users know the status and errors for the requests made to the server. Infi-IoT REST-API uses the following request status code when you make an HTTP request:

|Code|Detail|
|---|---|
|200|Ok -- Successful request.|
|201|Created -- Successful request + an item (device or variable) was created.|
|202|Accepted -- The request has been accepted for processing, but the processing has not been completed.|
|204|One of the fields is incorrect and the request is not saved -- Please verify it's a valid JSON string and that the fields are the ones expected by the endpoint (string, object, or float).|
|400|	Bad Request -- Error due to an invalid body in your request. Please verify it's a valid JSON string and that the fields are the ones expected by the endpoint (string, object, or float).|
|401|Invalid API key -- Please verify your API Key.|
|402|Payment required -- Please verify your balance.|
|403|Forbidden -- This token is not valid. Please verify your token.|
|404|Not Found -- We couldn’t find the URL you're trying to access. This might be due to a wrong device label or ID, a wrong variable label or ID, or simply a typo in the URL of the request.|
|405|	Method Not Allowed -- This API endpoint does not accept the method used. Check our API docs to see the allowed methods.|
|415|Unsupported media type -- The payload is in a format not supported by this method on the target resource.|
|420|You have exceeded your API limits -- To upgrade your API limits contacts the Ubidots team|
|423|Device does not receive data because it is disabled|
|429|Too many requests -- Many requests sent in a given amount of time ("rate limiting")|
|50|Internal Error -- We're having issues with our servers.|


# Limits

Configuration updates are limited to 1 update per second, per device. However, for best results, device configuration should be updated much less often — at most, once every 5 seconds.

The update rate is calculated as the time between the most recent server acknowledgment and the next update request.


