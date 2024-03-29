# Fio for RPMA Installation
Install the rpma that apply RDMA method on pmem. Use fio to test the performance of the program. ***Please install the mellanox drivers and pmem drivers correctly before this installation.***
## Environment

**System:** Fedora 31
**Kernel:** 5.3.7-301.fc31.x86_64
**Hardware:** pmem, RNIC
**Software:** rpma, fio, ipmctl, ndctl ... (Enough softwares to support pmem and RNIC)

## Requirements

Please go to https://github.com/pmem/rpma/blob/master/INSTALL.md for rpma and  https://github.com/pmem/fio for fio to get the latest version of installation.

**rpma:**

	$ yum -y install cmake pkg-config libibverbs-devel librdmacm-devel libcmocka-devel libpmem-devel libprotobuf-c-devel
	$ git clone https://github.com/pmem/rpma
	$ cd rpma && mkdir build && cd build
	$ cmake ..
	$ make -j
	$ make install

**fio:**

	$ git clone https://github.com/pmem/fio
	$ cd fio && ./configure
	$ make
	$ make install

***Note:***  if your kernel version or system version is too low, you may not have or support **cmake**. You should manually compile cmake package or update your versions. 


***Check** the configuration of fio engines in case that the fio fails. 
	
	$./configure | grep librpma
	librpma                 yes
	# in ...fio/ directory 

***Note:*** If the librpma in fio configure is 'no', it means the rpma's path is not detected by the fio. You should try either the two ways below:
* Add ***prefix=/usr/local*** after the $make install of the **rpma**  

		$ make install prefix=/usr/local
* Add ***pkgs_path*** to the ~/.bashrc file.

		$ export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib64/
		$ source ~/.bashrc

## Fio Test

Before testing the real cases' performance, you may first test the max bind width of the transation:
	
	$ numactl -N 1 ib_read_bw -a -d mlx5_0 -p 999
	# on server
	# -N [number] should be the cpu where your RNIC is on.
	$ numactl -N 1 ib_read_bw -a -F 192.168.01 -d mlx5_0 -p 999
	# on client
	# 192.168.0.1 should be the virtual address of server.

On the **server**( listener ) server, in the fio directory:

	$ cd examples 
	$ fio librpma-server.fio
	# check the librpma-server.fio and fill in the blanks

On the **client**( sender ) server, in the fio directory:
	
	$ cd examples 
	$ fio librpma-client.fio
	# check the librpma-client.fio and fill in the blanks

***Note:*** if there is something wrong with the send-receive process, the listening process can't be interrupted by Ctrl+C. You should use **kill** to finish it before next try.

### Possible improvements

*You may also control the fio arguments by command line:

	$ vim librpma-client.fio
	iodepth=${depth}
	$ depth=8 fio librpma-client.fio

*You may also use numactl to improve performance:
	
	$ cat /sys/class/net/enp24s0f0/device/numa_node
	 0              #RNIC_NUMA
	$ numactl -N $RNIC_NUMA ./benchmark args
	# e.g. numactl -N 0 depth=8 fio librpma-client.fio
