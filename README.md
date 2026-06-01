# CrossOver Wine 26.1.0 with WineGDK Patches

## License and Legal Notice

This repository is a derivative distribution of the Wine project and CodeWeavers CrossOver source snapshot, with additional patches from the WineGDK project applied.

All original Wine source code is licensed under the GNU Lesser General Public License version 2.1 or later (LGPL-2.1-or-later). A copy of the full license text can be found in the `LICENSE` file in the root directory, and copyright notices are available in the `upstream/wine` directory.

This repository does NOT relicense upstream code.

Modifications and integration work in this repository are provided under the same LGPL-2.1-or-later terms, unless otherwise stated.

No warranty is provided.

---

## Project Overview

This repository provides a unified patch and build structure to integrate the **WineGDK** modifications into the official **CodeWeavers CrossOver** source tree. Because both projects modify overlapping parts of the Wine source, this repository contains the necessary manual conflict resolutions to make them work together.

## Component Versions and Sources

To ensure absolute reproducibility and legal transparency, this repository is built upon exact, unmodified source releases and specific Git commits.

### 1. CodeWeavers CrossOver Source (Base)
*   **Version:** CrossOver 26.1.0
*   **Archive:** `crossover-sources-26.1.0.tar.gz`
*   **Source URL:** [CodeWeavers Source Code Page](https://www.codeweavers.com/crossover/source)
*   *Note:* This archive serves as the unmodified upstream base code. It should be extracted into the `upstream/wine` directory.

### 2. WineGDK Patchset
*   **Source Repository:** [Weather-OS/WineGDK](https://github.com/Weather-OS/WineGDK)
*   **Target State (Commit):** `5cffa07a43a61a2fd1695d714b85b5b44852d55b`
*   **Base State (Commit):** `af147f53f5e6b50a1cf31d78da443ac018e5a7b9`
*   *Note:* The patch provided in this repository (`patches/0001-winegdk.patch`) isolates the exact modifications made by the WineGDK project between the specified base commit and target commit.

## Modifications and Conflict Resolution

Applying the isolated WineGDK changes to the CrossOver 26.1.0 source tree results in merge conflicts, particularly in the Windows App Core API components. 

These conflicts have been manually resolved. Specifically, adjustments were made to the following files to ensure compatibility:
*   `dlls/twinapi.appcore/main.c`
*   `dlls/twinapi.appcore/classes.idl`
*   `dlls/twinapi.appcore/private.h`

## How to Apply and Build

1. **Prepare the Base:** 
   Ensure that the extracted CrossOver 26.1.0 source code is located in the `upstream/wine` directory.

2. **Apply the Patch:**
   Navigate to the source directory and apply the combined, conflict-resolved patch:
```bash
   cd upstream/wine
   patch -p1 -l < ../patches/0001-winegdk.patch
```

3. **Compile (General):**
   Building Wine requires heavily specialized dependencies (such as Vulkan headers, FreeType, and specific compiler flags). On a fully configured Linux environment, a standard build might look like this:
```bash
   ./configure
   make
```
   For general build prerequisites, please refer to the official [WineHQ Building Guide](https://wiki.winehq.org/Building_Wine).

4. **Compile (macOS / Apple Silicon via Rosetta):**
   Building this branch on macOS, specifically on Apple Silicon (M1/M2/M3) using Rosetta for x86_64 architecture, requires a highly customized configure command.
   Ensure you have all dependencies (like LLVM, Vulkan loader, and FreeType) installed via Homebrew or manually, and export their paths appropriately (e.g., `$STAGE`, `$VULKAN_HEADER_DIR`, etc.). Then use the following configuration:
```bash
   arch -x86_64 ./configure \
     --prefix="$STAGE" \
     CC=clang \
     CXX=clang++ \
     "CFLAGS=-O3 -I/usr/local/include -I$VULKAN_HEADER_DIR/include --target=x86_64-apple-darwin" \
     "LDFLAGS=-L$VULKAN_LOADER_DIR/lib -L/usr/local/lib -Wl,-rpath,/usr/local/lib" \
     "x86_64_CC=$LLVM_BINDIR/clang" \
     "x86_64_CXX=$LLVM_BINDIR/clang++" \
     "x86_64_CFLAGS=-O3" \
     "x86_64_LDFLAGS=-fuse-ld=lld" \
     --enable-archs=x86_64 \
     --build=x86_64-apple-darwin \
     --host=x86_64-apple-darwin \
     FREETYPE_CFLAGS=-I/usr/local/include/freetype2 \
     FREETYPE_LIBS=/usr/local/lib/libfreetype.dylib \
     build_alias=x86_64-apple-darwin \
     host_alias=x86_64-apple-darwin

   arch -x86_64 make -j"$(sysctl -n hw.ncpu)"
```
