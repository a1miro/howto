# Steps for setting up SmartV2 board

## Setting up ethernet MAC addresses

At the moment the MAC addresses are randomly regenerated in the u-boot after each reboot. For avoiding the IP address reassignment after each reboot, you can set the MAC addresses in the u-boot environment variables. To do that, follow these steps:
1. Connect to the board via serial console and power it on.
2. Stop the booting process by pressing any key when you see the message **"Hit any key to stop autoboot: 0"**.

3. Check the current MAC addresses for both ethernet interfaces by running the following commands:
```shell
printenv ethaddr
printenv eth1addr
```
you should see the randomly generated MAC addresses:

```shell
ethaddr=12:b3:e8:a4:0e:6e
eth1addr=f2:83:ad:63:d0:18
```
Be notice that your values will be different from the above ones, as they are randomly generated at each boot.

4. Set the MAC addresses for both ethernet interfaces by running the following commands

```shell   
setenv ethaddr 00:11:22:33:44:55
setenv eth1addr 00:11:22:33:44:56
saveenv
```
restart the board
```shell
reset
```

After this the MAC addresses will be set to the values you have defined and they will persist after each reboot.


## Setting up loop-back can0 --> can1 test

1. Create the CAN interfaces initialisation script `can_init.sh` with the following content:
```shell
#!/bin/bash
ip link set can0 down
ip link set can1 down
ip link set can0 type can bitrate 2500000 restart-ms 100 berr-reporting on
ip link set can1 type can bitrate 2500000 restart-ms 100 berr-reporting on
ip link set can0 up
ip link set can1 up
```
this script will set up the CAN interfaces with a bitrate of 2.5 Mbps and enable error reporting. You can adjust the bitrate and other parameters as needed for your specific use case.

2. Make the script executable:
```shell
chmod +x can_init.sh
```
3. Run the script to initialize the CAN interfaces:
```shell
./can_init.sh
```
4. Now you can test the loop-back connection between can0 and can1 using the `cansend` and `candump` utilities from the `can-utils` package. Open two terminal windows, in the first one run:
```shell
candump -e can1
```
-e option is used to display the error frames, you can omit it if you are not interested in them. In the second terminal window, send a CAN frame from can0 to can1:
```shell
cangen can0 -I 123 -L 8 -D r -g 5
```
This command will generate a CAN frame with ID 123, DLC 8, random data bytes, and a gap of 5ms between frames. You can adjust the parameters as needed for your specific use case. For example, you can use the following command to generate frames with random IDs, DLCs, and data bytes:
``` shell
cangen can0 -g 10        # gap 10ms between frames
cangen can0 -I r         # random IDs (default)
cangen can0 -L r         # random DLC (default)
cangen can0 -D r         # random data bytes (default)
cangen can0 -n 100       # send exactly 100 frames then stop
```

You should see the frame being received in the first terminal window:
```shell
  can1  123   [8]  67 A7 14 10 7C 77 93 04
  can1  123   [8]  61 B1 DD 45 AF 3E 49 50
  can1  123   [8]  2A 0C 12 69 C7 BB 71 4C
  can1  123   [8]  56 12 F2 20 91 18 E7 48
  can1  123   [8]  71 67 54 5F F3 70 2D 00
```

