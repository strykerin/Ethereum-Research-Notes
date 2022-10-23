# Notes

This repos contains my notes about my research on the EVM.

## Overview

<https://ethervm.io/>

> The Ethereum VM is a stack-based, big endian VM with a word size of 256 bits and is used to run the smart contracts.
>
> Smart contracts are like regular accounts, except they run EVN bytecode when receiving a transaction, allowing them to perform calculations and further transactions. Transactions can carry a payload of 0 or more bytes of data, which is used to specify the type of interaction with a contract and any additional information.

## Opcodes

> Each opcode is encoded as one byte, except for the `PUSH` opcodes.
>
> All opcodes pop their operands from the top of the stack and push their result.

## Machine space of EVM

<https://www.youtube.com/watch?v=kCswGz9naZg>

- Stack: 256 bits x 1024 elements
- Memory: volatile memory. byte addressing. linear memory
- Storage: persistent memory. 256 bits to 256 bits. key-value store
