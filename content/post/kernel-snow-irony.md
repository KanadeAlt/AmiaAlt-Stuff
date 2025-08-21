---
title: "Kernel-Snow-Irony-Vince"
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


**DEVICES** :VINCE

**BUILD DATE :** 21 August 2025

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

**Prerequisites**
<ol>
    <li>Unlocked Bootloader</li>
</ol>


**Download**

[bla.zip](https://drive.google.com/file/d/1-lvZfze9TwtPFIfLVaWBeG9qem3wsqKo/view?pli=1)
