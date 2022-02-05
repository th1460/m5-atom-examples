# M5stack Atom Examples

## Install firmware

```
# Install esptool
pip3 install esptool

# Erase device
esptool.py --port /dev/ttyUSB0 erase_flash

# Install firmware
esptool.py --chip esp32 --port /dev/ttyUSB0 --baud 750000 write_flash -z 0x1000 esp32-20210902-v1.17.bin
```

## Examples

### Led

```
import machine, neopixel
np = neopixel.NeoPixel(machine.Pin(27), 1)
np[0] = (0,255,78)
np.write()
```

### Button

```
import machine, neopixel
button = machine.Pin(39, machine.Pin.IN, machine.Pin.PULL_UP)
np = neopixel.NeoPixel(machine.Pin(27), 1)

while True:
    if (button.value()):
        np[0] = (255,0,0) # red
        np.write()
    else: # press button
        np[0] = (0,255,0) # green
        np.write()
```

### Web server

```
# Configure the ESP32 wifi
# as STAtion mode.
import network

sta = network.WLAN(network.STA_IF)
if not sta.isconnected():
    print('connecting to network...')
    sta.active(True)
    sta.connect('ssid', 'pass')
    while not sta.isconnected():
        pass
print('network config:', sta.ifconfig())

# Configure the socket connection
# over TCP/IP
import socket

# AF_INET - use Internet Protocol v4 addresses
# SOCK_STREAM means that it is a TCP socket.
# SOCK_DGRAM means that it is a UDP socket.
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.bind(('',80)) # specifies that the socket is reachable 
#                 by any address the machine happens to have
s.listen(5)     # max of 5 socket connections

# Function for creating the
# web page to be displayed
import machine, neopixel
np = neopixel.NeoPixel(machine.Pin(27), 1)

def web_page():

    html_page = """   
      <html>   
      <head>   
       <meta content="width=device-width, initial-scale=1" name="viewport"></meta>   
      </head>   
      <body>
      <center><h2>M5Stack Atom Web Server in MicroPython </h2></center>
      <center>   
         <form>   
          <button name="LED" type="submit" value="1"> LED ON </button>   
          <button name="LED" type="submit" value="0"> LED OFF </button>   
         </form>   
      </center>   
      <center>
      </body>   
      </html>"""  
    return html_page   

while True:
    # Socket accept() 
    conn, addr = s.accept()
    print("Got connection from %s" % str(addr))
    
    # Socket receive()
    request=conn.recv(1024)
    print("")
    print("")
    print("Content %s" % str(request))

    # Socket send()
    request = str(request)
    led_on = request.find('/?LED=1')
    led_off = request.find('/?LED=0')

    if led_on == 6:
        np[0] = (0,255,0)
        np.write()
    elif led_off == 6:
        np[0] = (0,0,0)
        np.write()
    
    response = web_page()
    conn.send('HTTP/1.1 200 OK\n')
    conn.send('Content-Type: text/html\n')
    conn.send('Connection: close\n\n')
    conn.sendall(response)
    
    # Socket close()
    conn.close()
```

## References
1. Led https://www.youtube.com/watch?v=_HfETjIYdig
2. Web Server https://techtotinker.blogspot.com/2020/10/015-esp32-micropython-web-server-esp32.html
