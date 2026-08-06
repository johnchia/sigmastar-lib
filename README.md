# sigmastar-lib

Prebuilt SigmaStar MI userspace libraries for thingino, kept out of the firmware
tree and fetched at build time by a pinned hash.

Companion to [`sigmastar-sdk`](https://github.com/johnchia/sigmastar-sdk)
(kernel-side) and
[`sigmastar-headers`](https://github.com/johnchia/sigmastar-headers)
(programming headers). The split follows thingino's Ingenic convention, where
`ingenic-lib` holds prebuilt userspace libraries and `ingenic-sdk` holds the
kernel side.

This repository exists so `thingino-firmware` does not redistribute proprietary
vendor payload on every clone. The package that consumes it declares
`LICENSE = PROPRIETARY` and `REDISTRIBUTE = NO`, which is only truthful if the
binaries live outside that tree.

## Layout

Mirrors `gtxaspec/ingenic-lib`: family, content type, then SDK release, C
library, and toolchain version.

```
INFINITY6E/
  lib/0607/glibc/9.1.0/     MI userspace libraries
PROVENANCE
```

`sigmastar-lib.mk` resolves this as

```make
$(@D)/$(SOC_FAMILY_CAPS)/lib/$(SDK_VERSION)/$(LIBC_NAME)/$(LIBC_VERSION)
```

so a second family, release or C library is a directory rather than a code
change — the same property `ingenic-lib` has, where `T31/lib/` carries seven SDK
versions side by side.

**Read `PROVENANCE` before changing anything here.** All four path components
are load-bearing: the vendor's own trees differ in content, not merely in
codegen, between C library and toolchain variants.

The Raptor HAL **dlopens** these rather than linking them, so nothing here is a
link-time dependency — but an image without them has daemons that start and then
fail at the first HAL call.

## Provenance summary

Alkaid `release_0607`, built 2022-06-07, GCC 9.1.0, glibc. Harvested from a
shipped camera firmware via OpenIPC's `sigmastar-osdrv-infinity6e` — not taken
from a vendor SDK release tarball, and matching no public drop exactly. Five of
the twenty carry no vendor build stamp; `PROVENANCE` names them.

No source is available and none is claimed here.

## Updating

Consumers pin a commit hash, never a branch. To move the pin, push here and bump
`SIGMASTAR_LIB_VERSION` in `package/sigmastar-lib/sigmastar-lib.mk`.
