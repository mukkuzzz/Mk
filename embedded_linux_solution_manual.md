# Embedded Linux System — Solution Manual

---

## Question (1): Shell Commands and File Operations

### Aim
To demonstrate the use of essential Linux shell commands for directory navigation, file creation, viewing, copying, renaming, and removal.

### Tools Required
- Host Linux workstation running bash terminal
- Core utilities: `pwd`, `mkdir`, `cd`, `touch`, `ls`, `echo`, `cat`, `cp`, `mv`, `rm`

### Theory
The Linux command-line shell provides a text interface to the underlying virtual file system. In Linux, all storage and device endpoints are organized in a single unified hierarchical tree starting at root (`/`). Navigation and manipulation commands operate by invoking kernel system calls:

- `mkdir` and `rmdir` invoke `sys_mkdir` and `sys_rmdir`.
- `touch` updates file modification timestamps via `sys_utimensat` or creates zero-byte files via `sys_openat` with `O_CREAT`.
- Output redirection (`>`) redirects the standard output file descriptor (FD 1) to a target file.
- `cp` copies data blocks from source to destination inode, while `mv` updates the directory entry without copying data blocks when within the same filesystem.

### Program
```bash
# 1. Check current directory and create working directory
pwd
mkdir demo && cd demo

# 2. Create empty files and view attributes
touch file1.txt file2.txt
ls -l

# 3. Write text into file1.txt and read it
echo "Hello, Linux" > file1.txt
cat file1.txt

# 4. Copy, rename, and remove files
cp file1.txt file1_copy.txt
mv file1_copy.txt file3.txt
rm file3.txt
ls -l
```

### Execution
Run the command sequence directly in the terminal:
```bash
./demo.sh   # Or execute the commands sequentially in the bash shell
```

### Inference
1. Running `ls -l` shows the file type, permission bits (`-rw-r--r--`), ownership, size, and timestamp.
2. The `cp` command preserves the source file while creating a new independent copy with a new inode.
3. The `mv` command atomically updates the directory reference to point to `file3.txt`.
4. The `rm` command unlinks the inode reference, releasing the storage block.

---

## Question (2): Bash Shell Scripting (sysinfo.sh)

### Aim
To write and execute a bash shell script (`sysinfo.sh`) demonstrating command substitution, directory inspection with conditionals, and loop iteration.

### Tools Required
- Host Linux workstation with bash shell
- Text editor (nano, vim, or gedit)

### Theory
A shell script is an interpreted plain text executable containing a sequence of commands.

- **Shebang (`#!/bin/bash`):** Tells the kernel loader which binary interpreter to execute.
- **Command Substitution (`$(...)`):** Executes a subshell command and captures its standard output into a string variable.
- **Conditionals (`if [ -f "$file" ]`):** Tests filesystem attributes where `-f` validates regular files and `-d` validates directories.
- **Loops (`for`, `while`):** Enable structured iteration over collections or numeric conditions using arithmetic expansion (`$((...))`).

### Program
Create script `sysinfo.sh`:
```bash
#!/bin/bash
# sysinfo.sh - displays system info and directory items
echo "===== System Information ====="
echo "Date     : $(date)"
echo "User     : $(whoami)"
echo "Hostname : $(hostname)"
echo "Kernel   : $(uname -r)"

echo -e "\n===== Directory Listing ====="
for file in *; do
    if [ -f "$file" ]; then
        echo "File : $file"
    elif [ -d "$file" ]; then
        echo "Dir  : $file"
    fi
done

echo -e "\n===== Iteration Check ====="
count=1
while [ $count -le 3 ]; do
    echo "Iteration $count"
    count=$((count + 1))
done
```

### Execution
```bash
chmod +x sysinfo.sh
./sysinfo.sh
```

### Inference
1. `chmod +x` sets the execution permission bit (`S_IXUSR`).
2. The script retrieves live system metrics via command substitution.
3. The `for` loop dynamically inspects directory contents and accurately classifies files and folders.

---

## Question (3): Cross Compilation: ARM64 Target

### Aim
To write a C program, cross-compile it for the 64-bit ARM architecture using the GNU cross-toolchain, and verify the binary.

### Tools Required
- Host system: x86_64 Linux workstation
- Cross-compiler: `gcc-aarch64-linux-gnu`
- Binary inspection tool: `file`

