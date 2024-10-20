# Stream audio from the PC to the Android phone

Stream audio from the PC to the Android phone with an ultra low latency audio relay.

This repository provides the PC audio forwarder and listening server in Rust. Also an Android app in Java is provided, with an audio receiver implemented as a native library in Rust.


## Building

The build is performed in Docker containers, and is entirely orchestrated by a CMake build system:

```
mkdir build
cd build
cmake ..
make
```

