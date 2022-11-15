# Swedish Embedded Platform SDK ok-dev Docker Image

See https://github.com/swedishembedded/develop for inspiration

This repository contains a docker image:

- **ok-dev Developer Image**: this image contains a standalone development environment based on VSCode Server which can be used for development by connecting to container from a local VSCode IDE.

## Support

- Community: https://swedishembedded.com/community

## Developer Docker Image

### Overview

The developer image includes all tools included in the build image as well as
additional tools that are useful for development:

- **VSCode Server**: use a familiar IDE from your desktop. Connect via the VSCode Docker extension.

- **GDB Dashboard**: for easy graphical GDB debugging.

### Installation

#### Using Pre-built Developer Docker Image

Pull the latest prebuilt docker image:

```
docker pull mmcc007/se-sdk-ok-dev:latest
docker run -ti -v <path to my git repo>/my_repo:/build/platform/my_repo \
           mmcc007/se-sdk-ok-dev:latest
```

The command above mounts your workspace under /build/platform inside the container so you can
work on your local files from within the container.

For a sample repo that uses the recommended SE SDK project layout use.  
https://github.com/ok-mo/zephyr-getting-started

In order to flash firmware over USB JTAG you also need to run docker in
privileged mode and mount usb within docker:

```
docker run -ti -v <path to my git repo>/my_repo:/build/platform/my_repo \
	--privileged -v /dev/bus/usb:/dev/bus/usb \
    mmcc007/se-sdk-ok-dev:latest
```

Note: due to incomplete support from Docker for macOS, USB pass-thru does not work for macOS.  
https://github.com/docker/for-mac/issues/900

A potential workaround using VirtualBox, which supports USB pass-thru, as an intermediary hypervisor to docker container, is being investigated. Otherwise, the workaround is to flash app image using west from the local env (and debugging locally also).

This will allow you to access JTAG adapter from inside the container. You can
also expose other resources to the container in the same way.

Check out the configuration like this:

```
cat ~/.bashrc
```

#### Building Developer Docker Image Locally

Developer docker image can be built locally using the supplied shell script:

```
./scripts/build
```

### Usage

#### Building a sample application

While in a container, follow the steps below to build and run a sample application:

A preliminary step is required to setup the zephyr sdk

```
/opt/toolchains/zephyr-sdk-0.14.2/setup.sh -t all -h -c
```

A demo:

```
cd /build/platform/sdk
west build -b custom_board -d build-apps/custom_board/shell/apps.shell.release/ -s apps/shell

# if on linux you can also flash to a physical board, eg:
west build -b stm32f429i_disc1 -s ../zephyr/samples/basic/blinky -t flash
```

To build in my_repo

```
cd /build/platform/my_repo
west build -b custom_board -d build-apps/custom_board/shell/apps.shell.release/ -s apps/shell
```

Note: assumes my_repo has been setup similar to https://github.com/swedishembedded/sdk

## Cleanup

If you find that your system runs out of space then you can always delete all modified docker data by pruning everything (be careful because this will delete any changes you have done to files inside a docker container. It will **not** however remove files you modified in a mounted local directory. So it's safe.):

```
docker system prune --volumes
```

## Questions

### Why is the docker image so big?

Because it includes **all essential tools** including a handful of different
compilers which are used for crosscompiling, source code and git repositories
used for demo. Every tool included in the base image is useful at some stage in
the build process. The build image is designed to be a versatile build image
that can be used to build not just firmware but multiple types of documentation
as well.

### Is it possible to reduce the size?

You can probably reduce the size but it is not practical because you will
always run into cases where you need extra tools and you will find that you
will need to add them back again. It is better to have a single image that
contains a complete and reproducible invironment so that it is possible to
easily scale development to multiple projects.

Besides, 15GB is not much considering that it is basically a full ubuntu setup
with everything included inside.
