# RISC-V

This github repository summarizes the progress made in the RISCV - ISA program. Quick links:

[Day 1 : Introduction to RISC-V ISA and GNU compiler toolchain](#day-1)

[Day 2 : Introduction to ABI and Basic Verification Flow](#day-2)

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

<details >
<summary >Lab for Combinational logic </summary>

Makerchip IDE

Makerchip is a free online environment for developing high-quality integrated circuits. You can code, compile, simulate, and debug Verilog designs, all from your browser. Your code, block diagrams, and waveforms are tightly integrated.

## Loading pythagorean Example on Makerchip IDE

![1](https://github.com/SahilSira/RISC-V/assets/140998855/cd411bd7-db16-434d-8659-11a668f51145)

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

