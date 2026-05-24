# SecuriGate

| | |
|-|-|
|`Author` Ciolac Vladislav

## Description
SecuriGate est un système de contrôle d'accès intelligent et IoT (Internet of Things) qui combine plusieurs protocoles de communication matérielle et sans fil. Le cœur du système repose sur un Raspberry Pi Pico W qui gère un environnement multitâche. L'authentification locale se fait via un lecteur RFID (communiquant par le bus SPI). L'interface utilisateur locale est assurée par un écran LCD 1602 (utilisant un adaptateur I2C pour optimiser les broches IO). 

Au-delà de l'accès physique, le système intègre une dimension "Smart Home". Le microcontrôleur établit une connexion BLE (Bluetooth Low Energy) avec un PC local faisant office de passerelle (Gateway). Ce PC héberge un serveur web permettant la surveillance en temps réel et le déverrouillage à distance depuis un smartphone. 

Le système gère également l'environnement : une photorésistance lue par le convertisseur analogique-numérique (ADC) active progressivement une LED de courtoisie via un signal PWM en fonction de la baisse de luminosité. Enfin, un bouton de sortie d'urgence déclenche une interruption matérielle (Hardware Interrupt) pour une ouverture immédiate, surpassant toute autre tâche en cours. Un timer non-bloquant garantit le reverrouillage automatique de la porte après cinq secondes.

## Motivation
Le besoin de systèmes de sécurité domestiques modernes est en constante augmentation. Ce projet est né de la volonté de concevoir une architecture IoT complète, allant du capteur physique jusqu'à l'interface utilisateur web. L'objectif pédagogique principal est de maîtriser et d'intégrer simultanément de multiples protocoles de communication matérielle (SPI, I2C, ADC, PWM, GPIO Interrupts) tout en gérant la communication sans fil (BLE) et l'architecture Client-Serveur. L'utilisation du Raspberry Pi Pico W permet de simuler un environnement industriel moderne, remplaçant les architectures basiques par des solutions connectées et asynchrones.

## Architecture
Le système est structuré en plusieurs sous-systèmes coordonnés par l'unité centrale :

<p><b>L'unité centrale (Raspberry Pi Pico W)</b> coordonne l'ensemble des opérations. Elle exécute une boucle principale non bloquante (utilisant la fonction `millis()` ou `asyncio`) pour gérer les capteurs simultanément et maintenir la connexion BLE active.</p>

<p><b>Le sous-système d'authentification</b> utilise le module RFID RC522 connecté via le bus SPI (MISO, MOSI, SCK, CS) pour lire l'UID des badges de sécurité et le comparer à une base de données locale.</p>

<p><b>Le sous-système d'affichage</b> repose sur un écran LCD 1602 alimenté en 3.3V, utilisant un adaptateur I2C (broches SDA, SCL) pour minimiser le câblage. Il affiche les statuts du système en temps réel.</p>

<p><b>Le sous-système de contrôle à distance (Gateway PC)</b> utilise la puce sans fil du Pico W pour établir une connexion Bluetooth Low Energy (BLE). Le PC local exécute un script Python qui se connecte au Pico W et déploie un serveur web local (via Flask ou FastAPI). L'utilisateur accède à ce serveur via le navigateur de son smartphone pour envoyer la commande de déverrouillage.</p>

<p><b>Le sous-système d'éclairage ambiant</b> comprend une photorésistance (LDR) agissant comme diviseur de tension pour l'entrée analogique (ADC), et une LED contrôlée par modulation de largeur d'impulsion (PWM) qui s'illumine de manière inversement proportionnelle à la luminosité ambiante.</p>

<p><b>Le sous-système d'actionnement et d'urgence</b> est composé d'un servomoteur SG90 (alimenté en 5V via la broche VBUS) pour actionner le pêne de la porte. Un bouton poussoir est configuré avec une interruption matérielle (Hardware Interrupt) déclenchée sur un front descendant (FALLING), forçant l'ouverture immédiate.</p>

### Block diagram

<img width="1024" height="559" alt="image" src="https://github.com/user-attachments/assets/de9a61d1-8583-4bad-86f1-b977120cfa17" />


### Schematic

<img width="693" height="419" alt="image" src="https://github.com/user-attachments/assets/b5b4d8df-4421-4d22-92ed-f00d7960f39a" />

### Components

