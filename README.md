# Robotic-Arm-with-Voice-Commands
The Robotic Arm project was upgraded using ESP32, modifying the code and adding Wi-Fi control, enabling it to execute voice commands The arm performs specific movements via voice input can be controlled remotely via a mobile app uses Servo Motors for precise joint motion and demonstrates seamless integration of speech recognition with motor control
#include <WiFi.h>
#include <ESP32Servo.h>

const char* ssid = "YOUR_WIFI_NAME";
const char* password = "YOUR_WIFI_PASSWORD";

WiFiServer server(80);

// Servo pins
#define basePin 16
#define armPin 17
#define forearmPin 2
#define gripperPin 4

Servo base, arm, forearm, gripper;

int baseAngle = 90;
int armAngle = 90;
int forearmAngle = 90;
int gripperAngle = 90;

void setup() {
  Serial.begin(9600);

  base.attach(basePin);
  arm.attach(armPin);
  forearm.attach(forearmPin);
  gripper.attach(gripperPin);

  base.write(90);
  arm.write(90);
  forearm.write(90);
  gripper.write(90);

  WiFi.begin(ssid, password);
  Serial.print("Connecting to WiFi");
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  Serial.println("\nWiFi Connected");
  Serial.print("ESP32 IP: ");
  Serial.println(WiFi.localIP());

  server.begin();
}

void loop() {
  WiFiClient client = server.available();
  if (!client) return;

  while (!client.available()) delay(1);
  String request = client.readStringUntil('\r');
  client.flush();

  Serial.println(request);

  // ====== أوامر صوتية (Siri) ======
  if (request.indexOf("/openGripper") != -1) gripperControl(180);
  if (request.indexOf("/closeGripper") != -1) gripperControl(0);
  if (request.indexOf("/armUp") != -1) armControl(180);
  if (request.indexOf("/armDown") != -1) armControl(0);
  if (request.indexOf("/baseLeft") != -1) baseControl(0);
  if (request.indexOf("/baseRight") != -1) baseControl(180);

  // ====== تحكم بالسلايدر ======
  if (request.indexOf("slider1=") != -1)
    baseControl(request.substring(request.indexOf("slider1=") + 8).toInt());

  if (request.indexOf("slider2=") != -1)
    armControl(request.substring(request.indexOf("slider2=") + 8).toInt());

  if (request.indexOf("slider3=") != -1)
    forearmControl(request.substring(request.indexOf("slider3=") + 8).toInt());

  if (request.indexOf("slider4=") != -1)
    gripperControl(request.substring(request.indexOf("slider4=") + 8).toInt());

  // ====== صفحة ويب ======
  client.println("HTTP/1.1 200 OK");
  client.println("Content-Type: text/html");
  client.println("Connection: close");
  client.println();
  client.println("<h1>ESP32 Robot Arm</h1>");
  client.println("<p>Voice control ready ✔️</p>");
}

// ================= Servo Functions =================

void baseControl(int angle) {
  int target = map(angle, 0, 180, 180, 0);
  moveServo(base, baseAngle, target);
  baseAngle = target;
}

void armControl(int angle) {
  int target = map(angle, 0, 180, 80, 180);
  moveServo(arm, armAngle, target);
  armAngle = target;
}

void forearmControl(int angle) {
  int target = map(angle, 0, 180, 30, 150);
  moveServo(forearm, forearmAngle, target);
  forearmAngle = target;
}

void gripperControl(int angle) {
  int target = map(angle, 0, 180, 85, 150);
  moveServo(gripper, gripperAngle, target);
  gripperAngle = target;
}

void moveServo(Servo &s, int from, int to) {
  if (from < to) {
    for (int i = from; i <= to; i++) {
      s.write(i);
      delay(10);
    }
  } else {
    for (int i = from; i >= to; i--) {
      s.write(i);
      delay(10);
    }
  }
}