### Theory
Cross-compilation generates executable code on a host machine (e.g. x86_64) that targets a foreign architecture (e.g. ARM64). The compiler uses a target triplet prefix (`aarch64-linux-gnu-gcc`):

- `aarch64`: Target instruction set architecture (64-bit ARMv8-A).
- `linux`: Target operating system kernel ABI.
- `gnu`: C standard library implementation (glibc).

The output binary cannot run natively on the host processor because the host CPU does not recognize ARM64 machine instructions.

### Program
1. Toolchain Installation:
```bash
sudo apt update && sudo apt install -y gcc-aarch64-linux-gnu
```

2. Source code (`main.c`):
```c
#include <stdio.h>

int main() {
    int a = 17;
    int b = 31;
    printf("%d + %d = %d\n", a, b, a + b);
    return 0;
}
```

3. Compilation and verification commands:
```bash
# Cross-compile for ARM64 target
aarch64-linux-gnu-gcc main.c -o main-arm64

# Verify binary architecture
file main-arm64
```

### Execution
```bash
file main-arm64
```

### Inference
1. Output reports `ELF 64-bit LSB executable, ARM aarch64, version 1 (SYSV)`.
2. `ELF 64-bit LSB` confirms 64-bit word size and Little Endian byte ordering.
3. `ARM aarch64` verifies that compilation targeted the 64-bit ARM ISA; attempting `./main-arm64` on the host x86_64 CPU yields an `Exec format error`.

---

## Question (4): Cross Compilation: 32-bit ARM (ARMHF) Target

### Aim
To cross-compile a C program for 32-bit ARM Hard-Float architecture and verify the resulting binary.

### Tools Required
- Host system: x86_64 Linux workstation
- Cross-compiler: `gcc-arm-linux-gnueabihf`
- Inspection utility: `file`

### Theory
The `armhf` (ARM hard-float) port targets 32-bit ARM processors (ARMv7-A) equipped with a hardware Floating Point Unit (VFPv3/NEON). Function arguments for floating-point values are passed directly in hardware FPU registers (`s0`-`s15`, `d0`-`d7`) rather than general-purpose CPU registers (`r0`-`r3`), significantly improving computational performance over soft-float (`armel`).

### Program
1. Toolchain Installation:
```bash
sudo apt update && sudo apt install -y gcc-arm-linux-gnueabihf
```

2. Compilation Procedure:
```bash
# Cross-compile for 32-bit ARM Hard-Float
arm-linux-gnueabihf-gcc main.c -o main-armhf

# Inspect binary architecture
file main-armhf
```

### Execution
```bash
file main-armhf
```

### Inference
1. Output reports `ELF 32-bit LSB executable, ARM, EABI5 version 1 (SYSV)`.
2. Confirms 32-bit ARM architecture using the Embedded ABI revision 5 (EABI5).
3. Hardware floating-point calling conventions are baked directly into the binary ABI tags.

---

## Question (5): Cross Compilation: RISC-V 64 Target

### Aim
To cross-compile a C program for RISC-V 64-bit architecture and verify the resulting binary.

### Tools Required
- Host system: x86_64 Linux workstation
- Cross-compiler: `gcc-riscv64-linux-gnu`
- Verification tool: `file`

### Theory
RISC-V is an open, modular standard instruction set architecture (ISA) based on reduced instruction set computer (RISC) principles. The `riscv64-linux-gnu-gcc` cross-compiler generates 64-bit RISC-V machine instructions (RV64GC) with standardized registers (`x0` through `x31`).

### Program
1. Toolchain Installation:
```bash
sudo apt update && sudo apt install -y gcc-riscv64-linux-gnu
```

2. Compilation Procedure:
```bash
# Cross-compile for RISC-V 64-bit
riscv64-linux-gnu-gcc main.c -o main-riscv64

# Verify binary architecture
file main-riscv64
```

### Execution
```bash
file main-riscv64
```

### Inference
1. Output reports `ELF 64-bit LSB executable, UCB RISC-V, version 1 (SYSV)`.
2. Confirms successful compilation for the 64-bit open RISC-V architecture.

---

## Question (6): Cross Compilation: Target vs Host Verification

