# Parcial2_spd_2023
<h2> Proyecto : Sistema de incendio con Arduino</h2>
<img src="./Img/Arduino.png" ><br>
<h2> Descripción</h2>
<p>EL proyecto es un modelo de un sistema de incendio utilizando Arduino que
detecta cambios de temperatura y activa un servo motor en caso de detectar un incendio.
Además, se mostrará la temperatura actual y la estación del año en un display LCD.</p>
<h2> Funcionamiento</h2>
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
<img src="./Img/MonitorSerial.png" ><br>
<p>Cuando el montacargas este subiendo, bajando o en pausa se informará en el monito Serial</p><br>
<h2> Funciones </h2>
<ul>
<li>La función display() se utiliza para controlar los segmentos del display de siete segmentos. Recibe como argumentos los valores de los segmentos (0 o 1) y utiliza la función digitalWrite() para establecer los valores correspondientes en los pines de los segmentos.</li><br>

<li>La función visualizacion() recibe un número como argumento y muestra ese número en el display de siete segmentos llamando a la función display() con los valores de segmentos correspondientes al número.</li>

<li>void subirMontacargas(): Esta función se encarga de subir el montacargas al siguiente nivel. Incrementa la variable nivel en uno y luego verifica si el nivel supera el valor máximo permitido (9). Si es así, establece nivel en 9, muestra el número 9 en el display llamando a la función visualizacion(), imprime un mensaje en la comunicación serial indicando que solo se puede subir hasta el nivel 9. En caso contrario, muestra el número actual del nivel en el display, enciende el LED verde utilizando digitalWrite(), imprime el número del nivel actual en la comunicación serial, espera 3 segundos utilizando delay(), apaga los segmentos del display llamando a display() con valores de segmento apagados, y finalmente apaga el LED verde.</li>
 
<li>void bajarMontacargas(): Esta función se encarga de bajar el montacargas al nivel anterior. Decrementa la variable nivel en uno y luego verifica si el nivel es menor que cero. Si es así, establece nivel en 0, muestra el número 0 en el display llamando a la función visualizacion(), e imprime un mensaje en la comunicación serial indicando que solo se puede bajar hasta el nivel 0. En caso contrario, muestra el número actual del nivel en el display, enciende el LED verde utilizando digitalWrite(), imprime el número del nivel actual en la comunicación serial, espera 3 segundos utilizando delay(), apaga los segmentos del display llamando a display() con valores de segmento apagados, y finalmente apaga el LED verde.</li><br>
<img src="./Img/LedVerde.png" ><br>
  
  <p>Así se ve cuando se encuentra activado</p>
<li>void pausarMontacargas(int estadoDelLed): Esta función se encarga de pausar o reanudar el montacargas. Recibe el estado del LED rojo como argumento (estadoDelLed). Si estadoDelLed es igual a 1, se enciende el LED rojo utilizando digitalWrite(), se imprime un mensaje en la comunicación serial indicando que el montacargas está en pausa, y se establece flagPausa en TRUE. Si estadoDelLed es diferente de 1, se apaga el LED rojo, se imprime un mensaje en la comunicación serial indicando que el montacargas está activo, y se establece flagPausa en FALSE. Esta variable flagPausa se utiliza para controlar si se permite o no el movimiento del montacargas en las funciones de subir y bajar, evitando que se mueva mientras está en pausa.</li><br>
<img src="./Img/LedRojo.png" ><br>

  <p>Así se ve cuando se encuentra desactivado</p>
</ul>
<h2> Link del proyecto </h2>
[Parcial 1 SPD](https://www.tinkercad.com/things/gzPuxQ7PXj4)

