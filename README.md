# Universal_Robot_Script

---

Universal Robot Script (UR Script) is the scripting language used command the Universal Robot Controller

---

## Setup

- [How to change the console keyboard layout?](https://www.universal-robots.com/articles/ur/application-installation/changing-the-console-keyboard-layout/)
  - E.g.
    1. Once in `Select keymap from arch list`
    2. choose `Generic 104 Keyboard`
    3. Language: `English(US)`
    4. `English(US)`
    5. `The default setting`
    6. `Yes to Ctrl + Alt + Backspace` to terminate controller.
    7. `Ok`
    8. `Save`

---

# Common issues

- When encountered `Not able to open socket [socket_name] to host: [Ip address ] at port: [PORT number]: Connection timed out`, check FIREWALL policy when a socket connection can't be established between the UR control box and the remote host. The remote host might have an inbound traffic blocked from unknow/unauthorized IP address.
- When encountered error `C210A0: Socket is read-only when the robot is in local (Teach pendant) control`, go to your Tach Pendant and change your mode to `Remote` instead of `Local`. Then, re-run to your remote host program (because the socket connection established during the Pendant is being in `local` mode would get lost as we switch to `Remote` mode. So it needs to create the socket connection once again.)

---

# Demos

## Hello World in UR Script

```python
def hello_world():
  textmsg("Hello, World!")
  sleep(3)
end
hello_world()
```

This program defines a function `hello_world()` that displays the message "Hello, World!" using the `textmsg()` function, and then calls that function to run the program.

> 💡 Note that this program assumes that you are running it on a Universal Robot controller. If you're running it on a different platform, you may need to modify it to use the appropriate display function for that platform.

## Read position in UR Script

```python

def read_pose():
  s=get_actual_tcp_pose()
  textmsg(s)
  sleep(3)
end

read_pose()
```

This program defines a function `read_pose()` that reads in the current position of the cobot, and print that position to the log.

## How to get tcp pose in feature coordinates?

To get the TCP pose in feature coordinates in Universal Robot Script, you can use the `pTq()` function to transform the TCP pose from the robot's base coordinates to the feature coordinates. Here is an example program that demonstrates how to do this:

```python
def get_tcp_pose_in_feature_coords(feature_pose):
  # Get the current TCP pose in the robot's base coordinates
  tcp_pose = get_actual_tcp_pose()

  # Define the feature coordinates
  feature_origin = [0.5, 0.5, 0.0]
  feature_x_axis = [1.0, 0.0, 0.0]
  feature_y_axis = [0.0, 1.0, 0.0]

  # Calculate the transformation matrix from the robot's base coordinates to the feature coordinates
  T_base_to_feature = pTq(feature_origin, feature_x_axis, feature_y_axis, [0,0,0], [0,0,0])

  # Apply the transformation to the TCP pose to get the TCP pose in the feature coordinates
  tcp_pose_in_feature_coords = T_base_to_feature * tcp_pose

  return tcp_pose_in_feature_coords
```

In this program, `get_actual_tcp_pose(`) is used to get the current TCP pose in the robot's base coordinates. The feature coordinates are defined using an origin point (`feature_origin`), an x-axis vector (`feature_x_axis`), and a y-axis vector (`feature_y_axis`). The `pTq()` function is used to calculate the transformation matrix `T_base_to_feature` from the robot's base coordinates to the feature coordinates. Finally, this transformation is applied to the TCP pose using matrix multiplication to get the TCP pose in the feature coordinates.

> 💡Note that this program assumes that the feature coordinates are fixed relative to the robot's base coordinates. If the feature coordinates move relative to the robot, the transformation matrix will need to be recalculated each time the feature moves.

## How to move the UR robot in a custom defined plane instead of the actual base?

To move the UR robot in a custom defined plane instead of the actual base, you can use the `set_tcp()` function to define a custom tool center point (TCP) for the robot. The TCP defines the origin of a new coordinate system that the robot will use for all its movements. By setting the TCP to a point that lies in a custom plane, you can move the robot within that plane.

Here is an example program that demonstrates how to define a custom TCP and move the robot within a custom plane:

```python

# Define the origin and orientation of the custom TCP
tcp_origin = [0.5, 0.5, 0.0]
tcp_x_axis = [1.0, 0.0, 0.0]
tcp_y_axis = [0.0, 1.0, 0.0]

# Set the custom TCP
set_tcp(tcp_origin, tcp_x_axis, tcp_y_axis)

# Move the robot within the custom plane
movej(p[0.5, 0.6, 0.0, 0, 3.14, 0], a=1.2, v=0.5)
movej(p[0.5, 0.7, 0.0, 0, 3.14, 0], a=1.2, v=0.5)
movej(p[0.5, 0.8, 0.0, 0, 3.14, 0], a=1.2, v=0.5)

# Reset the TCP to the robot's base
set_tcp(p[0, 0, 0, 0, 0, 0])

```

