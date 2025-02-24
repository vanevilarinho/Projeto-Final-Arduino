# Projeto-Final-Arduino

#include <SPI.h>
#include <MFRC522.h>

#define SS_PIN 10
#define RST_PIN 9

#define GREEN_LED_PIN 5  // Pino do LED verde
#define RED_LED_PIN 6    // Pino do LED vermelho

MFRC522 mfrc522(SS_PIN, RST_PIN);  // Criando uma instância do MFRC522

unsigned long timerStart = 0;   // Variável para armazenar o tempo do timer
const unsigned long timeout = 2000;  // 2 segundos

bool cardDetected = false;  // Variável para controlar o estado do cartão
bool timerStarted = false;  // Flag para verificar se o timer foi iniciado

void setup() {
  Serial.begin(9600);    // Inicia a comunicação serial
  SPI.begin();           // Inicia a comunicação SPI
  mfrc522.PCD_Init();    // Inicia o módulo RC522
  
  pinMode(GREEN_LED_PIN, OUTPUT);  // Configura o LED verde como saída
  pinMode(RED_LED_PIN, OUTPUT);    // Configura o LED vermelho como saída

  // Acende o LED verde inicialmente
  digitalWrite(GREEN_LED_PIN, HIGH);
  digitalWrite(RED_LED_PIN, LOW);
  
  Serial.println("Aproxime o cartão ou tag RFID.");
}

void loop() {
  // Verifica se uma nova tag RFID foi detectada
  if (mfrc522.PICC_IsNewCardPresent()) {
    // Verifica se o cartão foi autenticado
    if (mfrc522.PICC_ReadCardSerial()) {
      if (!cardDetected) {
        // Se o cartão não foi detectado antes, acende o LED vermelho
        digitalWrite(RED_LED_PIN, HIGH);
        digitalWrite(GREEN_LED_PIN, LOW);  // Apaga o LED verde
        cardDetected = true;  // Marca o cartão como detectado
        timerStarted = true;  // Marca que o timer foi iniciado
        timerStart = millis();  // Inicia o timer de 2 segundos

        Serial.print("UID do cartão: ");
        
        // Exibe o UID do cartão no monitor serial
        for (byte i = 0; i < mfrc522.uid.size; i++) {
          Serial.print(mfrc522.uid.uidByte[i], HEX);
          Serial.print(" ");
        }
        Serial.println(); // Nova linha após exibir o UID
      }
    }
  }

  // Verifica se 2 segundos se passaram desde que o cartão foi detectado
  if (timerStarted && (millis() - timerStart >= timeout)) {
    // Verifica novamente se o cartão ainda está presente
    if (mfrc522.PICC_IsNewCardPresent()) {
      // Se o cartão ainda estiver presente, mantém o LED vermelho aceso
      if (mfrc522.PICC_ReadCardSerial()) {
        // Reacende o timer e mantém o LED vermelho
        timerStart = millis();
        Serial.println("Cartão ainda presente. Mantendo LED vermelho aceso.");
      }
    } else {
      // Se o cartão foi removido, apaga o LED vermelho e acende o verde
      digitalWrite(RED_LED_PIN, LOW);  // Apaga o LED vermelho
      digitalWrite(GREEN_LED_PIN, HIGH);  // Acende o LED verde
      cardDetected = false;  // Marca que o cartão foi removido
      timerStarted = false;  // Desativa o timer
      Serial.println("Cartão removido. LED verde aceso.");
    }
  }
}