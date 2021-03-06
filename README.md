# Corundum Readme

GitHub repository: https://github.com/ucsdsysnet/corundum

## Introduction

Corundum is an open-source, high-performance FPGA-based NIC.  Features include
a high performance datapath, 10G/25G/100G Ethernet, PCI express gen 3, a
custom, high performance, tightly-integrated PCIe DMA engine, many (1000+)
transmit, receive, completion, and event queues, MSI interrupts, multiple
interfaces, multiple ports per interface, per-port transmit scheduling
including high precision TDMA, flow hashing, RSS, checksum offloading, and
native IEEE 1588 PTP timestamping.  A Linux driver is included that integrates
with the Linux networking stack.  Development and debugging is facilitated by
an extensive simulation framework that covers the entire system from a
simulation model of the driver and PCI express interface on one side to the
Ethernet interfaces on the other side.

Corundum has several unique architectural features.  First, transmit, receive,
completion, and event queue states are stored efficiently in block RAM or
ultra RAM, enabling support for thousands of individually-controllable
queues.  These queues are associated with interfaces, and each interface can
have multiple ports, each with its own independent scheduler.  This enables
extremely fine-grained control over packet transmission.  Coupled with PTP time
synchronization, this enables high precision TDMA.

Corundum currently supports Xilinx Virtex 7, Ultrascale, and Ultrascale+ series
devices.  Designs are included for the following FPGA boards:

*  Alpha Data ADM-PCIE-9V3 (Xilinx Virtex Ultrascale+ XCVU3P)
*  Exablaze ExaNIC X10 (Xilinx Kintex Ultrascale XCKU035)
*  Exablaze ExaNIC X25 (Xilinx Kintex Ultrascale+ XCKU3P)
*  NetFPGA SUME (Xilinx Virtex 7 XC7V690T)
*  Xilinx VCU108 (Xilinx Virtex Ultrascale XCVU095)
*  Xilinx VCU118 (Xilinx Virtex Ultrascale+ XCVU9P)
*  Xilinx VCU1525 (Xilinx Virtex Ultrascale+ XCVU9P)

For operation at 10G and 25G, Corundum uses the open source 10G/25G MAC and
PHY modules from the verilog-ethernet repository, no extra licenses are
required.  However, it is possible to use other MAC and/or PHY modules.
Operation at 100G currently requires using the Xilinx CMAC core with RS-FEC
enabled, which is covered by the free CMAC license on Xilinx Ultrascale+ parts.

## Documentation

### Modules

#### cmac_pad module

Frame pad module for 512 bit 100G CMAC TX interface.  Zero pads transmit
frames to minimum 64 bytes.

#### cpl_op_mux module

Completion operation multiplexer module.  Merges completion write operations
from different sources to enable sharing a single cpl_write module instance.

#### cpl_queue_manager module

Completion queue manager module.  Stores device to host queue state in block
RAM or ultra RAM.

#### cpl_write module

Completion write module.  Responsible for writing completion and event entries
into host memory.

#### desc_fetch module

Descriptor fetch module.  Responsible for reading descriptors from host memory.

#### desc_op_mux module

Descriptor operation multiplexer module.  Merges descriptor fetch operations
from different sources to enable sharing a single cpl_write module instance.

#### event_mux module

Event mux module.  Enables multiple event sources to feed the same event queue.

#### interface module

Interface module.  Contains the event queues, interface queues, and ports.

#### port module

Port module.  Contains the transmit and receive engines

#### queue_manager module

Queue manager module.  Stores host to device queue state in block RAM or ultra
RAM.

#### rx_checksum module

Receive checksum computation module.  Computes 16 bit checksum of Ethernet
frame payload to aid in IP checksum offloading.

#### rx_engine module

Receive engine.  Manages receive descriptor dequeue and fetch via DMA, packet
reception, data writeback via DMA, and completion enqueue and writeback via
DMA.  Handles PTP timestamps for inclusion in completion records.

#### rx_hash module

Receive hash computation module.  Extracts IP addresses and ports from packet
headers and computes 32 bit Toeplitz flow hash.

#### tdma_ber_ch module

