PONG GAME

*1.0 Introduction*

  Pong is a table tennis–themed arcade sports video game with rudimentary two-dimensional graphics that was released in 1972 by Atari. It was one of the first arcade video games, developed by Allan Alcorn as a practice run for Atari co-founder Nolan Bushnell. Pong was the first commercially viable video game, and it, together with the Magnavox Odyssey, helped to start the video game industry. Several businesses began making games that closely resembled its gameplay soon after its release. Afterwards, Atari's competitors released new sorts of video games that departed from Pong's original style to varied degrees, prompting Atari to encourage its workers to look beyond Pong and create other innovative games. Pong's sequels, which added new features to the original's gameplay, were released by Atari. Atari offered a home version of Pong exclusively through Sears retail shops during the 1975 Christmas season. The home version was also a commercial hit, spawning a slew of clones. Following its initial release, the game was remade for a variety of home and portable systems. Because of its cultural significance, Pong is part of the Smithsonian Institution's permanent collection in Washington, D.C.



*2.0 Overview*

A ball bounces on a screen in the pong game. The user can make the ball bounce back up by using a paddle (which is operated by a mouse in this case).

Our group uses a MAX II EPM240T100C5N but any other FPGA would work.

![1](https://user-images.githubusercontent.com/87557430/126026390-b7854876-8414-4f96-8cf6-e91dcebeafcc.JPG)

Control Unit : Using our mouse to control the movement of the paddle to bounce the ball from the paddle to the wall/object.
Datapath Unit: The movement of the ball after we bounce it back using a paddle.



*3.0 Procedures to create the coding for the Pong Game (Software Part)*

3.1 Drawing a VGA monitor

To display a picture on a VGA monitor, 5 signals are required:
![image](https://user-images.githubusercontent.com/87557430/126026639-72b245b0-3f80-4c01-b012-45ac5e26a251.png)

a. R, G and B (red, green and blue signals).
b. HS and VS (horizontal and vertical synchronization).

- The R, G and B are analog signals, while HS and VS are digital signals.

3.2 Creating a VGA video signal from FPGA pins

![image](https://user-images.githubusercontent.com/87557430/126026682-61a73b14-ff35-4efd-929d-f9908bd141e6.png)

a. Because pins 13 and 14 of the VGA connector (HS and VS) are digital signals, they can be driven directly from two FPGA pins (or via low-value resistors such as 100).
b. Pins 1, 2, and 3 (R, G, and B) are 75Ω analogue signals with 0.7V rated value. Use three 270 series resistors with 3.3V FPGA outputs. With the 75Ω resistors in the monitor inputs, the resistors form voltage dividers, such that 3.3V becomes 3.3*75/(270+75)=0.72V, which is very close to 0.7V. We can get up to 8 colours by driving these RGB pins with different combinations of 0's and 1's.
c. Ground pins are pins 5, 6, 7, 8 and 10.


3.3 Frequency Generator

- A monitor typically shows a picture line by line, top to bottom. Each line is drawn from the left to the right. That is hard-coded and cannot be changed. However, you may control when the sketching begins by transmitting brief pulses on HS and VS at predetermined intervals. HS draws a new line to begin drawing, whereas VS indicates that the bottom has been reached (makes the monitor go back up to the top line). The pulse frequencies for the conventional 640x480 VGA visual transmission should be:

![image](https://user-images.githubusercontent.com/87557430/126026887-94c28f29-d611-411c-8ba8-f197cabb9e1a.png)


3.4 The first Video Generator

Presently, VGA monitors are multisync, so they can handle non-standard frequencies - there's no need to generate exact 60Hz and 31.5KHz frequencies (but if you're using an old (non-multisync) VGA monitor, you will). Let's begin with the X and Y counters.

![image](https://user-images.githubusercontent.com/87557430/126027034-6a3562ee-6c84-4faf-867d-bbab1e7c3887.png)

- CounterX counts 768 numbers (from 0 to 767) while CounterY counts 512. (0 to 511). 

- CounterX is now utilised to generate HS, and CounterY is used to generate VS. We got 32.5KHz for HS and 63.5Hz for VS using a 25MHz clock. The pulses must be active for a long enough period of time for the monitor to identify them. For HS, we'll use a 16 clocks pulse (0.64s) and a complete horizontal line length pulse (768 clocks or 30s). That's less than the VGA spec requires for, but it'll suffice. D flip-flops are used to create the HS and VS pulses (to get glitch free outputs).

![image](https://user-images.githubusercontent.com/87557430/126027012-65967a1e-4123-4f71-ae4d-9c815dfa27ec.png)


- Because the VGA outputs must be negative, we flip the signals.

![image](https://user-images.githubusercontent.com/87557430/126027055-70d2f625-eeac-49be-9400-e74d243c309b.png)

- Finally, the R, G, and B signals can be driven. As a first cut, we can use some X and Y counter bits to create attractive square colour patterns.

![image](https://user-images.githubusercontent.com/87557430/126027115-33460194-fc3b-495a-bde5-77a4e52b81f8.png)

- We will get a picture on the monitor.

3.5 Drawing the picture

- The sync generator is best rebuilt to be utilised as an HDL module where R, G, and B are generated outside. Also, if the X and Y counters begin counting from the drawing area, they are more useful. The new file is available here. 
We can now use it to create a border around the screen.

![image](https://user-images.githubusercontent.com/87557430/126027300-a3284276-b524-4a2b-beaf-5dd9335d618c.png)

3.6 Drawing a paddle

- Let's move the paddle on the screen left and right with a mouse.

- The secret is revealed on the quadrature decoder page. The syntax is as follows:

![image](https://user-images.githubusercontent.com/87557430/126027924-9209e065-8da8-4aae-9f47-962d19bccf46.png)


- We can now display the paddle because we already done the PaddlePosition value.

![image](https://user-images.githubusercontent.com/87557430/126028008-a7ca9f3f-f458-4f82-b61d-b306c33db007.png)


3.7 Drawing the ball

- The ball must move around the screen and bounce back when it comes into contact with an object (border or paddle).

- We begin by displaying the ball. It's a 16x16 pixel square. When CounterX and CounterY reach the ball's coordinates, we activate the drawing.

![image](https://user-images.githubusercontent.com/87557430/126028045-2212f198-e59e-4732-891f-6e29f4afd4da.png)


- Now comes the fun part: the collisions. That is the most difficult aspect of this project.

- We could compare the coordinates of the ball to each object on the screen to see if there is a collision. However, as the number of objects grows, this becomes a nightmare.

- Instead, we define four "hot-spot" pixels, one on each side of the ball. If an object (border or paddle) redraws itself at the same time the ball redraws one of its "hot-spots," we know there is a collision on that side of the ball.

![image](https://user-images.githubusercontent.com/87557430/126028104-1b08dcba-ec5a-4190-a2e2-f1d9d99abe45.png)


- We now update the ball position only once per video frame.

![image](https://user-images.githubusercontent.com/87557430/126028117-147613c2-249a-41bc-aba2-540f0832b0f7.png)


- Finally, we can put everything together.

![image](https://user-images.githubusercontent.com/87557430/126028123-c55e5424-af38-48a4-b391-ab0b73f9e35d.png)


Very difficult. But, we managed to go throughh with it. That is all the steps we needed.


*4.0 Equipments and Apparatus*

a) FPGA board MAX II EPM240T100C5N

b) USB blaster

c) VGA to HDMI adaptor

d) Jumpers/Wires

e) Laptop

f) Resistors

g) Microcontroller Power Board (5.0 V)

h) Breadboard

i) Mouse

g) VGA female header



*5.0 Procedures to connect the Hardware Parts to the Laptop*

a) Firstly, we connected the FPGA to the VGA female cable that are placed on the breadboard by using the wires/jumpers.

b) FPGA is powered by a microcontroller power board with 5.0 V.

c) The VGA cable was connected to the laptop using a VGA to HDMI cable adapter.

d) USB blaster is connected to the FPGA and to the laptop to detect the FPGA software in the laptop.

e) Software is run using Quatus II. FPGA was connected and detected. 

f) Run the quartus simulation and the image of the Pong game will pop up.

![2](https://user-images.githubusercontent.com/87557430/126028253-8555514d-b378-4f4c-b0fc-683e3515705f.jpg)




*6.0 Acknowledgement*

We would like to thank the lecturer for offering advices and assistance whenever we were confused when completing all of the tasks assigned to us especially Dr.  Muhammad Mun'im bin Ahmad Zabidi. Finally, the great teamwork spirit and contributions of all the participants in Group CADNoKokyu Section 1 of this CAD with HDL subject are greatly appreciated.



*7.0 References*

a) Mohamed Khalil-Hani. Modern Digital System Design With FPGA School Of Electrical Engineering Universiti Teknologi Malaysia, PP. 1-275, January 2021.

b) Harris, D. M., &amp; Harris, S. L. (2013). Digital design and computer architecture. Morgan Kaufmann. 

c) Intro to Verilog - MIT. (n.d.). http://web.mit.edu/6.111/www/f2017/handouts/L03_4.pdf. 

d) Sutherland, S. (2002). Verilog-2001: a guide to the new features of the Verilog hardware description language (3rd ed., Vol. 2, Ser. 5). Kluwer Academic. 