### Aim
To compare native host compilation against cross-compiled target binaries using the `file` utility.

### Tools Required
- Host system: x86_64 Linux workstation
- Native compiler: `gcc`
- Cross-compiler: `gcc-aarch64-linux-gnu`
- Inspection utility: `file`

### Theory
Native compilation builds binaries that execute directly on the compiler's host architecture. Cross-compilation translates source code into machine code for an alternative ISA. The `file` utility reads the 16-byte `e_ident` array and the `e_machine` field in the ELF header to report the target ISA without executing the file.

### Program
1. Toolchain Installation:
```bash
sudo apt update && sudo apt install -y gcc gcc-aarch64-linux-gnu
```

2. Compilation Procedure:
```bash
# 1. Compile natively on host
gcc main.c -o main-x86_64

# 2. Cross-compile for ARM64 target
aarch64-linux-gnu-gcc main.c -o main-arm64

# 3. Inspect and compare both binaries
file main-x86_64
file main-arm64
```

### Execution
```bash
file main-x86_64
file main-arm64
```

### Inference
1. `file main-x86_64` reports `ELF 64-bit LSB pie executable, x86-64`.
2. `file main-arm64` reports `ELF 64-bit LSB executable, ARM aarch64`.
3. The native binary executes directly on the host (`./main-x86_64`), while the ARM64 binary requires foreign hardware or an instruction emulator such as QEMU.

---

## Question (7): Kernel Configuration: Pi 4 (BCM2711) Defconfig

### Aim
To configure the Linux kernel for Raspberry Pi 4 (64-bit) using `menuconfig` and append a custom local release string.

### Tools Required
- Linux kernel source tree (Raspberry Pi tree `rpi-6.6.y`)
- Toolchain: `aarch64-linux-gnu-`
- Build packages: `libncurses-dev`, `bison`, `flex`, `make`

### Theory
The Linux kernel configuration system (Kconfig) manages thousands of compile-time options. Baseline board settings are pre-packaged as defconfigs. For the Raspberry Pi 4 (Broadcom BCM2711), `bcm2711_defconfig` sets up CPU architecture features, peripherals, and base bus drivers. The `CONFIG_LOCALVERSION` symbol allows developers to tag kernel releases with unique strings (e.g. for lab testing or production tracking).

### Program
1. Prerequisites and Toolchain Installation:
```bash
sudo apt update
sudo apt install -y git bc build-essential bison flex libssl-dev libncurses-dev gcc-aarch64-linux-gnu
```

2. Clone Kernel Source:
```bash
git clone --depth 1 -b rpi-6.6.y https://github.com/raspberrypi/linux.git
cd linux
```

3. Configuration Procedure:
```bash
export ARCH=arm64
export CROSS_COMPILE=aarch64-linux-gnu-

# 1. Load Pi 4 default configuration
make bcm2711_defconfig

# 2. Open menuconfig interface
make menuconfig
# Navigate: General setup > Local version - append to kernel release
# Enter: -rpi-custom-v1 > Select <Save> > <Exit>

# 3. Verify configuration output
grep "CONFIG_LOCALVERSION" .config
```

### Execution
```bash
grep "CONFIG_LOCALVERSION" .config
```

### Inference
1. The `.config` file confirms `CONFIG_LOCALVERSION="-rpi-custom-v1"`.
2. The generated kernel image appends this tag to `uname -r` upon boot, distinguishing it from upstream releases.

---

## Question (8): Kernel Configuration: Pi 5 (BCM2712) Defconfig

### Aim
To configure the Linux kernel for Raspberry Pi 5 using `menuconfig` and strip unused drivers (sound card).

### Tools Required
- Linux kernel source tree
- Toolchain: `aarch64-linux-gnu-`
- Build dependencies: `libncurses-dev`, `make`

### Theory
Embedded systems often operate with strict memory and storage constraints. Removing unnecessary device drivers from the kernel configuration (driver stripping) reduces the size of the compiled kernel image (`Image`), speeds up boot time, and decreases runtime memory footprint.

### Program
1. Prerequisites and Toolchain Installation:
```bash
sudo apt update
sudo apt install -y git bc build-essential bison flex libssl-dev libncurses-dev gcc-aarch64-linux-gnu
```

