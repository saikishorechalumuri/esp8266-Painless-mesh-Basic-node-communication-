# esp8266-Painless-mesh-Basic-node-communication-
Learn how to use ESP-MESH networking protocol to build a mesh network with the ESP32 and ESP8266 NodeMCU boards. ESP-MESH allows multiple devices (nodes) to communicate with each other under a single wireless local area network.
It is supported on the ESP32 and ESP8266 boards.
In this guide, we’ll show you how to get started with ESP-MESH using the Arduino core

![image](https://user-images.githubusercontent.com/93335682/171430790-58bdc526-73c0-49bd-bae5-6a53eab5848d.png)


ESP-MESH Network Architecture
With ESP-MESH, the nodes don’t need to connect to a central node. Nodes are responsible for relaying each others transmissions. 
This allows multiple devices to spread over a large physical area.
The Nodes can self-organize and dynamically talk to each other to ensure that the packet reaches its final node destination.
If any node is removed from the network, it is able to self-organize to make sure that the packets reach their destination..
![image](https://user-images.githubusercontent.com/93335682/171431009-936c2999-8d8a-4363-8be9-21cffd89b449.png)

the required esstential libraries 
If this window doesn’t show up, you’ll need to install the following library dependencies:

ArduinoJson (by bblanchon)
TaskScheduler
ESPAsyncTCP (ESP8266)
AsyncTCP (ESP32)



Main image

![image](https://user-images.githubusercontent.com/93335682/171431314-e63f073b-fdda-44e0-a81e-2cff6f7223bf.png)






DESCRIPTION OF THE CODE  IN DETAIL 



Before uploading the code, you can set up the MESH_PREFIX (it’s like the name of the MESH network) and the MESH_PASSWORD variables (you can set it to whatever you like).

Then, we recommend that you change the following line for each board to easily identify the node that sent the message. For example, for node 1, change the message as follows:

String msg = "Hi from node 1 ";
How the Code Works
Start by including the painlessMesh library.

#include "painlessMesh.h"
MESH Details
Then, add the mesh details. The MESH_PREFIX refers to the name of the mesh. You can change it to whatever you like.

#define MESH_PREFIX "whateverYouLike"
The MESH_PASSWORD, as the name suggests is the mesh password. You can change it to whatever you like.

#define MESH_PASSWORD "somethingSneaky"

All nodes in the mesh should use the same MESH_PREFIX and MESH_PASSWORD.

The MESH_PORT refers to the the TCP port that you want the mesh server to run on. The default is 5555.

#define MESH_PORT 5555
Scheduler
It is recommended to avoid using delay() in the mesh network code. To maintain the mesh, some tasks need to be performed in the background. Using delay() will stop these tasks from happening and can cause the mesh to lose stability/fall apart.

Instead, it is recommended to use TaskScheduler to run your tasks which is used in painlessMesh itself.

The following line creates a new Scheduler called userScheduler.

Scheduler userScheduler; // to control your personal task
painlessMesh
Create a painlessMesh object called mesh to handle the mesh network.

Create tasks
Create a task called taskSendMessage responsible for calling the sendMessage() function every second as long as the program is running.

Task taskSendMessage(TASK_SECOND * 1 , TASK_FOREVER, &sendMessage);
Send a Message to the Mesh
The sendMessage() function sends a message to all nodes in the message network (broadcast).

void sendMessage() {
  String msg = "Hi from node 1";
  msg += mesh.getNodeId();
  mesh.sendBroadcast( msg );
  taskSendMessage.setInterval(random(TASK_SECOND * 1, TASK_SECOND * 5));
}
The message contains the “Hi from node 1” text followed by the board chip ID.

String msg = "Hi from node 1";
msg += mesh.getNodeId();
To broadcast a message, simply use the sendBroadcast() method on the mesh object and pass as argument the message (msg) you want to send.

mesh.sendBroadcast(msg);

Every time a new message is sent, the code changes the interval between messages (one to five seconds).

taskSendMessage.setInterval(random(TASK_SECOND * 1, TASK_SECOND * 5));
Mesh Callback Functions
Next, several callback functions are created that will be called when specific events happen on the mesh.

The receivedCallback() function prints the message sender (from) and the content of the message (msg.c_str()).

void receivedCallback( uint32_t from, String &msg ) {
  Serial.printf("startHere: Received from %u msg=%s\n", from, msg.c_str());
}
The newConnectionCallback() function runs whenever a new node joins the network. This function simply prints the chip ID of the new node. You can modify the function to do any other task.

void newConnectionCallback(uint32_t nodeId) {
  Serial.printf("--> startHere: New Connection, nodeId = %u\n", nodeId);
}
The changedConnectionCallback() function runs whenever a connection changes on the network (when a node joins or leaves the network).

void changedConnectionCallback() {
  Serial.printf("Changed connections\n");
}
The nodeTimeAdjustedCallback() function runs when the network adjusts the time, so that all nodes are synchronized. It prints the offset.

void nodeTimeAdjustedCallback(int32_t offset) {
  Serial.printf("Adjusted time %u. Offset = %d\n", mesh.getNodeTime(),offset);
}
setup()
In the setup(), initialize the serial monitor.

void setup() {
  Serial.begin(115200);

Choose the desired debug message types:

//mesh.setDebugMsgTypes( ERROR | MESH_STATUS | CONNECTION | SYNC | COMMUNICATION | GENERAL | MSG_TYPES | REMOTE ); // all types on

mesh.setDebugMsgTypes( ERROR | STARTUP );  // set before init() so that you can see startup messages
Initialize the mesh with the details defined earlier.

mesh.init(MESH_PREFIX, MESH_PASSWORD, &userScheduler, MESH_PORT);
Assign all the callback functions to their corresponding events.

mesh.onReceive(&receivedCallback);
mesh.onNewConnection(&newConnectionCallback);
mesh.onChangedConnections(&changedConnectionCallback);
mesh.onNodeTimeAdjusted(&nodeTimeAdjustedCallback);
Finally, add the taskSendMessage function to the userScheduler. The scheduler is responsible for handling and running the tasks at the right time.

userScheduler.addTask(taskSendMessage);
Finally, enable the taskSendMessage, so that the program starts sending the messages to the mesh.

taskSendMessage.enable();
To keep the mesh running, add mesh.update() to the loop().

void loop() {
  // it will run the user scheduler as well
  mesh.update();
}
Demonstration
Upload the code provided to all your boards. Don’t forget to modify the message to easily identify the sender node

With the boards connected to your computer, open a serial connection with each board. You can use the Serial Monitor, or you can use a software like PuTTY and open multiple windows for all the boards.

You should see that all boards receive each others messages. For example, these are the messages received by Node 1. It receives the messages from Node 2, 3 and 4.


REFERED ARTICLE :https://randomnerdtutorials.com/esp-mesh-esp32-esp8266-painlessmesh/

