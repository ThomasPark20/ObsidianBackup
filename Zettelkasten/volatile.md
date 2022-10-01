2022-09-29 20:15
source: [source]()
tags: #scratch

#  2022-09-29 4crpmtc8Ppm4

## Volatile


**Memory models** say..
- registers are faster to access than RAM.
- Without volatile, variables may be saved on multiple registers if multiple functions reference it.
- which has the correct variable then???

**volatile disables this optimization. forcing the values to be loaded from and stored to memory with each use**


