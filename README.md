# enigmasetup

# Hardware

1. Go to your BIOS menu
2. Enable SGX (Software controlled is not enough)
3. Disable Secure Boot

# Software

First, make sure you have Rust installed: https://www.rust-lang.org/tools/install

- Once Rust is installed, install the `nightly` toolchain:

   ```bash
   rustup toolchain install nightly
   ```

- And install `cbindgen`:

    ```bash
    cargo install bindgen
    ```

Then you can use this script (or run the commands one-by-one), which was tested on Ubuntu 18.04 with SGX driver/sdk version 2.6:


