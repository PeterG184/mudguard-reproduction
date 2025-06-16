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



### Installation

## Running
