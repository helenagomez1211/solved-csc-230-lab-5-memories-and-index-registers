Download Link: https://assignmentchef.com/product/solved-csc-230-lab-5-memories-and-index-registers
<br>
In this course, we will work with program memory (flash) and data memory (RAM), but not EEPROM.

The diagram of the program (flash) memory from page 20 of the ATmega2560 datasheet:

What is the largest program memory address of ATmega2560?

How many bits required to address all of the program memory?

Answers: Based on Table 2-1, the flash memory size is 256KB -&gt; 2<sup>18</sup>Bytes -&gt;2<sup>17</sup>words. Recall that the flash memory (that is, the program memory) is word addressable; therefore, the largest word address is 0x1FFFF, which requires 17bits (or 3 bytes) to store.

The diagram of the data (SRAM) memory from pages 22 and 27 of the ATmega2560 datasheet:

ATmega2560 Internal memory (SRAM) contains the data memory (.dseg) and the stack. There is no external memory on the AVR boards in this lab.

The stack is necessary for working with interrupts (covered later in this course) and for the CALL, RET, PUSH, and POP instructions. CPU manages the stack automatically via the stack pointer (SP), which must be initialized before using the stack. For detailed information about the stack pointer, see page 15 in the ATmega2560 datasheet.

<ol>

 <li><strong> Simple Function Call.</strong></li>

</ol>

In the first-year programming language courses such as CSC 110, CSC 111, we wrote many functions. In assembly language, the terms procedure, function, and subroutine are interchangeable. A function call implies a transfer of control flow (like branch and jump) to an address representing the entry point of the function. Next, the processor executes the body of the function. When it reaches the end of the function, another transfer occurs to resume execution at the instruction following the initial call. The first transfer is the function call (for invocation), the second transfer is the return (to get back to the calling program). Together, this constitutes the processor’s call-return mechanism.

How does a processor know where to return to after the function completes?

When CPU executes the CALL instruction, it stores the address of the instruction immediately after the CALL on the stack (at the end of the data memory). At the same time, it decrements the Stack Pointer (SP), which holds the address of the stack’s next available memory location, by 3. When CPU executes the RET instruction, the opposite happens: it writes the 3-byte value on top of the stack to the PC register and increments the SP register by 3.

Why 3 bytes?<a href="#_ftn1" name="_ftnref1"><sup>[1]</sup></a>

For function calls to work properly, <strong>the Stack Pointer must be initialized</strong> at the beginning of every program. On the ATmega2560, SRAM ends at 0x21FF; therefore, we initialize the stack pointer to that address. It is named RAMEND in the m2560def.inc file, which is automatically included by the Atmel Studio 7 when you build your program.

Consider a simple function – no parameter passing, no return value. Create a new project named lab5 and write the following code (also available as function.asm from conneX):

<table width="658">

 <tbody>

  <tr>

   <td width="658"> .cseg.org 0; Initialize the Stack Pointer       ldi r16, high(RAMEND)         out SPH, r16         ldi r16, low(RAMEND)out SPL, r16loop:ldi r16, 0         call AddOne         cpi r16, 10brne loopdone:rjmp doneAddOne:inc r16         ret </td>

  </tr>

 </tbody>

</table>

Build the code and go to the menu, then click on <em>Debug -&gt; Start Debugging and Break</em>. The Program Counter (PC) in the Processor Status window contains the memory address of the next instruction that is about to execute. Press the F11 key and observe how the value of PC changes with each keypress. Notice when the CALL instruction executes, the PC is incremented by more than the usual 1, and when the RET instruction executes, the PC is decremented and points at the instruction immediately after the CALL instruction. At the same time, keep an eye on the stack memory and check the values being placed there, they should correspond to the program memory addresses (as described above). Also, pay attention to the Stack Pointer value when CALL and RET instructions are executed.

<strong>III. Data Memory Addressing (Direct &amp; Indirect). </strong>

<u>Data Direct Addressing:</u>

Consider the following code fragment:

<table width="658">

 <tbody>

  <tr>

   <td width="199"> .cseg.org 0 ldi r16, 7    sts variable, r16</td>

   <td width="459">; store the value in r16 at memory location labeled “output”</td>

  </tr>

  <tr>

   <td width="199">        lds r16, variable  .dseg  .org 0x0200variable: .db 1 </td>

   <td width="459">; load the value from a data memory location which is; labeled “input” into the register r16</td>

  </tr>

 </tbody>

</table>

In general, the instruction takes the form of “lds Rd, (k)”, where the value of k is a 16-bit unsigned integer representing the memory address of the SRAM (data memory). The content (1 byte) stored in the memory address k is loaded into register Rd. Note that data memory is byte-addressable, which means that even though the memory address is two bytes, the value stored at that location is 1 byte. For example, in the above code fragment, the memory address labeled “variable” is 0x0200 (two bytes), the value stored there is 0x07, which is one byte.

<u>Data Indirect Addressing:</u>

The AVR processor has three register pairs that can be used for data indirect addressing. The three register pairs (also called index registers) are:

<ul>

 <li>-&gt; R27:R26 or XH :XL</li>

 <li>-&gt; R29:R28 or YH :YL</li>

 <li>-&gt; R31:R30 or ZH :ZL</li>

</ul>

The address to be accessed must be preloaded into either X, Y, or Z register. What do the following statements do?

ld r6, X      st -Y, r16

lpm r23, Z


Indirect addressing is especially suited for accessing arrays, tables, and Stack Pointer.

<ol>

 <li>Download lab5.asm from conneX and finish the program:</li>

</ol>

A C-style string (C string) is stored in the program memory (flash memory). Write a short program to calculate the length of the string and copy the string from the program memory (flash) to the data memory (SRAM).

<ol start="2">

 <li>Extend the above program by writing a function that uses the stack to reverse a c-string at a data memory location, which is provided to the function via the Z pointer. The value of Z pointer must be set before calling the function. The function would iterate (loop) over the characters in the string (in memory) while pushing each one onto the stack; then, it would iterate over the same memory space while popping each character from the stack and placing it in memory. After the function completes, the string in memory would be reversed.</li>

</ol>

<a href="#_ftnref1" name="_ftn1">[1]</a> For answer, see the previous section of this lab.