# Qemu/KVM Virtual Machine Mitigation Techniques
# Index
- [Spoofing System Information](github.com/jraffstar/vm-mitigations/blob/main/README.md#spoofing-system-information)
- [HyperV Enlightenments](https://github.com/jraffstar/vm-mitigations/blob/main/README.md#hyperv-enlightenments) 

## Spoofing System Information
One of the methods that can be used to assist with software not detecting that you're running it in a virtual machine is spoofing the hardware information.
By default, in your virtual machine it will report information like the manufacturer for example as "QEMU" or "VMware" etc. Software with anti virtual machine measures in place will likely check these details and if they return with a value that is a guaranteed virtual machine value, it will determine that the software is running in a virtual machine.
To work around this we can replace these values with values that will be usually reported if ran in an actual PC, we will be using our own PCs information for this:
Firstly, we will acquire our system information by running these commands:
```
dmidecode --type bios
dmidecode --type baseboard
dmidecode --type system
```
Using the information acquired from these commands, you can now add this to your libvirt XML configuration, preferably replacing the information with your own systems information
```xml
  <sysinfo type="smbios">
    <bios>
      <entry name="vendor">American Megatrends Inc.</entry>
      <entry name="version">F31o</entry>
      <entry name="date">12/03/2020</entry>
    </bios>
    <system>
      <entry name="manufacturer">Gigabyte Technology Co., Ltd.</entry>
      <entry name="product">X570 AORUS ULTRA</entry>
      <entry name="version">x.x</entry>
      <entry name="serial">BASEBOARD SERIAL HERE (or "Default string")</entry>
      <entry name="uuid">BASEBOARD UUID HERE</entry>
      <entry name="sku">BASEBOARD SKU HERE (or "Default string")</entry>
      <entry name="family">X570 MB</entry>
    </system>
  </sysinfo>
```
You could also alternatively, or alongside this, add certain qemu arguments to spoof system information
```xml
<qemu:commandline>
  <qemu:arg value="-smbios"/>
  <qemu:arg value="type=0,version=UX305UA.201"/>
  <qemu:arg value="-smbios"/>
  <qemu:arg value="type=1,manufacturer=ASUS,product=UX305UA,version=2021.1"/>
  <qemu:arg value="-smbios"/>
  <qemu:arg value="type=2,manufacturer=Intel,version=2021.5,product=Intel i9-12900K"/>
  <qemu:arg value="-smbios"/>
  <qemu:arg value="type=3,manufacturer=XBZJ"/>
  <qemu:arg value="-smbios"/>
  <qemu:arg value="type=17,manufacturer=KINGSTON,loc_pfx=DDR5,speed=4800,serial=000000,part=0000"/>
  <qemu:arg value="-smbios"/>
  <qemu:arg value="type=4,manufacturer=Intel,max-speed=4800,current-speed=4800"/>
  <qemu:arg value="-cpu"/>
  <qemu:arg value="host,family=6,model=158,stepping=2,model_id=Intel(R) Core(TM) i9-12900K CPU @ 2.60GHz,vmware-cpuid-freq=false,enforce=false,host-phys-bits=true,hypervisor=off"/>
</qemu:commandline>
```
If done correctly, when you check your system information in the guests operating system, it should report the values you entered in here.

## HyperV Enlightenments
Very important values to add to your libvirt XML configuration are these values
```xml
  <kvm>
    <hidden state="on"/>
  </kvm>
```
```xml
 <features>
    <hyperv mode="custom">
      <relaxed state="on"/>
      <vapic state="on"/>
      <spinlocks state="on" retries="8191"/>
      <vendor_id state="on" value="GenuineIntel"/>
    </hyperv>
    <kvm>
      SEE ABOVE
    </kvm>
    <vmport state="off"/>
    <smm state="on"/>
    <ioapic driver="kvm"/>
  </features>
  <cpu mode="host-passthrough" check="none" migratable="on">
    <feature policy="disable" name="hypervisor"/>
  </cpu>
```
Adding these values will enable HyperV Enlightenments and allow for Nested Virtualization in your virtual machine. Make sure you enable HyperV in the virtual machine after making these changes for them to fully take action.
### What exactly is "Hidden State"?
Probably one of the most important values we're adding here is "<hidden state="on"/>". KVM exposes extra vCPU internals, This includes things like: VMXON/VMCS data (Intel), VMCB/nested state (AMD), Internal APIC state, FPU extended xsave components that aren’t part of user-visible state and more. The reasoning for it exposing these in the first place is to stop the virtual machine state from breaking when doing tasks such as migrating the virtual machine, or suspending it. This information isn't usually reported, so if it is, that's a very likely sign that it is a virtual machine. Enabling Hidden State stops it from exposing this information. It also may potentially hide some parts of the vCPUs identity.

### Downsides:
- You may experience varying amounts of performance loss after enabling these
- You wont be able to suspend your virtual machine anymore

### Negating the performance loss:
The other values we added such as "<relaxed state="on"/>" etc. are features that will increase performance in other ways, in effect slightly reducing the performance loss that enabling HyperV Enlightenments may cause. The one value just mentioned "Relaxed State" when enabled, tells the Guest OS that it is running on a virtual CPU so it can not waste time on waiting for certain hardware events that will not occur in a virtual machine. Even though it is telling the Guest OS that it is a virtual CPU, this shouldnt cause any virtual machine detection. However this may be subject to change in the future.

# Sources + Extra resources + Tools
- <https://docs.vrchat.com/docs/using-vrchat-in-a-virtual-machine>
- <https://github.com/zhaodice/qemu-anti-detection>
- <https://deepwiki.com/dsecuma/qemu-anti-detection/3-anti-detection-techniques>
- <https://github.com/nbviet300689/VM-Undetected>
- <https://github.com/Scrut1ny/Hypervisor-Phantom>
- <https://github.com/iaoedsz2008/libvirt-stealth>
- <https://github.com/zhaodice/proxmox-ve-anti-detection>
- <https://www.youtube.com/watch?v=Iass2FMHHng> / <https://pastebin.com/raw/w2UZd5GR>
- <https://www.qemu.org/docs/master/system/i386/hyperv.html>


- <https://github.com/a0rtega/pafish>
- <https://github.com/kernelwernel/VMAware>
- <https://github.com/Banaxi-dev/Vm-Detector>
