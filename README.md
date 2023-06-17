# Practica DHT22 con node-red
Este repositorio muestra como podemos programar una ESP32 con un DHT22 y poder observar los datos con node-red

## Introducción

### Descripción

En esta practica utilizamos la ```Esp32``` con un un DHT-22 para medir temperatura y humedad, en node-red hacemos un programa para captar los datos de temperatura y humedad para mostrarlos en una grafica y unos indicadores,para eso se utilisa un mq publico

## Material Necesario

Para realizar esta practica necesitas lo siguiente

- [WOKWI](https://https://wokwi.com/)
- Tarjeta ESP 32
- Sensor DHT-22
- node-red



## Instrucciones


Para poder usar este repositorio necesitas entrar a la plataforma [WOKWI](https://https://wokwi.com/).


### Instrucciones de preparación de entorno 

1. Abrir la terminal de programación y colocar la siguente programación:

```
#include <ArduinoJson.h>
#include <WiFi.h>
#include <PubSubClient.h>
#define BUILTIN_LED 2
#include "DHTesp.h"
const int DHT_PIN = 15;
DHTesp dhtSensor;
// Update these with values suitable for your network.

const char* ssid = "Wokwi-GUEST";
const char* password = "";
const char* mqtt_server = "44.195.202.69";
String username_mqtt="DavidVargas";
String password_mqtt="1234";

WiFiClient espClient;
PubSubClient client(espClient);
unsigned long lastMsg = 0;
#define MSG_BUFFER_SIZE  (50)
char msg[MSG_BUFFER_SIZE];
int value = 0;

void setup_wifi() {

  delay(10);
  // We start by connecting to a WiFi network
  Serial.println();
  Serial.print("Connecting to ");
  Serial.println(ssid);

  WiFi.mode(WIFI_STA);
  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  randomSeed(micros());

  Serial.println("");
  Serial.println("WiFi connected");
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());
}

void callback(char* topic, byte* payload, unsigned int length) {
  Serial.print("Message arrived [");
  Serial.print(topic);
  Serial.print("] ");
  for (int i = 0; i < length; i++) {
    Serial.print((char)payload[i]);
  }
  Serial.println();

  // Switch on the LED if an 1 was received as first character
  if ((char)payload[0] == '1') {
    digitalWrite(BUILTIN_LED, LOW);   
    // Turn the LED on (Note that LOW is the voltage level
    // but actually the LED is on; this is because
    // it is active low on the ESP-01)
  } else {
    digitalWrite(BUILTIN_LED, HIGH);  
    // Turn the LED off by making the voltage HIGH
  }

}

void reconnect() {
  // Loop until we're reconnected
  while (!client.connected()) {
    Serial.print("Attempting MQTT connection...");
    // Create a random client ID
    String clientId = "ESP8266Client-";
    clientId += String(random(0xffff), HEX);
    // Attempt to connect
    if (client.connect(clientId.c_str(), username_mqtt.c_str() , password_mqtt.c_str())) {
      Serial.println("connected");
      // Once connected, publish an announcement...
      client.publish("outTopic", "hello world");
      // ... and resubscribe
      client.subscribe("inTopic");
    } else {
      Serial.print("failed, rc=");
      Serial.print(client.state());
      Serial.println(" try again in 5 seconds");
      // Wait 5 seconds before retrying
      delay(5000);
    }
  }
}

void setup() {
  pinMode(BUILTIN_LED, OUTPUT);     // Initialize the BUILTIN_LED pin as an output
  Serial.begin(115200);
  setup_wifi();
  client.setServer(mqtt_server, 1883);
  client.setCallback(callback);
  dhtSensor.setup(DHT_PIN, DHTesp::DHT22);
}

void loop() {


delay(1000);
TempAndHumidity  data = dhtSensor.getTempAndHumidity();
  if (!client.connected()) {
    reconnect();
  }
  client.loop();

  unsigned long now = millis();
  if (now - lastMsg > 2000) {
    lastMsg = now;
    //++value;
    //snprintf (msg, MSG_BUFFER_SIZE, "hello world #%ld", value);

    StaticJsonDocument<128> doc;

    doc["DEVICE"] = "ESP32";
    //doc["Anho"] = 2022;
    //doc["Empresa"] = "Educatronicos";
    doc["TEMPERATURA"] = String(data.temperature, 1);
    doc["HUMEDAD"] = String(data.humidity, 1);
   

    String output;
    
    serializeJson(doc, output);

    Serial.print("Publish message: ");
    Serial.println(output);
    Serial.println(output.c_str());
    client.publish("Vargas95", output.c_str());
  }
}
```
2. Realizar la conecion del circuito

![](https://github.com/DavidVar95/Practica_Nivel_de_agua/blob/main/Captura%20de%20pantalla%202023-06-10%2012.58.28.png?raw=true)


### Instrucciónes de operación

1. Iniciar simulador.
2. Selecciona el sensor"HC-SR04" Y modifica la distancia.
3. los led enciende deacuerdo el aumento de distancia 

## Resultados

cuando la distancia es de 0 a 2 ningun led enciende.

![](https://github.com/DavidVar95/Practica_Nivel_de_agua/blob/main/Captura%20de%20pantalla%202023-06-10%2013.06.29.png?raw=true)


led 1 enciande con el nivel de 3 a 10


![](https://github.com/DavidVar95/Practica_Nivel_de_agua/blob/main/Captura%20de%20pantalla%202023-06-10%2013.08.14.png?raw=true))

led 1 y 2 enciande con el nivel de 11 a 20

![](https://github.com/DavidVar95/Practica_Nivel_de_agua/blob/main/Captura%20de%20pantalla%202023-06-10%2013.10.56.png?raw=true))

led 1,2 y 3 enciande con el nivel de 21 a 30

![](https://github.com/DavidVar95/Practica_Nivel_de_agua/blob/main/Captura%20de%20pantalla%202023-06-10%2013.12.48.png?raw=true))

led 1,2,3 y 4 enciande con el nivel de 31 o mas

![](https://github.com/DavidVar95/Practica_Nivel_de_agua/blob/main/Captura%20de%20pantalla%202023-06-10%2013.15.24.png?raw=true))

# Créditos

Desarrollado por Ing. David Vargas Gonzalez

