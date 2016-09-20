Etcher Extended Archives
========================

Etcher Extended Archives are ZIP files including an image and a special `.meta`
directory including information that describes the image, device type, and
vendor in detail, enabling Etcher to make use of such information to deliver
advanced features and a more pleasant experience to the user.

**WARNING: At the moment of this writing, only a small subset of the features
discussed in this document have been implemented in the product.**

A typical layout for an extended archive might look something like this:

```
Raspbian-Jessie.zip
├── .meta
│   └── manifest.json
│   └── instructions.markdown
│   └── device-type.jpg
│   └── vendor-logo.svg
│   └── raspbian-jessie.img.bmap
└── raspbian-jessie.img
```

The actual image file, `raspbian-jessie.img`, lives in the top level of the
archive, and a special `.meta` directory contains many files, which will be
explained in detail in the remaining of this document.

The most important file inside `.meta` is `manifest.json`, which declares
information about the image, the target device type, and the vendor, and
references all the other files living inside the `.meta` directory. This file
may contain the following top level properties:

- `vendor`: Information about the image vendor
- `device`: Information about the image device type
- `image`: Information about the image itself

For example:

```json
{
  "vendor": { ... },
  "device": { ... },
  "image": { ... }
}
```

Each of the top level properties are described in detail below.

Vendor
------

> A vendor represents a manufacturer of a device type and potential publisher
  of official images.

_Some examples of vendors are: Beagleboard, Raspberry Pi
Foundation, Intel, and Tizen._

Here's an example of a real-world vendor manifest for Beagleboard:

```json
{
  "name": "Beagleboard",
  "url": "http://beagleboard.org",
  "supportUrl": "http://beagleboard.org/support/",
  "logoPath": "vendor.svg",
  "colorScheme": {
    "backgroundColor": "#535760",
    "foregroundColor": "#FFFFFF",
    "primaryColor": "#5793db"
  }
}
```

A vendor may be described with any of the following properties:

### `name (String)`

The display human-friendly name of the vendor.

### `url (String)`

The main url of the vendor, usually the landing page.

### `supportUrl (String)`

The vendor url where users can get general support.

This could be a link to a forum, IRC room, troubleshooting page, support form,
etc.

### `logoPath (String)`

The path to the vendor's logo in SVG format.

The SVG format requirement ensures that the logo looks good in all types of
screen resolutions.

### `colorScheme (Object)`

The vendor specific color scheme, if any.

Declaring a vendor-specific color scheme allows a certain degree of branding to
be applied to the application on the fly as soon as a vendored image is
selected.

You may declare the following colors, in hexadecimal format:

- `backgroundColor`: The application background color
- `foregroundColor`: The application foreground color
- `primaryColor`: The application primary color

Device
------

> A device represents a specific device type manufactured by a certain vendor.

_Some examples of devices are: Beaglebone Black, Beaglebone Green, Intel Edison,
and Raspberry Pi 3._

Here's an example of a real-world device manifest for Raspberry Pi 3:

```json
{
  "displayName": "Raspberry Pi 3",
  "arch": "armv7hf",
  "slug": "raspberrypi3",
  "slugAliases": [
    "raspberry-pi3",
    "rpi3"
  ],
  "url": "https://www.raspberrypi.org/products/raspberry-pi-3-model-b/",
  "photoPath": "raspberry-pi-3-photo.jpg"
}
```

A device may be described with any of the following properties:

### `displayName (String)`

The human-friendly name of the device.

### `arch (String[])`

The device architecture.

The current possible values are: `aarch64`, `amd64`, `armv5e`, `armv7hf`,
`i386`, and `rpi`.

### `slug (String)`

The device slug.

### `slugAliases (String[])`

The device slug aliases, if any.

### `url (String)`

The url to the landing page of the device.

### `photoPath (String)`:

The path to a photo of the device in JPEG format.

### `compatibleWith (String[])`

An array of the device slugs this device is compatible with.

Declaring compatibility allows images that target a specific device to be also
used in other device types that are known to work with it.

Image
-----

> An image represents a downloadable operating system image that can be booted
  in a device or family of devices.

_Some examples of images are: Raspbian Jessie, ResinOS, CoreOS, and Ostro._

