# Theseus OS

Theseus is a new OS written from scratch in [Rust](https://www.rust-lang.org/) to experiment with novel OS structure, better state management, and how to leverage **intralingual design** principles to shift OS responsibilities like resource management into the compiler.

For more info, check out Theseus's [documentation](#Documentation) or our [published academic papers](http://kevinaboos.web.rice.edu/publications.html), which describe Theseus's design and implementation. 

Theseus is under active development, and although it is not yet mature, we envision that Theseus will be useful in high-end embedded systems or edge datacenter environments. 
We are continually working to improve the OS, including its fault recovery abilities for higher system availability without redundancy, as well as easier and more arbitrary live evolution and runtime flexbility.


# Quick start
On Linux (Debian-like distros), do the following:
 1. Obtain the Theseus repository (with all submodules):    
    ```
    git clone --recurse-submodules https://github.com/theseus-os/Theseus.git
    ```
 2. Install Rust:
    ```
    curl https://sh.rustup.rs -sSf | sh
    ```
 3. Install dependencies:
    ```
    sudo apt-get install make gcc nasm pkg-config grub-pc-bin mtools xorriso qemu qemu-kvm
    ```
 4. Build and run (in QEMU):
    ```
     cd Theseus
     make run
     ```

See below for more detailed instructions.


# Building and Running Theseus
**Note:** when you first check out the project, be sure to get all the submodule repositories too:
```
git submodule update --init --recursive
```

Currently, we support building Theseus on the following platforms:
 * Linux, 64-bit Debian-based distributions like Ubuntu, tested on Ubuntu 16.04, 18.04, 20.04. 
   - Arch Linux has also been reported to work correctly. 
 * Windows, using the Windows Subsystem for Linux (WSL), tested on the Ubuntu version of WSL and WSL2.
 * MacOS, tested on versions High Sierra (10.13) and Catalina (10.15.2).
 * Docker, atop any host OS that can run a Docker container.

`
## Setting up the build environment

First, install Rust by following the [setup instructions here](https://www.rust-lang.org/en-US/install.html). On Linux, just run:
```sh
curl https://sh.rustup.rs -sSf | sh
```

### Building on Linux or WSL (Windows Subsystem for Linux)
Install the following dependencies using your package manager:
```sh
sudo apt-get install make gcc nasm pkg-config grub-pc-bin mtools xorriso qemu qemu-kvm
```

  * Or on Arch Linux:
    ```bash
    sudo pacman -S nasm pkg-config grub mtools xorriso qemu
    ```

If you're on WSL, also do the following steps:
  * Install an X Server for Windows; we suggest using [Xming](https://sourceforge.net/projects/xming/).
  * Setup an X display as follows:
    * on original WSL (version 1), run:
      ```
      export DISPLAY=:0
      ```
    * on WSL2 (version 2), run:
      ```
      export DISPLAY=$(cat /etc/resolv.conf | grep nameserver | awk '{print $2}'):0
      ```

    You'll need to do this each time you open up a new WSL terminal, so it's best to add it to the end of your `.bashrc` or `.profile` file in your `$HOME` directory.
  * If you get an error like `Could not initialize SDL (No available video device) ...`, then make sure that your X Server is running and that you have set the `DISPLAY` environment variable above.
  * **NOTE**: WSL and WSL2 do not currently support using KVM.

### Building on MacOS
  * Install [MacPorts](https://www.macports.org/install.php) and [HomeBrew](https://brew.sh/). 
  * run the MacOS build setup script:
    ```
    ./scripts/mac_os_build_setup.sh
    ```
  * If things later go wrong, run the following and then start over.
    ```
    rm -rf $HOME/theseus_tools_src $HOME/theseus_tools_opt /opt/local/bin/make
    ```

### Building using Docker
Note: building and running Theseus within a Docker container may be slower than on a native host OS (e.g., QEMU+KVM). 
1. *(Skip if docker is already installed.)*  Install [Docker Engine](https://docs.docker.com/engine/install/). We provide a convenience script for this on Ubuntu:
    ```
    ./docker/install_docker_ubuntu.sh
    ``` 
    * After docker installs, enable your user account to run docker without root privileges:   
      `sudo groupadd docker; sudo usermod -aG docker $USER`    
      Then, **log out and log back in** (or restart your computer) for the user/group changes to take effet.
2. Build the docker image:     
    ```
    ./docker/build_docker.sh
    ```    
    This does not build Theseus, but rather only creates a docker image that contains all the necessary dependencies to build and run Theseus. 
3. Run the new docker image locally as a container:    
    ```
    ./docker/run_docker.sh
    ```   
    Now you can run `make run` or other Theseus-specific build/run commands from within the docker container's shell.

Notes on Docker usage:    
  * The docker-based workflow should only require you to re-run the `run_docker.sh` script multiple times when re-building or running Theseus after modifying its code. You shouldn't need to re-run `build_docker.sh` multiple times, though it won't hurt.
  * KVM doesn't currently work in docker. To run Theseus in QEMU using KVM, you can build Theseus within docker, exit the container, and then run `make orun host=yes` on your host machine.


## Building and Running
To build and run Theseus in QEMU, simply run:   
```
make run
```

Run `make help` to see other make targets and the various command-line options.


## Using QEMU
QEMU allows us to run Theseus quickly and easily in its own virtual machine.
To exit Theseus in QEMU, press `Ctrl+Alt+G` (or `Ctrl+Alt` on some systems), which releases your keyboard and mouse focus from the QEMU window. Then press `Ctrl+C` in the terminal window that you ran `make run` from originally to kill QEMU, or you can also quit QEMU using the GUI `(x)` button on the title bar.

To investigate the hardware/machine state of the running QEMU VM, you can switch to the QEMU console by pressing `Ctrl+Alt+2`. Switch back to the main window with `Ctrl+Alt+1`. On Mac, manually select `VGA` or `compact_monitor0` under `View` from the QEMU menu bar.

To access/expose a PCI device in QEMU using PCI passthrough via VFIO, see [these instructions](book/src/pci_passthrough.md).

### KVM Support
While not strictly required, KVM will speed up the execution of QEMU.
To install KVM, run the following command:    
```
sudo apt-get install kvm
```  
To enable KVM support, add `host=yes` to your make command, e.g.,    
```
make run host=yes
```

# Documentation
Theseus includes two forms of documentation:
1. The [source-level documentation](https://theseus-os.github.io/Theseus/doc/___Theseus_Crates___/index.html), generated from code and inline comments (via *rustdoc*).
    * Intended for Theseus developers and contributors, or those who want low-level details.
2. The [book-style documentation](https://theseus-os.github.io/Theseus/book/index.html), written in Markdown.
    * Useful for high-level descriptions of design concepts and key components.

The above links may be out of date, so it's best to build the docs yourself, as shown below. 
The [latest version of the book is available here](book/src/SUMMARY.md).

Once your build environment is fully set up, you can build and view Theseus's source-level docs:
```
make view-doc
```

To build the Theseus book:
```
make view-book
```

# Other

## Booting on Real Hardware
We have tested Theseus on a variety of real machines, including Intel NUC devices, various Thinkpad laptops, and Supermicro servers. 
Currently, the only limiting factor is that the device support booting via USB or PXE using traditional BIOS rather than UEFI; support for UEFI is a work-in-progress. 

To boot over USB, simply run `make boot usb=sdc`, in which `sdc` is the device node for the USB disk itself *(**not a partition** like sdc2)* to which you want to write the OS image.
On WSL or other host environments where `/dev` device nodes don't exist, you can simply run `make iso` and burn the `.iso` file in the `build/` directory to a USB, e.g., using [Rufus](https://rufus.ie/) on Windows.

To boot Theseus over PXE (network boot), see [this set of separate instructions](book/src/pxe.md).




## Debugging Theseus on QEMU
GDB has built-in support for QEMU, but it doesn't play nicely with OSes that run in 64-bit long mode. In order to get it working properly with our OS in Rust, we need to patch it and build it locally. The hard part has already been done for us ([details here](https://os.phil-opp.com/set-up-gdb/)), so we can just quickly set it up with the following commands.  

1. Install the following packages:
    ```
    sudo apt-get install texinfo flex bison python-dev ncurses-dev
    ```

2. From the base Theseus directory, run this script to download and build GDB from an existing patched repo:
    ```
    curl -sf https://raw.githubusercontent.com/phil-opp/binutils-gdb/rust-os/build-rust-os-gdb.sh | sh
    ```
    After that, you should have a `rust-os-gdb` directory that contains the `gdb` executables and scripts. 

3. Run Theseus in QEMU using `make run` (or `make run_pause` to pause QEMU until we attach GDB).

4. In another terminal window, run the following to start GDB and attach it to the running QEMU instance:
    ```
    make gdb 
    ```
    QEMU will be paused until we move the debugger forward, with standard GDB commands like `n` to step through the next instruction or `c` to continue execution. Any standard GDB commands will now work.


## IDE Setup  
Our personal preference is to use Visual Studio Code (VS Code), which has excellent cross-platform support for Rust. Other options are available [here](https://areweideyet.com/).

For VS Code, recommended plugins are:
 * **rust-analyzer**, by matklad
 * **Better TOML**, by bungcip
 * **x86 and x86_64 Assembly**, by 13xforever

### Fixing Rustup, Rust Toolchain, or RLS Problems
Sometimes things just don't want to behave, especially if there were issues with the currently-chosen Rust nightly version.
In that case, try the following steps to fix it:
 * Set your default Rust toolchain to the one version in the `rust-toolchain` file, for example:
    ```
    rustup default $(cat rust-toolchain)
    ```
 * With your newly-set default toolchain, add the necessary components:    
    ```
    rustup component add rls rust-analysis rust-src
    ```
 * In VS Code (or whatever IDE you're using), uninstall and reinstall the Rust-related extension(s), restarting the IDE each time.
 * Check your IDE's settings to make sure that no weird Rust settings have been selected; building Theseus doesn't require any special settings. 
 * If you're still having issues, remove all other Rust toolchain versions besides the default one and try again. You can see other installed toolchains with `rustup toolchain list`.


## Acknowledgements
We would like to express our thanks to the [OS Dev wiki](https://wiki.osdev.org/) and its community and to Philipp Oppermann's [blog_os](https://os.phil-opp.com/) for serving as excellent starting points for Theseus. The early days of Theseus's development progress are indebted to these resources. 


# License
Theseus's source code is licensed under the MIT License. See the [LICENSE-MIT](LICENSE-MIT) file for more. 

# Contributing
We adhere to similar development and code style guidelines as the core Rust language project. See more [here](book/src/ch01.md)

PRs and issues are welcome from anyone; because Theseus is an experimental OS, certain features may be deprioritized or excluded from the main branch. Don't hesitate to ask or mention something though! :smile:
