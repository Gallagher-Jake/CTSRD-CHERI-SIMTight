# SIMTight

SIMTight is a prototype GPGPU being developed on the [CAPcelerate
project](https://gow.epsrc.ukri.org/NGBOViewGrant.aspx?GrantRef=EP/V000381/1)
to explore the impact of [CHERI capabilities](http://cheri-cpu.org) on
SIMT-style accelerators popularised by NVIDIA and AMD.

The SIMTight SoC consists of a scalar CPU and a 32-lane 64-warp GPGPU
sharing DRAM, both supporting the CHERI-RISC-V ISA, though CHERI
support is entirely optional.

<img src="doc/SoC.svg" width="450">

The SoC is optimised for high performance density on FPGA (MIPS per
LUT).  A sample project is provided for the
[DE10-Pro](http://de10-pro.terasic.com) development board.  There is
also a [CUDA-like C++ library](soc/SIMTight/inc/NoCL.h) and a set of
sample [compute kernels](soc/SIMTight/apps/) ported to this library.
When CHERI is enabled, the kernels all run in pure capability mode.

## Standard Build

We'll need Verilator, the RISC-V SDK, and a fairly recent version
of GHC (8.6.1 or later).

On Ubuntu 20.04, we can simply do:

```sh
$ sudo apt install verilator
$ sudo apt install gcc-riscv64-unknown-elf
$ sudo apt install ghc-8.6.5
```

Now, we recursively clone the repo:

```sh
$ git clone --recursive https://github.com/CTSRD-CHERI/SIMTight
```

Inside the repo, there are various things to try.  For example, to
build and run the SIMTight simulator:

```sh
$ cd sim
$ make
$ ./sim
```

While the simulator is running, we can build and run the test suite
in a separate terminal:

```sh
$ cd apps/TestSuite
$ make test-cpu-sim     # Run on the CPU
$ make test-simt-sim    # Run on the SIMT core
```

Alternatively, we can run one of the SIMT kernels:

```sh
$ cd apps/Histogram
$ make RunSim
$ ./RunSim
```

To run all tests and benchmarks, we can use the test script.  This
script will launch the simulator automatically, so we first make sure
it's not already running.

```sh
$ killall sim
$ cd test
$ ./test.sh            # Run in simulation
```

To build an FPGA image (for the
[DE10-Pro](http://de10-pro.terasic.com) board):

```sh
$ cd de10-pro
$ make                 # Assumes quartus is in your PATH
$ make download-sof    # Assumes DE10-Pro is connected via USB
```

We can now run a SIMT kernel, almost exactly how we did so via the
simulator.

```sh
$ cd apps/Histogram
$ make Run
$ ./Run
```

To run the test suite and all benchmarks on FPGA:

```sh
$ cd test
$ ./test.sh --fpga     # Assumes FPGA image built and FPGA connected via USB
```

Notice that when running on FPGA, performance stats are also emitted.

## CHERI Build

To enable CHERI, a little bit of additional preparation is required.
First, edit [inc/Config.h](inc/Config.h) and:

  * change `#define EnableCHERI 0` to `#define EnableCHERI 1`;
  * change `#define UseClang 0` to `#define UseClang 1`.

Second, install the CHERI-Clang compiler using
[cheribuild](https://github.com/CTSRD-CHERI/cheribuild).  Assuming all
of [cheribuild's
dependencies](https://github.com/CTSRD-CHERI/cheribuild#pre-build-setup)
are met, we can simply do:

```sh
$ git clone https://github.com/CTSRD-CHERI/cheribuild
$ cd cheribuild
$ ./cheribuild.py sdk-riscv64-purecap
```

By default, this will install the compiler into `~/cheri/`.  We then
need to add the compiler to our `PATH`:

```sh
export PATH=~/cheri/output/sdk/bin:$PATH
```

At this point, all of the standard build instructions should run as
before.
