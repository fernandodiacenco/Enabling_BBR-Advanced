# Enabling BBR Advanced

A how-to on how to enable BBR-Advanced congestion control on your Linux Server, a improved version of Google's BBR congestion control with reduced re-transmissions and improved fairness.

---

<b>INTRODUCTION</b>

Congestion control algorithms are used on the network for the purpose of maintaining operations under adverse conditions, and today we are presented with a wide variety of choices for our use cases

This is a tutorial on how to install BBR Advanced congestion control, an improvement over Google's original BBR focusing on reduced retransmissions and fairness, made by Dr. Imtiaz Mahmud.

There are caveats however:

It does not compile on current kernels, I managed to compile it only on (LTS) Kernel version 4.14, it did not compile on any newer LTS version tried 4.19, 5.4, 5.10)

There is a need to compile the Kernel, which is time consuming and requires higher than average knowledge

The instructions bellow worked virtual machines running Debian 10 and Ubuntu 20.04 with a very slight difference at the start (installing prerequisites), my advice is to try and test on virtual machines first before applying on production

Note that at the time of this writing the latest LTS Kernel 4.14 is 4.14.232, it is likely to be updated as time goe on, so be sure to check kernel.org and adapt the instructions accordingly

---

<b>INSTALLING AND COMPILING THE KERNEL</b>

PREREQUISITES FOR DEBIAN 10

    sudo apt install build-essential bison flex libelf-dev libssl-dev libncurses-dev bc git wget

PREREQUISITES FOR UBUNTU 20.04

    sudo apt install libncurses-dev libssl-dev libelf-dev

And then

    cd /usr/src && sudo wget https://cdn.kernel.org/pub/linux/kernel/v4.x/linux-4.14.232.tar.xz

    sudo tar -xvf linux-4.14.232.tar.xz

    sudo git clone https://github.com/imtiaztee/BBR-Advanced--BBR-A--Linux-Kernel-Code && cd BBR-Advanced--BBR-A--Linux-Kernel-Code

    sudo cp tcp_bbr.c /usr/src/linux-4.14.232/net/ipv4/

    sudo nano /usr/src/linux-4.14.232/include/net/inet_connection_sock.h

Go to this line

    #define ICSK_CA_PRIV_SIZE      (11 * sizeof(u64))

And change it to

    #define ICSK_CA_PRIV_SIZE      (12 * sizeof(u64))

    sudo nano /usr/src/linux-4.14.232/Makefile

Add this text to the Extraversion line

    EXTRAVERSION = -with-BBR-Advanced

    cd /usr/src/linux-4.14.232/

    sudo make localmodconfig

Just hold enter for the questions until finish

    sudo make menuconfig

Go to: Networking support → Networking options → TCP: advanced congestion control

Go to BBR TCP and change from <M> to <*> with the spacebar

Go to Default TCP congestion control and choose (X) BBR

Save and exit

    sudo make -j$(nproc) modules_prepare

    sudo make -j$(nproc) modules

    sudo make -j$(nproc) modules_install

    sudo make -j$(nproc)

    sudo make install

Reboot and on Grub Menu, select Advanced Options -> Linux 4.14.232-with-BBR-Advanced

Check if is running with

    sudo cat /proc/sys/net/ipv4/tcp_congestion_control

If it shows bbr, congratulations


---

<b>CONCLUSION</b>

BBR-Advanced if probably useful in cases where the default BBR is not achieving the desired results, but having to use and specially compile an earlier kernel version makes it less attractive due to the time and knowlege invested to make it work, and also due to the presumably less Kernel features compared to newer versions.

I can only hope someday the author updates the code to work as a kernel module, which might make it Kernel version agnostic.

Thanks for Alex Sparda for the tip on this congestion control.

---

<b>REFERENCES</b>

BBR-A Github: https://github.com/imtiaztee/BBR-Advanced--BBR-A--Linux-Kernel-Code</br>
BBR-A Paper: https://www.sciencedirect.com/science/article/pii/S2405959520301296?via%3Dihub</br>
Google's BBR Announcement - https://cloud.google.com/blog/products/networking/tcp-bbr-congestion-control-comes-to-gcp-your-internet-just-got-faster</br>
