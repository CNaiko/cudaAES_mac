
# AES Encryption and Decryption on Tesla C2075
by Zhibin Zhang, Lerong Chen

## Conclusion of our experiment
By proper configuration to maximize the throughput, we got the following maximum throughput:
  - AES Encryption: 19678Mbps
  - AES Decryption: 18926Mbps

Since the experiment shows that this application throughputs are not bounded by the memory and I/O operations, we optimistically estimate that the throughputs of this application can be 4x-6x using Titan X.(Although the architecture is different, we estimate it by the number of CUDA cores.)

The left of this report shows the detailed results.
## Experiment Setup
This experiment is setup using CUDA-5.5 on the Tesla C2075. This Fermi C2075 GPU has *448 CUDA cores*, interconnected by *PCI-E 2.0 x 16* bus interface.

## Experiment Result

This AES CUDA application can have several parameters:
  - the massage size as a input batch
  - the number of massages
  - the number of streams

We measure AES performance by varying number of messages in a batch. We use 16KB message which is the maximum record size in the SSL protocol. For AES operations, all the data is copied from the host memory to graphics card's memory, processed in it, and copied back to the host memory.

Using the *nvprof* to measure the time and time percentages of memory operations, we have got the following results:

### Encryption Results
This is the throughput results under stream = 1, an the maximum throughput in this case is 8467Mbps.
```bash
[zzb@Workstation-4 bin]$ nvprof ./aes_test -m ENC -s 1
------------------------------------------
AES-128-CBC ENC, Size: 16KB
------------------------------------------
#msg #stream latency(usec) thruput(Mbps)
==57608== NVPROF is profiling process 57608, command: ./aes_test -m ENC -s 1
   1       1          7363            17
   2       1          7677            34
   4       1          8417            62
   8       1         10406           100
  16       1         13837           151
  32       1         18363           228
  64       1         19351           433
 128       1         20453           820
 256       1         24846          1350
 512       1         26261          2555
1024       1         28953          4635
2048       1         34472          7787
4096       1         63400          8467


==57608== Profiling application: ./aes_test -m ENC -s 1
==57608== Profiling result:
Time(%)      Time     Calls       Avg       Min       Max  Name
 84.92%  24.2857s      1300  18.681ms  7.3703ms  42.619ms  AES_cbc_128_encrypt_kernel_SharedMem(unsigned char const *, unsigned char*, unsigned int const *, unsigned char const *, unsigned char*, unsigned int, unsigned char*)
  7.85%  2.24375s      1300  1.7260ms  5.6950us  11.214ms  [CUDA memcpy HtoD]
  7.23%  2.06862s      1300  1.5913ms  5.5030us  10.344ms  [CUDA memcpy DtoH]

```

If I increase the number of streams to 4, the throughput is maximized to 19678Mbps . The following result show this observation:

```bash
[zzb@Workstation-4 bin]$ nvprof ./aes_test -m ENC -s 4
------------------------------------------
AES-128-CBC ENC, Size: 16KB
------------------------------------------
#msg #stream latency(usec) thruput(Mbps)
==57621== NVPROF is profiling process 57621, command: ./aes_test -m ENC -s 4
   1       4          2199            59
   2       4          2256           116
   4       4          2428           215
   8       4          2913           359
  16       4          3783           554
  32       4          4882           859
  64       4          5081          1650
 128       4          5843          2871
 256       4          6057          5539
 512       4          6448         10407
1024       4          9319         14402
2048       4         14013         19156
4096       4         27270         19687
==57621== Profiling application: ./aes_test -m ENC -s 4
==57621== Profiling result:
Time(%)      Time     Calls       Avg       Min       Max  Name
 88.10%  33.6707s      1339  25.146ms  8.1991ms  102.74ms  AES_cbc_128_encrypt_kernel_SharedMem(unsigned char const *, unsigned char*, unsigned int const *, unsigned char const *, unsigned char*, unsigned int, unsigned char*)
  6.07%  2.31816s      1339  1.7313ms  5.6640us  11.976ms  [CUDA memcpy HtoD]
  5.83%  2.22795s      1339  1.6639ms  5.4080us  12.241ms  [CUDA memcpy DtoH]
```  
On the other hand, the nvprof information shows that the memory copy time is around 11%-15% among the application execution time. So we can conclude that this AES application on Tesla C2075 is not bounded by the memory I/O speed. The maximum transfer speed of the PCI-E 2.0 x16, 8GB/s, also is not the bottleneck of this application. So that this application performance can be improved using the GPU with more computing resources.

### Decryption results

We also measure the AES Decryption performance using the Tesla C2075. The results are shown blow:

```bash
[zzb@Workstation-4 bin]$ nvprof ./aes_test -m DEC -s 4
------------------------------------------
AES-128-CBC DEC, Size: 16KB
------------------------------------------
#msg #stream latency(usec) thruput(Mbps)
==57654== NVPROF is profiling process 57654, command: ./aes_test -m DEC -s 4
   1       4            46          2849
   2       4            47          5577
   4       4            48         10922
   8       4            57         18396
  16       4           113         18558
  32       4           221         18978
  64       4           437         19195
 128       4           880         19065
 256       4          1754         19130
 512       4          3503         19157
1024       4          7089         18933
2048       4         14180         18930
4096       4         28366         18926
==57654== Profiling application: ./aes_test -m DEC -s 4
==57654== Profiling result:
Time(%)      Time     Calls       Avg       Min       Max  Name
 48.74%  4.85594s      1338  3.6293ms  43.781us  23.338ms  AES_cbc_128_decrypt_kernel_SharedMem(unsigned char const *, unsigned char*, unsigned char*, unsigned char*, unsigned short*, unsigned long, unsigned char*)
 27.28%  2.71817s      1338  2.0315ms  6.0160us  13.762ms  [CUDA memcpy HtoD]
 23.98%  2.38897s      1338  1.7855ms  5.4080us  12.279ms  [CUDA memcpy DtoH]

```
