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
   -Install gcc:
   ```
   sudo apt update
   sudo apt install build-essential
   ```

- And install `cbindgen`:

    ```bash
    cargo install bindgen
    ```

Then you can use this script (or run the commands one-by-one), which was tested on Ubuntu 18.04 with SGX driver/sdk version 2.6:

```
#! /bin/bash

UBUNTUVERSION=$(lsb_release -r -s | cut -d '.' -f 1)

if (($UBUNTUVERSION < 16)); then
	echo "Your version of Ubuntu is not supported. Must have Ubuntu 16.04 and up. Aborting installation script..."
	exit 1
elif (($UBUNTUVERSION < 18)); then
	DISTRO='xenial'
else
	DISTRO='bionic'
fi


echo "\n\n#######################################"
echo "##### Installing missing packages #####"
echo "#######################################\n\n"

# Install needed packages for script
sudo apt install -y lynx parallel gdebi

# Create a working directory to download and install the SDK inside
mkdir -p "$HOME/.sgxsdk"

(
   # In a new sub-shell cd into our working directory so to no pollute the
   # original shell's working directory
   cd "$HOME/.sgxsdk"

   echo "\n\n################################################"
   echo "##### Downloading Intel SGX driver and SDK #####"
   echo "################################################\n\n"

   # Download the SGX Driver and SDK:
   wget https://download.01.org/intel-sgx/sgx-linux/2.9/distro/ubuntu16.04-server/sgx_linux_x64_driver_2.6.0_95eaa6f.bin
   wget https://download.01.org/intel-sgx/linux-2.6/ubuntu18.04-server/sgx_linux_x64_sdk_2.6.100.51363.bin
   wget https://download.01.org/intel-sgx/linux-2.6/ubuntu18.04-server/libsgx-enclave-common_2.6.100.51363-bionic1_amd64.deb
   wget https://download.01.org/intel-sgx/linux-2.6/ubuntu18.04-server/libsgx-enclave-common-dev_2.6.100.51363-bionic1_amd64.deb
	 wget https://download.01.org/intel-sgx/linux-2.6/ubuntu18.04-server/libsgx-enclave-common-dbgsym_2.6.100.51363-bionic1_amd64.ddeb

   # Make the driver and SDK installers executable
   chmod +x ./sgx_linux_*.bin

   echo "\n\n###############################################"
   echo "##### Installing Intel SGX driver and SDK #####"
   echo "###############################################\n\n"

   # Install the driver
   sudo ./sgx_linux_x64_driver_*.bin


   # Remount /dev as exec, also at system startup
   sudo tee /etc/systemd/system/remount-dev-exec.service >/dev/null <<EOF
[Unit]
Description=Remount /dev as exec to allow AESM service to boot and load enclaves into SGX

[Service]
Type=oneshot
ExecStart=/bin/mount -o remount,exec /dev
RemainAfterExit=true

[Install]
WantedBy=multi-user.target
EOF
   sudo systemctl enable remount-dev-exec
   sudo systemctl start remount-dev-exec

   # Install the SDK inside ./sgxsdk/ which is inside $HOME/.sgxsdk
   echo yes | ./sgx_linux_x64_sdk_*.bin

   # Setup the environment variables for every new shell
   echo "source '$HOME/.sgxsdk/sgxsdk/environment'" |
      tee -a "$HOME/.bashrc" "$HOME/.zshrc" > /dev/null
)

echo "\n\n##############################################"
echo "##### Installing additional dependencies #####"
echo "##############################################\
n\n"

# Add Intels's SGX PPA
echo 'deb [arch=amd64] https://download.01.org/intel-sgx/sgx_repo/ubuntu bionic main' | sudo tee /etc/apt/sources.list.d/intel-sgx.list
wget -qO - https://download.01.org/intel-sgx/sgx_repo/ubuntu/intel-sgx-deb.key | sudo apt-key add -

echo "deb http://apt.llvm.org/bionic/ llvm-toolchain-bionic-7 main" | sudo tee /etc/apt/sources.list.d/llvm-toolchain-bionic-7.list
wget -qO - https://apt.llvm.org/llvm-snapshot.gpg.key | sudo apt-key add -

echo "deb [arch=amd64] https://packages.microsoft.com/ubuntu/18.04/prod bionic main" | sudo tee /etc/apt/sources.list.d/msprod.list
wget -qO - https://packages.microsoft.com/keys/microsoft.asc | sudo apt-key add -
echo "deb https://download.01.org/intel-sgx/sgx_repo/ubuntu $(lsb_release -cs) main" | sudo tee -a /etc/apt/sources.list.d/intel-sgx.list >/dev/null
curl -sSL "https://download.01.org/intel-sgx/sgx_repo/ubuntu/intel-sgx-deb.key" | sudo -E apt-key add -
sudo apt-get update


# Install all the additional necessary dependencies (besides the driver and the SDK)
# for building a rust enclave
wget -O /tmp/libprotobuf10_3.0.0-9_amd64.deb http://ftp.br.debian.org/debian/pool/main/p/protobuf/libprotobuf10_3.0.0-9_amd64.deb
(sleep 3 ; echo y) | sudo gdebi /tmp/libprotobuf10_3.0.0-9_amd64.deb


sudo apt install -y libsgx-enclave-common libsgx-enclave-common-dev libsgx-urts sgx-aesm-service libsgx-enclave-common-dbgsym libsgx-uae-service libsgx-launch libsgx-aesm-launch-plugin libsgx-ae-le autoconf libtool

```



