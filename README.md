# EOSS Specific Changes

The `eoss` branch adds the following changes/features so that the dump1090-fa release can work relatively seamlessly on the EOSS SDR system:
- Updates to the web display and the underlying Javascript to Redirect the map source to the locally running OSM map server.  This way map tiles are fetched from the local system and removes the need for a dedicated Internet connection.
- Apache is used as the web server of choice instead of lighttpd.  The Apache configuration is modified to use an alias (/dump1090-fa) pointing to the /user/share/dump1090-fa/html directory.
- Parameter changes for the dump1090-fa daemon so that it looks to use only those RTL-SDR USB dongles that have been labeled with "ADSB" as their serial number.  This allows 
multiple RTL-SDR dongles to be used on the SDR system (dump1090-fa uses one while the SDR system can use another).


# dump1090-fa Debian/Raspbian packages

dump1090-fa is a ADS-B, Mode S, and Mode 3A/3C demodulator and decoder that
will receive and decode aircraft transponder messages received via
a directly connected software defined radio, or from data provided over a
network connection.

It is the successor to
[dump1090-mutability](https://github.com/mutability/dump1090) and is
maintained by [FlightAware](http://flightaware.com/).

It can provide a display of locally received aircraft data in a terminal or
via a browser map. Together with [PiAware](http://flightaware.com/adsb/piaware)
it can be used to contribute crowd-sourced flight tracking data to FlightAware.

It is designed to build as a Debian package, but should also be buildable on
many other Linux or Unix-like systems.

## Building under buster

```bash
$ sudo apt-get install build-essential fakeroot debhelper librtlsdr-dev pkg-config dh-systemd libncurses5-dev libbladerf-dev libhackrf-dev liblimesuite-dev
$ dpkg-buildpackage -b --no-sign
```

## Building under stretch

```bash
$ sudo apt-get install build-essential debhelper librtlsdr-dev pkg-config dh-systemd libncurses5-dev libbladerf-dev
$ dpkg-buildpackage -b --no-sign
```

## Building under jessie

### Dependencies - bladeRF

You will need a build of libbladeRF. You can build packages from source:

```bash
$ git clone https://github.com/Nuand/bladeRF.git  
$ cd bladeRF  
$ git checkout 2017.12-rc1  
$ dpkg-buildpackage -b
```

Or Nuand has some build/install instructions including an Ubuntu PPA
at https://github.com/Nuand/bladeRF/wiki/Getting-Started:-Linux

Or FlightAware provides armhf packages as part of the piaware repository;
see https://flightaware.com/adsb/piaware/install

### Dependencies - rtlsdr

This is packaged with jessie. `sudo apt-get install librtlsdr-dev`

### Actually building it

Nothing special, just build it (`dpkg-buildpackage -b`)

## Building with limited dependencies

The package supports some build profiles to allow building without all
required SDR libraries being present. This will produce a package with
limited SDR support only.

Pass `--build-profiles` to `dpkg-buildpackage` with a comma-separated list of
profiles. The list of profiles should include `custom` and zero or more of
`rtlsdr`, `bladerf`, `hackrf`, `limesdr` depending on what you want:

```bash
$ dpkg-buildpackage -b --no-sign --build-profiles=custom,rtlsdr          # builds with rtlsdr support only
$ dpkg-buildpackage -b --no-sign --build-profiles=custom,rtlsdr,bladerf  # builds with rtlsdr and bladeRF support
$ dpkg-buildpackage -b --no-sign --build-profiles=custom                 # builds with _no_ SDR support (network support only)
```

## Building manually

You can probably just run "make" after installing the required dependencies.
Binaries are built in the source directory; you will need to arrange to
install them (and a method for starting them) yourself.

``make BLADERF=no`` will disable bladeRF support and remove the dependency on
libbladeRF.

``make RTLSDR=no`` will disable rtl-sdr support and remove the dependency on
librtlsdr.

``make HACKRF=no`` will disable HackRF support and remove the dependency on 
libhackrf.

``make LIMESDR=no`` will disable LimeSDR support and remove the dependency on
libLimeSuite.

## Building on OSX

Minimal testing on Mojave 10.14.6, YMMV.

```
$ brew install librtlsdr
$ brew install libbladerf
$ brew install hackrf
$ brew install pkg-config
$ make
```

## Building on FreeBSD

Minimal testing on 12.1-RELEASE, YMMV.

```
# pkg install gmake
# pkg install pkgconf
# pkg install rtl-sdr
# pkg install bladerf
# pkg install hackrf
$ gmake
```

## Generating wisdom files

dump1090-fa uses [starch](https://github.com/flightaware/starch) to build
multiple versions of the DSP code and choose the fastest supported by the
hardware at runtime. The implementations chosen can been seen by running
`dump1090-fa --version`.

The implementations used are controlled by "wisdom files", a list of
implementations to use in order of priority. For each DSP function, the first
implementation listed that's supported by the current hardware is used.
By default dump1090-fa provides compiled-in wisdom for [x86](wisdom.x86),
[ARM 32-bit](wisdom.arm), and [ARM 64-bit](wisdom.aarch64). If the defaults
are not suitable for your hardware or if you're building on a different
architecture, you may want to generate your own external wisdom file.

Ideally, to get stable results, you want to do this on an idle system
with CPU frequency scaling disabled. Running the benchmarks will take
some time (10s of minutes).

### Package installs

Run `/usr/share/dump1090-fa/generate-wisdom`. Wait.

Follow the instructions to copy the resulting wisdom file to `/etc/dump1090-fa/wisdom.local`.

Restart dump1090.

### Manual installs

Run `make wisdom.local`. Wait.

Copy the resulting `wisdom.local` file somewhere appropriate.

Update the dump1090-fa command-line options to include `--wisdom /path/to/wisdom.local`
