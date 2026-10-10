# BC250-DXEv3-DXEBUILD

Unified build script for my DXE drivers, put in a repo to allow for proper
versioning and prevent drift across drivers.

## Layout

`build_ffs.sh` is the canonical, shared build helper. Each driver repository
includes this repository as a `DXEBUILD` git submodule and keeps a thin
`build_ffs.sh` wrapper at its root that runs the shared script with
`REPO_DIR` set to the driver repository root (see `build.conf.example` for a
template configuration).

Per-project settings live in each driver's `build.conf` next to its wrapper.
The shared script reads that file, so no driver-specific logic belongs here.

## Usage

From a driver repository (for example `BC250-DXEv3-Core-Unlock`):

```bash
git submodule update --init --recursive
./build_ffs.sh
```

The build produces, under `Build/Output/` in the driver repository:

- `<MODULE_NAME>.efi` - the raw DXE PE32 image
- `<MODULE_NAME>.ffs` - the driver-type firmware file for UEFITool

See the shared script header for the recognized environment overrides
(`REPO_DIR`, `ARCH`, `TARGET`, `TOOL_CHAIN_TAG`, `CONTAINER_ENGINE`,
`EDK2_IMAGE`, `EDK2_REPO`, `EDK2_REF`, `NO_CONTAINER`, `EDK2_DIR`).

## `build.conf`

`build.conf` is a bash fragment that the wrapper's shared script sources from
the repository root. It is the only file a driver needs to add: it selects the
package, the module to build, and the GUID used when packing the `.ffs`.

Start from `build.conf.example` and edit the values. All paths are relative to
the driver repository root.

### Required variables

| Variable | Meaning |
| --- | --- |
| `PKG_DIR` | Package directory that contains the platform DSC (must exist). |
| `PLATFORM_DSC` | Platform `.dsc` passed to `build -p` (must exist). |
| `BUILD_PACKAGE_DIR` | Output subdirectory name under the edk2 `Build/` tree where the `.efi` is looked up. |
| `MODULE_NAME` | Driver name; also the base name of every generated artifact. |
| `MODULE_INF` | Module `.inf` (must exist); used with `-m` when `BUILD_MODULE=1`. |
| `MODULE_GUID` | File GUID passed to `GenFfs -g` for the `.ffs`. |

### Optional variables

| Variable | Default | Meaning |
| --- | --- | --- |
| `BUILD_MODULE` | `0` | When `1`, build only `MODULE_INF` (`build -m`); otherwise build the whole `PLATFORM_DSC`. |
| `SUBMODULES_RECURSIVE` | `0` | When `1`, `git submodule update --init --recursive` in edk2; otherwise only the minimal submodule set plus `SUBMODULES_EXTRA`. |
| `SUBMODULES_EXTRA` | empty | Space-separated extra edk2 submodule paths to initialize in the non-recursive case (for example a package's own brotli copy). |
| `PREBUILD_CMD` | empty | Shell command run in the repository root (via `eval`) before the edk2 build, in both container and host modes. Use it to generate sources or blobs. |

### Example

```bash
PKG_DIR="BC250DXEv3ColdBootPkg"
PLATFORM_DSC="BC250DXEv3ColdBootPkg/BC250DXEv3ColdBootPkg.dsc"

BUILD_PACKAGE_DIR="MeiMeiDXEv3_ColdBoot"

MODULE_NAME="MeiMeiDXEv3_ColdBoot"
MODULE_INF="BC250DXEv3ColdBootPkg/BC250DXEv3ColdBootDxe/BC250DXEv3ColdBootDxe.inf"
MODULE_GUID="6c98e4f3-9d8b-4e5a-b2e7-31a45b9c5d81"
```

A module built from an INF (needs `-m`) that also pulls an extra submodule and
runs a generator, as used by the ACPI and menu drivers:

```bash
BUILD_MODULE=1
SUBMODULES_EXTRA="MdeModulePkg/Library/BrotliCustomDecompressLib/brotli"
PREBUILD_CMD="python3 BC250DXEv3ACPIAutoInjectPkg/prepare_acpi_blobs.py"
```
