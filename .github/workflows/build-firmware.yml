name: Build Firmware

on:
  push:
    tags:
      - 'v*'
  workflow_dispatch:
    inputs:
      tag:
        description: 'Release tag (e.g. v1.0.0)'
        required: true
        default: 'v1.0.0'

jobs:
  build:
    name: Build ${{ matrix.target }}
    runs-on: ubuntu-latest
    strategy:
      matrix:
        target: [esp32c3, esp32s3]

    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          submodules: recursive

      - name: Build firmware (${{ matrix.target }})
        uses: espressif/esp-idf-ci-action@v1
        with:
          esp_idf_version: v5.4.1
          target: ${{ matrix.target }}
          path: '.'
          command: idf.py build

      - name: Rename binaries for release
        run: |
          cp build/bootloader/bootloader.bin bootloader-${{ matrix.target }}.bin
          cp build/partition_table/partition-table.bin partition-table-${{ matrix.target }}.bin
          cp build/bitle.bin bitle-${{ matrix.target }}.bin

      - name: Upload build artifacts
        uses: actions/upload-artifact@v4
        with:
          name: firmware-${{ matrix.target }}
          path: |
            bootloader-${{ matrix.target }}.bin
            partition-table-${{ matrix.target }}.bin
            bitle-${{ matrix.target }}.bin

  release:
    name: Create Release
    runs-on: ubuntu-latest
    needs: build
    permissions:
      contents: write

    steps:
      - name: Download all firmware artifacts
        uses: actions/download-artifact@v4
        with:
          path: firmware
          merge-multiple: true

      - name: Create GitHub Release
        uses: softprops/action-gh-release@v2
        with:
          tag_name: ${{ github.ref_name || inputs.tag }}
          name: Bitle Firmware ${{ github.ref_name || inputs.tag }}
          body: |
            ## Bitle Node Firmware

            Pre-built binaries for flashing with the [Bitle Node Flasher](https://bitle-flasher.replit.app).

            | Board | Description |
            |-------|-------------|
            | ESP32-C3 | BLE-only mesh relay node |
            | ESP32-S3 | BLE + LoRa long-range backhaul |

            ### Flashing
            Use the Bitle Node Flasher to flash these binaries directly from your browser — no command-line tools needed.
          files: firmware/*.bin
          draft: false
          prerelease: false
