---
layout: default
title: PIC
nav_order: 5
grand_parent: Arşivlenen Dosyalar
parent: Microcontrollers
permalink: /assets/Microcontrollers/PIC
---

**Step 1:**  
Install the setup file for your operating system from the zip file named  
[**mikroprog-pic-dspic-pic32-drivers.zip**]({{(https://em5l.github.io)}}/home/assetsMicrocontrollers/PIC/mikroprog-pic-dspic-pic32-drivers.zip)

**Step 2:**  
Install the setup file from the zip file named  
[**mikroprog-suite-pic-dspic-pic32-programming-software-setup-v290.zip**]({{(https://em5l.github.io)}}/home/assetsMicrocontrollers/PIC/mikroprog-suite-pic-dspic-pic32-programming-software-setup-v290.zip)

**Step 3:**  
The program should be opened **without** the microcontroller being inserted.  
(On some computers, if it is inserted before opening the program, an error may occur.)

**Step 4:**  
As shown in the photo, click on the **Load** button to load the `.hex` file you created.  
Then, click on the **Write** button to program the microcontroller.  
Finally, press the **Verify** button to check if the code was successfully written.

Example Code:

<div class="code-example" markdown="1">
```
#include <30F6014A.h>

// Configuration fuses
#fuses NOWDT, HS, NOPROTECT, NOPUT, NOBROWNOUT

// Clock frequency 
#use delay(clock=20000000)  // 20 MHz external crystal (adjustable)

void main() {
   set_tris_b(0xFFFD);       // Set RB1 as output or set_tris_b(0b1111101)
   output_low(PIN_B1);       

   while(TRUE) {
      output_toggle(PIN_B1); // Toggle the RB1
      delay_ms(500);         
   }
}
```
