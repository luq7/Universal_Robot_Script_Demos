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

# Demos

## Hello World in UR Script

```python
def hello_world():
  textmsg("Hello, World!")
end
hello_world()
```

This program defines a function `hello_world()` that displays the message "Hello, World!" using the `textmsg()` function, and then calls that function to run the program.

> 💡 Note that this program assumes that you are running it on a Universal Robot controller. If you're running it on a different platform, you may need to modify it to use the appropriate display function for that platform.

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