2. Clone Kernel Source:
```bash
git clone --depth 1 -b rpi-6.6.y https://github.com/raspberrypi/linux.git
cd linux
```

3. Configuration Procedure:
```bash
export ARCH=arm64
export CROSS_COMPILE=aarch64-linux-gnu-

# 1. Load Pi 5 default configuration
make bcm2712_defconfig

# 2. Open menuconfig to strip sound drivers
make menuconfig
# Navigate: Device Drivers > Sound card support
# Press 'N' to uncheck CONFIG_SOUND > Select <Save> > <Exit>

# 3. Verify driver status in .config
grep "CONFIG_SOUND" .config
```

### Execution
```bash
grep "CONFIG_SOUND" .config
```

### Inference
1. The resulting `.config` file confirms `# CONFIG_SOUND is not set`.
2. All dependent ALSA audio drivers and codecs are stripped, reducing compiled kernel binary size and runtime memory consumption.

---

## Question (9): Kernel Configuration: 32-bit Pi Defconfig

### Aim
To configure the Linux kernel for 32-bit Raspberry Pi boards with compiler size optimization (`-Os`).

### Tools Required
- Linux kernel source tree
- Toolchain: `arm-linux-gnueabihf-`
- Build packages: `libncurses-dev`, `make`

### Theory
By default, the Linux kernel compiles with speed optimizations (`-O2`). For memory-constrained 32-bit embedded targets, the kernel can be instructed to compile with `-Os` (Optimize for size). This prioritizes smaller code generation over raw execution speed, reducing instruction cache pressure.

### Program
1. Prerequisites and Toolchain Installation:
```bash
sudo apt update
sudo apt install -y git bc build-essential bison flex libssl-dev libncurses-dev gcc-arm-linux-gnueabihf
```

2. Clone Kernel Source:
```bash
git clone --depth 1 -b rpi-6.6.y https://github.com/raspberrypi/linux.git
cd linux
```

3. Configuration Procedure:
```bash
export ARCH=arm
export CROSS_COMPILE=arm-linux-gnueabihf-

# 1. Load 32-bit Pi defconfig
make bcm2711_defconfig

# 2. Open menuconfig and enable size optimization
make menuconfig
# Navigate: General setup > Compiler optimization level
# Select: Optimize for size (-Os) > <Save> > <Exit>

# 3. Verify configuration
grep "CONFIG_CC_OPTIMIZE_FOR_SIZE" .config
```

### Execution
```bash
grep "CONFIG_CC_OPTIMIZE_FOR_SIZE" .config
```

### Inference
1. The `.config` file confirms `CONFIG_CC_OPTIMIZE_FOR_SIZE=y`.
2. The build system passes `-Os` to GCC during compilation, yielding a compact binary tailored for memory-constrained 32-bit hardware.

---

## Question (10): Kernel Configuration: Pi 1 / Zero (BCM2835)

### Aim
To configure the Linux kernel for legacy Raspberry Pi 1 / Zero (BCM2835) boards and customize local release strings.

### Tools Required
- Linux kernel source tree
- Toolchain: `arm-linux-gnueabihf-`
- Build packages: `libncurses-dev`, `make`

### Theory
The Raspberry Pi 1 and Raspberry Pi Zero are based on the Broadcom BCM2835 SoC featuring an ARM1176JZF-S (ARMv6) core. Because ARMv6 uses an older instruction set than ARMv7/v8, using the specific `bcm2835_defconfig` ensures that incompatible instruction extensions are not emitted by the compiler.

### Program
1. Prerequisites and Toolchain Installation:
```bash
sudo apt update
sudo apt install -y git bc build-essential bison flex libssl-dev libncurses-dev gcc-arm-linux-gnueabihf
```

2. Clone Kernel Source:
```bash
git clone --depth 1 -b rpi-6.6.y https://github.com/raspberrypi/linux.git
cd linux
```

3. Configuration Procedure:
```bash
export ARCH=arm
export CROSS_COMPILE=arm-linux-gnueabihf-

# 1. Load BCM2835 defconfig
make bcm2835_defconfig

# 2. Open menuconfig to set local version
make menuconfig
# Navigate: General setup > Local version - append to kernel release
# Enter: -rpi-zero-v1 > Save and Exit

# 3. Verify configuration
grep "CONFIG_LOCALVERSION" .config
```