Here's an example of a real-world image manifest for Raspbian Jessie:

```json
{
  "displayName": "Raspbian Jessie",
  "version": "May 2016",
  "url": "https://www.raspbian.org",
  "releaseNotesUrl": "http://downloads.raspberrypi.org/raspbian/release_notes.txt",
  "logoPath": "image-logo.svg",
  "checksumType": "sha1",
  "checksum": "64c7ed611929ea5178fbb69b5a5f29cc9cc7c157",
  "recommendedDriveSize": 4294967296,
  "bmapPath": "raspbian-jessie.bmap",
  "instructionsPath": "raspbian-getting-started.markdown",
  "updateUrl": "https://downloads.raspberrypi.org/raspbian_latest",
  "etag": "c0170-53152af2-533d18ef29fc0",
  "supportedDevices": [
    "raspberrypi",
    "raspberrypi2",
    "raspberrypi3"
  ]
}
```

A image may be described with any of the following properties:

### `displayName (String)`

The human-friendly name of the image.

### `version (String)`

The version of the image.

### `url (String)`

The main url of the image.

### `releaseNotesUrl (String)`

The url to the image's release notes or CHANGELOG.

### `logoPath (String)`

The path to the image specific logo in SVG format.

The SVG format requirement ensures that the logo looks good in all types of
screen resolutions.

### `checksumType (String)`

The checksum type. The current possible values are: `sha1`, `crc32`, and `md5`.

### `checksum (String)`

The actual checksum according to the type declared in `checksumType`.

### `bytesToZeroOutFromTheBeginning (Number)`

The number of bytes to zero out from the beginning before starting the flash
process. This option currently only applies when using bmaps.

### `recommendedDriveSize (Number)`

The recommended drive size to flash this image, in bytes.

The use case for this option is that while a drive might be large enough to
contain the image, it might not be large enough to deliver a good experience
when actual using the application or operating system contained in the image.

### `bmapPath (String)`

Path to a [bmap][bmap] file.

### `instructionsPath (String)`

Path to a markdown file showing instructions to the user for when the flash
completes.

### `supportedDevices (String[])`

An array of the device slugs this image supports.

### `updateUrl (String)`

The url where the client can request the latest version of the image.

The way this works is that you provide a generic url that redirects to the
latest image resource. The client will follow all redirects until it finds the
final image, and will check if the resource matches the currently selected
image.

### `etag (String)`

The [HTTP Etag][etag] of the current image.

This can be used to check if the image resolved by `updateUrl` is different to
the one we have locally.

### `expiryDate (String)`

The timestamp to determine at which point the image may be expired. The date
should conform to [ISO 8601][iso8601] standard.

Clients may use this information to refuse flashing the image.

Client
------

> A client represents the program making use of the information about the other
  entities. This is not something publishers declare themselves.

_At the moment of this writing, the only existent client is the Etcher
application._

Here's an example of a real-world client manifest for Etcher running on Mac OS
X:

```json
{
  "name": "etcher",
  "version": "1.0.0-beta.14",
  "os": "darwin",
  "arch": "x64"
}
```

A client may be described with any of the following properties:

### `name (String)`

The client name.

### `version (String)`

The client version.

### `os (String)`

The operating system running the client.

The current possible values are: `aix`, `darwin`, `freebsd`, `linux`,
`openbsd`, `sunos`, and `win32`.

### `arch (String)`

The client operating system architecture.

The current possible values are: `arm`, `arm64`, `ia32`, `mips`, `mipsel`,
`ppc`, `ppc64`, `s390`, `s390x`, `x32`, `x64`, and `x86`.

Extensions
----------

Each of the entities described above accept an object property called
`extensions`, which you can use to provide your own custom properties to the
manifests, in a way that are specific to your application. It is every
encouraged that you put those custom properties inside the `extensions`
namespace to avoid potential clashes with other properties that may be
introduced in the future.

[iso8601]: https://en.wikipedia.org/wiki/ISO_8601
[etag]: https://en.wikipedia.org/wiki/HTTP_ETag
[bmap]: https://source.tizen.org/documentation/reference/bmaptool/bmap-tools-project
