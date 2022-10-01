2022-09-29 20:15
source: [source]()
tags: #scratch

#  2022-09-29 4crpmtc8Ppm4

## Volatile


**Memory models** say..
- registers are faster to access than RAM.
- Without volatile, variables may be saved on multiple registers if multiple functions reference it.
- which has the correct variable then???



C keyword ex. volatile int x
which tells the compiler to strictly load and store the value on every use, rather than storing the value in a register.

This makes sure the value isn't lost due to other computations on the register.

