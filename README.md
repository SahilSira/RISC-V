# RISC-V

This github repository summarizes the progress made in the RISCV - ISA program. Quick links:

[Day 1 : Introduction to RISC-V ISA and GNU compiler toolchain](#day-1)

[Day 2 : Introduction to ABI and Basic Verification Flow](#day-2)

[Day 3 : Digital Logic with TL-Verilog and Makerchip](#day-3)

[Day 4 : Basic RISC-V CPU Micro-Architecture](#day-4)

[Day 5 : Pipelined RISC-V CPU Micro-architecture](#day-5)

## Day 1 

<details> 
<summary> Installation </summary>

Install the dependencies using the following command :
```
sudo apt-get install libboost-regex-dev
```

**Steps to install the toolchain**
```
git clone https://github.com/kunalg123/riscv_workshop_collaterals.git
cd riscv_workshop_collaterals
chmod +x run.sh
./run.sh
```

Once you run it you will get make error. ignore it  and type the following command
```
cd ~/riscv_toolchain/iverilog/
git checkout --track -b v10-branch origin/v10-branch
git pull 
chmod 777 autoconf.sh 
./autoconf.sh 
./configure 
make
sudo make install 
```

- To set the PATH variable in .bashrc
```
gedit .bashrc

#Type at last line
export PATH="/home/sahil/riscv_toolchain/riscv64-unknown-elf-gcc-8.3.0-2019.08.0-x86_64-linux-ubuntu14/bin:$PATH" 

# close the bashrc and type in terminal
source .bashrc
```

</details>

<details>
  <summary>Introduction to RISC-V ISA </summary>

RISC-V ISA is a base integer ISA and must be present in any implemenatation along with some optional extension. The RISC-V has been designed to support extensive customization and specialization which can be extended  with  one  or  more  optional  instruction-set  extensions,  but  the  base  integer instructions cannot be redefine. The different instructions included in RISC-V are listed below.

1. Pseudo instructions - For e.g- mv,li,ret etc
2. Base integer instruction (RV64I, RV32I)-For e.g-lui,addi etc
3. Multiply extension (RV64M) -For e.g- mulw,divw etc
4. Single and double floating point instruction (RV64F, RV64D) -For e.g- flw,fadd etc
5. Application binary instruction 
6. Memory allocation and stack pointer

To know more about RISC-V check on this link [here](https://riscv.org/wp-content/uploads/2017/05/riscv-spec-v2.2.pdf).

Each base integer set is characterized by the  width  of the register (XLEN) and size of the user address space. The most important advantage of RISC-V is that it is an open standard instruction which is easily available for academics and commercial purposes free of cost.

</details>

<details>
  <summary>Application Software on Hardware Flow</summary>

![1](https://github.com/SahilSira/RISC-V/assets/140998855/fd9b355d-a776-4c1f-a3f8-d699f0d7f882)

When a C program needs to run on a hardware chip, it goes through a series of steps. First, the C program is turned into assembly language, specifically RISCV assembly language in this case. Then, this assembly language is transformed into machine language consisting of 0s and 1s. These binary instructions are what the chip understands and executes. There's a bridge between the RISCV assembly language and the physical layout of the chip. This bridge is made using Hardware Description Language, which is more closely related to the hardware's workings. To create a RISC specification, the architecture needs to be implemented in a way that registers transfer data. This process involves converting from RTL (Register Transfer Level) to the layout, a process known as RTL to GDSII flow. This ensures that all applications function properly on the hardware.

To make an application work on the hardware, it has to pass through the software system. Here, the system software comes into play, which includes the Operating System (OS), compiler, and assembler. The OS handles tasks like input/output and memory allocation, while the compiler turns the high-level code (like C or C++) into a set of instructions. These instructions depend on the hardware's structure. For a RISC-V system, the instructions follow the RISC-V architecture. The assembler then takes these instructions and turns them into a binary form, which is basically a machine language program. This binary representation is what the hardware ultimately receives and processes. These instructions act as a link between the C language and the intricate hardware components. This link is formally called the Instruction Set Architecture (ISA). In hardware's language, only 0s and 1s make sense, and they serve as the foundation for communication between software and hardware.

</details>

<details>
  <summary>Illustration of the RISC-V gnu toolchain</summary>

#### O1 mode 
Consider the simple C program given below which calculates the sum of the number form 1 to n. 

```
#include<stdio.h>
int main()
{
    int i ,sum=0,n=9;
    for (i=1;i<=n;i++)
        sum+=i;
    printf("The sum of numbers from 1 to %d is %d\n",n,sum);
    return 0;
}
```
In order to map this command to riscv based assembly language compile it using the riscv-gnu-toolchain shown below

```
riscv64-unknown-elf-gcc -O1 -mabi=lp64 -march=rv64i -o sum1ton_O1.o sum1ton.c
riscv64-unknown-elf-objdump -d sum1ton_O1.o | less
spike pk sum1ton_O1.o 
```

**Output of the disassembled file**

![dis_main](https://github.com/SahilSira/RISC-V/assets/140998855/a6af741e-50aa-4a54-bbd8-8d1fbef1d441)

To debug line by line
```
spike -d pk sum1ton_O1.o 
until pc 0 10184
reg 0 sp
reg 0 a2
```

**Output of the spike in debug mode is shown below :**

![spike](https://github.com/SahilSira/RISC-V/assets/140998855/a0229bb1-e7c1-450f-87cb-169e8da8d282)

#### Ofast mode
Consider the same [C program] given in the O1 mode.

In order to map this command to riscv based assembly language compile it in Ofast mode using the riscv-gnu-toolchain shown below

```
riscv64-unknown-elf-gcc -Ofast -mabi=lp64 -march=rv64i -o sum1ton_Ofast.o sum1ton.c
riscv64-unknown-elf-objdump -d sum1ton_Ofast.o | less
spike pk sum1ton_Ofast.o 
```

**Output of the disassembled file**

![dis_main_ofast](https://github.com/SahilSira/RISC-V/assets/140998855/0dcf8144-41a0-4c0a-ad53-605b73bd277a)

**Observation** - The same C code compiled in Ofast mode used less number of instruction compared to the O1 mode.

</details>

<details>
  <summary>Data Representation</summary>

![2](https://github.com/SahilSira/RISC-V/assets/140998855/dbda3b8e-dd17-45af-a7c0-dab81170aa2e)

In RISC-V and computer architecture in general, several terms relate to data representation and storage. Let's explore them:

1. **Byte:** - A byte is the fundamental unit of data storage and representation in computers. It consists of 8 bits and can represent a single character or value.

2. **Word:** - A word typically refers to the natural data size that a processor operates with. In RISC-V, the term "word" can vary based on the architecture. For example, in RV32 (32-bit architecture), a word is 4 bytes (32 bits), while in RV64 (64-bit architecture), a word is 8 bytes (64 bits).

3. **Double Word:** - A double word is twice the size of a word. In RISC-V, for example, in RV32, a double word is 8 bytes (64 bits), and in RV64, a double word is 16 bytes (128 bits).

4. **Least Significant Bit (LSB):** -  The least significant bit is the lowest-order bit in a binary representation. 

5. **Most Significant Bit (MSB):** -  The most significant bit is the highest-order bit in a binary representation. It has the greatest influence on the overall value of a number. The MSB is the bit that represents the largest power of two.


6. **Endianess:** - Endianess refers to how multi-byte data is stored in memory. In a big-endian system, the most significant byte is stored at the lowest memory address, while in a little-endian system, the least significant byte is stored at the lowest memory address. RISC-V supports both big-endian and little-endian modes.

7. **Byte addressing** -  is a memory addressing scheme used in computer systems to identify and access individual bytes of data within the computer's memory. In byte addressing, each individual byte in the memory has a unique address, allowing direct access to and manipulation of single bytes of data. In RISC-V, like in many other computer architectures, memory is byte-addressable.

Understanding these terms is crucial when working with data representation, memory allocation, and programming in computer systems, including the RISC-V architecture.

</details>

<details>
  <summary>Representation of Signed and Unsigned Numbers</summary>

#### Unsigned Numbers
Unsigned numbers don’t have any sign, these can contain only magnitude of the number. So, representation of unsigned binary numbers are all positive numbers only.
Since there is no sign bit in this unsigned binary number, so N bit binary number represent its magnitude only. Zero (0) is also unsigned number. Every number in unsigned number representation has only one unique binary equivalent form, so this is unambiguous representation technique. The range of unsigned binary number is from  **0 to ((2^n)-1)**.

Consider the C code given below which demostrates the maximum unsigned number the RV64I can store. 

```
#include<stdio.h>
#include<math.h>

int main()
{
    unsigned long long int max = (unsigned long long int)(pow(2,64)-1); //Line 1
    // unsigned long long int max = (unsigned long long int)(pow(2,127)-1);// Line 2
    // unsigned long long int max = (unsigned long long int)(pow(2,64)*-1);// Line 3
    // unsigned long long int max = (unsigned long long int)(pow(-2,64)-1);// Line 4
    // unsigned long long int max = (unsigned long long int)(pow(-2,63)-1);// Line 5
    // unsigned long long int max = (unsigned long long int)(pow(2,10)-1);// Line 6
    printf("Highest number represented by unsigned long long  int is %llu \n",max);
    return 0;
}
```
___
***Note***</br>

**%llu** - is the format specifier for 64-bit unsigned integer.

**%lld** - is the format specifier for 64-bit signed integer.

Uncomment the lines in the code appropriately and view the result.
___

- Line 1 will execute and give the result of (2^64)-1.</br>
- Line 2 will execute and give the result of (2^64)-1 instead of (2^127)-1 since the maximum unsigned value that can be stored in the 64 bit register is (2^64)-1.</br>
- Line 3 will execute and give the result of 0 instead of -(2^64) since the minimum unsigned value that can be stored in 64 bit register is 0.</br>
- Line 4 will execute and give the result of 0 instead of (2^64)-1.</br>
- Line 5 will execute and give the result of 0 instead of -(2^64) since the minimum unsigned value that can be stored in 64 bit register is 0.</br>
- Line 6 will execute and give the result of 1024 since the value of max is less that (2^64)-1.

To compile and execute the C code in RISC-V gnu toolchain follow the steps given below:

```
riscv64-unknown-elf-gcc -Ofast -mabi=lp64 -march=rv64i -o unsign_num_new.o unsign_num.c 
spike  pk unsign_num_new.o 
```

**Output of the execution**

![unsigne](https://github.com/SahilSira/RISC-V/assets/140998855/13cc63f4-60fb-4932-9d1b-a8f458e5ad27)

![unsigned out](https://github.com/SahilSira/RISC-V/assets/140998855/5adbc7cf-f28c-49e0-aac2-7321cda3554b)


#### Signed Numbers
Generally 2's complement representation is used for the signed numbers. 2’s complement of a number is obtained by inverting each bit of given number plus 1 to least significant bit (LSB). So, positive numbers are represented in binary form and negative numbers are represented in 2’s complement form. There is extra bit for sign representation. If value of sign bit is 0, then number is positive and you can directly represent it in simple binary form, but if value of sign bit 1, then number is negative and 2’s complement of given binary number should be taken. In this representation, zero (0) has only one (unique) representation which is always positive. The range of 2’s complement form is from  **(-2^(n-1))  to ((2^(n-1))-1)**.

Consider the C code given below which demostrates the maximum and minimum signed number the RV64I can store. 


```
#include<stdio.h>
#include<math.h>

int main()
{
    long long int max = (long long int)(pow(2,63)-1);
    long long int min = (long long int)(pow(-2,63));
    printf("Highest number represented by long long  int is %lld \n",max);
    printf("Smallest number represented by long long  int is %lld \n",min);
    return 0;
}
```
To compile and execute the C code in RISC-V gnu toolchain follow the steps given below:

```
riscv64-unknown-elf-gcc -Ofast -mabi=lp64 -march=rv64i -o sign_num_new.o sign_num.c 
spike  pk sign_num_new.o 
```

**Output of the execution**

![signed](https://github.com/SahilSira/RISC-V/assets/140998855/be7ba122-9ab4-4366-be44-971c9d7913a5)

**Different types of format specifiers**

![3](https://github.com/SahilSira/RISC-V/assets/140998855/70fe7fb5-e5ba-4a54-a4ce-4b22fb1b6349)

</details>


## Day 2

<details>
  <summary>RV64I Base Integer Instruction Set</summary>

RV64I is the base integer instruction set for the 64-bit architecture, which builds upon the RV32I variant. RV64I shares most of the instructions with RV32I but the width of registers is different and there are a few additional instructions only in RV64I. The base integer instruction set has 47 instructions (35 instructions from RV32I and 12 instructions from RV64I). The instructions and their format is shown below :

![1](https://github.com/SahilSira/RISC-V/assets/140998855/1098a0b8-77f6-429e-abc0-3a472da07ea4)

There are 31 general-purpose registers x1–x31, which hold integer values. Register x0 is hardwired to the constant 0. There is no hardwired subroutine return address link register, but the standard software calling convention uses register x1 to hold the return address on a call. For RV32, the x registers are 32 bits wide, and for RV64, they are 64 bits wide. The term XLEN to refer to the current width of an x register in bits (either 32 or 64).

![2](https://github.com/SahilSira/RISC-V/assets/140998855/c748f0f7-99ef-404a-af05-cc5856c3f227)

![3](https://github.com/SahilSira/RISC-V/assets/140998855/d7f3058b-dec9-402c-b3ef-5ae19426ae9f)

In the RISC-V instruction set architecture, instructions are categorized into different formats based on their opcode and operand types. These formats are denoted by single-letter abbreviations. Here's an explanation of each type:

- **R-Type (Register Type)** -  These instructions involve operations that operate on two source registers and store the result in a destination register. They include arithmetic, logical, and bitwise operations. The typical format is: opcode rd, rs1, rs2.

- **I-Type (Immediate Type)** - These instructions have an immediate (constant) value as one of their operands, and they work with a source register to perform operations like arithmetic, logical, and memory operations. The typical format is: opcode rd, rs1, imm.

- **S-Type (Store Type)** - S-type instructions are used for storing data into memory. They combine a source register, a destination address (base register), and an immediate offset to determine where the data should be stored. The typical format is: opcode rs2, imm(rs1).

- **B-Type (Branch Type)** - B-type instructions are used for conditional branching. They compare values from two source registers and use an immediate offset to determine the branching target. These instructions support operations like equality, inequality, and comparison. The typical format is: opcode rs1, rs2, imm.

- **U-Type (Upper Immediate Type)** - U-type instructions are used for loading immediate values into registers. They include unconditional jump instructions. These instructions operate on a single source register and use an immediate value to specify the upper bits of the result. The typical format is: opcode rd, imm.

- **J-Type (Jump Type)** J-type instructions are used for unconditional jumps. They involve using an immediate offset to determine the target address of the jump. These instructions are typically used for implementing function calls and other control flow changes. The typical format is: opcode rd, imm.

The instruction format for all types is shown below :

![4](https://github.com/SahilSira/RISC-V/assets/140998855/935d7e6a-0140-4bda-8d55-abd4237816e1)

To know more about the instructions check this [link](https://riscv.org/wp-content/uploads/2017/05/riscv-spec-v2.2.pdf).

</details>

<details>
  <summary>Application Binary Interface (ABI)</summary>

The ABI is a set of rules that govern how software components, like programs and libraries, interact with each other at the binary level. It defines things like how data is passed between different parts of a program, how function calls are made, and how data structures are organized in memory. The ABI ensures compatibility between different parts of a software ecosystem, making it possible for programs to work together seamlessly even if they're written in different languages or compiled by different compilers. The application program can directly access the registers of the RISC V architecture using system calls. The ABI also known as system call interface enables the application to access the hardware resources via registers.A system call is a specific request your program makes to the operating system to perform a task it can't do on its own. For example, if your program needs to read a file, it would make a system call to ask the operating system to read the file and give it the data. System calls are a way for programs to access the more powerful features of the operating system while staying within the rules defined by the ABI. The ISA is inherently divided into two parts: User & System ISA and User ISA the latter is available to the user directly by system calls.

</details>

<details>
  <summary>Illustration of ABI</summary>

Consider the C code given below which calculates the sum from 1 to 9 :
```
#include<stdio.h>

extern int load(int x, int y);

int main()
{
    int result = 0;
    int count = 9;
    result = load(0x0,count+1);
    printf("Sum of numbers from 1 to %d is %d\n",count,result);
    
}
```

Consider the assembly code (ASM) given below :
```
.section .text 
.global load
.type load, @function

load:
    add a4, a0, zero
    add a2, a0, a1
    add a3, a0, zero
loop : add a4, a3, a4
       addi a3, a3, 1
       blt a3, a2, loop
       add a0, a4, zero
       ret
```
The flow chart of the function performed by ASM code is shown below :

![5](https://github.com/SahilSira/RISC-V/assets/140998855/d7decab2-afef-4811-8358-4006ee4ef0a2)

To illustrate the ABI the C code shown above will send the values to the ASM code through the function load and the ASM code will perform the function and return the value to C code and the value is displayed by the C code.

**Steps to perform the lab task mentioned above**
```
riscv64-unknown-elf-gcc -Ofast -mabi=lp64 -march=rv64i -o custom_1_to_9.o custom_1_to_9.c load.S
riscv64-unknown-elf-objdump -d custom_1_to_9.o | less
spike pk custom_1_to_9.o
```

**Outputs of the Lab**

![sum](https://github.com/SahilSira/RISC-V/assets/140998855/5e7c8222-8795-42a9-80be-765a51c297b6)

![sum_disasselbe](https://github.com/SahilSira/RISC-V/assets/140998855/ef60e8f4-e052-4174-8bd1-c96c9c6be1d9)

</details>

<details>
  <summary>RISC-V Basic Verification flow using iverilog demo</summary>

For verification of the RISC-V CPU the C code will be converted into HEX file and it will be given to the RISC-V CPU and the output will be displayed and verified. The block diagram is shown below :

![6](https://github.com/SahilSira/RISC-V/assets/140998855/83a88fd3-abeb-4e79-9b9b-04d5b9efae12)

For demo go to the lab directory using the command given below :
```
cd ~/riscv_workshop_collaterals/labs/
chmod 777 rv32im.sh
./rv32im.sh  # Contains necessary commands to convert C to hex
```

**Output, Script(rv32im.sh) and firmare.hex**

![lab](https://github.com/SahilSira/RISC-V/assets/140998855/f042481b-14cc-4f30-9bb3-51b48d1bd85e)

![rvsim](https://github.com/SahilSira/RISC-V/assets/140998855/4adcd9eb-beae-4522-886d-0950ce2fc2c1)

![hex](https://github.com/SahilSira/RISC-V/assets/140998855/7ed48be5-3751-4add-98de-e778a3590819)

</details>


## Day 3 
<details >
<summary >Introduction</summary>

TL-Verilog was used as the HDL of choice for this project. Projects on Makerchip can be completely designed using TL-Verilog. Transaction Level - Verilog standard is an extension of Verilog which has various advantages like simpler syntax, shorter codes and easy pipelining. You can learn more about TL-Verilog [here](http://tl-x.org/).

Timing abstract can be done in TL-Verilog. This model is specified for pipelines where the sequential elements are generated by tools from the pipelined specification. This allows for easy retiming without the risk of introduction of any functional bugs. More information on timing abstract in TL-Verilog can be found in the IEEE paper ["Timing-Abstract Circuit Design in Transaction-Level Verilog" by Steven Hoover](https://ieeexplore.ieee.org/document/8119264).

</details>

<details>
  <summary>Example on Makerchip IDE </summary>

  ## Makerchip IDE: 
Makerchip is a free online environment for developing high-quality integrated circuits. You can code, compile, simulate, and debug Verilog designs, all from your browser. Your code, block diagrams, and waveforms are tightly integrated.

## Loading pythagorean Example on Makerchip IDE

![1](https://github.com/SahilSira/RISC-V/assets/140998855/cd411bd7-db16-434d-8659-11a668f51145)

## Conway's Game of Life

![29](https://github.com/SahilSira/RISC-V/assets/140998855/39e97598-bd45-49b1-ac7e-af36c490817c)

</details>

<details >
<summary >Lab for Combinational logic </summary>


We will first implement some basic logic gates on Makerchip IDE to gain understanding of the platform. In TL verilog we simply code the logic itself without requiring to declare the variables separately and $in assignment is also not required

## NOT Gate Example on Makerchip IDE

![2](https://github.com/SahilSira/RISC-V/assets/140998855/b0ad6e8c-887a-4922-9446-17bdac2f4c3c)

## OR Gate Example on Makerchip IDE

![4](https://github.com/SahilSira/RISC-V/assets/140998855/086a970a-30c9-4c8b-9390-4f7c783eacb0)

## AND Gate Example on Makerchip IDE

![3](https://github.com/SahilSira/RISC-V/assets/140998855/58673dbf-2399-4bb1-9dad-e46f6138fdea)

## XOR Gate Example on Makerchip IDE

![5](https://github.com/SahilSira/RISC-V/assets/140998855/bcb69a84-ae96-4aa8-a105-47963d91ad54)

## Vector Addition on Makerchip IDE

![6](https://github.com/SahilSira/RISC-V/assets/140998855/d6853ec4-6f34-412f-a060-059dba347529)

## 2:1Multiplexer on Makerchip IDE

![7](https://github.com/SahilSira/RISC-V/assets/140998855/57a9160f-5bb4-4eec-b176-ed99114cae06)

## 2:1 Vector Multiplexer on Makerchip IDE

![8](https://github.com/SahilSira/RISC-V/assets/140998855/588447ab-df8d-4396-9fa4-ec78041aff21)

## Calculator on Makerchip IDE

This is a combinational calculator that can perform +, -, *, / on two input values.

![9](https://github.com/SahilSira/RISC-V/assets/140998855/45b8b2f5-90de-496c-a870-5230d0431ee6)

</details>
<details >
  
<summary >Lab for Sequential  logic </summary>

Sequential logic refers to a type of digital logic circuit or system in which the output depends not only on the current inputs but also on the previous states of the circuit. Unlike combinational logic, which only considers the current inputs to generate outputs, sequential logic incorporates memory elements to store information and generate outputs based on both current inputs and past history.

## Fibonacci Series

The block diagram of the fibonacci series generator is shown below :

![10](https://github.com/SahilSira/RISC-V/assets/140998855/0f25c81a-da3c-4554-a846-938e7781fb29)

Output:

![15](https://github.com/mavi62/IIITB_VLSI/assets/57127783/3d1e3090-6c65-46c6-b2d7-8c732c5fd7cf)


## Free running counter

The block diagram of the free running counter is shown below :

![12](https://github.com/SahilSira/RISC-V/assets/140998855/8578d8f5-2521-4452-b76d-55f3f2580a35)

Output:

![13](https://github.com/SahilSira/RISC-V/assets/140998855/e6f66dc3-a8a3-4af4-8100-e568993b59fd)

## Sequential Calculator`

It works like a normal calculator in which the result of the previous operation is considered as one of the operand for the next operation. Upon reset the result becomes zero.

![15](https://github.com/SahilSira/RISC-V/assets/140998855/f96b7566-cb29-404e-b139-5a3d5818b15b)

</details>

<details> <summary > Pipelining </summary>
	
Pipelining or timing abstract is an important feature in TL verilog as it can be implemented very easily with fewer codes as compared to system verilog which reduces bugs to a great extent. An example of the pipeling for pythogoras theorem using both TL verilog and system verilog in this repo . In TL verilog pipeling can be implemented by defining the pipeline as |calc and the different pipeline stages should be properly align and are indicated by @1, @2 and so on.

## Pipelined pythagorean theorem

![17](https://github.com/SahilSira/RISC-V/assets/140998855/987f6d61-2fe4-47e1-ab79-918fa5f15af4)

## Error detection:

![18](https://github.com/SahilSira/RISC-V/assets/140998855/988420b7-b4f5-4a56-87ec-ce75a4b3dd59)

## Counter and Calculator in Pipeline

The block diagram of the counter with calculator in pipeline is shown below :

![19](https://github.com/SahilSira/RISC-V/assets/140998855/94385eba-fefb-4f19-a6fe-c30a3c2c258e)

The TL-Verilog code is given below :

```
   $reset = *reset;
   $op[1:0] = $random[1:0];
   $val2[31:0] = $rand2[3:0];
   
   |calc
      @1
         $val1[31:0] = >>1$out;
         $sum[31:0] = $val1+$val2;
         $diff[31:0] = $val1-$val2;
         $prod[31:0] = $val1*$val2;
         $div[31:0] = $val1/$val2;
         $out[31:0] = $reset ? 32'h0 : ($op[1] ? ($op[0] ? $div : $prod):($op[0] ? $diff : $sum));
         
         $cnt[31:0] = $reset ? 0 : (>>1$cnt + 1); 

```

![20](https://github.com/SahilSira/RISC-V/assets/140998855/4f0b6dff-762f-45b9-b6a9-e33520b0c7fb)

## 2 Cycle Calculator

The block diagram of the 2 cycle calculator is shown below:

![21](https://github.com/SahilSira/RISC-V/assets/140998855/18697c74-184b-4b17-a11d-881a4675c5b7)

The TL-verilog code is shown below :
```
   $reset = *reset;
   $op[1:0] = $random[1:0];
   $val2[31:0] = $rand2[3:0];
   
   |calc
      @1
         $val1[31:0] = >>2$out;
         $sum[31:0] = $val1+$val2;
         $diff[31:0] = $val1-$val2;
         $prod[31:0] = $val1*$val2;
         $div[31:0] = $val1/$val2;
         $valid = $reset ? 0 : (>>1$valid + 1);
      @2
         $out[31:0] = ($reset | ~($valid))  ? 32'h0 : ($op[1] ? ($op[0] ? $div : $prod):($op[0] ? $diff : $sum));
```

![22](https://github.com/SahilSira/RISC-V/assets/140998855/95b6c099-4926-43ee-bd79-27940d63c158)

</details>

<details>
<summary>Validity</summary>
<br />
First, we shall see a distance accumulator coupled with a Pythagorean pipeline as shown below.
<br />
	
![23](https://github.com/SahilSira/RISC-V/assets/140998855/d57295c6-8baf-4483-ae88-fca692967cce)
<br />
The TL-verilog code is shown below :
```
$reset = *reset;
   |calc
      @1
         $reset = *reset;
      ?$valid
         @1
            $aa_sqr[31:0] = $aa[3:0] * $aa[3:0];
            $bb_sqr[31:0] = $bb[3:0] * $bb[3:0];
         @2
            $cc_sqr[31:0] = $aa_sqr + $bb_sqr;
         @3
            $out[31:0] = sqrt($cc_sqr);
      @4
         $total_distance[63:0] = $reset ? '0 : ($valid ? >>1$total_distance + $out: >>1$total_distance);  
   
```
<br />
The output for it generated in MakerChip is given below.<br />
![24](https://github.com/SahilSira/RISC-V/assets/140998855/1f1131a9-be8d-41f8-a065-b78c17ccd78d)
<br />

## Cycle Calculator with Validity

The following design is what we are required to create.<br />
![25](https://github.com/SahilSira/RISC-V/assets/140998855/fec454fb-f712-4043-a6e0-a761a234be7a)
<br />

The TL-verilog code is shown below :
```
  $reset = *reset;
   |calc
      @1
         $valid = $reset ? 0 : >>1$valid+1;
         $valid_or_reset = $valid || $reset;
      ?$valid_or_reset
         @1
            $val1[31:0] = >>2$out;
            $sum[31:0] = $val1+$val2;
            $diff[31:0] = $val1-$val2;
            $prod[31:0] = $val1*$val2;
            $div[31:0] = $val1/$val2;
            $valid = $reset ? 0 : (>>1$valid + 1);
         @2
            $out[31:0] = $reset  ? 32'h0 : ($op[1] ? ($op[0] ? $div : $prod):($op[0] ? $diff : $sum));

```
<br />
The output in Makerchip is given below.<br />
![26](https://github.com/SahilSira/RISC-V/assets/140998855/9a789f97-f055-48ba-aba3-dc0a0540a6b8)

## Cycle Calculator with Validity and memory

The design we have to implement is given below.<br />
![27](https://github.com/SahilSira/RISC-V/assets/140998855/e54b4bc4-7d86-4ee9-be19-8e0d498fff3b)

The TL-verilog code is shown below :
```
   |calc
      @0
         $reset = *reset;
         
      @1
         $val1 [31:0] = >>2$out;
         $val2 [31:0] = $rand2[3:0];
         
         $valid = $reset ? 1'b0 : >>1$valid + 1'b1 ;
         $valid_or_reset = $valid || $reset;
         
      ?$vaild_or_reset
         @1   
            $sum [31:0] = $val1 + $val2;
            $diff[31:0] = $val1 - $val2;
            $prod[31:0] = $val1 * $val2;
            $div[31:0] = $val1 / $val2;
            
         @2   
            $mem[31:0] = $reset ? 32'b0 :
                         ($op[2:0] == 3'b101) ? $val1 : >>2$mem ;
            
            $out [31:0] = $reset ? 32'b0 :
                          ($op[2:0] == 3'b000) ? $sum :
                          ($op[2:0] == 3'b001) ? $diff :
                          ($op[2:0] == 3'b010) ? $prod :
                          ($op[2:0] == 3'b011) ? $quot :
                          ($op[2:0] == 3'b100) ? >>2$mem : >>2$out ;
```
<br />
The output in Makerchip is given below.<br />

![28](https://github.com/SahilSira/RISC-V/assets/140998855/d3260de8-0949-4490-8f24-8498f3622aa5)

</details>

## Day 4

<details>
<summary>Introduction to simple RISC-V micro-architecture</summary>
The block diagram of a basic RISC-V microarchitecture is as shown in figure below. Now, using the Makerchip platform the implementation of the RISC-V microarchitecture or core is done. For starting the implementation a starter code present in reference is used. The starter code consist of -

- A simple RISC-V assembler.
- An instruction memory containing the sum 1..9 test program.
- Commented code for register file and memory.
- Visualization.
![1](https://github.com/SahilSira/RISC-V/assets/140998855/92ae0e6e-a07d-40a8-9d37-cb951aa4973a)

It's important to note that RISC-V is an instruction set architecture, and microarchitectures 
based on RISC-V can vary widely depending on the design goals of the processor manufacturer. 
Different companies and research institutions may develop their own microarchitectures that 
implement the RISC-V ISA in unique ways, tailored to specific use cases and performance goals.

Here we are designing the basic processor of 3 stages fetch, decode and execute based on 
RISC-V ISA. For starting the implementation a starter code is present in the github repository 
provided.
```bash
 https://github.com/stevehoover/RISC-V_MYTH_Workshop
```
We will follow a test driven development, ie. develop first and then test functionality.

- Template for starting point code of RISC-V CPU.
```bash
\m4_TLV_version 1d: tl-x.org
\SV
   // This code can be found in: https://github.com/stevehoover/RISC-V_MYTH_Workshop
   
   m4_include_lib(['https://raw.githubusercontent.com/BalaDhinesh/RISC-V_MYTH_Workshop/master/tlv_lib/risc-v_shell_lib.tlv'])

\SV
   m4_makerchip_module   // (Expanded in Nav-TLV pane.)
\TLV

   // /====================\
   // | Sum 1 to 9 Program |
   // \====================/
   //
   // Program for MYTH Workshop to test RV32I
   // Add 1,2,3,...,9 (in that order).
   //
   // Regs:
   //  r10 (a0): In: 0, Out: final sum
   //  r12 (a2): 10
   //  r13 (a3): 1..10
   //  r14 (a4): Sum
   // 
   // External to function:
   m4_asm(ADD, r10, r0, r0)             // Initialize r10 (a0) to 0.
   // Function:
   m4_asm(ADD, r14, r10, r0)            // Initialize sum register a4 with 0x0
   m4_asm(ADDI, r12, r10, 1010)         // Store count of 10 in register a2.
   m4_asm(ADD, r13, r10, r0)            // Initialize intermediate sum register a3 with 0
   // Loop:
   m4_asm(ADD, r14, r13, r14)           // Incremental addition
   m4_asm(ADDI, r13, r13, 1)            // Increment intermediate register by 1
   m4_asm(BLT, r13, r12, 1111111111000) // If a3 is less than a2, branch to label named <loop>
   m4_asm(ADD, r10, r14, r0)            // Store final result to register a0 so that it can be read by main program
   
   // Optional:
   // m4_asm(JAL, r7, 00000000000000000000) // Done. Jump to itself (infinite loop). (Up to 20-bit signed immediate plus implicit 0 bit (unlike JALR) provides byte address; last immediate bit should also be 0)
   m4_define_hier(['M4_IMEM'], M4_NUM_INSTRS)

   |cpu
      @0
         $reset = *reset;



      // YOUR CODE HERE
      // ...

      // Note: Because of the magic we are using for visualisation, if visualisation is enabled below,
      //       be sure to avoid having unassigned signals (which you might be using for random inputs)
      //       other than those specifically expected in the labs. You'll get strange errors for these.

   
   // Assert these to end simulation (before Makerchip cycle limit).
   *passed = *cyc_cnt > 40;
   *failed = 1'b0;
   
   // Macro instantiations for:
   //  o instruction memory
   //  o register file
   //  o data memory
   //  o CPU visualization
   |cpu
      //m4+imem(@1)    // Args: (read stage)
      //m4+rf(@1, @1)  // Args: (read stage, write stage) - if equal, no register bypass is required
      //m4+dmem(@4)    // Args: (read/write stage)
      //m4+myth_fpga(@0)  // Uncomment to run on fpga

   //m4+cpu_viz(@4)    // For visualisation, argument should be at least equal to the last stage of CPU logic. @4 would work for all labs.
\SV
   endmodule
```

</details>

<details>
<summary>Fetch and decode</summary>

Here we gonna design RiscV Cpu Core for which block diagram is given below :
![2](https://github.com/SahilSira/RISC-V/assets/140998855/5a9e72ed-3d4f-4c45-bedf-50dd82874e33)

**Program Counter Logic**

The Program Counter, often referred to as the "PC," is a fundamental component of a processor 
that keeps track of the address of the next instruction to be executed. In the RISC-V 
architecture, the PC is typically called "pc" or "pc_reg." Overall, the PC logic is crucial 
for the control flow of a program. It determines which instruction will be executed next and 
how the program progresses. RISC-V, as a RISC (Reduced Instruction Set Computer) architecture, 
emphasizes simplicity and regularity in its design, which extends to its PC handling 
mechanisms.

![3](https://github.com/SahilSira/RISC-V/assets/140998855/02ad5965-f6be-44c5-bf49-d363ce080b50)

```bash
|cpu
      @0
         $reset = *reset;
         
         $pc[31:0] = >>1$reset ? 32'b0 : >>1$pc + 32'd4;
```
- program counter implementation on makerchipIDE
![4](https://github.com/SahilSira/RISC-V/assets/140998855/0c22672c-a039-4d62-b7e2-e0cc928c69f0)

**Fetch implementation Logic**
During the fetch stage, processors fetches the instruction from the memory to the address 
pointed by the program counter. The program counters holds the address of the next stage, in 
our case it is after 4 cycle and the instruction memory holds the set of instruction to be 
executed. The snapshot of the fetch stage is shown below.
         
- logic diagram for Fetch
![5](https://github.com/SahilSira/RISC-V/assets/140998855/1f2e5846-31cd-4044-b3a9-09389258065a)

```bash
|cpu
      @0
         $reset = *reset;
         $pc[31:0] = >>1$reset ? 32'b0 : >>1$pc + 32'd4;

// Assert these to end simulation (before Makerchip cycle limit).
*passed = *cyc_cnt > 40;
*failed = 1'b0;

|cpu
      m4+imem(@1)    // Args: (read stage)

m4+cpu_viz(@4)
```
- Implementation of Fetch Logic on Makerchip along with Diagram and Visualisation.
![6](https://github.com/SahilSira/RISC-V/assets/140998855/0400b3bf-cce5-4825-9a6d-811cb59b8780)

The current implementations have errors such as the variables are not being used. Fetch Logic 
to be implemented

![7](https://github.com/SahilSira/RISC-V/assets/140998855/3e28a9fa-ec37-44b1-87b2-2469d5731cce)

- code for implementation of Fetch
```bash
@0
         $reset = *reset;
         $pc[31:0] = >>1$reset ? 32'b0 : >>1$pc + 32'd4;
      @1
         $imem_rd_addr[M4_IMEM_INDEX_CNT-1:0] = $pc[M4_IMEM_INDEX_CNT+1:2];
         $imem_rd_en = !$reset;
         $instr[31:0] = $imem_rd_data[31:0];
      
      ?$imem_rd_en
         @1
            $imem_rd_data[31:0] = /imem[$imem_rd_addr]$instr;          
```

- Final implementation of Fetch Logic

![8](https://github.com/SahilSira/RISC-V/assets/140998855/062d8cc2-2681-4345-9a34-d8400f0aa165)

![9](https://github.com/SahilSira/RISC-V/assets/140998855/20dc0019-9990-43ea-b447-183edbe40cc3)

**Decode Logic**

Under this section, we look into how to decode the instruction code we fetched from memory.

- Logic Diagram for Decode stage.

![10](https://github.com/SahilSira/RISC-V/assets/140998855/0fe2cfb1-071f-4763-90bb-dd8275364ab6)

Before moving on to Decode logic implementation, it is important to understand how the 
instruction set and opcode are defined in RISC-V. We have dicussed before the different types 
of the instruction types. The various types of instrutcion types are summarised in the table 
along with the binary code. There are 6 instructions type in RISC-V :

- Register (R) type
- Immediate (I) type
- Store (S) type
- Branch (B) type
- Upper immediate (U) type
- Jump (J) type

![11](https://github.com/SahilSira/RISC-V/assets/140998855/c4559145-fb41-47d0-81fc-51afacf6f836)




-code for decode logic
```bash
@1
         $is_i_instr = $instr[6:2] ==? 5'b0000x || 
                       $instr[6:2] ==? 5'b001x0 || 
                       $instr[6:2] == 5'b11001;
         $is_r_instr = $instr[6:2] ==? 5'b011x0 || 
                       $instr[6:2] == 5'b01011 || 
                       $instr[6:2] == 5'b10100;
         $is_s_instr = $instr[6:2] ==? 5'b0100x;
         $is_b_instr = $instr[6:2] ==? 5'b11000;
         $is_j_instr = $instr[6:2] ==? 5'b11011;
         $is_u_instr = $instr[6:2] ==? 5'b0x101;
```
- Implementation for fetch logic
![12](https://github.com/SahilSira/RISC-V/assets/140998855/5c75c993-18dd-4f50-9de3-df9601e20169)

![13](https://github.com/SahilSira/RISC-V/assets/140998855/18000745-442c-4595-bb4f-089d63efadf6)

*Lab on instruction immediate code*
![14](https://github.com/SahilSira/RISC-V/assets/140998855/47dbff7d-075e-4310-9e67-e4438939abd3)

- Code or determining *immediate* for decode logic implementaion
```bash
		      $imm[31:0] = $is_i_instr ? {{21{$instr[31]}}, $instr[30:20]} :
                      $is_s_instr ? {{21{$instr[31]}}, $instr[30:25], $instr[11:7]} :
                      $is_b_instr ? {{20{$instr[31]}}, $instr[7], $instr[30:25], $instr[11:8], 1'b0} :
                      $is_u_instr ? {$instr[31:12], 12'b0} :
                      $is_j_instr ? {{12{$instr[31]}}, $instr[19:12], $instr[20], $instr[30:21], 1'b0} :
                                    32'b0;
```
- Implementaion of the immediate instruction set
![15](https://github.com/SahilSira/RISC-V/assets/140998855/a3d3a2cf-7dfc-4a31-b602-5e1bd7a9e7c2)
- VIZ
![16](https://github.com/SahilSira/RISC-V/assets/140998855/e4c9a80a-a4c8-4e06-9228-902cb0e8a2a3)

*Lab on instruction decode*
![17](https://github.com/SahilSira/RISC-V/assets/140998855/7b9f689b-8c94-46bd-82da-ea70ecbe1974)

- Code fo nstruction code implmentaion
```bash
	 $rs2[4:0] = $instr[24:20];
         $rs1[4:0] = $instr[19:15];
         $rd[4:0]  = $instr[11:7];
         $opcode[6:0] = $instr[6:0];
         $func7[6:0] = $instr[31:25];
         $func3[2:0] = $instr[14:12];
```

- Output
![18](https://github.com/SahilSira/RISC-V/assets/140998855/28b828b4-db35-45b7-a21e-b34c383d970e)
- VIZ
![19](https://github.com/SahilSira/RISC-V/assets/140998855/c80b3faa-488d-4651-995f-c5844c50b411)

*Lab on Decoding Instruction Field Set*
![20](https://github.com/SahilSira/RISC-V/assets/140998855/91a71f5e-b9dd-4c9d-8901-4c9045fe33c9)

-code for instruction based field set
```bash
	$rs2_valid = $is_r_instr || $is_s_instr || $is_b_instr;
         ?$rs2_valid
            $rs2[4:0] = $instr[24:20];
            
         $rs1_valid = $is_r_instr || $is_i_instr || $is_s_instr || $is_b_instr;
         ?$rs1_valid
            $rs1[4:0] = $instr[19:15];
         
         $funct3_valid = $is_r_instr || $is_i_instr || $is_s_instr || $is_b_instr;
         ?$funct3_valid
            $funct3[2:0] = $instr[14:12];
            
         $funct7_valid = $is_r_instr ;
         ?$funct7_valid
            $funct7[6:0] = $instr[31:25];
            
         $rd_valid = $is_r_instr || $is_i_instr || $is_u_instr || $is_j_instr;
         ?$rd_valid
            $rd[4:0] = $instr[11:7];
```     
- Output
![21](https://github.com/SahilSira/RISC-V/assets/140998855/1384fb2f-c1a7-41bd-a221-3b8fbbc8f917)
- VIZ
![22](https://github.com/SahilSira/RISC-V/assets/140998855/d4a1b2fc-a70d-4fca-b8b3-4c95bbfaa006)

*Lab on individual instruction decode*

- code
```bash
	 $dec_bits [10:0] = {$funct7[5], $funct3, $opcode};
         $is_beq = $dec_bits ==? 11'bx_000_1100011;
         $is_bne = $dec_bits ==? 11'bx_001_1100011;
         $is_blt = $dec_bits ==? 11'bx_100_1100011;
         $is_bge = $dec_bits ==? 11'bx_101_1100011;
         $is_bltu = $dec_bits ==? 11'bx_110_1100011;
         $is_bgeu = $dec_bits ==? 11'bx_111_1100011;
         $is_addi = $dec_bits ==? 11'bx_000_0010011;
         $is_add = $dec_bits ==? 11'b0_000_0110011;
```
![23](https://github.com/SahilSira/RISC-V/assets/140998855/96c07a93-7613-4b0c-a73e-c1b92ff7abb0)

- Implementaion of individual instruction decode
![24](https://github.com/SahilSira/RISC-V/assets/140998855/c6437bb3-58a0-4032-bd35-ea536aeedc52)

- VIZ
![25](https://github.com/SahilSira/RISC-V/assets/140998855/4d21afce-a0e4-45e8-976d-9d0055036205)

</details>

<details>
<summary>RISC-V control logic</summary>
	
Under this section, we will look into the implementation of RISC-V CPU from register file read 
onwards.	
**Execute and Register file read/write**

- Pipelined Logic diagram for implementation.
![26](https://github.com/SahilSira/RISC-V/assets/140998855/318f7922-a20a-4946-9c11-e1ab331332d0)

- Structure of the register design for implementation. Two read operations and one write operation to be performed.
![27](https://github.com/SahilSira/RISC-V/assets/140998855/2c70da0c-84bb-4e1b-b3ae-cee1a9aaf44b)

- Code for implementaion
```bash
	 $rf_wr_en = 1'b0;
         $rf_wr_index[4:0] = 5'b0;
         $rf_wr_data[31:0] = 32'b0;
         $rf_rd_en1 = $rs1_valid;
         $rf_rd_en2 = $rs2_valid;
         
         $rf_rd_index1[4:0] = $rs1;
         $rf_rd_index2[4:0] = $rs2;
```
- Now, we have read the register files, we will connect up the values we have read to the signals we will send to the ALU.
- Code for connection.
```bash
	 $src1_value[31:0] = $rf_rd_data1;
         $src2_value[31:0] = $rf_rd_data2;
```

*ALU Imlimentaion*
We move to the next stage of implementation, ie. ALU. The logic diagram.

![28](https://github.com/SahilSira/RISC-V/assets/140998855/e69e68a3-f50a-4569-bb51-eea8f0959fd3)

- code to implement the arithmatic and logic functionalities of the ALU.
```bash
	$result[31:0] = $is_addi ? $src1_value + $imm :
			$is_add ? $src1_value + $src2_value :
			32'bx ;
```

- Implementation upto ALU on Makerchip IDE
![29](https://github.com/SahilSira/RISC-V/assets/140998855/313f3e1b-1414-4f11-83f6-54d7800559c5)

*Register File Write Implementation*

Under this we go over the implementation of Write funcction after the ALU has performed.

- Logic Diagram
![30](https://github.com/SahilSira/RISC-V/assets/140998855/0c5ebe2d-f9bb-4cfc-ba27-5832df718f0d)

- Code for writing register file.
```bash
	$rf_wr_en = $rd_valid && $rd != 5'b0;
	$rf_wr_index[4:0] = $rd;
	$rf_wr_data[31:0] = $result;
```
- Implementaion of the register file logic
![31](https://github.com/SahilSira/RISC-V/assets/140998855/90960cb3-ebe2-4be9-8480-c795a41eb0cd)

**Note** : We will look into arrays under RISC-V.

- Arrays are collection of data of same datatypes.
- RISC-V processors support arrays through their memory access instructions and addressing modes. In the RISC-V architecture, arrays are commonly managed using a combination of memory locations and registers.
- Arrays are represented as contiguous blocks of memory in RISC-V.
- Pointers are used to track the memory address where the array starts.
- Arrays are accessed using pointers and offsets calculated from the index and element size.
- Load and store instructions are used to manipulate array elements.
- Pointer arithmetic involves adding offsets to pointers for accessing different elements.
- Pointers are initialized with the memory address of the array's first element.
- Array operations are performed through load-store instructions and pointer manipulation.

![32](https://github.com/SahilSira/RISC-V/assets/140998855/73a26720-4b66-4375-9e96-da9b6495eb80)

*Branch Instruction Imlementaion*
We will look into the implementations for the various branch instructions, we have decoded earlier.

- Logic Diagram
![33](https://github.com/SahilSira/RISC-V/assets/140998855/b07faf85-28b8-4cad-9b3f-c5a0b152e7e0)

- All instr with B initials are branch statements. Logic for the branches are as follow
![34](https://github.com/SahilSira/RISC-V/assets/140998855/7e4d2931-1a05-47e7-b0ed-a560450148bd)

- Code for implementing when to take a branch.
```bash
$taken_branch = $is_beq ? ($src1_value == $src2_value):
		$is_bne ? ($src1_value != $src2_value):
		$is_blt ? (($src1_value < $src2_value) ^ ($src1_value[31] != $src2_value[31])):
		$is_bge ? (($src1_value >= $src2_value) ^ ($src1_value[31] != $src2_value[31])):
		$is_bltu ? ($src1_value < $src2_value):
		$is_bgeu ? ($src1_value >= $src2_value):
		1'b0;
```

- Now, we will look into how to figure out where to branch to.
- Code to determine the path to Branch to
```bash
$br_target_pc[31:0] = $pc + $imm;
```

- implemenataion of branch instructions
![35](https://github.com/SahilSira/RISC-V/assets/140998855/df850853-06e9-4c02-9287-56afde52a38f)


</details>


## Day 5

<details> 
<summary>Pipelining the CPU</summary>

Now pipelining of the CPU core is done, which allows easy retiming and reduces functional bug to a great extent . Pipelining allows faster computaion. For pipelining as mentioned earlier we simply need to add @1, @2 and so on. The snapshot of the pipelining is as shown below. In TL verilog, another advantage is defining of pipeline in systematic order is not necessary. More inforamtion on timming abstract can be found in the IEEE paper "Timing-Abstract Circuit Design in Transaction-Level Verilog"  by Steeve Hoover in makerchip platform itself or else [here](https://ieeexplore.ieee.org/document/8119264).

**Implementing Cycle valid signal**

We are required to get implement the logic in the following block diagram:
![1](https://github.com/SahilSira/RISC-V/assets/140998855/7cb09515-9f77-40ca-ac43-b142f5a05d68)

- the following code is used for implementaion
```bash

$valid = $reset ? 1'b0 : ($start) ? 1'b1 : (>>3$valid) ;
         $start_int = $reset ? 1'b0 : 1'b1;
         $start = $reset ? 1'b0 : ($start_int && !>>1$start_int);
```

**Taking care of invalid cycles**

![4](https://github.com/SahilSira/RISC-V/assets/140998855/6eb6b771-ad66-4a7b-a196-849a25ddcbce)

- The code snippet required to implement is.
```bash
 $pc[31:0] = (>>1$reset) ? 32'b0 : (>>3$valid_taken_branch) ? (>>3$br_tgt_pc) :  (>>3$int_pc)  ;
 $valid_taken_branch = $valid && $taken_br;
```

**Distributing logic**

Pipelining is done in this step. Code is distributed and output is obtained.
![5](https://github.com/SahilSira/RISC-V/assets/140998855/73708584-c1a7-420c-96cd-208f9f56e469)


</details>

<details>
<summary>Correcting pipeline hazards</summary>

**Register file Bypass to address rd-after-wr hazard**

We are required to implement the logic as per the given figure.

![6](https://github.com/SahilSira/RISC-V/assets/140998855/f7ca74cf-737a-462c-9b80-1f9c54f9b14a)

The logic code required is given below.
```bash
$src1_value[31:0] = ((>>1$rf_wr_en) && (>>1$rd == $rs1 )) ? (>>1$result): $rf_rd_data1; 
$src2_value[31:0] = ((>>1$rf_wr_en) && (>>1$rd == $rs2 )) ? (>>1$result) : $rf_rd_data2;
```

**Correcting the branch target path**

The logic snippet required is given below.
```bash
 $pc[31:0] = (>>1$reset) ? 32'b0 : (>>3$valid_taken_br) ? (>>3$br_tgt_pc) :  (>>3$int_pc)  ;
         //$valid = $reset ? 1'b0 : ($start) ? 1'b1 : (>>3$valid) ; no need for this
```

</details>

<details>
<summary>Finalising the RISC-V CPU</summary>

Similar to branch,load will also have 3 cycle delay. So, added a Data Memory 1 write/read 
memory. Added test case to check the functionality of load/store.

```bash
	uncomment enable m4+dmem(@4)    // Args: (read/write stage)
 	connect interface signals using address bits[5:2] to perform load and store (when valid)
	Additionally Incorporation of Jump feature (JAL and JALR instructions).
```


Final Output : 

![2](https://github.com/SahilSira/RISC-V/assets/140998855/e99ca585-869a-4fee-a224-35508c09d0c1)

![3](https://github.com/SahilSira/RISC-V/assets/140998855/bf4bd47b-2179-459b-9fe6-72cad8b25b2e)

</details> 

## Word of Thanks
I sciencerly thank **Mr. Kunal Gosh**(Founder/**VSD**) for helping me out to complete this flow smoothly.

## Acknowledgement
- Kunal Ghosh, VSD Corp. Pvt. Ltd.
- Chatgpt
- Simarjeet Singh Thethi, Colleague, IIIT B
- Sumanto Kar, Sr. Project Technical Assistant, IIT Bombay
- Emil Jayanth Lal,Colleague, IIIT B
- Alwin Shaju, Colleague, IIITB
- Tuhsar Mavi, Colleague, IIITB
## Reference 
- https://www.vsdiat.com
- https://en.wikipedia.org/wiki/Toolchain
- https://en.wikipedia.org/wiki/GNU_toolchain
- https://github.com/riscv/riscv-gnu-toolchain
- https://redwoodeda.com
- https://github.com/stevehoover
