# MUDGuard Reproduction
This guide will allow you to reproduce MUDGuard, developed at the University of York Systron Lab.

MUDGuard is an interface to allow end users to easily change [MUD files](https://datatracker.ietf.org/doc/html/rfc8520) for devices on their network.

## Requirements
This is the minimum hardware and software required to reproduce MUDGuard.

### Hardware
The device that will be running the software we are using:
- Raspberry Pi 3B
- Linux based computer - WSL or any Linux Distro

### Software
The software that we will be running to setup the system
- [OpenWRT](https://github.com/openwrt/openwrt)
- [osMUD](https://github.com/osmud/osmud)
- [MUDGuard](https://github.com/PeterG184/osMUD-UI)

## System Description
MUDGuard runs on a router device, in this scenario a Raspberry Pi. This Pi runs OpenWRT, an open source Linux based operating system designed for networking. osMUD runs as a service on this operating system, alongside an API that acts as an interface between the router and a frontend that runs on a PC.

### OpenWRT

#### Build OpenWRT in Docker Container
1. Clone the osmud repo using `git clone https://github.com/osmud/osmud`
1. Build the Dockerfile located at osmud/build/Dockerfile `docker build -t osmud/build-env .`
1. Run & download docker image and components as a daemon `docker run -d -ti --name=osmud-build-env osmud/build-env`
1. Find docker container id: `docker ps -a`
1. Enter running docker container: `docker exec -i -t container_ID_from_docker_ps /bin/bash`
1. Change directory to the "lede" directory: `cd lede`
1. If this is your first time into the docker image you will need to update and install all feeds.
    * Update all feeds: `./scripts/feeds update -a`
    * Install all source feeds: `./scripts/feeds install -a`
1. Create make config file: `make menuconfig`
    1. Select your target system. The platform is BCM2710
    1. Select your target profile. The model is Raspberry Pi 3B+
    1. Tab over to the `<Exit>` command and save the configuration
1. Build only openwrt: `make`
1. View the compiled binaries `~/lede/bin/targets/bcm2710/generic`

#### Install OpenWRT on the router

1. tar the binary file using `tar czf {binary file}.tar.gz {binary-file}.bin`
1. install the .gz file onto a microSD card using [Raspberry Pi Imager](https://github.com/raspberrypi/rpi-imager)

### osMUD

#### Build osmud for OpenWRT
1. Enter the docker container as in the openwrt installation process
1. Create an osmud directory for OpenWRT at `~/lede/package/network/config/osmud`
1. Copy `~osmud/opewrt_toolchain/Makefile` to `~/lede/package/network/config/osmud/Makefile`
1. `cd ~/lede`
1. run `make menuconfig`
1. Select osmud in under `Base System -> osmud` (move to the osmud line and hit "y" to include in the build)
1. Run `make package/network/config/osmud/compile` to compile only osMUD

This builds osMUD to `bin/packages/[platform]/base/osmud_[version].ipk`

#### Install osmud on the router
1. Scp osmud to the router `scp osmud_[version].ipk root@192.168.1.1:/tmp/osmud_[version].ipk`
1. Ensure the router is update `opkg update`
1. Install osmud `opkg install /tmp/osmud_[version].ipk`

## Building the Mudguard API for the RPi

The MUDGuard API can be found in [this repository](https://github.com/PeterG184/osMUD-UI)

The go binary has to built using specific arguments for the Raspberry Pi, defined in the Makefile:

```bash
  CGO_Enabled = 0 # OpenWRT does not include the C Standard Library
  CC=arm-linux-gnueabi-gcc # Toolchain for this hardware and version of OpenWRT
  GOOS=linux 
  GOARM=7 # Using a 32-bit OS so ARMv7 not v8
```

## Running
### osMUD on the RPi
The osmud service is set to autostart on the RPi, however if need to restart it you can use

```bash
$ service osmud restart
```

alternatively the help menu for the osmud service can be brought up with 

```bash
$ service osmud
```

### Launching the API on the RPi
Use 

```bash
$ ./usr/bin/osmud_startup.sh
```

to start the API.

#### What this script does

Before launching the API the database `store.db` must be moved to the location that osmud expects it to be in. This can be done with the commands

```bash
$ mkdir /var/lib/osmud
$ cp /etc/osmud/store.db /var/lib/osmud/store.db
$ cp /etc/osmud/mud_file.json /var/lib/osmud/mud_file.json
```

osmud also expects a file for the dhcp events log to exist. This can be created with

```bash
$ touch /var/log/osmud-dhcp-events.log
```

The API is an executable script stored in

```bash
$ ~/usr/bin/api
```
And can be launched with 

```bash
$ ./usr/bin/api
```

from the root directory. This starts a server running at `192.168.2.1:8080`

### Launching the UI

The UI is launched on your machine. Use `cd ui` to navigate to the folder.

Run the command 

```bash 
$ npm install
```

 to install all dependancies for the UI

Then use 
```bash
$ npm run dev
```
 to start the UI. This will run at `localhost:3000` in your browser.

## Tools

### MUD File Maker 
Python script for generating arbitrary length MUD files for testing. Specify number of rules to generate. Found in root folder

### Batch Create MUD DB Entry
Run 
```bash
$ ./batch_create_mud_db_entry.sh
```
in /scripts/osmud/ with a number of devices to add to add that number of devices to the database. All devices have name test_device_n and a sequential MAC address.
