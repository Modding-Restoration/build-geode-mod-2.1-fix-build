# build-geode-mod
An easy to use action to build a geode mod within github actions.

This repository contains two actions, one for just building the mods, and another for combining builds for multiple platforms into a single .geode file.

# Usage
```yml
- uses: Modding-Restoration/build-geode-mod-2.1-fix-build@main
  with:
    # Which version of the SDK to use. Use nightly to specify latest commit
    # Default: latest
    sdk: ''

    # Which CLI version to use.
    # Default: latest
    cli: ''

    # Extra arguments passed to CMake when configuring. Not required.
    configure-args: ''

    # Extra arguments passed to CMake when building. Not required.
    build-args: ''

    # Which build configuration to use.
    # Default: Release
    build-config: ''

    # Path to the project which to build. Defaults to current directory.
    path: ''

    # Set this to true if you plan on merging .geode files later. See the README for more info.
    # Default: false
    combine: ''
```

# Examples

## Building and uploading a mod on latest sdk
```yml
- uses: Modding-Restoration/build-geode-mod-2.1-fix-build@main
  id: build

- uses: actions/upload-artifact@v4
  with:
    name: My mod
    path: ${{ steps.build.outputs.build-output }}
```

## Building with RelWithDebInfo
Note: the pdb is discarded for now
```yml
- uses: geode-sdk/build-geode-mod@main
  id: build
  with:
    build-config: RelWithDebInfo
```

# Combine
It is also possible to build mods for different platforms, and then afterwards combine them into a single .geode file.

Usually this is done using a matrix, and due to limitations on how much actions can do, you will have to add another job for the combining.

To do this, make sure to set `combine: true` on the build action!

## Building a mod on mac and windows, and then combining it
Full workflow:
```yml
name: Build Geode Mod

on:
  workflow_dispatch:
  push:
    branches:
      - "main"

jobs:
  build:
    strategy:
      fail-fast: false
      matrix:
        config:
        - name: Windows
          os: windows-latest
          
        - name: macOS
          os: macos-latest

    name: ${{ matrix.config.name }}
    runs-on: ${{ matrix.config.os }}

    steps:
      - uses: actions/checkout@v4

      - name: Build the mod
        uses: Modding-Restoration/build-geode-mod-2.1-fix-build@main
        with:
          combine: true
      
  package:
    name: Package builds
    runs-on: ubuntu-latest
    needs: ['build']

    steps:
      - uses: Modding-Restoration/build-geode-mod-2.1-fix-build@combine
        id: build

      - uses: actions/upload-artifact@v4
        with:
          name: Build Output
          path: ${{ steps.build.outputs.build-output }}

```
