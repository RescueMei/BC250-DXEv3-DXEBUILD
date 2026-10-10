# BC250-DXEv3-DXEBUILD

Unified build script for my DXE drivers, put in a repo to allow for proper
versioning and prevent drift across drivers.

## Layout

`build_ffs.sh` is the canonical, shared build helper. Each driver repository
includes this repository as a `DXEBUILD` git submodule and keeps a thin
`build_ffs.sh` wrapper at its root that runs the shared script with
`REPO_DIR` set to the driver repository root.

Per-project settings (package directory, platform DSC, module name/INF/GUID,
and optional hooks or edk2 submodule extras) live in each driver's `build.conf`
next to its wrapper. The shared script reads that file, so no driver-specific
logic belongs here.

## Usage

From a driver repository (for example `BC250-DXEv3-Core-Unlock`):

```bash
git submodule update --init --recursive
./build_ffs.sh
```

See the shared script header for the recognized environment overrides
(`REPO_DIR`, `ARCH`, `TARGET`, `TOOL_CHAIN_TAG`, `CONTAINER_ENGINE`,
`EDK2_IMAGE`, `EDK2_REPO`, `EDK2_REF`, `NO_CONTAINER`, `EDK2_DIR`).
