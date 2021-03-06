# SDR Setup

<!-- markdown-toc start - Don't edit this section. Run M-x markdown-toc-generate-toc again -->
**Table of Contents**

- [SDR Setup](#sdr-setup)
    - [Connections](#connections)
    - [Software Requirements](#software-requirements)
    - [Configuration](#configuration)
    - [Running](#running)
    - [Docker](#docker)
    - [Next Steps](#next-steps)
    - [Further Information](#further-information)
        - [Manual Installation of SDR Software](#manual-installation-of-sdr-software)

<!-- markdown-toc end -->

## Connections

The SDR setup is connected as follows:

![SDR Connections](img/sdr_connections.png?raw=true "SDR Connections")

- Connect the RTL-SDR USB dongle to your host PC.
- Connect the **non-powered** port of the power supply (labeled as “Signal to
  IRD”) to the RTL-SDR using an SMA cable and an SMA-to-F adapter.
- Connect the **powered** port (labeled “Signal to SWM”) of the power supply to
  the LNB using a coaxial cable (an RG6 cable is recommended).

**IMPORTANT**: Do NOT connect the powered port of the power supply to the SDR
interface. Permanent damage may occur to your SDR and/or your computer.

## Software Requirements

The SDR-based setup relies on the applications listed below:

- [leandvb](http://www.pabr.org/radio/leandvb/leandvb.en.html): a software-based
  DVB-S2 demodulator.
- [rtl_sdr](https://github.com/osmocom/rtl-sdr): reads samples taken by the
  RTL-SDR and feeds them into
  [leandvb](http://www.pabr.org/radio/leandvb/leandvb.en.html).
- [TSDuck](https://tsduck.io/): unpacks the output of leandvb and produces
  IP packets to be fed to [Bitcoin Satellite](bitcoin.md).
- [Gqrx](https://gqrx.dk): useful for spectrum visualization during antenna
  pointing.

To install them all at once, run:

```
blocksat-cli deps install
```

> NOTE: This command assumes and has been tested with Ubuntu 18.04 and
> Fedora 31. Please adapt if necessary in case you are using another Linux
> distribution or version.

If you prefer to build all software components manually, please refer to the
[manual installation section](#manual-installation-of-sdr-software).

## Configuration

After installing, you can generate the configurations that are needed for gqrx
by running:

```
blocksat-cli gqrx-conf
```

> NOTE: this assumes you are going to use gqrx with an RTL-SDR dongle.

## Running

You should now be ready to launch the SDR receiver. You can run it by executing:

```
blocksat-cli sdr
```

More specifically, as thoroughly explained in the [Antenna Pointing
Guide](antenna-pointing.md#sdr-based), you might want to run with specific gain
and de-rotation parameters that are suitable to your setup, like so:

```
blocksat-cli sdr -g [gain] --derotate [freq_offset]
```

where `[gain]` and `[freq_offset]` should be substituted by the appropriate
values.

Furthermore, in the SDR setup, you must choose which stream you would like to
receive from the two streams that are simultaneously broadcast through the
Blockstream Satellite network (refer to more information in the [Antenna
Pointing Guide](antenna-pointing.md#optimize-snr)).

By default, the application will try to decode the low-throughput stream. To try
decoding the high-throughput stream, run with option `-m high`, as follows:

```
blocksat-cli sdr -g [gain] --derotate [freq_offset] -m high
```

## Docker

There is a Docker image available in this repository for running the SDR
setup. Please refer to instructions in the [Docker guide](../docker/README.md).

## Next Steps

At this point, if your antenna is already correctly pointed, you should be able
to start receiving data in Bitcoin Satellite. Please follow the [instructions
for Bitcoin Satellite configuration](bitcoin.md). If your antenna is not pointed
yet, please follow the [antenna alignment guide](antenna-pointing.md).

## Further Information

### Manual Installation of SDR Software

If you do not wish to rely on the automatic installation handled by command
`blocksat-cli deps install`, you can build and install all applications
manually.

By default, the CLI will look for the locally built apps on directory
`~/.blocksat/bin`. Hence, to start, create this directory, as well as the
directory for source files:

```
mkdir -p ~/.blocksat/bin
mkdir -p ~/.blocksat/src
```

To build leandvb from source, first install the dependencies:

```
apt install git make g++ libx11-dev
```
or
```
dnf install git make g++ libX11-devel
```

Then, run:

```
cd ~/.blocksat/src
git clone --recursive https://github.com/Blockstream/leansdr.git
cd leansdr/src/apps
make
install leandvb ~/.blocksat/bin
```

Next, build and install `ldpc_tool`, which is used as an add-on to `leandvb`:

```
cd ../../LDPC/
make CXX=g++ ldpc_tool
install ldpc_tool ~/.blocksat/bin
```

To install the RTL-SDR application, run:

```
apt-get install rtl-sdr
```
or
```
dnf install rtl-sdr
```

Next, to build TSDuck from source, run:

```
cd ~/.blocksat/src
git clone https://github.com/tsduck/tsduck.git
cd tsduck
build/install-prerequisites.sh
make NOTELETEXT=1 NOSRT=1 NOPCSC=1 NOCURL=1 NODTAPI=1
```

And then add the following to your `.bashrc`:

```
source ~/.blocksat/src/tsduck/src/tstools/release-x86_64/setenv.sh
```
or
```
source ~/.blocksat/src/tsduck/src/tstools/release-aarch64/setenv.sh
```
depending on whether you are running on x86_64 or aarch64 processor.

By adding this `setenv.sh` script to your `.bashrc`, the environmental variables
that are necessary in order to use TSDuck become available on every terminal
session.

Finally, install Gqrx from binary:

```
sudo apt install gqrx-sdr
```
or
```
sudo dnf install gqrx
```

