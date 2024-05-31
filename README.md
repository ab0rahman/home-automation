
# Home Automation using Telegram bot

## How the Code Works:
Home automation using Telegram with NodeMCU involves controlling devices like lamps through text messages. NodeMCU, based on the ESP8266 Wi-Fi SoC, is used for this IoT project, integrating with the Telegram app for remote control. The system architecture includes GPIO pins for communication, and the Telegram Bot API handles user commands. The process involves setting up Telegram, installing the Telegram library in Arduino IDE, and coding to interpret commands and control hardware components. This method offers a user-friendly way to manage home automation via smartphones


## Installation

Start by importing the required libraries.

```bash
#ifdef ESP32
  #include <WiFi.h>
#else
  #include <ESP8266WiFi.h>
#endif
#include <WiFiClientSecure.h>
#include <UniversalTelegramBot.h>   // Universal Telegram Bot Library written by Brian Lough: https://github.com/witnessmenow/Universal-Arduino-Telegram-Bot
#include <ArduinoJson.h>

```
    
## Network Credentials:

Insert your network credentials in the following variables.

```bash
    const char* ssid = "REPLACE_WITH_YOUR_SSID";
    const char* password = "REPLACE_WITH_YOUR_PASSWORD"; 
```

## Define Output:
Set the GPIO you want to control. In our case, we’ll control GPIO 2 (built-in LED) and its state is LOW by default.
```bash
    const int ledPin = 2;
    bool ledState = LOW;
```


## Telegram Bot Token:
Insert your Telegram Bot token you’ve got from Botfather on the BOTtoken variable.
```bash
    #define BOTtoken "XXXXXXXXXX:XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"       // your Bot Token (Get from Botfather)
```


## Telegram User ID:
Insert your chat ID. The one you’ve got from the IDBot.
```bash
    #define CHAT_ID "XXXXXXXXXX"
```
Create a new WiFi client with WiFiClientSecure.
```bash
    WiFiClientSecure client;
```
Create a bot with the token and client defined earlier.
```bash
    UniversalTelegramBot bot(BOTtoken, client);
```
The botRequestDelay and lastTimeBotRan are used to check for new Telegram messages every x number of seconds. In this case, the code will check for new messages every second (1000 milliseconds). You can change that delay time in the botRequestDelay variable.
```bash
    int botRequestDelay = 1000;
    unsigned long lastTimeBotRan;
```


## handleNewMessages():
The handleNewMessages() function handles what happens when new messages arrive.
```bash
void handleNewMessages(int numNewMessages) {
  Serial.println("handleNewMessages");
  Serial.println(String(numNewMessages));

  // Check the available messages
  for (int i = 0; i < numNewMessages; i++) {
    // Get the chat ID for that particular message and store it in the chat_id variable
    String chat_id = String(bot.messages[i].chat_id);

    // If the chat_id is different from your chat ID (CHAT_ID), ignore the message and wait for the next message
    if (chat_id != CHAT_ID) {
      bot.sendMessage(chat_id, "Unauthorized user", "");
      continue;
    }

    // Save the message in the text variable and check its content
    String text = bot.messages[i].text;
    Serial.println(text);

    // Save the name of the sender
    String from_name = bot.messages[i].from_name;

    // If it receives the /start message, send the valid commands to control the ESP32/ESP8266
    if (text == "/start") {
      String welcome = "Welcome, " + from_name + ".\n";
      welcome += "Use the following commands to control your outputs.\n\n";
      welcome += "/led_on to turn GPIO ON \n";
      welcome += "/led_off to turn GPIO OFF \n";
      welcome += "/state to request current GPIO state \n";
      bot.sendMessage(chat_id, welcome, "");
    }

    // Send a message to the bot
    // You just need to use the sendMessage() method on the bot object and pass as arguments the recipient’s chat ID, the message, and the parse mode
    bot.sendMessage(chat_id, welcome, "");

    // If it receives the /led_on message, turn the LED on and send a message confirming we’ve received the message
    if (text == "/led_on") {
      bot.sendMessage(chat_id, "LED state set to ON", "");
      ledState = HIGH;
      digitalWrite(ledPin, ledState);
    }

    // Do something similar for the /led_off message
    if (text == "/led_off") {
      bot.sendMessage(chat_id, "LED state set to OFF", "");
      ledState = LOW;
      digitalWrite(ledPin, ledState);
    }

    // If the received message is /state, check the current GPIO state and send a message accordingly
    if (text == "/state") {
      if (digitalRead(ledPin)) {
        bot.sendMessage(chat_id, "LED is ON", "");
      } else {
        bot.sendMessage(chat_id, "LED is OFF", "");
      }
    }
  }
}
```

