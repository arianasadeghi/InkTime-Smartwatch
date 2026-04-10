# InkTime - Open Source Smartwatch

InkTime este un proiect de smartwatch accesibil, open-source, conceput cu un accent puternic pe eficiența energetică și autonomia bateriei. Dispozitivul este construit în jurul ecosistemului nRF52840 (oferind conectivitate Bluetooth Low Energy) și folosește un ecran E-Ink pentru a asigura o vizibilitate excelentă în lumina soarelui cu un consum de energie aproape nul în stand-by.

Acest repository conține fișierele de design hardware pentru faza **EVT (Engineering Validation Test)**.

## 1. Diagrama Bloc a Sistemului

```mermaid
graph TD
    %% Power Management
    USB[Mufă USB-C / Pogo Pins] -->|5V| CHG[IC Încărcare BQ24040]
    CHG --> BATT[Acumulator LiPo 3.7V]
    BATT --> LDO[Regulator LDO 3.3V Ultra-Low Iq]
    
    %% Core
    LDO -.->|3.3V| MCU[SoC nRF52840]
    
    %% Interfaces & Peripherals
    MCU <-->|SPI| EINK[Display E-Ink 1.54"]
    MCU <-->|I2C| IMU[Accelerometru BMA400]
    MCU <-->|I2C| HR[Senzor Puls MAX30102]
    MCU -->|GPIO| VIB[Motor Vibrații]
    MCU -->|GPIO| BTN[Butoane Navigare]
    
    %% Power lines
    LDO -.->|3.3V| EINK
    LDO -.->|3.3V| IMU
    LDO -.->|3.3V| HR
```

## 2. Bill Of Materials (BOM)

Tabelul de mai jos conține componentele principale folosite în designul plăcii. Lista completă (inclusiv rezistențe și condensatoare) se găsește în fișierul .csv din folderul Manufacturing.
Componentă	Rol în sistem	JLC Part #	Link Datasheet
nRF52840-QIAA	Microcontroller principal (BLE, Cortex-M4F)	C206001	Datasheet nRF52840
BQ24040DSQR	IC Încărcare Acumulator LiPo (1A)	C43012	Datasheet BQ24040
TPS7A0533PDQNR	LDO 3.3V (Consum Quiescent extrem de mic - 1µA)	C396492	Datasheet TPS7A05
BMA400	Accelerometru Ultra-Low Power (Pedometer)	C383215	Datasheet BMA400
MAX30102	Senzor Optic Puls și SpO2	C84666	Datasheet MAX30102
Ecran E-Ink 1.54"	Display principal (SPI)	N/A (Modul)	Datasheet Waveshare
## 3. Descrierea Funcționalității Hardware

Procesare și Conectivitate:
Inima ceasului InkTime este SoC-ul Nordic nRF52840. A fost ales datorită suportului nativ pentru Bluetooth 5.0 (BLE), esențial pentru sincronizarea notificărilor cu telefonul mobil. Arhitectura ARM Cortex-M4F permite procesarea eficientă a algoritmilor de numărare a pașilor și calcul al ritmului cardiac.

Managementul Consumului de Energie (Power Tree):
Fiind un dispozitiv wearable, constrângerile de baterie sunt critice.

    Sistemul este alimentat de o baterie LiPo de mici dimensiuni (ex. 200mAh).

    Încărcarea se face la 5V printr-un circuit BQ24040, reglat din rezistențe pentru a oferi un curent de încărcare mic și sigur (ex. 100mA).

    Pentru a coborî tensiunea la 3.3V am folosit un LDO din seria TPS7A05, care are un curent de repaus (Iq) de doar 1µA, esențial pentru a nu drena bateria când ceasul este în Deep Sleep.

Periferice și Interfețe:

    Display-ul E-Ink: Comunică prin protocol SPI. Deși are un framerate mic, este perfect pentru un ceas deoarece consumă 0mA pentru a menține imaginea afișată.

    Senzorii (IMU & HR): Comunicația se realizează pe o magistrală I2C partajată. BMA400 are un mod special de "step counter" hardware care poate trezi microcontroller-ul din sleep doar când utilizatorul face un pas, economisind masiv energia.

## 4. Alocarea Pinilor nRF52840

Microcontroller-ul nRF52840 permite maparea flexibilă a pinilor (orice funcție digitală pe orice pin), lucru care a facilitat o rutare curată (fără suprapuneri) pe PCB.
Pin nRF52840	Nume Net / Funcție	Justificare / Detalii
P0.13 / P0.14 / P0.15	SPI_SCK, SPI_MOSI, SPI_MISO	Magistrala SPI dedicată ecranului E-Ink. Pinii au fost aleși pe aceeași latură a chip-ului cu conectorul FPC al ecranului.
P0.16	EINK_CS (Chip Select)	Activare magistrală SPI pentru ecran.
P0.17 / P0.18	EINK_DC, EINK_RST	Pini de control adiționali (Data/Command și Reset) necesari controller-ului de E-Ink.
P0.26 / P0.27	I2C_SDA, I2C_SCL	Magistrala I2C pentru accelerometru (BMA400) și senzorul de puls (MAX30102). Conține rezistențe de pull-up de 4.7k.
P1.02	IMU_INT (Interrupt)	Pin setat ca Input. BMA400 trimite un semnal aici pentru a trezi SoC-ul când detectează mișcare.
P1.06	MOTOR_PWM	Pin configurat ca ieșire PWM, conectat la poarta unui tranzistor MOSFET N-Channel pentru a acționa motorul de vibrații (notificări haptice).
P0.11 / P0.12	BTN_UP, BTN_DOWN	Pini conectați la butoanele laterale, configurați cu Input Pull-Up intern.
## 5. Design Log & Integrare Mecanică (EVT Phase)

    Construcția PCB-ului: Cablajul a fost realizat pe 2 straturi. Pentru a asigura performanța antenei, a fost respectată zona de keepout cerută în datasheet-ul antenei ceramice (sau PCB trace), neavând planuri de masă sub aceasta.

    Constrângeri Mecanice: Forma PCB-ului a fost dictată strict de fișierul .step al carcasei primite. Decupajele plăcii se aliniază perfect cu pinii de montare ai carcasei.

    Stack-up 3D: Modelul 3D final validează faptul că placa de bază, bateria (poziționată sub PCB) și display-ul E-Ink (deasupra PCB-ului) intră în carcasa de smartwatch fără coliziuni de componente.