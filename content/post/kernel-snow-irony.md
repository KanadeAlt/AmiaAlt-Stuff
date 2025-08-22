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
    <li>Use prlmk for memory killer (@neophyte Sugestion)</li>
    <li>Undervolt 70mV CPU Voltage</li>
    <li>Undervolt PMIC Based On Xiaomi-YSL MSM8953MTP-PMI8940</li>
    <li>Update New Synaptic_TD4310 Touchscreen Drivers from msm-4.19</li>
    <li>Set efficiency size based on SDM450</li>
    <li>Increase GPU Mempool size to 256K based on MSM8956/8976</li>
    <li>Kgsl drop Adreno 3xx/4xx/6xx & snapshot/corsight/trace (don't make Adreno 506 load some useless stuff lol)</li>
    <li>Reduced gpu jump busy penalty to 8000 nsec Based on MSM8956/8976</li>
    <li>Disable Some Unused PMIC & CPU Drivers On Vince (AOSP Sugestion)</li>
    <li>Drop Qcom mem_dump_v2 (no need to allocate mem_dump, we don't have much memory)</li>
    <li>Hardcode MSM8953 CLK (Reduce kernel & cpu workload)</li>
    <li>Change tick rate to 300 HZ (AOSP Sugestion)</li>
    <li>Reduce rootwait time to 5ms (Samsung Sugestion)</li>
    <li>Disable some kernel debugs</li>
    <li>fix all errors in dmesg log</li>
    <li>And More</li>
</ul>

## Snow-Irony Datasheet

### CMA Reserved Memory Region

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

thanks to [@skhife](https://t.me/skhife) [@dnxnin](https://t.me/dnxnin) on telegram for helping to solve this

### Prlmk

A memory-saving program for Android, based on the process reclaim driver. It's especially suitable for devices with 4GB of RAM or less. It replaces Android Low Memory Killer.

prlmk configuration :
```bash
CONFIG_ANDROID_LOW_MEMORY_KILLER=n
CONFIG_PRLMK=y
```

Yes, this is really much better than Android's low memory killer, thanks for the suggestion [@k4ngcaribug](https://t.me/k4ngcaribug)

### Undervolt CPU & PMIC Voltage

Undervolt is the process of reducing the voltage supplied to certain components such as CPU, GPU, PMIC.

The impact of undervolt :
- much better battery backup without affecting/deteriorating performance.
- much better temperature.

Bad effects of undervolt :
- Each hardware has its own voltage limit value. If you lower the voltage incorrectly, it will have a bad impact on the hardware.

I have almost all xiaomi-msm8953 devices and I noticed that the devices using MSM8953QRD + PMI8950 use higher voltage compared to MSM8953MTP + PMI8940. and mido, vince, rosy, daikura, tissot are QRD Devices and ysl, tiffany, oxygen are MTP devices

Some Differences in the MSM8953 QRD & MSM8953 MTP Regulator Tables, see here :
```
 -----------------------------
     MSM8953QRD Regulator
 -----------------------------
 PMIC Data     |      Voltage
 -----------------------------
 lab_reg       |      5.700V           
 pm8953_l10    |      2.850V           
 pm8953_l19    |      1.380V          
 pm8953_l23    |      1.200V          
```

```
 -----------------------------
     MSM8953MTP Regulator
 -----------------------------
 PMIC Data     |      Voltage
 -----------------------------
 lab_reg       |      5.500V           
 pm8953_l10    |      2.800V           
 pm8953_l19    |      1.200V          
 pm8953_l23    |      0.975V
```

so i applied the `xiaomi-ysl` PMIC voltage to the `xiaomi-vince` and also some other undervolt PMICs see this :
```
 -------------------------------------------------------------------------------------
                         Vince-LPP LDO
 -------------------------------------------------------------------------------------
 PMIC Data     |           Voltage           |                Interface
 -------------------------------------------------------------------------------------
 lab_reg       |      5.700V -> 5.500V       |        Lab,IBB
 pm8953_l6     |      1.800V -> 1.750v       |        MDSS, DSI PLL, Display
 pm8953_l10    |      2.800V -> 2.750V       |        Sensor
 pm8953_l19    |      1.380V -> 1.200V       |        WCNS
 pm8953_l23    |      1.200V -> 0.975V       |
```
<tr>
  <td>
    <img src="https://raw.githubusercontent.com/mizuenaAlt/Amia-Lab/refs/heads/main/after.png" width="290" height="580" align="center" />
  </td>
</tr>

and because of this change I managed to get a good battery backup, also a reduction in excessive heat :v

this is the original schema data of MSM8953 from QCOM : [MSM8953 Schematics](https://file.elecfans.com/web2/M00/0A/DF/pYYBAGD7uHOABTBQAA96XdA1FDw057.pdf)

### CONFIG_HZ / Timer frequency

An interrupt causes the CPU to temporarily stop the execution of the current process and allocate its resources to the process or hardware that issued the interrupt. For example, when you press the volume button, an interrupt is triggered by the volume button and the CPU then changes the volume level and displays the changed volume level on the screen. Similarly, interrupts are triggered when you touch the screen or press any hardware button. There is a time gap between two consecutive interrupts. We call this the timer frequency.

A timer frequency of 100Hz means the interrupt will be triggered every 10 milliseconds and a timer frequency of 1000Hz means the interrupt will be triggered after 1 millisecond. Which one is better? 1000Hz will provide the lowest latency and smoothest experience. But it will increase overhead on CPU and increase battery consumption. 100Hz will keep CPU overhead minimal but may increase latency. In my tests on the Deco kernel on vince, reimu-lpp on ysl & tenshin on spesn, I didn't notice any lag caused by this change, but I did feel the battery lasted a little longer than usual. So, for now, stick with 300Hz.

changing the Timer frequency is also one of the suggestions from source.android.com [Identify jitter-related jank](https://source.android.com/docs/core/tests/debug/jank_jitter#long_threads)

### Optimize boot times & Reduce CPU & GPU workload

To make the device boot faster and reduce CPU workload, I did some things suggested by source.android.com, such as using lz4 on Ramdisk, disabling UART Serial Console and removing useless drivers for our device.

change GZIP to LZ4 example configuration :
```bash
CONFIG_RD_LZ4=y
```

To disable Serial Console do like this :
```bash
# CONFIG_SERIAL_EARLYCON is not set
# CONFIG_SERIAL_MSM_CONSOLE is not set
```

also add this `console=null` in boot configuration, see this example :
```bash
chosen {
    bootargs = "core_ctl_disable_cpumask=0-7 kpti=0 cgroup.memory=nokmem,nosocket cgroup_disable=pressure noirqdebug nodebugmon console=null";
};
```

Disable some useless drivers for xiaomi-vince, look at this :
```bash
# EFI
# CONFIG_EFI is not set

# Unused CPU Drivers
# CONFIG_ARCH_SDM450 is not set
# CONFIG_ARCH_SDM632 is not set
# CONFIG_CORESIGHT is not set
# CONFIG_CORESIGHT_LINK_AND_SINK_TMC is not set
# CONFIG_CORESIGHT_QCOM_REPLICATOR is not set
# CONFIG_CORESIGHT_STM is not set
# CONFIG_CORESIGHT_TPDA is not set
# CONFIG_CORESIGHT_TPDM is not set
# CONFIG_CORESIGHT_CTI is not set
# CONFIG_CORESIGHT_EVENT is not set
# CONFIG_CORESIGHT_HWEVENT is not set

# Unused PMIC Drivers
# CONFIG_SMB135X_CHARGER is not set
# CONFIG_SMB1355_SLAVE_CHARGER is not set
# CONFIG_SMB1351_USB_CHARGER is not set
# CONFIG_QPNP_SMB5 is not set
# CONFIG_QPNP_QG is not set
# CONFIG_QPNP_TYPEC is not set
# CONFIG_LEDS_QTI_TRI_LED is not set
# CONFIG_LEDS_QPNP_VIBRATOR_LDO is not set

# Unused Hardware Support
# CONFIG_NFC_NQ is not set
# CONFIG_INPUT_HBTP_INPUT is not set
# CONFIG_SCSI_UFSHCD is not set
# CONFIG_SCSI_UFSHCD_PLATFORM is not set
# CONFIG_SCSI_UFS_QCOM is not set
# CONFIG_SCSI_UFS_QCOM_ICE is not set

# Debug
# CONFIG_FB_MSM_MDSS_XLOG_DEBUG is not set
# CONFIG_MSM_SMD_DEBUG is not set
# CONFIG_RMNET_DATA_DEBUG_PKT is not set
# CONFIG_MMC_PERF_PROFILING is not set
# CONFIG_MSM_CAMERA_DEBUG is not set
# CONFIG_MSMB_CAMERA_DEBUG is not set
# CONFIG_IOMMU_DEBUG is not set
# CONFIG_IOMMU_DEBUG_TRACKING is not set

# Reduce size
CONFIG_IKHEADERS=n
CONFIG_SLUB_DEBUG=n
```

remove some things that GPU doesn't need see this commit reference : [81a0753](https://github.com/Snow-Irony/android_kernel_qcom_msm8953/commit/81a07535ef7285db4844f744b528073a807af114) [d87b07a](https://github.com/Snow-Irony/android_kernel_qcom_msm8953/commit/d87b07ab1150302c359bc72c9d3376b8dcb7e638) [3d2cfe9](https://github.com/Snow-Irony/android_kernel_qcom_msm8953/commit/3d2cfe962787dca659e89cf653fe6df924a492da)

This change is also a suggestion from source.android.com in this document [Optimize boot times android](https://source.android.com/docs/core/perf/boot-times)

Drop qcom mem dump v2 (since we don't have much memory, deleting this can also speed up booting since we don't need to create mem_dump_region on reserved memory which we know reserved memory is loaded first during boot)

Detailed log reserved memory in dmesg :
```bash
vince:/ # dmesg | grep -i Reserved memory
[    0.000000] Reserved memory: created CMA memory pool at 0x000000008f800000, size 8 MiB
[    0.000000] Reserved memory: created CMA memory pool at 0x00000000c7800000, size 8 MiB
[    0.000000] Reserved memory: created CMA memory pool at 0x00000000ff000000, size 16 MiB
[    0.000000] Reserved memory: created CMA memory pool at 0x00000000fdc00000, size 20 MiB
[    0.000000] OF: reserved mem: initialized node linux,cma, compatible id shared-dma-pool
[    0.000000] Reserved memory: created CMA memory pool at 0x00000000f2800000, size 180 MiB
[    0.000000] Reserved memory: created CMA memory pool at 0x00000000f2400000, size 4 MiB
[    0.000000] Reserved memory: created CMA memory pool at 0x00000000f2000000, size 4 MiB
[    0.000000] Reserved memory: created CMA memory pool at 0x00000000f1c00000, size 4 MiB
```

Disabled qcom mem dump v2 look at this :
```bash
# CONFIG_QCOM_MEMORY_DUMP_V2 is not set
```

Drop qcom mem dump v2 on DTS :
```bash
dump_mem: mem_dump_region {
    compatible = "shared-dma-pool";
    reusable;
    size = <0 0x400000>;
};

mem_dump {
    compatible = "qcom,mem-dump";
    memory-region = <&dump_mem>;

    rpm_sw_dump {
        qcom,dump-size = <0x28000>;
        qcom,dump-id = <0xea>;
    };

    pmic_dump {
        qcom,dump-size = <0x10000>;
        qcom,dump-id = <0xe4>;
    };

    vsense_dump {
        qcom,dump-size = <0x10000>;
        qcom,dump-id = <0xe9>;
    };

    tmc_etf_dump {
        qcom,dump-size = <0x10000>;
        qcom,dump-id = <0xf0>;
    };

    tmc_etr_reg_dump {
        qcom,dump-size = <0x1000>;
        qcom,dump-id = <0x100>;
    };

    tmc_etf_reg_dump {
        qcom,dump-size = <0x1000>;
        qcom,dump-id = <0x101>;
    };

    misc_data_dump {
        qcom,dump-size = <0x1000>;
        qcom,dump-id = <0xe8>;
    };
};
```

reduce rootwait time to 5ms (Samsung Sugestion)

For several devices, the rootwait time is sensitive because it directly
affects booting time.  The polling interval of rootwait is currently
100ms.  To save unnessesary waiting time, reduce the polling interval to
5 ms.

reference : [beffec8](https://github.com/Snow-Irony/android_kernel_qcom_msm8953/commit/beffec86f272e3e401ad0e6f14fd03c2005eab13)

Yes, all these changes really speed up booting, and also reduce CPU workload test on vince-exthmui also android 16 btw :v

## Download

[Kernel-Snow-Irony-Vince-LPP-SukiSU.zip](https://t.me/MI8953/16/5589)

[SukiSU_v3.1.8_13250-release.apk](https://t.me/MI8953/16/5550)
