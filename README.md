# GenomeWorks

## ATTENTION
This is a fork intended to add `meson` build support. This is work in progress.

For questions about GenomeWorks, please check out the original repo
https://github.com/clara-parabricks/GenomeWorks

## Build instructions
Requirements
 - [CUDA](https://developer.nvidia.com/CUDA-zone)
 - [ninja](https://ninja-build.org/)
 - python

Checkout this branch

```bash
git clone --recursive https://github.com/armintoepfer/GenomeWorks.git -b mesonify
```

Get meson with necessary meson patches. Will be included upstream at some point.

```bash
git clone https://github.com/SoapGentoo/meson.git -b cuda-fixes
```

Build and install library for Volta, Turing, and Ampere
```bash
cd GenomeWorks
mkdir build && cd build
../../meson/meson.py --prefix=${PWD}/install
sed -i -e 's/nvcc $ARGS/nvcc -DNDEBUG $ARGS/g' build.ninja
ninja
```

