# ava-doc

# Running a C program on RISC-V

As the first step we'll write a C program and compile it to 32 bit RISC-V 
architecture (RV32I) and  run the program using the Spike RISC-V ISA simulator. 

## Build the Toolchain

Get the source and enter in to the source directory
```
git clone https://github.com/riscv/riscv-gnu-toolchain.git
cd riscv-gnu-toolchain
```
Initialize the submodules
```
git submodule init
git submodule update --init --recursive --progress
```

We'll use the `VERSION="620887bea83ed9016c7552f72ac72e908b2c105a"` of the toolchain.
```
git checkout $VERSION
```

I'll use `$RISCV = /opt/riscv` as the installation directory.

Configure and start the build
``` 
./configure --disable-linux --disable-multilib --disable-gdb --prefix=$RISCV --with-arch=rv32imc --with-abi=ilp32
make -j4
```

NOTE: 
Recent versions of each which don't support the RISC-V Vector ISA, 
or older versions which do have RVV support.

## Compile a C Program

Programs can be compiled and run in three [different modes](https://lowrisc.org/docs/untether-v0.2/riscv_compile/):
- Bare metal mode: supervisor programs with no I/O accesses. Programs run in this mode have no peripheral support. 
- Newlib mode: supervisor programs **with access to I/O devices**. single-threaded. Bootloaders are run in this mode.
- Linux mode: user programs with Linux support.

We will compile a simple hello world program in Linux mode.
```
#include <stdio.h>

void main()
{
    const char *s = "Hello.\n";
    while (*s) putchar(*s++);
    while(1);
}
```

Compile the program.
```
riscv32-unknown-elf-gcc -o hello hello.c
```

## Test the Program

There are four ways to test a program:
- RISC-V ISA Simulator (Spike).
- RTL Simulation (Verilator): No I/O devices is available in RTL simulation.
- FPGA Simulation: simulate the program using Xilinx ISim
- FPGA Board: Full I/O support (UART and SD).

The ISA simulation is sufficient for us to test the program. 
Follow the instruction to install Spike.

Install dependencies
```
sudo apt update
sudo apt install device-tree-compiler -y 
```

Clone the source
```
git clone https://github.com/riscv/riscv-isa-sim.git
```

Create the build directory
```
mkdir -p ./riscv-isa-sim/build
cd ./riscv-isa-sim/build
```

Configure the build and start building 
```
../configure --prefix=$RISCV
make -j4
```

Install
```
sudo make install -j4
```

As we compiled the program for Linux mode, we need a kernel.
The RISC-V Proxy Kernel([pk](https://github.com/riscv-software-src/riscv-pk)), 
is a lightweight application execution environment that can 
host statically-linked RISC-V ELF binaries. Install `pk` from source.

Clone source
```
git clone https://github.com/riscv/riscv-pk.git
```

Create the build directory
```
mkdir -p ./riscv-pk/build
cd ./riscv-pk/build
```

Configure the build, start building and install
```
../configure --prefix=$RISCV --host=riscv32-unknown-elf
make -j4
sudo make install
```
Now everything is ready to run the program. 

Spike uses the `RV64IMAFDC` ISA by default. So to run a 32bit ISA run it as follows,
```
spike --isa=RV32IMAFDC pk hello
```

Program output:
![hello-world-output](img/hello-world.png)

# Next

Use a real processor with Verilator: CV32E40P

Generate the Verilator model of the Standard CV32E40P.
```
git clone https://github.com/openhwgroup/core-v-verif.git
cd ./core-v-verif/cv32e40p/sim/core
make
```
