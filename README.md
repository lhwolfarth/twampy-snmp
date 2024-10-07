# Python tools for TWAMP and TWAMP light
Twampy is a Python implementation of the Two-Way Active Measurement
Protocol (TWAMP and TWAMP light) as defined in RFC5357. This tool
has small customizations from original repository https://github.com/nokia/twampy

## Supported features
* unauthenticated mode
* IPv4 and IPv6
* Support for DSCP, Padding, JumboFrames, IMIX
* Support to set DF flag (don't fragment)
* Basic Delay, Jitter, Loss statistics (jitter according to RFC1889)

##  Modes of operation
* TWAMP Controller
* TWAMP Control Client
* TWAMP Test Session Sender
* TWAMP light Reflector

## Install Python3
sudo apt install python3

## Installation
```
$ git clone https://github.com/lhwolfarth/twampy-snmp
Cloning into 'twampy'...
```

##  Usage Notes
Use padding to configure unirectional packet/frame sizes:

Agent | IP Version | Packet Size | Frame Size
:---:|:---:| --- | ---
Sender | IPv4 | Padding+14 | Padding+28
Responder | IPv4 | Padding+38 | Padding+52
Sender | IPv6 | Padding+14 | Padding+28
Responder | IPv6 | Padding+38 | Padding+52

Default Padding is 50 bytes.

Use padding value '-1' for IMIX traffic generation:

L2 Size | Packets | Ratio(Packets) | Ratio(Volume)
---:|:---:| ---:| ---:
64 | 7 | 58% | 10%
590 | 4 | 33% | 55%
1514 | 1 | 8% | 35%

TOS/DSCP user settings neet to be enabled on WINDOWS:
1. Open Registry Editor
2. Go to key:
      HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\TcpIp\Parameters
3. Create new DWORD value:

EntryName | Value
--- | ---
DisableUserTOSSetting | 0x00000000 (0)

4. Quit Registry Editor
5. Restart you computer
6. Command prompt for validation (capture needed)

      $ ping <ipaddress> -v 8
      
Reference: http://support.microsoft.com/kb/248611

DF flag implementation supports Linux und Windows. To support other
Operating Systems such as OS X (darwin) or FreeBSD the according
code such as sockopts need to be added and validated.

## Possible Improvements
* authenticated and encrypted mode
* sending intervals variation
* enhanced statistics
  * bining and interim statistics
  * late arrived packets
  * smokeping like graphics
  * median on latency
  * improved jitter (rfc3393, statistical variance formula):
    jitter:=sqrt(SumOf((D[i]-average(D))^2)/ReceivedProbesCount)
* daemon mode: NETCONF/YANG controlled, ...
* enhanced failure handling (catch exceptions)
* per probe time-out for statistics (late arrival)
* Validation with other operating systems (such as FreeBSD)
* Support for RFC 5938 Individual Session Control
* Support for RFC 6038 Reflect Octets Symmetrical Size

## Error codes (as per RFC 4656)
Error Code | Description
--- | ---
0 | OK
1 | Failure, reason unspecified (catch-all).
2 | Internal error.
3 | Some aspect of request is not supported.
4 | Cannot perform request due to permanent resource limitations.
5 | Cannot perform request due to temporary resource limitations.

## Usage example: getting help
Help on modes of operation:
```
$ sudo python3 ./twampy.py --help
usage: twampy.py [-h] [-v]
                 {responder,sender,controller,controlclient,dscptable} ...

positional arguments:
  {responder,sender,controller,controlclient,dscptable}
                        twampy sub-commands
    responder           TWL responder
    sender              TWL sender
    controller          TWAMP controller
    controlclient       TWAMP control client
    dscptable           print DSCP table

optional arguments:
  -h, --help            show this help message and exit
  -v, --version         show program's version number and exit
```

Specific help:
```
$ sudo python3 ./twampy.py sender --help
usage: twampy.py sender [-h] [-l filename] [-q | -v | -d]
                        [--tos type-of-service] [--dscp dscp-value]
                        [--ttl time-to-live] [--padding bytes]
                        [--do-not-fragment] [-i msec] [-c packets]
                        [remote-ip:port] [local-ip:port]

optional arguments:
  -h, --help            show this help message and exit
  -q, --quiet           disable logging
  -v, --verbose         enhanced logging
  -d, --debug           extensive logging

Debug Options:
  -l filename, --logfile filename
                        Specify the logfile (default: <stdout>)

IP socket options:
  --tos type-of-service        IP TOS value
  --dscp dscp-value            IP DSCP value
  --ttl time-to-live           [1..128]
  --padding bytes              IP/UDP mtu value
  --do-not-fragment            keyword (do-not-fragment)

TWL sender options:
  remote-ip:port [default port: 862]
  local-ip:port [default: 127.0.0.1:862]
  -i msec, --interval msec     [100,1000] [default: 200]
  -c packets, --count packets  [1..9999] [default: 50]
```



## Usage example against Datacom DmOS TWAMP server
Switch configuration:
```
# show running-config oam twamp reflector
----------------------------------------------
oam
 twamp
  reflector
   ipv4
    client-address 192.168.255.2
   !
   ipv6
    client-address 2001::1
   !
  !
 !
!
----------------------------------------------
```
Running the test:
```
$ sudo python3 ./twampy.py controller 192.168.255.2
===============================================================================
Direction         Min         Max         Avg          Jitter     Loss
-------------------------------------------------------------------------------
  Outbound:        0.30ms      0.35ms      0.33ms      0.03ms      0.0%
  Inbound:         0.26ms      0.29ms      0.28ms      0.03ms      0.0%
  Roundtrip:       0.57ms      0.64ms      0.61ms      0.01ms      0.0%
-------------------------------------------------------------------------------
                                                    Jitter Algorithm [RFC1889]
===============================================================================
```
