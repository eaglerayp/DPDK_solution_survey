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

6. set mtcp.conf and config/route.conf(arp.conf) per app 
```
/*
Route.conf and arp.conf settings example
(Connect to Internet have to set static roule and must set their arp same as proxy in this version.)
*/
ROUTES 1   // how many routing rules , route.conf choose which dpdk port to send pkts
173.194.72.100/32 dpdk0

ARP_ENTRY 2
10.128.83.254/32 48:0F:CF:0A:A7:E3   //proxy
173.194.72.100/32 48:0F:CF:0A:A7:E3  
//because mtcp lack of gateway settings, we have to set gateway by setting outer network's mac same as gateway, 
//so port know to send pkts to gateway, then gateway forward them.
```  



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
netdp> ip addr del 2.2.2.2/24 dev eth0
netdp> ip addr add 10.128.80.44/22 dev eth0
netdp> ip route add 0.0.0.0/0 via 10.128.83.254
quit
// can ping
```

## libos-nuse

* Github: https://github.com/libos-nuse/net-next-nuse
* run libnuse application control nic.
* Usage: test protocol in its document instead of performance.

## rumprun

* Github: https://github.com/rumpkernel/drv-netif-dpdk
* have to build rumprunkernel first, because rumprun-dpdk run under rumprun kernel.
