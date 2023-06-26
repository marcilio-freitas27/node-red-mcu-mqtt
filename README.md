## Atividade: Construa uma aplicação IoT integrada com uma rede social (2BIM)
### Proposta
- Construa uma aplicação IoT que atende aos seguintes requisitos.
-   Comunica o NodeMCU com MQTT;  
-   Comunica o NodeMCU via MQTT com o Node-red e recebe comandos do Dashboard;
-   Comunica o NodeMCU via MQTT com um rede social (Ex: Telegram).  

Escreva os passos realizados em um arquivo markdown.md no seu repositório.  
Os seguintes aspectos também serão considerados na avaliação: criatividade, organização, qualidade da documentação e aplicabilidade. O material em anexo serve de apoio ao desenvolvimento da tarefa.

1. **Comunica o NodeMCU com MQTT**

Para estabelecer esta comunicação, conectamos o NodeMCU no computador com um cabo usb do tipo b na entrada COM9 e alterando o tipo de ferramenta para NodeMCU 1.0.  Esta alteração depende da importação de uma nova biblioteca para o NodeMCU na IDE do Arduino. 
Após isso, instalamos o módulo PubSubClient na IDE do Arduino e usamos como base o script abaixo:

```
#include <ESP8266WiFi.h>  
#include <PubSubClient.h>  
  
// Defina as credenciais do Wi-Fi  
const char* ssid = "Galaxy A312E16";  
const char* password = "12345678";  
  
// Defina as informações do servidor MQTT  
const char* mqtt_server = "[test.mosquitto.org](http://test.mosquitto.org/)";  
const int mqtt_port = 1883;  
const char* mqtt_username = "";  
const char* mqtt_password = "";  
  
// Defina os tópicos MQTT  
const char* ligarTopico = "ligar";  
const char* desligarTopico = "desligar";  
const char* estadoTopico = "estado";  
  
// Pino do LED  
const int ledPin = D4;  
  
WiFiClient espClient;  
PubSubClient client(espClient);  
  
// Função para reconexão com o servidor MQTT  
void reconectar() {  
while (!client.connected()) {  
		Serial.print("Conectando ao servidor MQTT...");  
		if (client.connect("NodeMCU_Client", mqtt_username, mqtt_password)) {  
			Serial.println("Conectado!");  
			client.subscribe(ligarTopico);  
			client.subscribe(desligarTopico);  
		} else {  
			Serial.print("Falha na conexão (rc=");  
			Serial.print(client.state());  
			Serial.println("). Tentando novamente em 5 segundos...");  
			delay(5000);  
		}  
	}  
}  
  
// Função chamada quando uma mensagem MQTT é recebida  
void callback(char* topic, byte* payload, unsigned int length) {  
	String msg;  
	for (int i = 0; i < length; i++) {  
		msg += (char)payload[i];  
	}  
  
	Serial.print("Mensagem recebida no tópico: ");  
	Serial.println(topic);  
	Serial.print("Conteúdo: ");  
	Serial.println(msg);  
	  
	// Verifica qual tópico recebeu a mensagem  
	digitalWrite(ledPin, HIGH); // Desliga o LED  
	if (strcmp(topic, ligarTopico) == 0) {  
		digitalWrite(ledPin, LOW); // Liga o LED  
		client.publish(estadoTopico, "ligado");  
	} else if (strcmp(topic, desligarTopico) == 0) {  
		digitalWrite(ledPin, HIGH); // Desliga o LED  
		client.publish(estadoTopico, "desligado");  
	}  
}  
  
void setup() {  
	pinMode(ledPin, OUTPUT);  
	digitalWrite(ledPin, LOW);  
	  
	Serial.begin(9600);  
	delay(100);  
	  
	// Conecta-se à rede Wi-Fi  
	WiFi.begin(ssid, password);  
	while (WiFi.status() != WL_CONNECTED) {  
		delay(500);  
		Serial.print(".");  
	}  
	Serial.println("");  
	Serial.print("Conectado à rede Wi-Fi ");  
	Serial.println(ssid);  
	Serial.print("Endereço IP: ");  
	Serial.println(WiFi.localIP());  
	  
	// Conecta-se ao servidor MQTT  
	client.setServer(mqtt_server, mqtt_port);  
	client.setCallback(callback);  
}  
  
void loop() {  
	if (!client.connected()) {  
		reconectar();  
	}  
	client.loop();  
}
```
Feito o passo anterior, precisamos fazer o Node se comunicar via WIFI. Criamos um acesso WIFI a um rede 4G roteando via SmarthPhone Android e identificando a SSID da rede e a Senha através da seguinte linha do script exibido anteriormente:

```
// Defina as credenciais do Wi-Fi  
const char* ssid = "Galaxy A312E16";  
const char* password = "12345678";  
```
2. -   **Comunica o NodeMCU via MQTT com o Node-red e recebe comandos do Dashboard;**

Para a comunicação com o Dashboard do Node-red via MQTT e NodeMCU, utilizamos widgets de entrada e saída para cada um dos tópicos e dois botões do Dashboard para ligar e desligar o led padrão (D4) no Node-Red. Os botões de entrada recebiam o servidor MQTT (test.mosquitto.org:1883) e o tópico, que eram ligar parar ligar, desligar para desligar e estado para verificar em qual das opções anteriores ele estava.

![NodeMCU](/imagens/dashboard.png)
Dashboard

![NodeMCU](/imagens/mcu-led-off.jpg)
NodeMCU com led desligada

![NodeMCU](/imagens/mcu-led-on.jpg)
NodeMCU com led ligada

![NodeMCU](/imagens/node-red-mqtt-conf1.png)
Node-Red configuração parte 1

![NodeMCU](/imagens/node-red-mqtt-conf2.png)
Node-Red configuração parte 2

![NodeMCU](/imagens/node-red-mqtt-conf3.png)
Node-Red configuração parte 3

![NodeMCU](/imagens/node-red.png)
Node-Red configuração final

3 -   **Comunica o NodeMCU via MQTT com um rede social (Ex: Telegram).** 

Para a comunicação via telegram, baixamos a biblioteca para usar o telegram no Node-red e utlizamos os widgets *sender*
e *receiver*. Esses widgets se comunicam para envio(sender + mqqt sender) e recebimento(receiiver + mqtt receiver) 
com mqtt e o mqtt com o nodeMCU, exibindo os resultados no debud do node-red. Criamos comandos /ligar e /desligar no 
telegram via bot do telegram, token e o chatid, e um /status para pegar quando o estado foi alterado. 
O envio e o armazenamento de dados depende de funções que tratam os valores enviados.

![NodeMCU](/imagens/mcu-red-telegram.png)
Node-Red + Mqtt com telegram e dashboard

![NodeMCU](/imagens/func-guarda-estado.png)
Função que guarda estado

![NodeMCU](/imagens/func-envia-mensagem.png)
Função que envia mensagem

![NodeMCU](/imagens/func-envia-estado.png)
Função que envia estado
