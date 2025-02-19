# cpuid
## This repository is a Fork of the [github.com/klauspost/cpuid](https://github.com/klauspost/cpuid) project by user [klauspost](https://github.com/klauspost)

Package cpuid provides information about the CPU running the current program.

CPU features are detected on startup, and kept for fast access through the life of the application.
Currently x86 / x64 (AMD64/i386) and ARM (ARM64) is supported, and no external C (cgo) code is used, which should make the library very easy to use.

You can access the CPU information by accessing the shared CPU variable of the cpuid library.

Package home: https://github.com/arthurnavah/cpuid

[![PkgGoDev](https://pkg.go.dev/badge/github.com/arthurnavah/cpuid)](https://pkg.go.dev/github.com/arthurnavah/cpuid/v2)

## installing

`go get -u github.com/arthurnavah/cpuid/v2` using modules.

Drop `v2` for others.

## example

```Go
package main

import (
	"fmt"
	"strings"

	"github.com/arthurnavah/cpuid/v2"
)

func main() {
	// Print basic CPU information:
	fmt.Println("Name:", cpuid.CPU.BrandName)
	fmt.Println("PhysicalCores:", cpuid.CPU.PhysicalCores)
	fmt.Println("ThreadsPerCore:", cpuid.CPU.ThreadsPerCore)
	fmt.Println("LogicalCores:", cpuid.CPU.LogicalCores)
	fmt.Println("Family", cpuid.CPU.Family, "Model:", cpuid.CPU.Model, "Vendor ID:", cpuid.CPU.VendorID)
	fmt.Println("Features:", strings.Join(cpuid.CPU.FeatureSet(), ","))
	fmt.Println("Cacheline bytes:", cpuid.CPU.CacheLine)
	fmt.Println("L1 Data Cache:", cpuid.CPU.Cache.L1D, "bytes")
	fmt.Println("L1 Instruction Cache:", cpuid.CPU.Cache.L1I, "bytes")
	fmt.Println("L2 Cache:", cpuid.CPU.Cache.L2, "bytes")
	fmt.Println("L3 Cache:", cpuid.CPU.Cache.L3, "bytes")
	fmt.Println("Frequency", cpuid.CPU.Hz, "hz")

	// Test if we have these specific features:
	if cpuid.CPU.Supports(cpuid.SSE, cpuid.SSE2) {
		fmt.Println("We have Streaming SIMD 2 Extensions")
	}

}
```

Sample output:
```
>go run main.go
Name: AMD Ryzen 9 3950X 16-Core Processor
PhysicalCores: 16
ThreadsPerCore: 2
LogicalCores: 32
Family 23 Model: 113 Vendor ID: AMD
Features: ADX,AESNI,AVX,AVX2,BMI1,BMI2,CLMUL,CMOV,CX16,F16C,FMA3,HTT,HYPERVISOR,LZCNT,MMX,MMXEXT,NX,POPCNT,RDRAND,RDSEED,RDTSCP,SHA,SSE,SSE2,SSE3,SSE4,SSE42,SSE4A,SSSE3
Cacheline bytes: 64
L1 Data Cache: 32768 bytes
L1 Instruction Cache: 32768 bytes
L2 Cache: 524288 bytes
L3 Cache: 16777216 bytes
Frequency 0 hz
We have Streaming SIMD 2 Extensions
```

# usage

The `cpuid.CPU` provides access to CPU features. Use `cpuid.CPU.Supports()` to check for CPU features.
A faster `cpuid.CPU.Has()` is provided which will usually be inlined by the gc compiler.

Note that for some cpu/os combinations some features will not be detected.
`amd64` has rather good support and should work reliably on all platforms.

Note that hypervisors may not pass through all CPU features.

## arm64 feature detection

Not all operating systems provide ARM features directly
and there is no safe way to do so for the rest.

Currently `arm64/linux` and `arm64/freebsd` should be quite reliable.
`arm64/darwin` adds features expected from the M1 processor, but a lot remains undetected.

A `DetectARM()` can be used if you are able to control your deployment,
it will detect CPU features, but may crash if the OS doesn't intercept the calls.
A `-cpu.arm` flag for detecting unsafe ARM features can be added. See below.

Note that currently only features are detected on ARM,
no additional information is currently available.

## flags

It is possible to add flags that affects cpu detection.

For this the `Flags()` command is provided.

This must be called *before* `flag.Parse()` AND after the flags have been parsed `Detect()` must be called.

This means that any detection used in `init()` functions will not contain these flags.

Example:

```Go
package main

import (
	"flag"
	"fmt"
	"strings"

	"github.com/arthurnavah/cpuid/v2"
)

func main() {
	cpuid.Flags()
	flag.Parse()
	cpuid.Detect()

	// Test if we have these specific features:
	if cpuid.CPU.Supports(cpuid.SSE, cpuid.SSE2) {
		fmt.Println("We have Streaming SIMD 2 Extensions")
	}
}
```

## commandline

Download as binary from: https://github.com/arthurnavah/cpuid/releases

Install from source:

`go install github.com/arthurnavah/cpuid/v2/cmd/cpuid@latest`

### Example

```
λ cpuid
Name: AMD Ryzen 9 3950X 16-Core Processor
Vendor String: AuthenticAMD
Vendor ID: AMD
PhysicalCores: 16
Threads Per Core: 2
Logical Cores: 32
CPU Family 23 Model: 113
Features: ADX,AESNI,AVX,AVX2,BMI1,BMI2,CLMUL,CLZERO,CMOV,CMPXCHG8,CPBOOST,CX16,F16C,FMA3,FXSR,FXSROPT,HTT,HYPERVISOR,LAHF,LZCNT,MCAOVERFLOW,MMX,MMXEXT,MOVBE,NX,OSXSAVE,POPCNT,RDRAND,RDSEED,RDTSCP,SCE,SHA,SSE,SSE2,SSE3,SSE4,SSE42,SSE4A,SSSE3,SUCCOR,X87,XSAVE
Microarchitecture level: 3
Cacheline bytes: 64
L1 Instruction Cache: 32768 bytes
L1 Data Cache: 32768 bytes
L2 Cache: 524288 bytes
L3 Cache: 16777216 bytes

```
### JSON Output:

```
λ cpuid --json
{
  "BrandName": "AMD Ryzen 9 3950X 16-Core Processor",
  "VendorID": 2,
  "VendorString": "AuthenticAMD",
  "PhysicalCores": 16,
  "ThreadsPerCore": 2,
  "LogicalCores": 32,
  "Family": 23,
  "Model": 113,
  "CacheLine": 64,
  "Hz": 0,
  "BoostFreq": 0,
  "Cache": {
    "L1I": 32768,
    "L1D": 32768,
    "L2": 524288,
    "L3": 16777216
  },
  "SGX": {
    "Available": false,
    "LaunchControl": false,
    "SGX1Supported": false,
    "SGX2Supported": false,
    "MaxEnclaveSizeNot64": 0,
    "MaxEnclaveSize64": 0,
    "EPCSections": null
  },
  "Features": [
    "ADX",
    "AESNI",
    "AVX",
    "AVX2",
    "BMI1",
    "BMI2",
    "CLMUL",
    "CLZERO",
    "CMOV",
    "CMPXCHG8",
    "CPBOOST",
    "CX16",
    "F16C",
    "FMA3",
    "FXSR",
    "FXSROPT",
    "HTT",
    "HYPERVISOR",
    "LAHF",
    "LZCNT",
    "MCAOVERFLOW",
    "MMX",
    "MMXEXT",
    "MOVBE",
    "NX",
    "OSXSAVE",
    "POPCNT",
    "RDRAND",
    "RDSEED",
    "RDTSCP",
    "SCE",
    "SHA",
    "SSE",
    "SSE2",
    "SSE3",
    "SSE4",
    "SSE42",
    "SSE4A",
    "SSSE3",
    "SUCCOR",
    "X87",
    "XSAVE"
  ],
  "X64Level": 3
}
```

### Check CPU microarch level

```
λ cpuid --check-level=3
2022/03/18 17:04:40 AMD Ryzen 9 3950X 16-Core Processor
2022/03/18 17:04:40 Microarchitecture level 3 is supported. Max level is 3.
Exit Code 0

λ cpuid --check-level=4
2022/03/18 17:06:18 AMD Ryzen 9 3950X 16-Core Processor
2022/03/18 17:06:18 Microarchitecture level 4 not supported. Max level is 3.
Exit Code 1
```

# license

This code is published under an MIT license. See LICENSE file for more information.
