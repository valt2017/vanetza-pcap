# Socktap

*socktap* is an example application demonstrating API usage of Vanetza libraries.
You can run this demo application on commodity hardware, i.e. no special V2X or Car2X hardware is required.
If you have an IEEE 802.11p compatible network interface card, though, socktap can also communicate with other ITS-G5 stations.

Consider *socktap* as an experimental application showcasing some of Vanetza's features but not as a full-grown ITS-G5 station.
However, *socktap* may be the starting point for your custom application.
Just keep in mind that Vanetza supports more features than those integrated in *socktap*.
For example, *socktap* omits *Decentralized Congestion Control* (DCC) entirely though Vanetza supports this feature.


## Link layer

At the moment, six link layer implementations exist for *socktap*.
You can choose via the `--link-layer` argument which implementation to use:

- *ethernet* runs on Linux raw packet sockets
- *cube-evk* runs nfiniity's link-layer implementation for the cube-evk
- *autotalks* uses the Autotalks API to run on Autotalks hardware
- *cohda* employs Cohda's LLC API
- *udp* runs GeoNetworking on top of IP/UDP multicast sockets
- *tcp* runs GeoNetworking on top of IP/TCP sockets
- *rpc* uses *Cap’n Proto* interchange format and RPC with external server


### Ethernet

The *ethernet* variant has been initially *socktap*'s only available link-layer implementation.
In this mode, *socktap* will send and receiver Ethernet frames on the specified network interface (see `--interface` argument).

Please be aware that *socktap* needs special privileges to access the raw sockets.
Either run *socktap* as root user or set the `CAP_NET_RAW` capabilities on the *socktap* executable.
You can do this via `sudo setcap cap_net_raw+ep bin/socktap`.
When `CAP_NET_RAW` is attached to the *socktap* binary you can run it as an ordinary user.


### CUBE-EVK

If you have access to a cube-evk from [nfiniity](https://www.nfiniity.com/#portfolio) you can use socktap on your personal computer and connect to the EVK. In this mode the EVK is used as wireless remote radio. Moreover, you can use socktap on the EVK natively as well. Please refer to our [Running socktap on nfiniity devices](../recipes/cube-evk-build.md) for more details.


### Cohda

If you have access to V2X hardware from Cohda Wireless, you can also run *socktap* on their units.
In the *cohda* mode, *socktap* uses Cohda's LLC API for sending and receiving data frames.
This mode is similar to *ethernet* but depends on the Cohda SDK.
Please refer to our [Cohda SDK building recipe](../recipes/cohda-sdk-build.md) for details.


### Autotalks

Another option involving dedicated V2X hardware uses the Autotalks API.
Please have a look at our [Building for Autotalks devices](../recipes/autotalks-sdk-build.md) guide how to incorporate the Autotalks SDK.

> [!WARNING]
> Our Autotalks link layer integration is deprecated. Please use the RPC link layer as a superior replacement.


### UDP

A relatively new addition is the *udp* mode, which allows running *socktap* without any privileges.
GeoNetworking packets are wrapped into UDP datagrams and sent to the IP multicast group **239.118.122.97** on UDP port **8947**.
Further *socktap* instances within the same IP multicast network exchange GeoNetworking packets then.
You can consider this as "GeoNetworking over IP/UDP".


### TCP

The TCP implementation is similiar to the UDP one.
However, TCP adds the arguments `--tcp-accept` and `--tcp-connect`, which allow the user to accept incoming TCP connections or connect to open TCP sockets, respectively.
Both arguments expect a comma separated list of `ip:port`.
Outgoing GeoNetworking packets will then be sent to all active TCP connections.

### RPC

RPC link layer lets socktap connect to an external RPC server over [Cap’n Proto](https://capnproto.org/).
You can refer to [MACH SYSTEMS's RPC link](https://github.com/mach-systems/RPC-link) for developing the server on your own V2X hardware, e.g. Autotalks EVKs.
The [cube V2X devices](https://cubesys.io) ship a compatible *cube-radio-rpc* service with *cube:os* 1.4 and later.

#### Command line flags
- `--rpc-host <HOST>` –⁠⁠⁠⁠⁠ hostname or IP of the RPC server (default: `localhost`)
- `--rpc-port <PORT>` –⁠⁠⁠⁠⁠⁠ TCP port on which the server is listening (default: `23057`)
- `--rpc-radio-technology <TECH>` –⁠⁠⁠⁠ radio technology to advertise to the server; valid values:  
  - `ITS-G5` –⁠⁠⁠⁠⁠⁠ 802.11p / bd
  - `LTE-V2X|C-V2X` –⁠⁠⁠⁠⁠⁠ LTE-V2X / 5G-V2X
- `--rpc-debug` –⁠⁠⁠⁠⁠⁠ enable debugging output


## Positioning

Many components of an ITS-G5 system depend on positioning data.
Ideally, you have a GPS receiver attached to the computer running [gpsd](http://catb.org/gpsd) along with *socktap*.
If you do not have a GPS receiver or no GPS signal in your environment, you can also set a static position manually.

The `--positioning` argument controls if *gpsd* provides live GPS position fixes or if socktap shall use a *static* position fix.
See also the `--latitude` and `--longitude` arguments for the latter option.


## Applications

Earlier, multiple variants of *socktap* existed as separate executables.
Nowadays, *socktap* is the unified executable which can be configured in many ways.
These configuration options substitute the deprecated executables.

You can choose from three simple example V2X applications to run with *socktap* via the `--applications` argument:

- *ca* sends Cooperative Awareness Messages (CAM)
- *hello* sends simple BTP-B message with the binary payload `0xc0ffee`
- *benchmark* counts the number of any received messages and prints the current message rate once per second


## Building and Running

You need to enable the CMake option `BUILD_SOCKTAP` so *socktap* will be built at all.
Two further CMake options control which optional features are included into the *socktap* executable.


### Build options

First, the option `SOCKTAP_WITH_COHDA_LLC` enables the link-layer variant for operation on Cohda V2X devices.
This option usually makes only sense if you are cross-compiling *socktap* for a Cohda device.

If CMake finds *gpsd* on your system, the option `SOCKTAP_WITH_GPSD` is enabled by default.
The integration of GPS receivers depends on this option.
If this option is disabled, you can only configure static positions with *socktap*.


!!! warning
    A bug in gpsd<=3.15 causes a segmentation fault when *socktap* tries to fetch GPS data.
    More recent versions include a bugfix, e.g. gpsd>=3.17 is known to work.
    See also the corresponding [issue ticket #69](https://github.com/riebl/vanetza/issues/69).


### First steps

You can locate *socktap* in your build directory at **bin/socktap**.
Run this executable with `--help` appended to see the list of available runtime configuration options.
With the *ethernet* link-layer you should specify the network device on which *socktap* should send and receive packets via `--interface`.
Usually, such devices are named *eth0* or *wlan0*. You can look up the available devices on your machine with the `ip link` command.

If you want to use the local loopback device (usually `lo`) you need to override the used MAC address using `--mac-address` to receive packets.
Otherwise, the MAC address is `00:00:00:00:00:00` for both sides and the router drops incoming packets matching its address.


## Acknowledgement

This demo application has been initially developed as part of a student's project at Hochschule Darmstadt in summer term 2016.
Participating students were in alphabetical order: Sachin Kashyap Bukkambudhi Satyanarayana, Alvita Marina Menezes, Mrunmayi Parchure, Subashini Rajan and Deeksha Venkadari Yogendra.
Since then, [@kelunik](https://github.com/kelunik) and [@glmax](https://github.com/glmax) have contributed a lot to *socktap*.
