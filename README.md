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

Family, content type, then SDK release, C library and toolchain version — the
shape `gtxaspec/ingenic-lib` uses, and the same family-first key as
`sigmastar-sdk`.

```
infinity6e/
  lib/0607/glibc/9.1.0/     MI userspace libraries
  PROVENANCE
infinity6c/
  lib/0907/glibc/11.1.0/    MI libraries + sigma_common_libs, merged
  lib/0907/uclibc/9.1.0/    same drop, different content -- see its PROVENANCE
  PROVENANCE
```

`sigmastar-lib.mk` resolves this as

```make
$(@D)/$(SOC_FAMILY)/lib/$(SIGMASTAR_DROP)/$(SIGMASTAR_LIBC)/$(SIGMASTAR_GCC)
```

so a second family, release or C library is a directory rather than a code
change — the same property `ingenic-lib` has, where `T31/lib/` carries seven SDK
versions side by side.

All four components are per family. The drop, C library and toolchain come from
`soc/sigmastar/<family>.mk`:

```make
SIGMASTAR_DROP := 0607
SIGMASTAR_LIBC := glibc
SIGMASTAR_GCC  := 9.1.0
```

which is also what `sigmastar-sdk` reads for its kernel modules. That is the
point of putting them there rather than in either package: the modules and these
libraries are two halves of one vendor build, `vermagic` will not catch a
mismatched pair, and a single definition means the two repository pins cannot
drift apart without the path failing to resolve.

Family names are lowercase, matching `SOC_FAMILY` literally, so no case
conversion happens on the way to a path.

Content type stays a single directory on purpose. The vendor splits i6c into
`mi_libs` and `sigma_common_libs`, but nothing downstream distinguishes them —
both install to `/usr/lib` and the Raptor HAL dlopens by name — so they are
merged under `lib/`, which is also the shape infinity6e already has. That keeps
`sigmastar-lib.mk` to one install glob. No filename appears in both vendor sets,
so the merge shadows nothing; `infinity6c/PROVENANCE` records which libraries
came from which.

**Read `PROVENANCE` before changing anything here.** All four path components
are load-bearing: the vendor's own trees differ in content, not merely in
codegen, between C library and toolchain variants.

The Raptor HAL **dlopens** these rather than linking them, so nothing here is a
link-time dependency — but an image without them has daemons that start and then
fail at the first HAL call.

## Provenance summary

**infinity6e** — Alkaid `release_0607`, built 2022-06-07, GCC 9.1.0, glibc.
Harvested from a shipped camera firmware via OpenIPC's
`sigmastar-osdrv-infinity6e` — not taken from a vendor SDK release tarball, and
matching no public drop exactly. Five of the twenty carry no vendor build stamp;
`PROVENANCE` names them.

**infinity6c** — Alkaid `release_0907`, built 2022-09-07, in both GCC 11.1.0
glibc and GCC 9.1.0 uclibc. Taken from the vendor SDK release tree rather than
reconstructed from a firmware image, so its provenance is the stronger of the
two. The variants differ in content, not only codegen. Nothing consumes it yet:
Infinity6C is a 5.10.61 family and the thingino SigmaStar port is 4.9.84
throughout. See `infinity6c/PROVENANCE`.

No source is available and none is claimed here.

## Updating

Consumers pin a commit hash, never a branch. To move the pin, push here and bump
`SIGMASTAR_LIB_VERSION` in `package/sigmastar-lib/sigmastar-lib.mk`.