In this program, `tcp_origin`, `tcp_x_axis`, and `tcp_y_axis` define the origin and orientation of the custom TCP. These vectors are specified in the robot's base coordinates. The `set_tcp()` function is used to set the custom TCP for the robot.

After setting the TCP, the `movej()` function is used to move the robot within the custom plane. The `p[]` function is used to specify the target positions for each movement. These positions are specified relative to the custom TCP, so the robot will move within the plane defined by the TCP.

Finally, the `set_tcp()` function is called again to reset the TCP to the robot's base coordinates.

> 💡Note that using a custom TCP can affect the accuracy of the robot's movements, since it introduces additional sources of error. If high precision is required, it may be necessary to calibrate the custom TCP using a calibration tool.

## Openning up a socket connection

### Client side which is the UR robot

```python

global PC_IP = ""
global PORT = 30033
global SOCKET_NAME = "socket_0"
def before_start():
  socket = socket_open(PC_IP,PORT,SOCKET_NAME)
  textmsg("socket opened")
end

def robot_program():
  socket = False
  while socket == False:
    socket = socket_open(PC_IP,PORT)
    Wait(0.5)
  socket_send_string("Successfully connected",SOCKET_NAME)
  Wait(0.5)
  content = socket_read_string()
  textmsg("content=",content)
end
```

### Server side which is whatever remote PC trying to talk to the UR robot

> ⚠️ Don't forget to turn off the Firewall, or add a new firewall policy so that
> the UR ip address and the dedicated port could reach the remote PC

```python
import socket
import sys
import time

# Define the host and port to listen on
HOST = ''  # Listen on all available network interfaces
PORT = 30033  # The same port as used by the robot controller

# Create a socket object
server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

# Bind the socket to a specific address and port
server_socket.bind((HOST, PORT))

# Listen for incoming connections
server_socket.listen(1)

# Wait for a client to connect
print('Waiting for a connection...')
client_socket, client_address = server_socket.accept()
print('Connected by', client_address)

# Send a message to the client
message = 'Hello, client!'
client_socket.sendall(message.encode())

# Receive data from the client
data = client_socket.recv(1024)

# Print the received data
print('Received:', data.decode())

# Close the connection
client_socket.close()
server_socket.close()

```

## Open connection via RTDE

### Python script:

```python

from rtde import RTDE

# establish RTDE connection
ROBOT_HOST = '192.168.1.100'  # replace with robot's IP address
ROBOT_PORT = 30004           # replace with robot's RTDE port
rtde = RTDE(ROBOT_HOST, ROBOT_PORT)
rtde.connect()

# send command to robot
command = "movej([0, 0, 0, 0, 0, 0], a=1.2, v=0.5)\n"
rtde.send(command)

# receive response from robot
data = rtde.receive()
print('Received:', data)

# close RTDE connection
rtde.disconnect()

```

### URScript program:

```python
rtde = rtde_init("192.168.1.100")

while True:
    data = rtde_receive()
    if data:
        # execute command
        exec(data)
        # send response
        rtde_send("Command executed successfully")
    else:
        break
end

rtde_stop()

```

---

## Official docu

- [REMOTE CONTROL VIA TCP/IP](https://www.universal-robots.com/articles/ur/interface-communication/remote-control-via-tcpip/)
- [REAL-TIME DATA EXCHANGE (RTDE) GUIDE](https://www.universal-robots.com/articles/ur/interface-communication/real-time-data-exchange-rtde-guide/)
- [ Remote Operation Guide](https://www.universal-robots.com/articles/ur/interface-communication/remote-operation-of-robots/)

---

## Third-party documentation.

- [Getting started - by Zacobria](https://www.zacobria.com/universal-robots-knowledge-base-tech-support-forum-hints-tips-cb2-cb3/index.php/ur-script-script-programming-from-the-teaching-pendant/)
  - _'It is possible to have the entire program as a script file... so the tree structure will only have one statement.'_
  - _'A script: statement where the entire program is in a script file and thereby loaded into the robot'_
- [UR Script: Send Commands from host (PC) to robot via socket connection.
  ](https://www.zacobria.com/universal-robots-knowledge-base-tech-support-forum-hints-tips-cb2-cb3/index.php/ur-script-send-commands-from-host-pc-to-robot-via-socket-connection/)
  - _On the robot a server on TCP port 30002 is running to receive script commands and thereby control the robot._
  - _this method the data flow is sort of “one way” i.e. it is possible to send commands to the robot, but it is not meant to read data back i.e. inputs or other read data cannot be send back to the host with this method. To send back data see the <a href="#client_server">“Client-Server” </a>example._
- <a id="client_server">[Universal-Robots Script **Client-Server example**.
  ](https://www.zacobria.com/universal-robots-knowledge-base-tech-support-forum-hints-tips-cb2-cb3/index.php/universal-robots-script-client-server-example/)</a>
- [UR Script Fieldbus input disconnected](https://forum.universal-robots.com/t/protective-stopfieldbus-input-disconnected/320)
