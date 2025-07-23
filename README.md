# Application-5
Alexander Barrass al855321
Q1:
Structure One: Synchronization was achieved by adding mutexs to protect shared resources and used semaphores for our events e.g. Alert for when the button is pressed.
Structure Two: Converting Hardware code into tasks that will run in our RTS, Sensor, Web and more are included with this change.
Structure Three: Changing our Single loop structure into a parallel multi task system, Not only did this help create RTS task specific code, but also enabled our blocking to work.

Example Code replacement:
Original:
 void setup() {
  pinMode(LED1, OUTPUT);
  pinMode(LED2, OUTPUT);
  WiFi.begin("Wokwi-GUEST", "");
  while (WiFi.status() != WL_CONNECTED) delay(100);
  
  server.on("/", handleRoot);
  server.on("/toggle/1", []() {
    led1State = !led1State;
    digitalWrite(LED1, led1State);
    handleRoot();
  });
  server.on("/toggle/2", []() {
    led2State = !led2State;
    digitalWrite(LED2, led2State);
    handleRoot();
  });
  server.begin();
Updated:
void TaskWebServer(void *pvParameters) {
  server.on("/", handleRoot);
  server.on("/toggle/1", []() {
    xSemaphoreTake(xLedMutex, portMAX_DELAY);
    led1State = !led1State;
    digitalWrite(LED1, led1State);
    xSemaphoreGive(xLedMutex);
    handleRoot();
  });
  server.on("/toggle/2", []() {
    xSemaphoreTake(xLedMutex, portMAX_DELAY);
    led2State = !led2State;
    digitalWrite(LED2, led2State);
    xSemaphoreGive(xLedMutex);
    handleRoot();
  });
  server.begin();

  for (;;) {
    server.handleClient();
    vTaskDelay(1);
  }
}

// Heartbeat task
void TaskHeartbeat(void *pvParameters) {
  for (;;) {
    digitalWrite(LED1, !digitalRead(LED1));
    vTaskDelay(pdMS_TO_TICKS(1000));
  }
}
void setup() {
  pinMode(LED1, OUTPUT);
  pinMode(LED2, OUTPUT);
  xLedMutex = xSemaphoreCreateMutex();
  
  WiFi.begin("Wokwi-GUEST", "");
  while (WiFi.status() != WL_CONNECTED) vTaskDelay(100);

  xTaskCreate(TaskWebServer,"WebTask",8192,NULL,2,NULL);
  xTaskCreate(TaskHeartbeat,"Heartbeat",1024,NULL,1,NULL);
}

Q2: 
Arduino + FreeRTOS: 
	Advantage 1: More familiar coding suites have helped me recognize the Arduino language better at times
	Advantage 2: Wifi setup seems significantly more concise
	Disadvantage: Time away from experience with Arduino leaves me with structural coding issues like setting up some basic operations.
ESP-IDF FreeRTOS
	Advantage 1: Memory is better managed rather than with the Arduino which is not always set up with RTS in mind.
	Advantage 2: Task monitoring can be layout better for structural design, helps when operating with hardware tasks.
	Disadvantage: I personally have had a very hard time reading the coding in order to understand complex systems.
Switching would allow us to increase the accuracy of our sampling with esp_timer down to micro-second accuracy, However, losing Arduino HTTP handling will greatly increase the amount of code required to handle this based on esp wifi handling.

Q3: Queue was sized 10 with 17ms available for each one summing to 170ms, I can test overflow by reducing this number to 2, so after 3 lagging messages, the print statement I had placed in response executed. I mitigated this simply by making sure there is adequate size and if that is not adequate, a timeout precaution can be coded in.

Q4: Having time stamping serial logging integrated was a new technique that proved useful for real time detection.
Example log:
[1023] TaskSensor: Value=3120 (Heap: 12433)  
[1025] TaskWeb: HTTP Request (Heap: 7798)  
[1026] TaskAlert: Threshold crossed!!

Q5: Design choice tied in with my theming was the remote AND manual access to the alert meaning both scientist inside and outside the danger area can alert in case of trouble.
Another practical choice would be to somehow implement a connection drop alert, whether it be a signal that while updating, powers an LED so that if for whatever reason the connection is lost, scientist inside danger zone can tell and leave before continuing.

Q6: 
Advantage 1: Better UI allows our real time system to be implemented into more user friendly environments, better and clearer information gathering rather than from a console.
Advantage 2: Real time sensor monitoring meaning that as long as connection is stable enough, no missed information and less queues are created for the sake of data accumulation.

Disadvantage 1: Connection can be inadequate and create new risks when considering the environment in which our system operates, lots of lost data or priority inversion can occur when data cannot be transmitted.
Disadvantage 2: Increased stack and heap usage requires more powerful CPUs and processing power in order to operate in the speed that is often required of RTS without introducing latency and a myriad of other issues stemming from that catalyst.

An example of one of the disadvantages occurring was when I was working on this project at my home, recently the connection has been unstable with periodic drops occurring every few minutes to my dismay, it interrupted my testing capabilities for this assignment and I had to wait until a more stable connection at my work was available for me to continue. A problem that can be avoided when not interacting with wireless connections.

A wired ethernet connection if possible often solves these difficult to solve issues because of the reliability of connection.
