---
title: "Kernel-Snow-Irony-Vince-LPP"
date: 2025-08-21T14:57:25+08:00
draft: false
---

![Snow_Irony Banner](https://raw.githubusercontent.com/mizuenaAlt/RenzAlt-Banner/refs/heads/main/snow-irony-vince.jpg)

```csharp
/*
 * I'm not responsible for bricked devices, dead SD cards, thermonuclear war, or you getting fired because the alarm app failed. 
 * Please do some research if you have any concerns about features included in the products you find here before flashing it! 
 * YOU are choosing to make these modifications, and if you point the finger at me for messing up your device, I will laugh at you. 
 * Your warranty will be void if you tamper with any part of your device / software.
 */
```

**DEVICES :** VINCE<br>
**BUILD DATE :** 21 August 2025<br>

**Changelog**
<ul>
    <li>SukiSU + SUSFS + KPM Intregrate</li>
    <li>Increase CMA region size 20MiB -> 32MiB (Qcom Sugestion)</li>
    <li>Undervolt 70mV CPU Voltage</li>
    <li>Undervolt PMIC Based On Xiaomi-YSL MSM8953MTP-PMI8940</li>
    <li>Update New Synaptic_TD4310 Touchscreen Drivers from msm-4.19</li>
    <li>Set efficiency size based on SDM450</li>
    <li>Increase GPU Mempool size to 256K based on MSM8956/8976</li>
    <li>Kgsl drop Adreno 3xx/4xx/6xx & snapshot/corsight/trace (don't make Adreno 506 load some useless stuff lol)</li>
    <li>Reduced gpu jump busy penalty to 8000 nsec Based on MSM8956/8976</li>
    <li>Disable Some Unused PMIC & CPU Drivers On Vince (Android Developer Sugestion)</li>
    <li>Drop Qcom mem_dump_v2 (no need to allocate mem_dump, we don't have much memory)</li>
    <li>Hardcode MSM8953 CLK (Reduce kernel & cpu workload)</li>
    <li>Change tick rate to 300 HZ (Android Developer Sugestion)</li>
    <li>Reduce rootwait time to 5ms (Samsung Sugestion)</li>
    <li>Disable some kernel debugs</li>
    <li>fix all errors in dmesg log</li>
    <li>And More</li>
</ul>

## Datasheet

**CMA Reserved Memory Region**
CMA (contiguous memory allocator) is a memory allocator within the kernel which allows allocating large chunks of memory with contiguous physical memory addresses.

Reasons to increase CMA Region Size :
- GPU memory is often allocated from the CMA pool, which is a shared memory region.

from my testing on the MSM8953 they have a CMA size of around 20MiB, look at this configuration :
```bash
linux,cma {
  compatible = "shared-dma-pool";
  alloc-ranges = <0x0 0x00000000 0x0 0xffffffff>;
  reusable;
  alignment = <0 0x400000>;
  size = <0 0x1400000>;
  linux,cma-default;
};
```

it seems 20MiB is not good because we are always running out of CMA memory based on this log :
```bash
vince:/ # dmesg | grep -i cma
[    0.000000] Reserved memory: created CMA memory pool at 0x00000000fdc00000, size 20 MiB
[    0.000000] cma: Not enough slots for CMA reserved regions!
[    0.000000] Reserved memory: unable to setup CMA region
[    0.000000] Memory: 2597260K/2979840K available (17916K kernel code, 2678K rwdata, 7500K rodata, 4096K init, 3034K bss, 132724K reserved, 249856K cma-reserved)

vince:/ # cat /proc/meminfo | grep Cma
CmaTotal:         249856 kB
CmaFree:               0 kB
```

Due to running out of CMA memory, the kernel floods the dmesg log with spam like this:
```bash
[24260.235503] cma: cma_alloc: alloc failed, req-size: 512 pages, ret: -12
```

to solve this problem i increased the size to <0 0x2000000> /32MiB and yes this proved to solve the problem see this :
```bash
vince:/ # dmesg | grep -i cma
[    0.000000] Reserved memory: created CMA memory pool at 0x00000000fdc00000, size 32 MiB
[    0.000000] Memory: 2597260K/2979840K available (17916K kernel code, 2678K rwdata, 7500K rodata, 4096K init, 3034K bss, 132724K reserved, 249856K cma-reserved)

vince:/ # cat /proc/meminfo | grep Cma
CmaTotal:         266240 kB
CmaFree:           11184 kB
```

yes but we will run out of CMA memory too, but it doesn't happen often, i maintain lmk to work together and help each other see this is what lmk does:
```bash
[  149.950550] lowmemorykiller: Killing 'd.process.media' (2846) (tgid 2846), adj 999,\x0ato free 81288kB on behalf of 'kswapd0' (136) because\x0acache 321644kB is below limit 322560kB for oom score 950\x0aFree memory is 160548kB above reserved.\x0aFree CMA is 6008kB\x0aTotal reserve is 45148kB\x0aTotal free pages is 211900kB\x0aTotal file cache is 491188kB\x0aGFP mask is 0x24000c0
```
Increasing the CMA reserved memory region is also one of Qualcomm's suggestions on this forum [Configure and manage memory qcom](https://docs.qualcomm.com/bundle/publicresource/topics/80-70020-3/memory.html)
thanks to @skhife @dnxnin on telegram for helping to solve this

**Prerequisites**
<ol>
    <li>Unlocked Bootloader</li>
</ol>


**Download**

[Kernel-Snow-Irony-Vince-LPP-SukiSU.zip](https://t.me/MI8953/16/5575)
