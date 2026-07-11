# MachinePM - Robot PCB
A versatile ESP32-based PCB for wireless robot control used in the 2026 Quebec Engineering Games by Polytechnique Montreal's [MachinePM](https://www.facebook.com/MachinePM/) robotics team
<p align="center">
  <img src="Images/pcb_view_0.png" width="500">
  &nbsp;&nbsp;
  <img src="Images/pcb_picture.jpg" width="420">
</p>

# Project overview
Every year, as part of the Quebec Engineering Games, the _Machine challenge_ takes place where each participating university has four months to design a robotics solution to a given challenge. As a member of MachinePM, the Polytechnique student robotics team responsible for taking on this challenge, I designed a PCB which was intended to be used on as many potential robots as possible for the 2026 challenge.

This single board, being a huge step forward compared to previous electronics solutions, had to meet combined needs of all potential future robots, which placed design focus on versatility and interoperability between robots in case of failure. The perfect expample would be the feature where, with a simple DIP-switch the user can switch the power supply to any servomotor channel from 5V to 7V for quick reconfiguration of a board.
<p align="center">
  <img src="Images/Competition/robot_0.jpg" width="300">
  &nbsp;&nbsp;
  <img src="Images/Competition/robot_1.jpg" width="300">
  &nbsp;&nbsp;
  <img src="Images/Competition/robot_2.jpg" width="300">
</p>

This electronics platform played a huge role in enabling the team to design a robotics solution on a larger scale than it had ever seen and contributed to the team winning **third place at the Machine Challenge of the Quebec Engineering Games 2026**. For more context on the project's results and associated awards, please see [this LinkedIn post](https://www.linkedin.com/posts/justin-lalonde-26bb49305_apr%C3%A8s-huit-mois-de-travail-intensif-je-suis-ugcPost-7419548842389381120-tLYX/?utm_source=share&utm_medium=member_desktop&rcm=ACoAAE3yFjMBjUuSvDgIEjEyuwl7U7Dr1T2H0Y8) (post is in French).

# Key features
The PCB acts as a robot motherboard which can control up to :
- Six 12V DC motors (up to 3.7A)
- Four 5V/7V servomors (PWM control);
- Two 12V stepper motors;

The board can be powered by any DC current source with voltages from 9.6V to 21.0V as long as the source can handle the combined power of every connected motor/peripheral. 3S lithium-ion 3S (11.1V nominal) were used during the competition.

Other than the motor channels, the board offers many other connectivity options such as :
- One I2C interface;
- Two headers for external switches;
- One USBC upstream facing port for programming / debugging of the ESP32;
- One USBA downstream facing port for connecting a wireless controller's USB dongle, among other possibilities;
- One external ligthing header (LED strips for example);

Along with the ESP32's Bluetooth / Wi-Fi antenna, an on-board RGB-LED and an on/off rocker switch, this board acts as a solid and versatile platform for quick robotics prototyping. 
<p align="center">
  <img src="Images/pcb_id_2.png" width="800">
</p>

# Hardware architecture
This project consists of a 4-layer PCB designed using Altium Designer. 
This PCB implements an ESP32 microcontroller module and many more integrated circuits.
<p align="center">
  <img src="Images/parts_id_0.png" width="500">
  &nbsp;&nbsp;
  <img src="Images/parts_id_1.png" width="300">
</p>

POWER STAGES :
1. The input voltage from the connected DC power supply (battery) first goes through an Input protection / switching stage. This input stage implements undervoltage and overvoltage lockouts, inrush current limiting and enable the rocker switch to disable power supply to the rest of the board ([LM74800](https://www.ti.com/lit/ds/symlink/lm7480-q1.pdf?HQS=dis-dk-null-digikeymode-dsf-pf-null-wwe&ts=1783701329776));
2. The input voltage then directly feeds the DC motor driver IC's ([DRV8231](https://www.ti.com/lit/ds/symlink/drv8231.pdf)) as well as the stepper motor driver IC's ([DRV8846](https://www.ti.com/lit/ds/symlink/drv8846.pdf?ts=1783761254264));
3. This same input voltage is also converted to 5V as well as 7V using two of the same switching regulator IC in a buck converter topology ([TPS56A37](https://www.ti.com/lit/ds/symlink/tps56a37.pdf?ts=1783737334153)).
4. The 5V and 7V are then multiplexed by two E-Fuse IC's ([TPS249474](https://www.ti.com/lit/ds/symlink/tps25947.pdf?ts=1783753086040)) per servomotor channel. By interacting with the 4-channel DIP-switch, the user can select which of the two supplies powers any one of the four servomotors connected to the board (see image above). These E-Fuse IC's also implement reverse current, overcurrent and inrush current protections independently for each supply;
5. The 5V supply is fed into the USBA host port for powering downstream devices and is multiplexed with the bus voltage of the USBC device port using a power multiplexer IC ([TPS2115](https://www.ti.com/lit/ds/symlink/tps2115.pdf?ts=1783787434649&ref_url=https%253A%252F%252Fwww.ti.com%252Fproduct%252FTPS2115)) before being regulated to 3.3V ([TLV75733](https://www.ti.com/lit/ds/symlink/tlv757p.pdf?ts=1783787499434&ref_url=https%253A%252F%252Fwww.ti.com%252Fproduct%252FTLV757P%252Fpart-details%252FTLV75733PDRVR)) to power many IC's present on the board, more importantly the ESP32 module itself. This power multiplexing is to ensure that the board can be programmed with a simple USBC connexion without needing the presence of the main power source;

DIGITAL IC's
1. Serial communication with the EPS32 requires a USB-To-UART bridge IC ([CP2102N](https://www.silabs.com/documents/public/data-sheets/cp2102n-datasheet.pdf));
2. Two Analog to digital conversion (ADC) IC's are also present on the board to read the power rails and analog signals generated by certain IC's ([ADS1015](https://www.ti.com/lit/ds/symlink/ads1013.pdf?HQS=dis-dk-null-digikeymode-dsf-pf-null-wwe&ts=1783787787332&ref_url=https%253A%252F%252Fwww.ti.com%252Fgeneral%252Fdocs%252Fsuppproductinfo.tsp%253FdistId%253D10%2526gotoUrl%253Dhttps%253A%252F%252Fwww.ti.com%252Flit%252Fgpn%252Fads1013))
3. A level-shifter / signal buffer circuit was also implemented to isolate the ESP32 gpio's from the servomotors and shift the PWM signal to 5V ([SN74LVC2G17](https://www.ti.com/lit/ds/symlink/sn74lvc2g17.pdf?ts=1783718395322));
4. A 1 to 2 demultiplexer was used to enable the user DIP-switch to enable only one of the two E-Fuses which handled the 5V and 7V supply to the servomotor channels ([SN74LVC1G18](https://www.ti.com/lit/ds/symlink/sn74lvc1g18.pdf?ts=1783788025016&ref_url=https%253A%252F%252Fwww.google.com%252F));

# Future improvements

# Documentation
You can find the PCB-schematics and the full project report (report is in French) under the [Documentation folder](https://github.com/justinlalonde/MachinePM---Robot-PCB/tree/main/Documentation).

<p align="center">
  <img src="Images/schematic_preview.png" width="600">
</p>