### Execution
```bash
grep "CONFIG_LOCALVERSION" .config
```

### Inference
1. The `.config` file confirms `CONFIG_LOCALVERSION="-rpi-zero-v1"`.
2. Baseline drivers from `arch/arm/configs/bcm2835_defconfig` ensure instruction compatibility with the legacy ARMv6 CPU core.

---

## Question (11): Character Device Driver (/dev/vled)

### Aim
To build, insert, test, and verify a virtual GPIO LED character device driver as a Loadable Kernel Module (LKM) using the Linux `miscdevice` framework.

### Tools Required
- Pre-installed driver source code `vled.c` (already available on the system computer)
- Host Linux system with kernel build headers (`linux-headers-$(uname -r)`)
- Build utilities and compiler: `gcc`, `make`
- Kernel module utilities: `insmod`, `rmmod`, `lsmod`, `dmesg`, `cat`, `echo`

### Theory
A character device driver streams data character-by-character to user space.

1. **Miscellaneous Device Subsystem (`miscdevice`):** A kernel convenience framework that automatically allocates Major Number 10 and creates the device node under `/dev/` upon registration via `misc_register()`, eliminating boilerplate `alloc_chrdev_region` and `class_create` code.
2. **User-Kernel Memory Isolation:**
   - `copy_from_user()`: Safely transfers command buffers across the hardware MMU boundary into kernel space.
   - `simple_read_from_buffer()`: Copies kernel-generated status strings safely to user buffers.
3. **State Transitions:** Decouples string parsing (`"1"`, `"0"`, `"on"`, `"off"`) into discrete `enum led_state` tokens (`LED_ON`, `LED_OFF`).

### Program
1. Kernel Headers and Build Tools Installation:
```bash
sudo apt update
sudo apt install -y build-essential linux-headers-$(uname -r)
```
> **Note:** The driver source file `vled.c` is already available on the system computer. Students inspect the source code and write the Makefile to compile the module.

2. Kernel Makefile (`Makefile`):
```makefile
obj-m = vled.o
KDIR = /lib/modules/$(shell uname -r)/build
PWD := $(shell pwd)

all:
	$(MAKE) -C $(KDIR) M=$(PWD) modules

clean:
	$(MAKE) -C $(KDIR) M=$(PWD) clean
```

### Execution
```bash
# 1. Build and insert module
make
sudo insmod vled.ko
lsmod | grep vled
ls -l /dev/vled

# 2. Test driver write and read operations
cat /dev/vled
echo "1" > /dev/vled
cat /dev/vled
echo "0" > /dev/vled
cat /dev/vled

# 3. Check kernel logs and unload
sudo dmesg | tail -n 10
sudo rmmod vled
```

### Inference
1. `misc_register()` registers the driver with Major Number 10 and dynamically creates `/dev/vled` with `0666` permissions.
2. `echo "1" > /dev/vled` invokes `fops.write`, safely transferring data into kernel memory via `copy_from_user()` and toggling driver state.
3. `cat /dev/vled` invokes `fops.read`, transferring synthesized hardware telemetry via `simple_read_from_buffer()`.

---

## Question (12): User-Space GPIO Control (GPIO 17)

### Aim
To configure and control a digital GPIO output pin directly from user space using `pinctrl` and automate toggling via a shell script.

### Tools Required
- Raspberry Pi 4B running Raspberry Pi OS
- 5mm LED and 330Ω current-limiting resistor
- Breadboard and jumper wires
- Utilities: `pinctrl`, `bash`

### Theory
Modern Linux kernels abstract SoC pins using the `pinctrl` subsystem. The `pinctrl` utility interacts directly with the SoC pin controllers without requiring manual sysfs exporting:

- `op`: Configures pin direction as Output.
- `dh`: Drives pin High (3.3V, logic level 1).
- `dl`: Drives pin Low (0V, ground).

A 330Ω resistor in series protects the Raspberry Pi SoC pin buffer from overcurrent.

### Hardware Wiring
- Connect LED Anode (+) through 330Ω resistor to GPIO 17 (Physical Pin 11).
- Connect LED Cathode (-) to GND (Physical Pin 6).