## setup():

In the setup(), initialize the Serial Monitor.
```bash
      Serial.begin(115200);
```
If you’re using the ESP8266, you need to use the following line:

```bash
    #ifdef ESP8266
    // Get UTC time via NTP
    configTime(0, 0, "pool.ntp.org");      
    // Add root certificate for api.telegram.org
    client.setTrustAnchors(&cert); 
    #endif

    // Set the LED as an output and set it to LOW when the ESP first starts
    pinMode(ledPin, OUTPUT);
    digitalWrite(ledPin, ledState);

    // Initialize Wi-Fi and connect the ESP to your local network
    WiFi.mode(WIFI_STA);
    WiFi.begin(ssid, password);
```


##  Init Wi-Fi:
Initialize Wi-Fi and connect the ESP to your local network with the SSID and password defined earlier.
```bash
    WiFi.mode(WIFI_STA);
    WiFi.begin(ssid, password);
    #ifdef ESP32
        client.setCACert(TELEGRAM_CERTIFICATE_ROOT); // Add root certificate for api.telegram.org
    #endif
    while (WiFi.status() != WL_CONNECTED) {
        delay(1000);
        Serial.println("Connecting to WiFi..");
    }

```


## loop():

In the loop(), check for new messages every second.
```bash
    void loop() {
    if (millis() > lastTimeBotRan + botRequestDelay)  {
        int numNewMessages = bot.getUpdates(bot.last_message_received + 1);


    When a new message arrives, call the handleNewMessages function.
    while(numNewMessages) {
        Serial.println("got response");
        handleNewMessages(numNewMessages);
        numNewMessages = bot.getUpdates(bot.last_message_received + 1);
        }
        lastTimeBotRan = millis();
    }
    }
```
<img src="https://github.com/ab0rahman/home-automation/blob/main/images/image10.jpg?raw=true" width="500" height="auto">
<img src="https://github.com/ab0rahman/home-automation/blob/main/images/image21.jpg?raw=true" width="500" height="auto">


## RESULTS
After uploading the code, press the ESP32/ESP8266 on-board EN/RST button so that it starts running the code. Then, you can open the Serial Monitor to check what’s happening in the background.

Go to your Telegram account and open a conversation with your bot. Send the following commands and see the bot responding:
```bash
    /start shows the welcome message with the valid commands.

    /led_on turns the LED on.

    /led_off turns the LED off.

    /state requests the current LED state.
```
<img src="https://github.com/ab0rahman/home-automation/blob/main/images/image20.png?raw=true" width="300" height="auto">

The on-board LED should turn on and turn off accordingly (the ESP8266 on-board LED works in reverse, it’s off when you send /led_on and on when you send /led_off).

<img src="https://github.com/ab0rahman/home-automation/blob/main/images/image11.png?raw=true" width="500" height="auto">

At the same time, on the Serial Monitor you should see that the ESP is receiving the messages.

<img src="https://github.com/ab0rahman/home-automation/blob/main/images/image3.png?raw=true" width="500" height="auto">

If you try to interact with your bot from another account, you’ll get the the “Unauthorized user” message.
<img src="https://github.com/ab0rahman/home-automation/blob/main/images/image8.png?raw=true" alt="proton search github" width="300" height="auto">



# Collaborators
  | |  |  |  |  |
  | ------------- | ------------- | ------------- | ------------- | ------------- |
  | Abdur Rahman | [GitHub](https://github.com/ab0rahman) | [Email](mailto:letsmail.him@gmail.com) | 
  | Shayam Kumar | [Github]() | [Email](mailto:) | 
  | N.V.S.L Srija  | [GitHub]() | [Email](mailto:) | 
  | N pravallika | [Github](https://github.com/NarsupalliPravallika) | [Email](mailto:narsupallipravallika.20.it@anits.edu.in) |  
