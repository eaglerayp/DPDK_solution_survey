# DPDK + TCP/IP solutions building step

## DPDK SDK
### LINUX
* reference: http://dpdk.org/doc/guides/linux_gsg/sys_reqs.html
* hugepage大小和數量的boot設定寫在kernel command line：/etc/default/grub e.g.,:
```
// /etc/default/grub 
GRUB_CMDLINE_LINUX_DEFAULT="text default_hugepagesz=2M hugepagesz=2M hugepages=1024 iommu=pt intel_iommu=on quiet splash"

$ sudo update-grub 
//會把設定寫入/boot/grub/grub.cfg
```
隨時調整2M hugepage數量：`echo 2048 > /sys/kernel/mm/hugepages/hugepages-2048kB/nr_hugepages`
因為hugepage是連續的memory, 所以儘量在boot的時候就先設定/預留下來, 像是1G hugepage因為太大,所以就只能在boot設定無法隨時調整數量,
從`cat /proc/cpCPU的flags`：如果pse存在，hugepage可用2M; pdpe1gb，可用1G。
* use hugepage by DPDK: add line:`nodev /mnt/huge hugetlbfs defaults 0 0` to the `/etc/fstab` file
* comment dpdkfolder/lib/librte_eal/linuxapp/kni/ethtool/igb/kcompat.h Line:3868 skb_set_hash if encouter build error
* use /tools/setup.sh  

## mtcp

* Github: https://github.com/eunyoung14/mtcp 

* setting dpdk (include compile and setup.sh)   mtcp support to dpdk-2.1.0
* sudo ./tools/setup_iface_single_process.sh 3    (**create /dev/dpdk-ifce, modify IP settings in /etc/network/interfaces, so arg useless**) modified example script:
```
//    /setup_iface_single_process.sh   argument is useless in my setting
.
while [ $counter -gt 0 ]
do
    ifconfig dpdk$(($counter - 1))  up
    ifup dpdk$(($counter - 1)) 
    let "counter=$counter - 1"
done
//    /etc/network/interfaces
iface dpdk0 inet static
address 10.128.80.44
netmask 255.255.252.0
gateway 10.128.83.254
```
* use ifup with /etc/network/interfaces  setup IP, routing
* create soft links   
```  
cd dpdk/
ln -s ../dpdk-2.1.0/x86_64-native-linuxapp-gcc/lib lib  
ln -s ../dpdk-2.1.0/x86_64-native-linuxapp-gcc/include include
```
* setup mtcp library 
```
./configure --with-dpdk-lib=$(pwd)/dpdk  
cd mtcp/src  
make
```

* set mtcp.conf and config/route.conf(arp.conf) per app 
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
/*
because mtcp lack of gateway settings, we have to set gateway by setting outer network's mac same as gateway, 
so port know to send pkts to gateway, then gateway forward them.
*/
```  



## opendp

* Github: https://github.com/opendp/dpdk-odp
* live project, commit frequently.

1. setting dpdk (include compile and setup.sh)  opendp support dpdk-2.1.0
2. compile opendp and its application (make in folder)
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
* The main purpose: test protocol from their document.
* build dpdk process learned from: https://github.com/libos-nuse/linux-libos-tools/issues/19
1. build dpdk-enable NUSE
```
cd nuse
make defconfig
make library ARCH=lib
./arch/lib/tools/dpdk-sdk-build.sh  //comment lib kcompat.h Line:3868 skb_set_hash if encouter build error
make library ARCH=lib DPDK=yes
```
2. set nuse.conf
```
interface dpdk0
	address 10.128.80.44
	netmask 255.255.252.0
	macaddr 00:15:17:8f:c2:45
	viftype DPDK

route
	network 0.0.0.0
	netmask 0.0.0.0
	gateway 10.128.83.254
```
3. set nic to dpdk mode
4. test 
```
sudo NUSECONF=nuse.conf ./nuse ping 210.242.127.88
```


## rumprun

* Github: https://github.com/rumpkernel/drv-netif-dpdk
* library usage: https://www.freelists.org/post/rumpkernel-users/Component-descriptions-was-Re-Question-Rumpkernel-Profiling-Rumpconf
* Now, we cannot compile kqueue in Linux.   rumpapi(/rump/include/rump/rump_syscalls.h)沒有包含kqueue使用所需的header檔,詢問作者的結果是只有在BSD寫過kqueue的program且在BSD上還無法build DPDK.   另外,因為rump不是把kqueue轉換成epoll所以在rump+Nginx上運行可能會有同時存在epoll和kqueue的情況.目前還想不到解法,先Stop!

### LINUX
```
git clone https://github.com/rumpkernel/drv-netif-dpdk.git
cd drv-netif-dpdk
git submodule update --init
chmod +x buildme.sh
comment dpdkfolder/lib/librte_eal/linuxapp/kni/ethtool/igb/kcompat.h Line:3868 skb_set_hash if encouter build error
export RTE_SDK=$(pwd)/dpdk
./buildme.sh   //it will update netBSD src, setup DPDK, rumpmake, compile example
//can run simple tcp, but no epoll and kqueue.
```

### FreeBSD  (temp, I cannot build DPDK!!)
```
pkg install bash
pkg install gmake
pkg install git
pkg install lang/gcc49  
// mv /usr/local/bin/gcc49 /usr/bin/cc  ln -s cc cc1
git clone https://github.com/rumpkernel/drv-netif-dpdk.git
cd drv-netif-dpdk
git submodule update --init   // NetBSD have to add CA
chmod +x buildme.sh

export RTE_SDK=$(pwd)/dpdk
mv /usr/bin/make /usr/bin/make_backup
cp /usr/local/bin/gmake /usr/bin/make
./buildme.sh
```
