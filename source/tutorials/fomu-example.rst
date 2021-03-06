.. _fomu-tutorial:

Renode, Fomu and EtherBone bridge example
=========================================

This tutorial shows how to run a hybrid simulation where part of the platform is run in Renode and part on FPGA hardware.

System architecture
-------------------

The system architecture consists of three main parts:

* `FOMU <https://github.com/im-tomu/fomu-hardware>`_ - Lattice iCE40UP5K-based board with RGB LED, connected to the host machine over USB;
* EtherBone bridge, translating wishbone packets between TCP and USB;
* Renode simulating the `LiteX <https://github.com/enjoy-digital/litex>`_ platform, running Zephyr OS that controls the RGB LED.

Prerequisites
-------------

Simulation
++++++++++

For the simulation part, you need to build Renode from sources according to the :ref:`building instructions <building-from-source>`.

Hardware
++++++++

For the hardware part, you need to have the `FOMU`_ board.

`FOMU`_ needs to be flashed with the `Foboot <https://github.com/im-tomu/foboot>`_ bitstream containing the `ValentyUSB <https://github.com/mithro/valentyusb>`_ IP core with a USB-Wishbone bridge.
The prebuilt version of the bitstream is `hosted by Antmicro <https://antmicro.com/projects/renode/foboot-bitstream.bin-s_104250-fc5f419372eb9a3a0baa5556483163bcfccb7d33>`_.

EtherBone bridge
++++++++++++++++

In order to connect Renode to `FOMU`_ you need to download the EtherBone-to-USB bridge shipped with `LiteX`_.

Clone the repository and initialize the environment:

.. code-block:: bash

    git clone https://github.com/enjoy-digital/litex
    cd litex
    ./litex_setup.py init  # this will clone dependencies
    export PYTHONPATH=`pwd`:`pwd`/litex:`pwd`/migen

Loading the bitstream
---------------------

Download and make `iceprog <https://github.com/cliffordwolf/icestorm/tree/master/iceprog>`_ - open source programming software for Lattice iCE40:

.. code-block:: bash

    git clone https://github.com/cliffordwolf/icestorm
    cd icestorm/iceprog
    make

Download the prebuilt bitstream:

.. code-block:: bash

    wget https://antmicro.com/projects/renode/foboot-bitstream.bin-s_104250-fc5f419372eb9a3a0baa5556483163bcfccb7d33 -O foboot-bitstream.bin

.. note::

    You can also build the bitstream yourself by following the instructions on the `Foboot`_ page.

Program the FPGA:

.. code-block:: bash

    sudo iceprog foboot-bitstream.bin

.. note::

    For convenience you can use `the Fomu Programmer <https://github.com/antmicro/fomu-programmer>`_ - the Open Hardware programming board for `FOMU`_ by Antmicro.

Runnning the demo
-----------------

Plug `FOMU`_ into a USB port and verify that it has been recognized by looking at ``dmesg`` logs:

.. code-block:: text

    [65038.250957] usb 2-1: new full-speed USB device number 16 using xhci_hcd
    [65038.409283] usb 2-1: New USB device found, idVendor=1209, idProduct=5bf0, bcdDevice= 1.01
    [65038.409286] usb 2-1: New USB device strings: Mfr=1, Product=2, SerialNumber=0
    [65038.409287] usb 2-1: Product: Fomu DFU Bootloader v1.7.2-3-g9013054
    [65038.409288] usb 2-1: Manufacturer: Foosn

Start the EtherBone bridge from the `LiteX`_ repository:

.. code-block:: bash

    cd litex
    sudo python3 litex/tools/litex_server.py --usb --usb-vid 0x1209 --usb-pid 0x5bf0

Run the Zephyr OS image in simulation using the script shipped with Renode:

.. code-block:: text

    (monitor) start @scripts/complex/fomu/renode_etherbone_fomu.resc

Now you can control the HW LED form Zephyr's shell using special commands:

.. code-block:: bash

    uart:~$ led_toggle
    uart:~$ led_breathe

`led_toggle`
    toggles the green led

`led_breathe`
    makes the blue led blink with a fade-in/fade-out effect
