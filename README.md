[![Review Assignment Due Date](https://classroom.github.com/assets/deadline-readme-button-22041afd0340ce965d47ae6ef1cefeee28c7c493a6346c4f15d667ab976d596c.svg)](https://classroom.github.com/a/MCJunYEq)
[![Open in Visual Studio Code](https://classroom.github.com/assets/open-in-vscode-2e0aaae1b6195c2367325f4f02e2d04e9abb55f0b24a779b69b11b9e10269abc.svg)](https://classroom.github.com/online_ide?assignment_repo_id=23741483&assignment_repo_type=AssignmentRepo)
# Lab06: Comunicación UART con PIC18F45K22

## Integrantes
* [Laura Alejandra Florian Orjuela](https://github.com/Laura-Florian)
* [Daniel Penagos Castro](https://github.com/Daniel-Penagos)
* [Juan de Jesus Calderon Comas](https://github.com/JuanCalderon22)
## Documentación
### Codigos 

### ◘ uart.h

#ifndef UART_H   // Si no está definido UART_H, entra

#define UART_H   // Define UART_H para evitar inclusión múltiple

#include <xc.h>  // Incluye cabecera específica del compilador XC8 para los PIC

#include <stdint.h> // Para tipos como uint16_t, etc.

#define _XTAL_FREQ 16000000UL    // Define la frecuencia del cristal/reloj de 16 MHz para usar __delay_ms y __delay_us

void UART_Init(void);    // Prototipo de función para inicializar UART

void UART_WriteChar(char data); // Envía un carácter por UART

void UART_WriteString(const char* str); // Envía una cadena

#endif  // Fin del guarda de inclusión

### ◘ uart.c

#include "uart.h"   // Incluye la cabecera con las declaraciones

#include <stdio.h>  // Para sprintf, aunque en este caso se usa en main.c, aquí no es necesario pero se incluye por si acaso


void UART_Init(void) {
    // Configura pines RC6 como salida (TX) y RC7 como entrada (RX)

    TRISC6 = 0;   // TRISC6 = 0 => pin RC6 como salida (TX)

    TRISC7 = 1;   // TRISC7 = 1 => pin RC7 como

    entrada (RX)  // Configuración del baudrate para UART1

    SPBRG1 = 25;   // Valor para generar 9600 baudios a 16 MHz (con BRGH=0 y BRG16=0)

    TXSTA1bits.BRGH = 0;   // BRGH = 0 => baja velocidad (modo asíncrono de baja velocidad)

    BAUDCON1bits.BRG16 = 0; // BRG16 = 0 => registro SPBRG de 8 bits

    RCSTA1bits.SPEN = 1;    // SPEN = 1 => habilita el puerto serie (UART)

    TXSTA1bits.SYNC = 0;    // SYNC = 0 => modo asíncrono

    TXSTA1bits.TXEN = 1;    // TXEN = 1 => habilita transmisión

    RCSTA1bits.CREN = 1;    // CREN = 1 => habilita recepción continua      // Configuración de interrupciones para recepción

    PIE1bits.RC1IE = 1;    // Habilita interrupción por recepción de UART1

    PIR1bits.RC1IF = 0;    // Limpia bandera de interrupción de recepción

    INTCONbits.PEIE = 1;   // Habilita interrupciones de periféricos

    INTCONbits.GIE = 1;    // Habilita interrupciones globales
}

void UART_WriteChar(char data) {
    // Espera hasta que el registro de transmisión esté vacío (TRMT=1)
    
    while (!TXSTA1bits.TRMT);   // TRMT es 1 cuando el buffer de transmisión está vacío (y el shift register también)

    TXREG1 = data;   // Carga el dato en el registro de transmisión, se envía
}

void UART_WriteString(const char* str) {
    // Mientras no se llegue al carácter nulo '\0'
    while (*str)
    
     {
        UART_WriteChar(*str++);   // Envía el carácter actual y avanza el puntero
    }
}
### ◘ main.c

#include <xc.h>      // Cabecera del compilador para PIC

#include <stdio.h>   // Para sprintf

#include <stdint.h>  // Para uint16_t, uint32_t

#include "uart.h"    // Para funciones UART
// Configuraciones de los fuse bits (configuración del microcontrolador)

#pragma config FOSC = INTIO67   // Oscilador interno, puertos RA6 y RA7 como I/O

#pragma config WDTEN = OFF      // Watchdog timer deshabilitado

#pragma config LVP = OFF        // Programación de bajo voltaje deshabilitada

void main(void) {
    // Configuración del oscilador interno a 16 MHz (según el datasheet del PIC)

    OSCCON = 0b01110000;   // bits: IRCF<2:0>=111 => 16 MHz, SCS=0 => oscilador primario, etc. 

    
    UART_Init();   // Inicializa UART con baudios 9600 (asumiendo 16 MHz)

    
    uint16_t contador = 0;   // Variable de 16 bits que simula un valor de ADC de 0 a 1023

    
    while(1) {   // Bucle infinito
        // Convierte el contador (0-1023) a un voltaje en mV suponiendo referencia de 5V y ADC de 10 bits (1024 pasos)

        uint32_t voltaje_mV = ((uint32_t)contador * 5000) / 1023;
        // Extrae la parte entera en voltios (dividiendo por 1000)
        uint16_t entero  = voltaje_mV / 1000;

        // Extrae los dos primeros dígitos decimales (centésimas de voltio). El residuo %1000 da mV restantes, /10 da centésimas de voltio.
        uint16_t decimal = (voltaje_mV % 1000) / 10;

        
        char buffer[30];   // Buffer para construir el string

        sprintf(buffer, "Voltage: %d.%02d\r\n", entero, decimal);  // Formatea: "Voltage: X.YY" con retorno de carro y salto de línea

        UART_WriteString(buffer);   // Envía la cadena por UART
        
        contador += 50;    // Incrementa el contador de 50 en 50

        if (contador > 1023) {  // Si supera 1023, vuelve a 0

            contador = 0;
        }
        
        __delay_ms(100);   // Espera 100 ms (depende de _XTAL_FREQ definido en uart.h)
    }
}
## ◘ Script de Python

#Importa la librería para comunicación serie (pySerial)
import serial

#Importa el módulo pyplot de matplotlib para hacer gráficos
import matplotlib.pyplot as plt

#Importa el submódulo de animación para actualizar el gráfico en tiempo real
import matplotlib.animation as animation

#Importa expresiones regulares para buscar patrones en el texto recibido
import re

#Importa deque (cola de doble extremo) para mantener un tamaño fijo automático
from collections import deque


#Define el puerto serie donde está conectado el PIC (en Linux suele ser /dev/ttyUSB0)
SERIAL_PORT = '/dev/ttyUSB0'

#Velocidad de comunicación, debe coincidir con la configurada en el PIC (9600 baudios)
BAUDRATE = 9600

#Número máximo de puntos que se mostrarán en el gráfico
MAX_POINTS = 100


#Abre el puerto serie con los parámetros dados; timeout=1 significa esperar máximo 1 segundo por datos
ser = serial.Serial(SERIAL_PORT, BAUDRATE, timeout=1)


#Crea una cola circular para almacenar los últimos MAX_POINTS voltajes
voltages = deque(maxlen=MAX_POINTS)

#Crea una cola circular para almacenar los últimos tiempos (número de muestra)
times = deque(maxlen=MAX_POINTS)

#Contador de muestras, se incrementa cada vez que se recibe un dato válido
time_counter = 0


#Compila una expresión regular que busca "Voltaje:" seguido de espacios y un número decimal
#El número se captura en un grupo (entre paréntesis)
regex = re.compile(r"Voltaje:\s*([0-9.]+)")


#Función que se llama periódicamente para actualizar la gráfica
#El parámetro 'frame' es obligatorio para FuncAnimation, pero no se usa
def update(frame):
    # Indica que vamos a modificar la variable global time_counter
    global time_counter

    # Lee una línea del puerto serie, la decodifica de bytes a string y elimina espacios/saltos de línea
    line = ser.readline().decode('utf-8').strip()

    # Busca en la línea el patrón definido en 'regex'
    match = regex.search(line)

    # Si se encontró el patrón (es decir, la línea contiene algo como "Voltaje: 2.34")
    if match:
        # Extrae el primer grupo capturado ( el número como string ) y lo convierte a float
        voltage = float(match.group(1))

        # Añade el voltaje a la cola circular (automáticamente elimina el más viejo si supera MAX_POINTS)
        voltages.append(voltage)

        # Añade el contador de tiempo actual a la cola de tiempos
        times.append(time_counter)

        # Incrementa el contador de tiempo para la próxima muestra
        time_counter += 1

        # Borra todo el contenido del eje (limpia el gráfico anterior)
        ax.clear()

        # Dibuja los tiempos en X y los voltajes en Y, con línea azul
        ax.plot(times, voltages, color='blue')

        # Fija los límites del eje Y de 0 a 5 voltios (asumiendo rango 0-5V)
        ax.set_ylim(0, 5)

        # Establece el título del gráfico
        ax.set_title("Voltaje leído por UART")

        # Etiqueta del eje X
        ax.set_xlabel("Tiempo (s)")

        # Etiqueta del eje Y
        ax.set_ylabel("Voltaje (V)")

        # Muestra una cuadrícula de fondo para facilitar la lectura
        ax.grid(True)


#Crea una figura y un objeto de ejes para el gráfico
fig, ax = plt.subplots()

#Crea una animación que llama a la función 'update' cada 1000 milisegundos (1 segundo)
#cache_frame_data=False evita almacenar datos innecesarios de frames
ani = animation.FuncAnimation(fig, update, interval=1000, cache_frame_data=False)

#Ajusta automáticamente los márgenes de la figura para que todo encaje bien
plt.tight_layout()

#Muestra la ventana con el gráfico en pantalla; el programa se queda aquí hasta cerrar la ventana
plt.show()

## Diagramas

## Evidencias de implementación