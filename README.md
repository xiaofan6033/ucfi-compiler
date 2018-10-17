# ucfi-compiler

## Introdocution

ucfi-compiler compiles project source code into a hardened version, so that ucfi-monitor can protect its execution from control-flow hijacking attacks. It contains three components: LLVM pass, X86 backend and ptwrite emulator.

#### LLVM pass

The related code of LLVM pass is in `llvm/lib/Transforms/NewCPSensitivePass`. The LLVM pass will complete the follows tasks:

1. Identify and instrument constraining data so that the monitor knows its value

2. Instrument each sensitive basic block (containing at least one control-instruction) to dump its unique ID

3. Change function attribute to avoid tail call optimization

4. Replace all indirect function call with a direct call to a specific function, where an indirect jump helps reach the real target

#### X86 backend

The X86 backend achieves two tasks

1. Redirect each RET instruction to an dedicated RET instruction

2. Provide a simple parallel shadow stack implementation proposed in [1] 

   Note: ucfi just adopts parallel shadow stack to demonstrate the compatibility with shadow stack solutions. We do not claim any contribution or guarantee on protecting the return address. All design novelties go to its orignal authors. This implementation is just a simple version written by ucfi authors. Any implementation bugs go to ucfi authors. Due to implementation difference, overhead of parallel shadow stack could be different from that reported in the original paper.

#### ptwrite emulator

ptwrite emulator helps dump arbitrary value (even non-control-flow data) into Intel PT trace. It mainly supports two features

1. A dedicated RET instrution to help achieve return for all functions

2. A dedicated JMP instruction to help achieve indirect function call

3. A code region of all RET instructions to achieve the ptwrite emulation

4. Help setup the parallel shadow stack

## Build Instructions

1. git pull this repo to local system

    `git pull git@github.com:uCFI-GATech/ucfi-compiler.git`
    
    `cd ucfi-compiler`
    
2. create build and install folder

    `make {build,install}`
    
    `cd build`
    
3. configuration the compilation

    `cmake -DCMAKE_INSTALL_PREFIX=../install -DCMAKE_BUILD_TYPE=Release -DLLVM_TARGETS_TO_BUILD=X86 ../llvm/`
    
4. build the compiler and install it

    `make -j 8`
    
    `make install`
    
Now you should have llvm & clang toolchains available in the `install` folder. run `source shrc` in the root directory to add the installation folder into `PATH`

## Compile a hello world

## References

[1] Thurston H.Y. Dang, Petros Maniatis, and David Wagner. 2015. The Performance Cost of Shadow Stacks and Stack Canaries. In Proceedings of the 10th ACM Symposium on Information, Computer and Communications Security (ASIA CCS '15). ACM, New York, NY, USA, 555-566. DOI: https://doi.org/10.1145/2714576.2714635
