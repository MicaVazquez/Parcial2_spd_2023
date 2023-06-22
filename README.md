# Parcial2_spd_2023
<h2> Proyecto : Sistema de incendio con Arduino</h2>

<img src="./img/sist_incendio_activado.png" ><br>
## Descripcion
El proyecto es un modelo de un sistema de incendio utilizando Arduino que
detecta cambios de temperatura y activa un servo motor en caso de detectar un incendio.
Además, se mostrará la temperatura actual y la estación del año en un display LCD.

## Funcionamiento
<p>    </p><br>

```bash
#include <IRremote.h> 
//permite recibir y transmitir señales de infrarrojos
#include <LiquidCrystal.h>
//funciones para escribir texto, números y controlar el cursor en la pantalla LCD
#include <Servo.h>
//para controlar servomotores

//SERVO
#define LED_ROJO 5
#define LED_VERDE 4
#define SENSOR A0
#define UMBRAL_INCENDIO 60
#define PIN_SERVO 3

int leds[] = {4,5};
Servo servo;//creo obj servo
int posicionServo;
bool direccionServo = false;

//LCD
#define RS 12
#define E  13
#define D4 6
#define D5 7
#define D6 8
#define D7 9
LiquidCrystal miLcd(RS,E,D4,D5,D6,D7);//creo obj de la clase y asigno pines
int actualizarLcd=1;
char estaciones[][10] = {"Invierno", "Otonio", "Primavera", "Verano","INCENDIO","         "};

//CONTROL REMOTO
#define Tecla_1 0xEF10BF00
#define Tecla_2 0xEE11BF00
//la biblioteca IRremote, que proporciona las funciones necesarias para recibir y decodificar señales infrarrojas
int IR = 11;// el receptor infrarrojo está conectado al pin 11 de la placa Arduino.
bool sistemaActivado = false;


void setup()
{
  for(int i=0;i<2;i++)
  {
    pinMode(leds[i], OUTPUT);//configuro los pines de salida
  }
  servo.attach(PIN_SERVO);//adjunto obj servo al pin
  
  miLcd.begin(16,2);//inicializa la pantalla LCD (16c x 2f)
  miLcd.setCursor(0,0);//posición del cursor- columna y fila
  
  IrReceiver.begin(IR, DISABLE_LED_FEEDBACK);//configura el ri  para recibir señales en el pin 11 y desactiva el LED de retroalimentación (DISABLE_LED_FEEDBACK).
}

void loop()
{
  manejarEntradaControlRemoto();
  
  if(sistemaActivado)
  {
    int temperatura = verificarTemperatura();

    if(temperatura > UMBRAL_INCENDIO)//cuando la temperatura sobrepase el umbral 
    {
      establecerEstadoLeds(1, 0);
      activarServo();
    }
    else 
    {
      establecerEstadoLeds(0, 1);
      desactivarServo();
    }

    if(actualizarLcd)
      printMenu();
      miLcd.setCursor(12,0);//columna y fila
      miLcd.print("   ");
      miLcd.setCursor(12,0);
      miLcd.print(temperatura);
      miLcd.setCursor(10,1);
      printEstacion();
      delay(100);
  }
  else if(sistemaActivado == false)
  {
    establecerEstadoLeds(0, 0);
    desactivarServo();
    miLcd.clear();
  }
}

//FUNCIONES
void activarServo()
{
 
  if (posicionServo < 180 && !direccionServo)
    {
      posicionServo++;
    }
    else if (posicionServo > 0 || direccionServo)
    {
      direccionServo = true;
      posicionServo--;
      if (posicionServo <= 0)
      {
        direccionServo = false;
        posicionServo = 0;
      }
    }

    servo.write(posicionServo);
    delay(50);
}

void desactivarServo()
{
  servo.write(posicionServo);  
  delay(35);      
}

void printMenu(){
  actualizarLcd=0;
  miLcd.clear();
  miLcd.print("Temperatura:");
  miLcd.setCursor(15,0);
  miLcd.print("C");
}
void printEstacion()
{
  int temperatura = verificarTemperatura();
  miLcd.setCursor(0,1);
  miLcd.print(estaciones[5]);
  miLcd.setCursor(0,1);
  
  
  if(temperatura > 60)//INCENDIO +60
  {
    miLcd.print(estaciones[4]);
  }
  
   else if(temperatura > 40)//40-60
  {
    miLcd.print(estaciones[3]);
  }
  
   else if(temperatura > 20)//20-40
  {
    miLcd.print(estaciones[2]);
  }
  
  else if(temperatura >= 5)//5-20
  {
    miLcd.print(estaciones[1]);
  }
  
  else if(temperatura < 5)//-40-5
  {
    miLcd.print(estaciones[0]);
  }
 
}

int verificarTemperatura()
{
  int lecturaSensor = analogRead(SENSOR);
  int temperatura;
  
  temperatura = map(lecturaSensor,20,358,-40,125);
  //mapear el valor leído del sensor (lecturaSensor) de un rango a otro. 
  return temperatura;
}

void establecerEstadoLeds(int estadoRojo, int estadoVerde)
{
  digitalWrite(LED_ROJO, estadoRojo);
  digitalWrite(LED_VERDE, estadoVerde);
}

void manejarEntradaControlRemoto()
{
  if (IrReceiver.decode()) //si se ha decodificado una señal infrarroja
  {
    unsigned long valor = IrReceiver.decodedIRData.decodedRawData;//asigna el valor de la señal infrarroja decodificada 
    
    if (valor == Tecla_1) 
    {
      sistemaActivado = true;
      actualizarLcd=1;
      
    } 
    else if (valor == Tecla_2) 
    {
      sistemaActivado = false;
    }
    
    IrReceiver.resume();//seguir recibiendo señales
  }

}
```


## Funciones


## Link del proyecto
[Parcial 2 SPD](https://www.tinkercad.com/things/jMK0zgReULG)

