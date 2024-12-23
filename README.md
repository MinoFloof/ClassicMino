# ClassicMino

ClassicMino is a project created as part of the "Introduction to Embedded Programming" classes, in order to acquire a grade.

The name "ClassicMino" was chosen to represent the idea of Classic NES Tetris using an ESP32-C3-DevKitM-1 circuit board, the NES Controller for controls, and an LCD 240x240 IPS display to graphically render the game.

## Hardware Components

The following hardware components were used:

* [ESP32-C3-DevKitM-1](https://amzn.eu/d/9a1SwUs)
* USB-A to Micro-USB-B Cable (Power for ESP32-C3-DevKitM-1)
* [NES Extension Cord](https://www.micomputer.es/en/nes/450-super-nintendo-extension-cable.html)
* [Dollatek TFT-LCD-Display (240x240 IPS, 3.3V with SPI Interface)](https://www.amazon.de/gp/product/B07QJY5H9G/)
* Male-To-Male Pin Connectors
* Breadboard
* Soldering Equipment

---

## Explanation of NES Controller Communication

The NES console uses a polling mechanism instead of interupts, with a frequency of 60Hz (this coincides with NES consoles rendering at 60FPS). Inside the controller is an internal 8-bit parallel-to-serial shift register (4021 IC). This allows to have all eight button states to be latched into the register simultaneously (parallel) and then read out one bit at the time (serial). [[1]](https://www.nesdev.org/wiki/Standard_controller#Hardware)

The NES console sends out a short HIGH-signal (12µs) through the Latch wire to the controller. This causes the shift register to store all eight button states simultaneously. After 6µs, the NES sends 8 HIGH-signals through the Clock wire to the controller, 12µs per full cycle, 50% duty cycle. At each clock cycle, the button states are read out from the shift register bit-by-bit in the following sequence: A, B, Select, Start, Up, Down, Left, Right. Data will assert ground if a button was pressed (= negative true). [[2]](https://tresi.github.io/nes/)

[![NES Controller Pinout](documentation/images/nes-data.gif)](https://tresi.github.io/nes/nes-data.gif)


### Figuring out the NES extension cord pinout

Given that there was no pinout issued by the manufacturer, it was required to open up the female end of the extension cord. For that, we need to look at the actual NES Controller pinout, so we can compare and figure out which wire is which.

[![NES Controller Pinout](documentation/images/nes-controller-pinout.png)](http://psmay.com/wp-content/uploads/2011/10/nes-controller-pinout.png)

Based on this, we can look at the pinout at the female end from the extension cord.

![NES Extension Cord Pinout](documentation/images/open_extension_cord_female_end.png)

---

# NES Tetris

## Piece Selection Algorithm

The piece selection algorithm implemented is equivalent to the one in Classic NES Tetris [[3]](https://meatfighter.com/nintendotetrisai/#Picking_Tetriminos) [[4]](https://www.reddit.com/r/Tetris/comments/6o6tvv/comment/dkf3uy1):
* Roll a number between 0-7
* If number is 7 OR previous rolled number is the same as the current rolled number: Reroll a number between 0-6 and keep the roll
  * This is to add bias by preventing the same piece to be selected over and over again in bad scenarios
* Otherwise keep the roll

### Roll representation in Tetrominos

[![Tetromino Representation in NES Classic Tetris](documentation/images/tetrominos.png)](https://meatfighter.com/nintendotetrisai/7.png)