### Program
1. Utility Installation:
```bash
sudo apt update && sudo apt install -y raspi-utils
```

2. Interactive `pinctrl` verification:
```bash
pinctrl get 17
pinctrl set 17 op
pinctrl set 17 dh   # LED ON
pinctrl set 17 dl   # LED OFF
```

3. Automated Blinker Script (`toggle.sh`):
```bash
#!/bin/bash
PIN=17
pinctrl set $PIN op

# Restore pin LOW on exit (Ctrl+C)
trap "pinctrl set $PIN dl; exit" SIGINT SIGTERM

echo "Blinking GPIO $PIN every 500 ms. Press Ctrl+C to stop."
while true; do
    pinctrl set $PIN dh
    sleep 0.5
    pinctrl set $PIN dl
    sleep 0.5
done
```

### Execution
```bash
chmod +x toggle.sh
./toggle.sh
```

### Inference
1. The script alternates GPIO 17 between HIGH (3.3V) and LOW (0V) at 1 Hz (500 ms half-period).
2. The signal trap guarantees that pressing Ctrl+C forces GPIO 17 LOW before exit, leaving the hardware in a safe state.

---

## Question (13): Digital Sensor Interfacing (DHT11 IIO on GPIO 4)

### Aim
To interface a DHT11 temperature and humidity sensor via the Linux Industrial I/O (IIO) kernel driver and process telemetry from user space.

### Tools Required
- Raspberry Pi running Raspberry Pi OS
- DHT11 digital sensor module
- Jumper wires
- Utilities: `dtoverlay`, `awk`, `bash`

### Theory
The DHT11 sensor uses a proprietary single-wire bi-directional protocol requiring microsecond timing. Rather than bit-banging in user space, the Linux kernel provides native driver support (`dht11.c`) via the Industrial I/O (IIO) subsystem. A Device Tree overlay (`dtoverlay=dht11,gpiopin=4`) informs the kernel to bind the hardware pin to the driver, creating standardized sysfs nodes under `/sys/bus/iio/devices/iio:device0/`:

- `in_temp_input`: Live temperature in milli-degrees Celsius (m°C).
- `in_humidityrelative_input`: Live relative humidity in milli-percent (m%RH).

### Hardware Wiring
- Connect sensor DATA line to GPIO 4 (Physical Pin 7).
- Connect VCC to 3.3V (Physical Pin 1) and GND to Pin 6.

### Program
1. Package Installation:
```bash
sudo apt update && sudo apt install -y gawk
```

2. Configure Device Tree overlay in `/boot/firmware/config.txt`:
```
dtoverlay=dht11,gpiopin=4
```
(Reboot the board using `sudo reboot`.)

3. Telemetry processing script (`read_sensor.sh`):
```bash
#!/bin/bash
IIO_DIR="/sys/bus/iio/devices/iio:device0"

if [ ! -d "$IIO_DIR" ]; then
    echo "Error: IIO device not found at $IIO_DIR"
    exit 1
fi

while true; do
    raw_temp=$(cat "$IIO_DIR/in_temp_input" 2>/dev/null)
    raw_humidity=$(cat "$IIO_DIR/in_humidityrelative_input" 2>/dev/null)

    if [ -n "$raw_temp" ] && [ -n "$raw_humidity" ]; then
        temp=$(awk "BEGIN {printf \"%.1f\", $raw_temp / 1000}")
        humidity=$(awk "BEGIN {printf \"%.1f\", $raw_humidity / 1000}")
        echo "[$(date '+%H:%M:%S')] Temp: ${temp} °C | Humidity: ${humidity} %RH"
    fi
    sleep 2
done
```

### Execution
```bash
dmesg | grep -i dht11
chmod +x read_sensor.sh
./read_sensor.sh
```

### Inference
1. Kernel ring buffer logs (`dmesg`) verify that the native IIO DHT11 driver bound to GPIO 4.
2. Reading virtual sysfs files (`in_temp_input`, `in_humidityrelative_input`) avoids user-space bit-banging and allows microsecond-accurate data acquisition directly from user space.

---

## Question (14): Ultrasonic Sensor & MCP23017 (OLAT = 0x14, MASK = 0x02)

