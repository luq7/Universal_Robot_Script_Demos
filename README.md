# Universal_Robot_Script_Demos

Universal Robot Script (UR Script) is the scripting language used command the Universal Robot Controller

---

## Hello World in UR Script

```python
def hello_world():
  textmsg("Hello, World!")

hello_world()
```

This program defines a function `hello_world()` that displays the message "Hello, World!" using the `textmsg()` function, and then calls that function to run the program.

> 💡 Note that this program assumes that you are running it on a Universal Robot controller. If you're running it on a different platform, you may need to modify it to use the appropriate display function for that platform.

---
