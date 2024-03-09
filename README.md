# Parallelism in Zmap

This document serves as the project report for zmen002 in CS254 2024 Winter.

[Zmap](https://github.com/zmap/zmap) is an open-source network scanner specifically to perform Internet-wide scans and capable of surveying the entire IPv4 address space. 

This project aims to explore the possibility to add the parallelism for Zmap, such as generic operations, data manipulation and etc, mostly in the preprocessing and the post-processing phase. More details regarding the implementation framework, please see project proposal, mid report and the video presentation. 

The project is conducted on a Linux machine running Ubuntu 22.04. Not tested on other machines. Please contact `zmen002@ucr.edu` if you have any issues running the code.

## Prerequisites

First pull this repository:

```bash
git clone git@github.com:RomaLzhih/zmap_x.git
cd zmap_x
```

### Installing the OpenCilk

Now we are under the directory of `zmap_x`.

The project utilizes `OpenCilk` to provide the parallelism, to install:

1. First download the tarball (for Linux x86_64 users):

```bash
wget https://github.com/OpenCilk/opencilk-project/releases/download/opencilk/v2.1/opencilk-2.1.0-x86_64-linux-gnu-ubuntu-22.04.tar.gz
```

- If you are a macOS user, check the [official document](https://www.opencilk.org/doc/users-guide/install/#download-1) in `opencilk` to download the tarball, which should be either the arm64 version:

  ```bash
  wget https://github.com/OpenCilk/opencilk-project/releases/download/opencilk/v2.1/opencilk-2.1.0-arm64-apple-darwin21.6.0.tar.gz
  ```

  or the x86_64 version:

  ```bash
  wget https://github.com/OpenCilk/opencilk-project/releases/download/opencilk/v2.1/opencilk-2.1.0-x86_64-apple-darwin21.6.0.tar.gz
  ```

2. After that, extract the OpenCilk 2.1 from the downloaded tarball to the current folder.

   For example:

   ```bash
   tar xvzf opencilk-2.1.0-x86_64-linux-gnu-ubuntu-22.04.tar.gz
   ```

​	For macOS user, please change the tarball name to the one that you downloaded. 

3. then rename it:

   ```bash
   mv opencilk-2.1.0-x86_64-linux-gnu-ubuntu-22.04 opencilk
   ```

4. Check whether things works:

   ```bash
   opencilk/bin/clang --version
   ```

   which should output the compiler version as clang 16.0.6.

### Installing dependencies for Zmap

For Ubuntu user:

```bash
sudo apt-get install build-essential cmake libgmp3-dev gengetopt libpcap-dev flex byacc libjson-c-dev pkg-config libunistring-dev libjudy-dev
```

For macOS user (using Homebrew):

```bash
brew install pkg-config cmake gmp gengetopt json-c byacc libunistring judy
```

For other OS user, checkout the [official document](https://github.com/zmap/zmap/blob/main/INSTALL.md#building-from-source) for Zmap.

## Compiling

First navigate to the directory of `zmap_x`.

To compile the code, try:

```{shell}
mkdir bin
mkdir build; cd build
cmake -DCMAKE_C_COMPILER=${PWD}/../opencilk/bin/clang -DCMAKE_INSTALL_PREFIX=../bin -DENABLE_DEVELOPMENT=OFF ..
make zmap
make install
```

The compiled binary is available in `zmap_x/bin/sbin/zmap`.  To verify, try:

```bash
../bin/sbin/zmap --version
```

 ## Running

Assume now we are in the root directory of `zmap_x`.

1. To send a `TCP SYN` packet to all IP's in the subnet `171.67.70.0/23` on port 80 with a packet send rate of 128 packets per second, try:

   ```bash
   bin/sbin/zmap -p 80 -r 128 171.67.70.0/23
   ```

   If the error messages says `Permission denied`, try:

   ```bash
   sudo bin/sbin/zmap -p 80 -r 128 171.67.70.0/23
   ```

   You should get some output similar to:

   ```bash
   Time(PARALLEL: iterator get sent) = 0.000006492 sec
   Time(PARALLEL: iterator get iterations) = 0.000000161 sec
   Time(PARALLEL: iterator get failed) = 0.000000171 sec
    0:00 0%; send: 4 1 p/s (109 p/s avg); recv: 0 0 p/s (0 p/s avg); drops: 0 p/s (0 p/s avg); hitrate: 0.00%
   Time(PARALLEL: iterator get sent) = 0.000003096 sec
   Time(PARALLEL: iterator get iterations) = 0.000000361 sec
   Time(PARALLEL: iterator get failed) = 0.000000310 sec
    0:01 9%; send: 131 127 p/s (126 p/s avg); recv: 0 0 p/s (0 p/s avg); drops: 0 p/s (0 p/s avg); hitrate: 0.00%
   Time(PARALLEL: iterator get sent) = 0.000002455 sec
   Time(PARALLEL: iterator get iterations) = 0.000000251 sec
   Time(PARALLEL: iterator get failed) = 0.000000230 sec
    0:02 17%; send: 259 128 p/s (127 p/s avg); recv: 0 0 p/s (0 p/s avg); drops: 0 p/s (0 p/s avg); hitrate: 0.00%
   Time(PARALLEL: translate fieldset) = 0.000000070 sec
   171.67.71.47
   Time(PARALLEL: fs free) = 0.001129358 sec
   Time(PARALLEL: translate fieldset) = 0.000000030 sec
   171.67.71.50
   Time(PARALLEL: fs free) = 0.000176310 sec
   Time(PARALLEL: fs free) = 0.001071660 sec
   Time(PARALLEL: translate fieldset) = 0.000000020 sec
   171.67.70.210
   ```

   **The output lines tagged with `Time(PARALLEL: )` are the running time of modules or subroutines that were re-implemented in parallel by this project.**

2. To find 5 HTTP servers, scanning at 10 Mb/s, on TCP port 80, try: 

   ```bash
   sudo bin/sbin/zmap -N 5 -B 10M -p 8
   ```

   which should output something similar to:

   ```bash
   Time(PARALLEL: iterator get sent) = 0.000007594 sec
   Time(PARALLEL: iterator get iterations) = 0.000000230 sec
   Time(PARALLEL: iterator get failed) = 0.000000161 sec
    0:00 0%; send: 2 1 p/s (40 p/s avg); recv: 0 0 p/s (0 p/s avg); drops: 0 p/s (0 p/s avg); hitrate: 0.00%
   Time(PARALLEL: translate fieldset) = 0.000000070 sec
   104.123.78.242
   Time(PARALLEL: fs free) = 0.001160206 sec
   Time(PARALLEL: fs free) = 0.000141496 sec
   Time(PARALLEL: translate fieldset) = 0.000000071 sec
   104.206.107.141
   Time(PARALLEL: fs free) = 0.000286628 sec
   Time(PARALLEL: translate fieldset) = 0.000000080 sec
   103.61.22.210
   Time(PARALLEL: fs free) = 0.000686648 sec
   Time(PARALLEL: fs free) = 0.000021210 sec
   Time(PARALLEL: translate fieldset) = 0.000000051 sec
   137.52.141.59
   Time(PARALLEL: fs free) = 0.001138877 sec
   Time(PARALLEL: fs free) = 0.000145443 sec
   Time(PARALLEL: translate fieldset) = 0.000000081 sec
   23.50.51.18
   Time(PARALLEL: fs free) = 0.000533701 sec
   Time(PARALLEL: iterator get sent) = 0.000003116 sec
   Time(PARALLEL: iterator get iterations) = 0.000000612 sec
   Time(PARALLEL: iterator get failed) = 0.000000360 sec
    0:01 100%; send: 15739 15.2 Kp/s (14.5 Kp/s avg); recv: 5 5 p/s (4 p/s avg); drops: 0 p/s (0 p/s avg); hitrate: 0.03%
   Mar 08 17:12:51.013 [INFO] zmap: completed
   ```

More possible commands are available at Zmap official sites (not all tested).  
