# Pista-C-Colores
Pista C Laberinto
#include <Wire.h>
#include "Adafruit_TCS34725.h"

Adafruit_TCS34725 colorSensor(TCS34725_INTEGRATIONTIME_50MS, TCS34725_GAIN_4X);

// Pines de motores
// Motor izquierdo
int IN1_izq = 32;
int IN2_izq = 30;
int IN3_izq = 28;
int IN4_izq = 26;
int ENA_izq = 7;
int ENB_izq = 8;

// Motor derecho
int IN1_der = 47;
int IN2_der = 29;
int IN3_der = 31;
int IN4_der = 33;
int ENA_der = 4;
int ENB_der = 3;

// LED RGB
int ledR = 13, ledG = 12, ledB = 11;

int trigFront = 38, echoFront = 39;
int trigLeft  = 40, echoLeft  = 41;
int trigRight = 37, echoRight = 36;

// Variables de color
uint16_t r, g, b, c;

// ---------------- FUNCIONES DE MOVIMIENTO ----------------
void adelante() {
  // Motor izquierdo
  digitalWrite(IN1_izq, 0); digitalWrite(IN2_izq, 1);
  digitalWrite(IN3_izq, 0); digitalWrite(IN4_izq, 1);
  analogWrite(ENA_izq, 200); analogWrite(ENB_izq, 200);

  // Motor derecho
  digitalWrite(IN1_der, HIGH); digitalWrite(IN2_der, LOW);
  digitalWrite(IN3_der, HIGH); digitalWrite(IN4_der, LOW);
  analogWrite(ENA_der, 200); analogWrite(ENB_der, 200);
}

void izquierda() {
  // Motor izquierdo retrocede
  digitalWrite(IN1_izq, LOW); digitalWrite(IN2_izq, HIGH);
  digitalWrite(IN3_izq, LOW); digitalWrite(IN4_izq, HIGH);
  analogWrite(ENA_izq, 200); analogWrite(ENB_izq, 200);

  // Motor derecho adelante
  digitalWrite(IN1_der, 0); digitalWrite(IN2_der, 1);
  digitalWrite(IN3_der, 0); digitalWrite(IN4_der, 1);
  analogWrite(ENA_der, 200); analogWrite(ENB_der, 200);
}

void derecha() {
  // Motor izquierdo adelante
  digitalWrite(IN1_izq, 0); digitalWrite(IN2_izq, 1);
  digitalWrite(IN3_izq, 0); digitalWrite(IN4_izq, 1);
  analogWrite(ENA_izq, 200); analogWrite(ENB_izq, 200);

  // Motor derecho retrocede
  digitalWrite(IN1_der, LOW); digitalWrite(IN2_der, HIGH);
  digitalWrite(IN3_der, LOW); digitalWrite(IN4_der, HIGH);
  analogWrite(ENA_der, 200); analogWrite(ENB_der, 200);
}

void parar() {
  // Motor izquierdo
  digitalWrite(IN1_izq, LOW); digitalWrite(IN2_izq, LOW);
  digitalWrite(IN3_izq, LOW); digitalWrite(IN4_izq, LOW);
  analogWrite(ENA_izq, 0); analogWrite(ENB_izq, 0);

  // Motor derecho
  digitalWrite(IN1_der, LOW); digitalWrite(IN2_der, LOW);
  digitalWrite(IN3_der, LOW); digitalWrite(IN4_der, LOW);
  analogWrite(ENA_der, 0); analogWrite(ENB_der, 0);
}

// ---------------- FUNCIONES AUXILIARES ----------------
long distancia(int trigPin, int echoPin) {
  digitalWrite(trigPin, LOW); delayMicroseconds(2);
  digitalWrite(trigPin, HIGH); delayMicroseconds(10);
  digitalWrite(trigPin, LOW);
  return pulseIn(echoPin, HIGH) * 0.034 / 2;
}

String detectarColor() {
  colorSensor.getRawData(&r, &g, &b, &c);
  if (r < 50 && g < 50 && b < 50) return "Negro";
  if (r > b && r > g) return "Rosa";
  if (g > r && g > b) return "Amarillo";
  if (b > r && b > g) return "Azul";
  return "Desconocido";
}

void mostrarColor(String color) {
  if (color == "Negro") {
    analogWrite(ledR, 0); analogWrite(ledG, 0); analogWrite(ledB, 0);
  } else if (color == "Azul") {
    analogWrite(ledR, 0); analogWrite(ledG, 0); analogWrite(ledB, 255);
  } else if (color == "Amarillo") {
    analogWrite(ledR, 255); analogWrite(ledG, 255); analogWrite(ledB, 0);
  } else if (color == "Rosa") {
    analogWrite(ledR, 255); analogWrite(ledG, 0); analogWrite(ledB, 150);
  } else {
    analogWrite(ledR, 50); analogWrite(ledG, 50); analogWrite(ledB, 50);
  }
}

// ---------------- SETUP ----------------
void setup() {
  Serial.begin(9600);
  colorSensor.begin();

  // Pines motores izquierdo
  pinMode(IN1_izq, OUTPUT); pinMode(IN2_izq, OUTPUT);
  pinMode(IN3_izq, OUTPUT); pinMode(IN4_izq, OUTPUT);
  pinMode(ENA_izq, OUTPUT); pinMode(ENB_izq, OUTPUT);

  // Pines motores derecho
  pinMode(IN1_der, OUTPUT); pinMode(IN2_der, OUTPUT);
  pinMode(IN3_der, OUTPUT); pinMode(IN4_der, OUTPUT);
  pinMode(ENA_der, OUTPUT); pinMode(ENB_der, OUTPUT);

  // LED RGB
  pinMode(ledR, OUTPUT); pinMode(ledG, OUTPUT); pinMode(ledB, OUTPUT);

  // Sensores ultrasónicos
  pinMode(trigFront, OUTPUT); pinMode(echoFront, INPUT);
  pinMode(trigLeft, OUTPUT); pinMode(echoLeft, INPUT);
  pinMode(trigRight, OUTPUT); pinMode(echoRight, INPUT);
}

// ---------------- LOOP ----------------
void loop() {
  // Leer color y mostrar en LED RGB
  String color = detectarColor();
  mostrarColor(color);
  Serial.println("Color: " + color);

  // Leer distancias
  long dFront = distancia(trigFront, echoFront);
  long dLeft  = distancia(trigLeft, echoLeft);
  long dRight = distancia(trigRight, echoRight);

  Serial.print("Front: "); Serial.print(dFront);
  Serial.print("  Left: "); Serial.print(dLeft);
  Serial.print("  Right: "); Serial.println(dRight);

  // Lógica de laberinto basada en la distancia más libre
  if(dFront >= dLeft && dFront >= dRight && dFront > 5) {
    adelante();
    Serial.println("Direccion: Adelante");
  } 
  else if(dLeft >= dFront && dLeft >= dRight && dLeft > 5) {
    izquierda();
    Serial.println("Direccion: Izquierda");
  } 
  else if(dRight >= dFront && dRight >= dLeft && dRight > 5) {
    derecha();
    Serial.println("Direccion: Derecha");
  } 
  else {
    parar();

    
  delay(300);
}
delay(500);
}
