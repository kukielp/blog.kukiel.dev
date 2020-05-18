---
layout: post
title:  "Running Cockroach DB on a Raspberry Pi"
date:   2020-05-16 08:38:34 +1000
categories: postgres database cockroach rpi raspberrypi docker
---
I have had my eye on [CockroachDB](https://www.cockroachlabs.com/) for some time.  Put simply Cockroach DB is a Postgres compatible Multi-Master database meaning you can have multiple write nodes.  I have set this up locally approximately 6 months ago.  It was effortless to get going on MacOS.  This weekend I was diving a little deeper and wanted to see if I could get Cockroach to run on my RaspberryPi 4.

The RaspberryPi 4 supports a 1.5GHz quad-core ARM Cortex-A72 CPU.  The key difference here is ARM, there are no compiled binaries availiable.  I was unable to compile the source code without some modifications to a few files which II will document eventually.  The end result was I have a cluster running across a Raspberry Pi4 and a Machine running Ubuntu on an intel i3.

## Result: 

![alt text](/assets/post/2020-05-16-cockroach-db-local/cockroachdb.png "CockroachDB on Rasberry Pi 4")

CPUs
{% highlight bash %}
ubuntu@ubuntu:~$ lscpu
Architecture:                    aarch64
CPU op-mode(s):                  32-bit, 64-bit
Byte Order:                      Little Endian
CPU(s):                          4
On-line CPU(s) list:             0-3
Thread(s) per core:              1
Core(s) per socket:              4
Socket(s):                       1
Vendor ID:                       ARM
Model:                           3
Model name:                      Cortex-A72
Stepping:                        r0p3
CPU max MHz:                     1500.0000
CPU min MHz:                     600.0000
BogoMIPS:                        108.00
Vulnerability Itlb multihit:     Not affected
Vulnerability L1tf:              Not affected
Vulnerability Mds:               Not affected
Vulnerability Meltdown:          Not affected
Vulnerability Spec store bypass: Vulnerable
Vulnerability Spectre v1:        Mitigation; __user pointer sanitization
Vulnerability Spectre v2:        Vulnerable
Vulnerability Tsx async abort:   Not affected
Flags:                           fp asimd evtstrm crc32 cpuid
{% endhighlight %}
{% highlight bash %}
paulk@server:~$ lscpu
Architecture:        x86_64
CPU op-mode(s):      32-bit, 64-bit
Byte Order:          Little Endian
CPU(s):              4
On-line CPU(s) list: 0-3
Thread(s) per core:  2
Core(s) per socket:  2
Socket(s):           1
NUMA node(s):        1
Vendor ID:           GenuineIntel
CPU family:          6
Model:               58
Model name:          Intel(R) Core(TM) i3-3110M CPU @ 2.40GHz
Stepping:            9
CPU MHz:             1197.181
CPU max MHz:         2400.0000
CPU min MHz:         1200.0000
BogoMIPS:            4788.67
Virtualisation:      VT-x
L1d cache:           32K
L1i cache:           32K
L2 cache:            256K
L3 cache:            3072K
NUMA node0 CPU(s):   0-3
Flags:               fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe syscall nx rdtscp lm constant_tsc arch_perfmon pebs bts rep_good nopl xtopology nonstop_tsc cpuid aperfmperf pni pclmulqdq dtes64 monitor ds_cpl vmx est tm2 ssse3 cx16 xtpr pdcm pcid sse4_1 sse4_2 x2apic popcnt tsc_deadline_timer xsave avx f16c lahf_lm cpuid_fault epb pti ssbd ibrs ibpb stibp tpr_shadow vnmi flexpriority ept vpid fsgsbase smep erms xsaveopt dtherm arat pln pts md_clear flush_l1d
{% endhighlight %}