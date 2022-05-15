---
layout: post
title: "Getting started with Migen | A python based HDL"
author_github: Utkar5hM
date: 2021-03-24 00:00:00
image: 'https://m-labs.hk/images/migen@2x.png'
description: A Python toolbox for building complex digital hardware
tags:
- IEEE NITK
- Blog
- Migen
- Verilog
- HDL
- VHDL
- LiteX
categories:
- Diode
github_username: 'Utkar5hM'
---

# Working with Migen | Python based Hardware Description Language(HDL)

## Introduction

In digital design, the complexity of designing a circuit gave birth to standard languages to describe digital circuits (i.e. Hardware Description Languages - HDL).  HDLs are used to model hardware elements very concurrently. Verilog HDL and VHDL are the most popular HDLs.

Digital circuits are described at Registers Transfer Level (RTL) using HDL. Then logic synthesis tool will generate details of gates and interconnection to implement courses. This synthesized result can be used for fabrication by having placement and routing components, and later its functionality can be verified using simulation.

Since the creation of traditional hardware descriptive languages(HDL) like Verilog and VHDL, there have not been many improvements.
They are old and provide a nonintuitive way to design hardware. 

Migen, on the other hand, created by M-Labs, is easier to learn and use.
It reduces syntax and helps to reduce error-prone by automating things like reset, allowing fast hardware prototyping.



In this blog, We will go through the steps of installing Migen on Linux, running a bare code for OR gate, and later see some fundamental basics of Migen.

## Installation

We will be installing Migen on Linux. As Migen is a based on Python, 
It is necessary to have Python installed. 
Python can be easily installed on Linux with respective package managers like Pacman or apt-get. 

We will first clone Migen's Git repository, Then Install it with Python3.

```bash
  git clone http://github.com/m-labs/migen
  cd migen
  python3 ./setup.py develop --user
```
    
## Getting Started With Migen

Basic Principle: We use migen to create a list of combinatorial and synchronous Assignments.

Like
```python
# for Combinational
self.comb += []
# for Synchronous
self.sync += [] 
```

each hardware module(in cotnext of Verilog) here is a Module object instatiated as a class.
So we can declare IO etc anywhere in the module(class) declaration.

subModules can be added to the current module with.
```python
my_submodule = MySubmodule()
self.submodules +=my_submodule
```

The Top Module can be converted into Verilog code with
```python
verilog.convert()
```

Signals (Registers /Wires) can be declared with 
```python
Signal(nbits)
#dont forget self. in actual code.
```
If we assign this to a sync list, then the signal becomes a register. like here
```python
counter = signal(26);
self.sync += counter.eq(counter +1)
```

if we assign signal to a comb list, then it acts like a wire. like here:
```python
counter = signal(26);
self.comb += led.eq(counter[25])
```

#### In Migen, All assignments in sync are effective on a rising edge of the clock.
By default, If we just use self.sync. 
the end design will have a single clock but if multiple clocks are required
then they can be setup.

## Usage/Example

To design a circuit with Migen, We will create a python file and code a simple combinational circuit( OR gate )
and see how it compares to the later generated Verilog Code.

### OR Gate Module
```python
from migen import *

class ORGate(Module):
  def __init__(self):
    self.a = Signal()
    self.b = Signal()
    self.x = Signal()

    ###

    self.comb += self.x.eq(self.a | self.b)

dut = ORGate()
```
### Testbench for the Module.
```python
def testbench():
  yield dut.a.eq(0)
  yield dut.b.eq(0)
  yield
  assert (yield dut.x) == 0

  yield dut.a.eq(0)
  yield dut.b.eq(1)
  yield
  assert (yield dut.x) == 1

```

### For Simulation (writing outputs to a file)
```python
run_simulation(dut, testbench(), vcd_name="file.vcd")
```

### Converting Python Code into a synthesizable Verilog Code
```python
if __name__ == "__main__":
  from migen.fhdl.verilog import convert
  convert(dut).write("my_design.v")
```

After having the code into a single file(ex- test.py). We can run it using python3 by using the following command.
```bash
python3 test.py
```
## Output File Generated

### my_design.v (Verilog Code Generated)
```verilog
/* Machine-generated using Migen */
module top(

);

reg a = 1'd0;
reg b = 1'd0;
wire x;

// synthesis translate_off
reg dummy_s;
initial dummy_s <= 1'd0;
// synthesis translate_on

assign x = (a | b);

endmodule

```



### Simulation Log File generated with testbench code.
```
$scope module ORGate $end
$var wire 1 ! a $end
$var wire 1 " b $end
$var wire 1 # x $end
$var wire 1 $ sys_clk $end
$enddefinitions $end
$dumpvars
$end
#0
0!
0"
0#
0$
#5
1$
#10
0$
#15
1"
1#
1$
#20
0$
#25
1$
0!
0"
0#
0$

```
We can easily confirm the proper working of the python code By analyzing the output log file generated with the testbench inputs.

## Conclusion

From the generated Verilog file, We can see that the generated code is similar to what we would manually write.

In the case of Sequential complex code, Python takes care of stuff like reset and synchronization with the clock.
Migen gives the developer more time to work on the logic of the hardware  while not having to deal with many of the restrictions that
are present while working with traditional Hardware Description Languages.

When Migen is used with tools like LiteX, which is also an open-source project/ It can be pretty powerful and directly implement our design onto SOCs and FPGAs. 


## Resources

- https://m-labs.hk/migen/manual/index.html
- https://github.com/enjoy-digital/litex
- https://github.com/m-labs/migen
 




