# compiler-rt-builtins (source package)

The routines a compiler emits calls to — division, comparison, the `__aeabi_*`
helpers — **compiled with the consuming program's own flags**.

```toml
[dependencies]
compiler-rt-builtins = "22.1.8"
```

mcpp reports `compiler-runtime compiler-rt (compiler-rt-builtins@22.1.8, graph)`.

## ⭐ Why it is a separate package from the C library

picolibc's `printf` formats floats through ryu, which calls routines no C
library defines — and on rv64 a 128-bit shift the instruction set has no
instruction for. A C library carrying its own copy would be wrong for anyone
supplying their own builtins. The dependency edge says the same thing and can be
overridden.

## ⚠️ compiler-rt does not recognise a `thumb*` triple

Configuring upstream's CMake with `thumbv6m-none-eabi` produces a build tree
with **no builtins target at all**: cmake succeeds, ninja reports "no work to
do", and the failure surfaces much later as a missing file. Only
`armv6m-none-eabi` works, and it then names the archive after *that* spelling.

A source package has neither problem: there is no archive to name and no triple
to translate.

## ⚠️ One list for all seven profiles, and it is the smallest one

Upstream picks per CPU, and the sets are nested rather than equal — 25 files on
armv6-m, 54 on armv7-m, 72 on armv7e-m, 90 on armv8-m.main. The extra ones are
Thumb-2 and VFP assembly, and assembling them on a smaller core fails outright:

```
umodsi3.S:35: error: conditional execution not supported in Thumb1
save_vfp_d8_d15_regs.S:29: error: instruction requires: fp registers
```

So the list is the **intersection**, and the routines it leaves out are supplied
by the generic C — those assembly files are optimisations, not the only
implementation. Measured: every file in the list assembles on `thumbv7m` as well
as on `thumbv6m`.

⚠️ That is a cost, stated: a Cortex-M33 gets the portable division routine rather
than its own. Seven profiles that all work was preferred to four that are faster.

## Licence

Apache-2.0 WITH LLVM-exception, as compiler-rt. See `LICENSE.compiler-rt`.
