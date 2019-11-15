# retdec-api-demo

Small demo that shows how to use RetDec API Poc in other projects.

## Requirements

* Works only on Linux.
* Everything that [RetDec](https://github.com/avast/retdec#requirements) requires.

## Build

```
mkdir build
cd build
cmake .. -DCMAKE_INSTALL_PREFIX=<install_path>
make -j<N>
make install
```

## Use

```
<install_path>/bin/demo -i tests/<file>
```