| Device | Usage | Price |
|--------|--------|-------|
| Raspberry Pi Pico WH | Microcontrôleur principal avec Wi-Fi/BLE intégré (Headers inclus) | [~39 RON](https://www.optimusdigital.ro/en/raspberry-pi-boards/13327-raspberry-pi-pico-2-w.html) |
| Module RFID RC522 | Authentification par badge (Communication SPI) | ~15 RON |
| Écran LCD 1602 (3.3V) | Affichage de l'état du système | ~20 RON |
| Adaptateur I2C pour LCD | Conversion série/parallèle pour réduire les câbles | ~5 RON |
| Servomoteur SG90 | Mécanisme de verrouillage de la porte (Actionneur PWM 50Hz) | ~12 RON |
| Photorésistance (LDR) | Capteur de luminosité ambiante | ~1 RON |
| Résistance 10kΩ | Diviseur de tension pour la photorésistance | ~0.5 RON |
| LED + Résistance 330Ω | Éclairage nocturne progressif | ~1.5 RON |
| Bouton poussoir tactile | Déclencheur d'urgence (Interruption matérielle) | ~1 RON |
| Breadboard 830 points | Plateforme de prototypage sans soudure | ~10 RON |
| Fils Dupont (M-M, M-F) | Connexions électriques entre les composants | ~15 RON |

### Libraries


**Pour le Raspberry Pi Pico W  :

| Library | Description | Usage |
|---------|-------------|-------|
| `Wire.h` | Bibliothèque standard I2C | Gère la communication série avec l'adaptateur de l'écran LCD. |
| `SPI.h` | Bibliothèque standard SPI | Gère la communication rapide avec le lecteur RFID. |
| `MFRC522.h` | MFRC522 by GithubCommunity | Bibliothèque complète pour l'interaction avec la puce NXP MFRC522. |
| `LiquidCrystal_I2C.h` | LCD I2C Library | Simplifie l'envoi de chaînes de caractères vers l'écran LCD. |
| `Servo.h` | Contrôle de servomoteurs | Génère le signal PWM spécifique à 50Hz pour contrôler l'angle du servomoteur SG90. |
| `ArduinoBLE.h` | Connectivité Bluetooth | Création des services et caractéristiques BLE pour écouter les commandes du PC. |

**Pour le PC Serveur / Gateway (Python) :**

| Library | Description | Usage |
|---------|-------------|-------|
| `bleak` | Bluetooth Low Energy client | Permet au PC (Windows/Linux/Mac) de scanner, se connecter et écrire sur les caractéristiques BLE du Pico W. |
| `asyncio` | Bibliothèque asynchrone | Nécessaire pour faire fonctionner `bleak` sans bloquer le reste du programme. |
| `Flask` (ou `FastAPI`) | Micro-framework Web | Hébergement de la page web locale (HTML/CSS) offrant l'interface utilisateur pour le smartphone. |



## Log
## Code
#include <SPI.h>
#include <MFRC522.h>
#include <Servo.h>
#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include <SerialBT.h> 
 
// Pini RFID (SPI)
#define RST_PIN   20 
#define SS_PIN    17 
#define MISO_PIN  16 
#define SCK_PIN   18 
#define MOSI_PIN  19 
 
#define SERVO_PIN 14 
 
#define I2C_SDA_PIN 4  
#define I2C_SCL_PIN 5  
 
#define LDR_PIN          26 // INTRARE: Senzor de lumină (Fotorezistor)
#define LED_LUMINA_PIN   15 // IEȘIRE: LED-ul de veghe (aprins la întuneric)
#define LED_URGENTA_PIN  13 // IEȘIRE: Cele 4 LED-uri pe tranzistor (doar la urgență)
#define BUZ_PIN          8  // IEȘIRE: Buzzere pe tranzistor (pentru alarme)
#define BTN_PIN          12 // INTRARE: Buton de urgență

 
MFRC522 mfrc522(SS_PIN, RST_PIN); 
Servo usaServo;
LiquidCrystal_I2C lcd(0x27, 16, 2); 

 
unsigned long timpDeschidere = 0;
const unsigned long durataDeschisa = 5000; 
bool usaEsteDeschisa = false;

unsigned long timpBuzzer = 0;
const unsigned long durataBuzzer = 3000; 
bool buzzerPornit = false;

volatile bool urgentaDeclansata = false;

 
byte cardAutorizat[4] = {0x25, 0xBF, 0x11, 0x06}; 
 
void isrButonUrgenta();
void verificaUrgenta();
void pornesteAlarma();
void verificaTimerBuzzer();
void verificaLumina();
void verificaBluetooth();
void verificaRFID();
void verificaTimerUsa();
void deblocheazaUsa();
void blocheazaUsa();
 
void setup() {
  Serial.begin(115200);
  
  // Inițializare Bluetooth
  SerialBT.setName("SecuriGate_BT"); 
  SerialBT.begin();
  delay(2000); 
   
  pinMode(LDR_PIN, INPUT);
  pinMode(LED_LUMINA_PIN, OUTPUT);
  pinMode(LED_URGENTA_PIN, OUTPUT);
  pinMode(BUZ_PIN, OUTPUT);
   
  pinMode(BTN_PIN, INPUT_PULLUP);
  attachInterrupt(digitalPinToInterrupt(BTN_PIN), isrButonUrgenta, FALLING);
 
  SPI.setRX(MISO_PIN);
  SPI.setTX(MOSI_PIN);
  SPI.setSCK(SCK_PIN);
  SPI.begin();
  mfrc522.PCD_Init();
 
  Wire.setSDA(I2C_SDA_PIN);
  Wire.setSCL(I2C_SCL_PIN);
  Wire.begin();
  lcd.init();
  lcd.backlight();
  
  usaServo.attach(SERVO_PIN);
  blocheazaUsa(); 
  
  Serial.println("Sistem Complet PORNIT!");
}

void loop() {
  verificaRFID();
  verificaTimerUsa();
  verificaLumina();     // Lumina de veghe pe timp de noapte
  verificaUrgenta();    // Monitorizare buton hardware
  verificaBluetooth();  // Comenzi de pe telefon
  verificaTimerBuzzer(); // Oprire automată a alarmei
}

void isrButonUrgenta() {
  urgentaDeclansata = true; 
}

void verificaUrgenta() {
  if (urgentaDeclansata) {
    urgentaDeclansata = false; 
    
    digitalWrite(LED_URGENTA_PIN, HIGH);
    
    pornesteAlarma(); 

    if (!usaEsteDeschisa) {
      deblocheazaUsa();
      SerialBT.println("ALERTA: Deschidere de urgenta prin buton!");
    } else {
      timpDeschidere = millis(); 
    }
  }
}

void pornesteAlarma() {
  digitalWrite(BUZ_PIN, HIGH);
  buzzerPornit = true;
  timpBuzzer = millis();
}

void verificaTimerBuzzer() {
  if (buzzerPornit && (millis() - timpBuzzer >= durataBuzzer)) {
    digitalWrite(BUZ_PIN, LOW); 
    buzzerPornit = false;
  }
}

void verificaLumina() {
  int nivelLumina = analogRead(LDR_PIN);
  int valoarePWM = map(nivelLumina, 200, 800, 255, 0);
  valoarePWM = constrain(valoarePWM, 0, 255); 
  
  analogWrite(LED_LUMINA_PIN, valoarePWM);
}

void verificaBluetooth() {
  if (SerialBT.available()) {
    char comanda = SerialBT.read();
    
    if (comanda == 'O' || comanda == 'o') { 
      if (!usaEsteDeschisa) {
        deblocheazaUsa();
        SerialBT.println("Usa deblocata prin Bluetooth.");
      } else {
        timpDeschidere = millis(); 
      }
    } 
    else if (comanda == 'C' || comanda == 'c') { 
      if (usaEsteDeschisa) {
        blocheazaUsa();
        SerialBT.println("Usa blocata fortat manual.");
      }
    }
  }
}

void verificaRFID() {
  if ( ! mfrc522.PICC_IsNewCardPresent()) return;
  if ( ! mfrc522.PICC_ReadCardSerial()) return;

  bool accesPermis = true;
  for (byte i = 0; i < 4; i++) {
    if (mfrc522.uid.uidByte[i] != cardAutorizat[i]) {
      accesPermis = false;
    }
  }

  if (accesPermis && !usaEsteDeschisa) {
    deblocheazaUsa();
    SerialBT.println("Usa deschisa local (Card Valid)."); 
  } else if (!accesPermis) {
    pornesteAlarma(); 
    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("Card Necunoscut!");
    lcd.setCursor(0, 1);
    lcd.print("Acces Respins");
    SerialBT.println("ALERTA DE SECURITATE: Card neautorizat detectat!"); 
    
    delay(1500); 
    if (!usaEsteDeschisa) blocheazaUsa(); 
  }
  mfrc522.PICC_HaltA(); 
}

void verificaTimerUsa() {
  if (usaEsteDeschisa && (millis() - timpDeschidere >= durataDeschisa)) {
    blocheazaUsa();
    SerialBT.println("Usa s-a reblocat automat.");
  }
}

void deblocheazaUsa() {
  usaServo.write(90); 
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("Acces Permis!");
  lcd.setCursor(0, 1);
  lcd.print("Usa Deschisa");
  usaEsteDeschisa = true;
  timpDeschidere = millis(); 
}

void blocheazaUsa() {
  usaServo.write(0); 
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("SecuriGate OK");
  lcd.setCursor(0, 1);
  lcd.print("Scanati Cardul");
  
  digitalWrite(LED_URGENTA_PIN, LOW);
  
  usaEsteDeschisa = false;
}

## Reference links

**Documentation officielle :**
* [Raspberry Pi Pico W - Datasheet & Documentation](https://www.raspberrypi.com/documentation/microcontrollers/raspberry-pi-pico.html)
* [MicroPython on Raspberry Pi Pico - Official Guide (PDF)](https://datasheets.raspberrypi.com/pico/raspberry-pi-pico-python-sdk.pdf)
* [Arduino Core for RP2040 (Earle F. Philhower)](https://github.com/earlephilhower/arduino-pico)

**Tutoriels Protocoles & Capteurs :**
* [Guide complet RFID RC522 avec Raspberry Pi Pico](
