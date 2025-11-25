# Monitoramento de Potência de Sinal WiFi com ESP32 e Envio via MQTT
Este projeto tem como objetivo medir continuamente a potência do sinal WiFi (em dBm) utilizando um ESP32 e publicar esses valores em uma plataforma IoT via protocolo MQTT. Para a visualização desses valores utilizamos a plataforma ubidots. 
## Objetivos do Projeto
Conectar o ESP32 à rede WiFi.


Medir o nível de sinal WiFi em dBm com o comando WiFi.RSSI().


Exibir localmente os valores na porta Serial da Arduino IDE.


Publicar cada valor de dBm na plataforma Ubidots via MQTT.


Criar um gráfico contínuo (tempo × dBm) na dashboard.


Realizar testes em cenários variados, incluindo ambientes com bloqueio de sinal.


Registrar a queda abrupta de sinal ao entrar no elevador (simulação de gaiola de Faraday).


Gravar um vídeo demonstrando toda a experiência e medições.


## Cenários Testados e Valores Obtidos
Os testes foram realizados em diferentes locais do campus, monitorando o nível de sinal por aproximadamente 10 segundos (ou 20 segundos no elevador).
Registors realizados: 

1) Sala de aula
-36 dBm
2) Catraca da recepção
-52 dBm
3) 1º andar — IT Bar
-45 dBm
4) Elevador fechado (20s) — Simulação de Gaiola de Faraday
-73 dBm
5) 2º andar — Laboratório André Leal
-76 dBm
6) Final do mezanino 2 — Última sala de reunião
-46 dBm
7) Volta pelo elevador (sem permanecer dentro)
-67 dBm
8) Lado de fora da rua, após sair pela recepção
-51 dBm

A queda expressiva registrada dentro do elevador comprova o bloqueio eletromagnético típico de uma gaiola de Faraday.

## Código Utilizado (ESP32 + MQTT + Ubidots)

```
#include "UbidotsEsp32Mqtt.h"

const char *WIFI_SSID = "Inteli.Iot"; 
const char *WIFI_PASS = "%(Yk(sxGMtvFEs.3"; 
const char *UBIDOTS_TOKEN = "BBUS-o3P7eUTIP2fU5qOt44jOgzFhLIhXVK"; 

const char *DEVICE_LABEL = "esp32_t17_rafa"; 
const char *VARIABLE_LABEL = "dbm";
const char *CLIENT_ID = "seu_id"; 

Ubidots ubidots(UBIDOTS_TOKEN, CLIENT_ID); 
const int PUBLISH_FREQUENCY = 3000; 
unsigned long timer;
unsigned long last_publish = 0;
const unsigned long PUBLISH_INTERVAL = PUBLISH_FREQUENCY;

void callback(char *topic, byte *payload, unsigned int length){
  Serial.print("Message arrived [");
  Serial.print(topic);
  Serial.print("] ");
  for (int i = 0; i < length; i++){
    Serial.print((char)payload[i]);
  }
  Serial.println();
}

void setup(){
  Serial.begin(115200);
  ubidots.setDebug(true); 
  ubidots.connectToWifi(WIFI_SSID, WIFI_PASS);
  ubidots.setCallback(callback);
  ubidots.setup();
  ubidots.reconnect();
  timer = millis();
}

void loop() {
  if (WiFi.status() == WL_CONNECTED) {
    int32_t dBm = WiFi.RSSI();
    if (millis() - last_publish > PUBLISH_INTERVAL) {
      ubidots.add(VARIABLE_LABEL, dBm);
      ubidots.publish(DEVICE_LABEL);
      Serial.printf("Nível de Sinal Wi-Fi: %d dBm\n", dBm);
      last_publish = millis();
    }
  } else {
    Serial.println("Wi-Fi desconectado! Tentando reconectar...");
    WiFi.begin(WIFI_SSID, WIFI_PASS);
  }
}
```
## Grafico das oscilações 
![image](https://res.cloudinary.com/dtxiyeitw/image/upload/v1764083781/Imagem_do_WhatsApp_de_2025-11-25_%C3%A0_s_12.16.11_8deaca2c_xzpdh1.jpg)

## Vídeo da Experiência
(https://drive.google.com/file/d/1veG104HWM8lyb9rJTi8sotJwnHTsPL8i/view?usp=drivesdk)

### Grupo:
Rafael Campos
Leonardo Lameda
Vinicius Cadena