### Aim
To interface an HC-SR04 ultrasonic sensor, an MCP23017 I2C expander, and an SSD1306 OLED display with Raspberry Pi to trigger an actuator and display status when obstacle distance falls below 15 cm.

### Tools Required
- Raspberry Pi running Raspberry Pi OS
- HC-SR04 ultrasonic sensor
- MCP23017 16-bit I2C I/O expander
- SSD1306 OLED display (128x64, I2C address 0x3C)
- Python libraries: `gpiozero`, `smbus`, `luma.oled`

### Theory
1. **HC-SR04 Ultrasonic Ranging:** Measures time-of-flight (*t*) of a 40 kHz ultrasonic wave.

   `d = (t × 34300) / 2 cm`

2. **MCP23017 I2C Expander:** Operates at slave address `0x20` on I2C bus 1. Direction register `IODIRA` (`0x00`) sets Port A pin directions (`0x00` = output). Output latch register `OLATA` (`0x14`) writes logic levels using bitmask `0x02` (bit 1 HIGH).
3. **SSD1306 OLED:** Communicates over I2C to render graphical telemetry and alert status strings.

### Hardware Wiring
- **HC-SR04:** Trigger on GPIO 10 (Pin 19), Echo on GPIO 9 (Pin 21).
- **MCP23017:** SDA on GPIO 2 (Pin 3), SCL on GPIO 3 (Pin 5), address `0x20`.
- **Actuator:** Pre-wired to MCP23017 Port A.
- **SSD1306:** SDA on GPIO 2, SCL on GPIO 3, address `0x3C`.

### Program
1. Enable I2C and Install Dependencies:
```bash
sudo raspi-config nonint do_i2c 0
sudo apt update
sudo apt install -y python3-gpiozero python3-smbus python3-pil python3-pip
pip3 install luma.oled --break-system-packages
```

2. Python Application (`main.py`):
```python
import time, smbus
from gpiozero import DistanceSensor
from luma.core.interface.serial import i2c
from luma.oled.device import ssd1306
from luma.core.render import canvas

ADDR = 0x20
OLAT = 0x14
MASK = 0x02

bus = smbus.SMBus(1)
oled = ssd1306(i2c(port=1, address=0x3c))
sensor = DistanceSensor(echo=9, trigger=10, max_distance=2)

# Set Port A to output
bus.write_byte_data(ADDR, 0x00, 0x00)

try:
    while True:
        dist = round(sensor.distance * 100, 1)
        alert = dist < 15
        bus.write_byte_data(ADDR, OLAT, MASK if alert else 0x00)
        status = "ALERT" if alert else "STATUS: SAFE"

        with canvas(oled) as draw:
            draw.text((0, 0), "Distance: ", fill="white")
            draw.text((0, 16), f"{dist} cm", fill="white")
            draw.text((0, 40), status, fill="white")

        print(f"Distance: {dist} cm | Status: {status}")
        time.sleep(0.5)
finally:
    bus.write_byte_data(ADDR, OLAT, 0x00)
```

### Execution
```bash
python3 main.py
```

### Inference
1. The ultrasonic sensor measures obstacle proximity continuously over GPIO pins 10 and 9.
2. When distance falls below 15 cm, writing bitmask `0x02` to latch `OLATA` (`0x14`) immediately drives the assigned actuator.
3. The SSD1306 OLED updates concurrently over I2C bus 1 with measured distance and alert status.

---

## Question (15): Ultrasonic Sensor & MCP23017 (OLAT = 0x13, MASK = 0x08)

### Aim
To interface an HC-SR04 ultrasonic sensor, an MCP23017 I2C expander, and an SSD1306 OLED display to trigger an actuator using Port B register `OLAT = 0x13` with `MASK = 0x08` below 15 cm.

### Tools Required
- Raspberry Pi running Linux
- HC-SR04 ultrasonic sensor
- MCP23017 I2C expander (address `0x20`)
- SSD1306 OLED display (address `0x3C`)
- Python libraries: `gpiozero`, `smbus`, `luma.oled`

### Theory
The MCP23017 expander contains two 8-bit ports (Port A and Port B). In this variant, Port B is used:

- Direction register `IODIRB` (`0x01`): Setting `0x00` configures all Port B pins as outputs.
- Output latch register `OLATB` (`0x13`): Writing `0x08` sets bit 3 HIGH (pin GPB3).