TDMA bit error ratio test channel module.  Controls PRBS logic in Ethernet PHY
and accumulates bit errors.  Can be configured to bin error counts by TDMA
timeslot.

#### tdma_ber module

TDMA bit error ratio test module.  Wrapper for a tdma_scheduler and multiple
instances of tdma_ber_ch.

#### tdma_scheduler module

TDMA scheduler module.  Generates TDMA timeslot index and timing signals from
PTP time.

#### tx_checksum module

Transmit checksum computation and insertion module.  Computes 16 bit checksum
of frame data with specified start offset, then inserts computed checksum at
the specified position.

#### tx_engine module

Transmit engine.  Manages receive descriptor dequeue and fetch via DMA, packet
data fetch via DMA, packet transmission, and completion enqueue and writeback
via DMA.  Handles PTP timestamps for inclusion in completion records.

#### tx_scheduler_ctrl_tdma module

TDMA transmit scheduler control module.  Controls queues in a transmit
scheduler based on PTP time, via a tdma_scheduler instance.

#### tx_scheduler_rr module

Round-robin transmit scheduler.  Determines which queues from which to send
packets.

### Source Files

    cmac_pad.v               : Pad frames to 64 bytes for CMAC TX
    cpl_op_mux.v             : Completion operation mux
    cpl_queue_manager.v      : Completion queue manager
    cpl_write.v              : Completion write module
    desc_fetch.v             : Descriptor fetch module
    desc_op_mux.v            : Descriptor operation mux
    event_mux.v              : Event mux
    event_queue.v            : Event queue
    interface.v              : Interface
    port.v                   : Port
    queue_manager.v          : Queue manager
    rx_checksum.v            : Receive checksum offload
    rx_engine.v              : Receive engine
    rx_hash.v                : Receive hashing module
    tdma_ber_ch.v            : TDMA BER channel
    tdma_ber.v               : TDMA BER
    tdma_scheduler.v         : TDMA scheduler
    tx_checksum.v            : Transmit checksum offload
    tx_engine.v              : Transmit engine
    tx_scheduler_ctrl_tdma.v : TDMA transmit scheduler controller
    tx_scheduler_rr.v        : Round robin transmit scheduler

## Testing

Running the included testbenches requires MyHDL and Icarus Verilog.  Make sure
that myhdl.vpi is installed properly for cosimulation to work correctly.  The
testbenches can be run with a Python test runner like nose or py.test, or the
individual test scripts can be run with python directly.

### Testbench Files

    tb/axi.py            : MyHDL AXI4 master and memory BFM
    tb/axil.py           : MyHDL AXI4 lite master and memory BFM
    tb/axis_ep.py        : MyHDL AXI Stream endpoints
    tb/eth_ep.py         : MyHDL Ethernet frame endpoints
    tb/ip_ep.py          : MyHDL IP frame endpoints
    tb/mqnic.py          : MyHDL mqnic driver model
    tb/pcie.py           : MyHDL PCI Express BFM
    tb/pcie_us.py        : MyHDL Xilinx Ultrascale PCIe core model
    tb/pcie_usp.py       : MyHDL Xilinx Ultrascale+ PCIe core model
    tb/ptp.py            : MyHDL PTP clock model
    tb/udp_ep.py         : MyHDL UDP frame endpoints
    tb/xgmii_ep.py       : MyHDL XGMII endpoints

## Publications

- A. Forencich, A. C. Snoeren, G. Porter, G. Papen, *Corundum: An Open-Source 100-Gbps NIC,* in FCCM'20

## Citation
If you use Corundum in your project please cite one of the following papers
and/or link to the github project:

```
@inproceedings{forencich2020fccm,
    author = {Alex Forencich and Alex C. Snoeren and George Porter and George Papen},
    title = {Corundum: An Open-Source {100-Gbps} {NIC}},
    booktitle = {28th IEEE International Symposium on Field-Programmable Custom Computing Machines},
    year = {2020},
}
```

## Dependencies

Corundum internally uses the following libraries:

*  https://github.com/alexforencich/verilog-axi
*  https://github.com/alexforencich/verilog-axis
*  https://github.com/alexforencich/verilog-ethernet
*  https://github.com/alexforencich/verilog-pcie
*  https://github.com/solemnwarning/timespec

