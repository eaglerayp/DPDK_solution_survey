# DPDK + TCP/IP solutions building step

## mtcp

* Github: https://github.com/eunyoung14/mtcp 

1. setting dpdk (include compile and setup.sh)
2. ./setup_iface_single_process.sh 3    (**create /dev/dpdk-ifce**)
3. use ifup with /etc/network/interfaces  setup IP, routing
4. create soft links   
```  
ln -s <path_to_dpdk_2_1_0_directory>/x86_64-native-linuxapp-gcc/lib lib  
ln -s <path_to_dpdk_2_1_0_directory>/x86_64-native-linuxapp-gcc/include include
```
5. setup mtcp library 
```
./configure --with-dpdk-lib=$<path_to_mtcp_release_v3>/dpdk  
cd mtcp/src  
make
```

6. set mtcp.conf and config/route.conf(arp.conf)  per app

## opendp

* Github: https://github.com/opendp/dpdk-odp

1.  setting dpdk (include compile and setup.sh)
2. compile opendp 
```
cd .../test/opendp/  
make  
```
3. export RTE_ODP=.../test
4. opendp application have to run **opendp** first, as a daemon-like program
```
sudo ./opendp -c 0x1 -n 4 -- -p 0x1 --config="(0,0,0)"  
```
5. [Configure iproute by commands] (https://github.com/opendp/dpdk-odp/wiki/Configure-iproute-by-commands)  
```
cd netdp_cmd  
sudo ./build/netdpcmd  
```