### Program
1. Enable I2C and Install Dependencies:
```bash
sudo raspi-config nonint do_i2c 0
sudo apt update
sudo apt install -y python3-gpiozero python3-smbus python3-pil python3-pip
pip3 install luma.oled --break-system-packages
```

2. Python Application (`main.py`):
```python
import time, smbus
from gpiozero import DistanceSensor
from luma.core.interface.serial import i2c
from luma.oled.device import ssd1306
from luma.core.render import canvas

ADDR = 0x20
OLAT = 0x13
MASK = 0x08

bus = smbus.SMBus(1)
oled = ssd1306(i2c(port=1, address=0x3c))
sensor = DistanceSensor(echo=9, trigger=10, max_distance=2)

# Set Port B direction to outputs (IODIRB = 0x01)
bus.write_byte_data(ADDR, 0x01, 0x00)

try:
    while True:
        dist = round(sensor.distance * 100, 1)
        alert = dist < 15
        bus.write_byte_data(ADDR, OLAT, MASK if alert else 0x00)

        status = "ALERT" if alert else "STATUS: SAFE"

        with canvas(oled) as draw:
            draw.text((0, 0), "Distance: ", fill="white")
            draw.text((0, 16), f"{dist} cm", fill="white")
            draw.text((0, 40), status, fill="white")

        print(f"Distance: {dist} cm | Status: {status}")
        time.sleep(0.5)
finally:
    bus.write_byte_data(ADDR, OLAT, 0x00)
```

### Execution
```bash
python3 main.py
```

### Inference
1. Direction register `IODIRB` (`0x01`) configures Port B pins as digital outputs.
2. Writing `0x08` to output latch `OLATB` (`0x13`) drives pin GPB3 HIGH when obstacle proximity drops below 15 cm.
3. Telemetry rendering on the SSD1306 OLED reflects real-time distance and alert states synchronously.

---

## Question (16): User Systemd Service & Timer Automation

### Aim
To create, configure, and manage user-level systemd service and timer units (`systemctl --user`) to automate periodic logging without root privileges and enable boot lingering.

### Tools Required
- Linux workstation running systemd
- Shell utilities: `systemctl --user`, `loginctl`, `bash`

### Theory
1. **User Systemd Services:** Run within an unprivileged user session under the user's UID. They require no `sudo` privileges, isolate environment variables (`$HOME`, `$PATH`), and cannot damage root system files.
2. **Service Unit (`.service`):** Configured with `Type=oneshot` to execute a script to completion.
3. **Timer Unit (`.timer`):** Provides a systemd-native cron replacement using monotonic timers (`OnBootSec`, `OnUnitActiveSec`).
4. **Session Lingering (`loginctl enable-linger`):** By default, user systemd daemons terminate when the user logs out. Enabling lingering allows the user daemon to launch at system boot and run indefinitely.

### Program
1. Logging script (`~/timelog.sh`):
```bash
#!/bin/bash
echo "$(date)" > "$HOME/file.txt"
```
Make executable: `chmod +x ~/timelog.sh`

2. Service unit (`~/.config/systemd/user/timelog.service`):
```ini
[Unit]
Description=Log timestamp to file (User Service)

[Service]
Type=oneshot
ExecStart=%h/timelog.sh

[Install]
WantedBy=default.target
```

3. Timer unit (`~/.config/systemd/user/timelog.timer`):
```ini
[Unit]
Description=Trigger timelog.service every 5 seconds

[Timer]
OnBootSec=5s
OnUnitActiveSec=5s
AccuracySec=1us

[Install]
WantedBy=timers.target
```

### Execution
```bash
# Reload user daemon configuration
systemctl --user daemon-reload

# Enable and start timer immediately
systemctl --user enable --now timelog.timer

# Check active timer schedule
systemctl --user list-timers

# View dynamically appended log entries
tail -f ~/file.txt

# Enable linger mode so service runs without interactive login
loginctl enable-linger $USER
```

### Inference
1. `systemctl --user list-timers` confirms that monotonic timers provide precise periodic task execution without cron.
2. Enabling linger mode via `loginctl` allows user services to launch at system boot and persist without active shell sessions.
3. Rootless execution guarantees environment isolation and system security.
