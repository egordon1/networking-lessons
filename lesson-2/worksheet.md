# Controlling physical devices across a network on two Raspberry Pis

There are three steps to setting up the Raspberry Pis so one can control the other: configuring the network, writing the server program, and setting up the hardware.

## Network configuration

Before the Raspberry Pis can communicate they need to be connected together via a network. Normally, when a device connects to a network, it is assigned a unique identifier called an IP address. As we only have two Raspberry Pis we have to give each Pi its own IP address.

1. Follow the [Static IP address setup guide](../rpi-static-ip-address.md) from lesson 1 to configure the IP address.

1. Repeat this procedure with your other Pi, giving this one the IP address `192.168.0.3`.

**Tip:** Use a Post-it note to physically label the Raspberry Pis with their IP addresses otherwise things will get confusing later!

### Testing your network

1. Connect the two Pis with an Ethernet cable
1. On the Pi that has the IP address ending `.2`, open the terminal and type:

    ```bash
    ping 192.168.0.3 -c5
    ```

You should see something like this:

```
PING 192.168.0.3 (192.168.0.3) 56(84) bytes of data.
64 bytes from 192.168.0.3: icmp_req=1 ttl=128 time=3.46 ms
[...four more PINGs ...]

--- 192.168.0.3 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4007ms
rtt min/avg/max/mdev = 3.466/3.788/4.380/0.322 ms
```

If not, check your edits and the network cable. Once the Raspberry Pis are successfully networked you are ready to write the control program.

## Setting up the control program
### On the Server Pi (192.168.0.2)

1. Create a new file with the nano editor by typing `nano thing-server.py`
1. Type in the following program:

   ```python
   from gpiozero import Button
   from time import sleep
   import network
   
   btn = Button(4)
   
   def heard(phrase):
      print("heard:" + phrase)
      for a in phrase:
         if a == "\r" or a == "\n":
            pass # strip it
         else:
            if btn.is_pressed():
               network.say("1")
            else:
               network.say("0")
   
   while True:
      print("waiting for connection")
      network.wait(whenHearCall=heard)
      print("connected")
      
      while network.isConnected():
         print("server is running")
         sleep(1)
         
      print("connection closed")
   ```

1. Save the file with `ctrl + o` followed by `Enter` and then exit nano with `ctrl + x`

### On the Client Pi (192.168.0.3)
1. Create a new file with the nano editor by typing `nano thing-client.py`
1. Type in the following program:
    ```python
    from gpiozero import LED
    from time import sleep
    import sys
    import network

    SERVER_IP = sys.argv[1]
    led = LED(17)
    gotResponse = False

    def heard(phrase):
      global gotResponse
      print("heard:" + phrase)

      for a in phrase:
         if a == "\r" or a == "\n":
            pass # skip it
         elif a == "0":
            led.off()
         else:
            led.on()
      gotResponse = True

    while True:
      while True:
         try:
            print("connecting to switch server")
            network.call(SERVER_IP, whenHearCall=heard)
            break
         except:
            print("refused")
            sleep(1)

    print("connected")

    while network.isConnected():
       gotResponse = False
       print("polling")
       network.say("?")

       while network.isConnected() and not gotResponse:
          print("waiting")
          sleep(1)

    print("connection closed")
    ```

    1. Save the file with `ctrl + o` followed by `Enter` and then exit nano with `ctrl + x`



    ## Setting up the hardware

    **Important**: do not connect hardware directly to the pins! Use female header wires that you can plug onto the GPIO pins and your hardware.

    ### 1. Set up the client machine with an LED (192.168.0.3)

    ![](images/client-led-setup.png)

    ### 2. Set up the server with a button (192.168.0.2)

    Note: you do not need to use an actual button, just something to connect the GPIO pin to the ground pin. It could be two paper clips or something similar. Again, use header wires that protect the GPIO pins.

    ![](images/server-button-setup.png)

    ## Running the program

    The **server** machine is connected to a button. It monitors data from the client machine, and when it receives a '?' character it sends back a '1' if the button is pressed and a '0' if it is not.

    The **client** machine is connected to an LED. It sends a '?' character every second to the server, and if it gets back a '1' in reply it turns the LED on.

1. Set the first Pi up as a **server** by typing:

    ```bash
    python3 thing-server.py
    ```

1. The second Pi will be the **client**. You need to tell it the IP address of the server that you want to connect to. For example, to connect to the Raspberry Pi that has the IP address ending in `.2`, type:

    ```bash
    python3 thing-client.py 192.168.0.2
    ```

1. You should now be able to press the button on the Raspberry Pi connected to the server, and the LED will flash on the client. Try it out!

### Things to think about:

- What is happening, physically and electrically, when you press the button?
- What happens at the other end, electrically, when the server receives a signal?

### Things to try:

- Can you break the program by pressing the button too fast?
- Try to change the frequency of the client requests (the `?` character). What happens if they get sent too fast?
- What happens if you stop the server program by pressing `CTRL-C`?

## What next?

- Change the message that appears when your program starts.
- Change the message that appears when a client connects.
- Change the character the client sends every second. Does it matter?
- Make the LED flash more quickly.
- Make the LED flash for a random length of time.
- Comment your code to explain what each section does.
- If the teacher asks you to, change the network configuration back to a dynamic IP address as shown in the "Clean up" section of the [Static IP address setup guide](../rpi-static-ip-address.md) from lesson 1.
