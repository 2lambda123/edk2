# UEFI Redfish EDK2 Implementation

## Introduction
UEFI Redfish EDK2 solution is an efficient and secure solution for the end-users to remote configure (in Out-of-band) UEFI platform configurations by leveraging the Redfish RESTful API.  It's simple for end-users to access the configurations of UEFI firmware which have the equivalent properties defined in Redfish schema.

Below are the block diagrams of UEFI Redfish EDK2 Implementation. ***[EDK2 Redfish Foundation[1]](#[0])*** in the lower part of the figure refers to the EDK2 Redfish Foundation, which provides the fundamental EDK2 drivers to communicate with Redfish service ***([[19]](#[0]) in the above figure)***. The Redfish service could be implemented in BMC to manage the system, or on the network for managing multiple systems.

***[EDK2 Redfish Client[2]](#[0])*** in the upper part of the figure refers to the EDK2 Redfish client, which is the EDK2 Redfish application used to configure platform configurations by consuming the Redfish properties. The EDK2 Redfish client can also provision the UEFI platform-owned Redfish properties, consume and update Redfish properties as well. The ***[EDK2 Redfish Feature DXE Drivers [17]](https://github.com/tianocore/edk2-redfish-client/blob/main/RedfishClientPkg/Readme.md)*** is the project in tianocore/edk2-redfish-client repository. Each EDK2 Redfish Feature DXE Driver is designed to communicate with the particular Redfish data model defined in the Redfish schema *(e.g. Edk2RedfishBiosDxe driver manipulates the properties defined in Redfish BIOS data model)*.

## <a name="[0]">EDK2 Redfish Implementation Diagrams</a>
![UEFI Redfish Implementation](https://github.com/tianocore/edk2/blob/master/RedfishPkg/Documents/Media/RedfishDriverStack.svg?raw=true)

## EFI EDK2 Redfish Driver Stack
Below are the EDK2 drivers implemented on EDK2,

### EDK2 Redfish Host Interface DXE Driver ***[[6]](#[0])***

The abstract EDK2 DXE driver to create SMBIOS type 42 record through EFI SMBIOS protocol according to the device descriptor and protocol type data (defined in SMBIOS type 42h ***[[7]](#[0])***) provided by platform level Redfish host interface library. On EDK2 open source implementation (**EmulatorPkg**), SMBIOS type 42 data is retrieved from EFI variables created by RedfishPlatformConfig.efi ***[[20]](#[0])*** under EFI shell. OEM may provide its own PlatformHostInterfaceLib ***[[11]](#[0])*** instance for the platform-specific implementation.

### EDK2 Refish Credential DXE Driver ***[[5]](#[0])***

The abstract DXE driver which incorporates with RedfishPlatformCredentialLib ***[[10]](#[0])*** to acquire the credential of Redfish service. On edk2 EmulatorPkg implementation, the credential is hardcoded using the fixed Account/Password in order to connect to Redfish service established by [Redfish Profile Simulator](https://github.com/DMTF/Redfish-Profile-Simulator). OEM may provide its own RedfishPlatformCredentialLib instance for the platform-specific implementation.

### EFI REST EX UEFI Driver for Redfish service ***[[4]](#[0])***

This is the network-based driver instance of EFI_REST_EX protocol [(UEFI spec 2.8, section 29.7.2)](http://uefi.org/specifications) for communicating with Redfish service using the HTTP protocol. OEM may have its own EFI REST EX UEFI Driver instance on which the underlying transport to Redfish service could be proprietary.

### EFI Redfish Discover UEFI Driver  ***[[3]](#[0])***

EFI Redfish Discover Protocol implementation (UEFI spec 2.8, section 31.1). Only support Redfish service discovery through Redfish Host Interface. The Redfish service discovery using SSDP over UDP ***[[18]](#[0])*** is not implemented at the moment.

### EFI REST JSON Structure DXE Driver  ***[[9]](#[0])***

EFI REST JSON Structure DXE implementation (UEFI spec 2.8, section 29.7.3). This could be used by EDK2 Redfish Feature DXE Drivers ***[[17]](#[0])***. The EDK2 Redfish feature drivers manipulate platform-owned Redfish properties in C structure format and convert them into the payload in JSON format through this protocol. This driver leverages the effort of [Redfish Schema to C Generator](https://github.com/DMTF/Redfish-Schema-C-Struct-Generator) to have the "C Structure" <-> "JSON" conversion.

### EDK2 Redfish Config Handler UEFI Driver ***[[15]](#[0])***

This is the centralized manager of EDK2 Redfish feature drivers, it initiates EDK2 Redfish feature drivers by invoking init() function of EDK2 Redfish Config Handler Protocol ***[[16]](#[0])*** installed by each EDK2 Redfish feature driver. EDK2 Redfish Config Handler driver is an UEFI driver which has the dependency with EFI REST EX protocol and utilizes EFI Redfish Discover protocol to discover Redfish service that manages this system.

### EDK2 Content Coding Library ***[[12]](#[0])***
The library is incorporated with RedfishLib ***[[13]](#[0])*** to encode and decode Redfish JSON payload. This is the platform library to support HTTP Content-Encoding/Accept-Encoding headers. EumlatorPkg use the NULL instance of this library because [Redfish Profile Simulator](https://github.com/DMTF/Redfish-Profile-Simulator) supports neither HTTP Content-Encoding header on the payload returned to Redfish client nor HTTP Accept-Encoding header.

## Other Open Source Projects
  The following libraries are the wrappers of other open source projects used in RedfishPkg

  * **RedfishPkg\PrivateLibrary\RedfishLib**  ***[[13]](#[0])***
  This is the wrapper of open source project ***[libredfish](https://github.com/DMTF/libredfish)***,
  which is the library to initialize the connection to Redfish service with the proper credential and execute Create/Read/Update/Delete (CRUD) HTTP methods on Redfish properties.

  * **RedfishPkg\Library\JsonLib** ***[[14]](#[0])***
  This is the wrapper of open source project ***[Jansson](https://digip.org/jansson)***,  which is the library that provides APIs to manipulate JSON payload.

## Platform Components
### **EDK2 EmulatorPkg**
![EDK2 EmulatorPkg Figure](https://github.com/tianocore/edk2/blob/master/RedfishPkg/Documents/Media/EmualtorPlatformLibrary.svg?raw=true)

  * **RedfishPlatformCredentialLib** <br>
    The EDK2 Emulator platform implementation of acquiring credential to build up the communication between
    UEFI firmware and Redfish service. ***[[10]](#[0])***

    The Redfish credential is hardcoded in the EmulatorPkg RedfishPlatformCredentialLib. The credential is
    used to access to the Redfish service hosted by [Redfish Profile Simulator](https://github.com/DMTF/Redfish-Profile-Simulator).

  * **RedfishPlatformHostInterfaceLib**<br>
    EDK2 Emulator platform implementation which provides the information of building up SMBIOS type 42h
    record. ***[[11]](#[0])***

    EmulatorPkg RedfishPlatformHostInterfaceLib library consumes the EFI Variable which is created
    by [RedfishPlatformConfig EFI application](https://github.com/tianocore/edk2/tree/master/EmulatorPkg/Application/RedfishPlatformConfig). RedfishPlatformConfig EFI application stores not all of SMBIOS
    Type42 record information but the necessary network properties of Redfish Host Interface in EFI
    Variable.

### **Platform with BMC and the BMC-Exposed USB Network Device**
![Platform with BMC Figure](https://github.com/tianocore/edk2/blob/master/RedfishPkg/Documents/Media/BmcExposedUsbNic.svg?raw=true)

Server platform with BMC as the server management entity may expose the [USB Network Interface Device (NIC)](https://www.usb.org/document-library/class-definitions-communication-devices-12)
to the platform, which is so called the in-band host-BMC transport interface. The USB NIC exposed by BMC is
usually a link-local network device which has two network endpoints at host and BMC ends. The endpoint at
host side is connected to the platform USB port, and it is enumerated by edk2 USB BUS driver. The edk2 USB
NIC driver then produces the **EFI Network Interface Identifier Protocol**
*(EFI_NETWORK_INTERFACE_IDENTIFIER_PROTOCOL_GUID_31)* and connected by EFI Simple
Network Protocol (SNP) driver and the upper layer edk2 network drivers. The applications can then utilize
the network stack built up on top of USB NIC to communicate with Redfish service hosted by BMC.<br>
BMC-exposed USB NIC is mainly designed for the communication between host and BMC-hosted Redfish service.
BMC-exposed USB NIC can be public to host through [Redfish Host Interface Specification](https://www.dmtf.org/sites/default/files/standards/documents/DSP0270_1.3.0.pdf) and discovered by edk2 Redfish discovery
driver. The [Redfish Host Interface Specification](https://www.dmtf.org/sites/default/files/standards/documents/DSP0270_1.3.0.pdf) describes the dedicated host interface between host and
BMC. The specification follows the [SMBIOS Type 42 format](https://www.dmtf.org/sites/default/files/standards/documents/DSP0134_3.6.0.pdf) and defines the host interface as:
- Network Host Interface type (40h)
- Redfish over IP Protocol (04h)

<br>

![Platform BMC Library Figure](https://github.com/tianocore/edk2/blob/master/RedfishPkg/Documents/Media/PlatformWihtBmcLibrary.svg?raw=true)

  * **RedfishPlatformCredentialLib** <br>
RedfishPlatformCredentialLib library instance on the platform with BMC uses Redfish Credential
Bootstrapping IPMI commands defined in [Redfish Host Interface Specification](https://www.dmtf.org/sites/default/files/standards/documents/DSP0270_1.3.0.pdf) to acquire the Redfish credential from BMC. edk2
Redfish firmware then uses this credential to access to Redfish service hosted by BMC.

  * **RedfishPlatformHostInterfaceLib** <br>
BMC-exposed USB NIC, a IPMI Message channel is reported by BMC as a "IPMB-1.0" protocol and "802.3 LAN"
medium channel. edk2 firmware can issue a series of IPMI commands to acquire the channel
application information (NetFn App), transport network properties (NetFn Transport) and other necessary
information to build up the SMBIOS type 42h record. <be>
In order to recognize the specific BMC-exposed USB NIC in case the platform has more than one USB NIC
devices attached, the MAC address specified in the EFI Device Path Protocol of SNP EFI handle is used to
match with the MAC address of IPMI message channel. Due to the network information such as the MAC
address, IP address, Subnet mask and Gateway are assigned by BMC, edk2 Redfish implementation needs a
programmatic way to recognize the BMC-exposed USB NIC.

    *  **MAC address:** Searching for the BMC-exposed USB NIC<br>
    The last byte of host-end USB NIC MAC address is the last byte of BMC-end USB NIC MAC address minus 1.
    RedfishPlatformHostInterfaceLib issues the NetFn Transport IPMI command to get the MAC address of each
    channel and checks them with the MAC address specified in the EFI Device Path Protocol.<br>

          **_For example:_**<br>
          BMC-end USB NIC MAC address: 11-22-33-44-55-00<br>
          Host-end USB NIC MAC address: 11-22-33-44-55-ff

    *  **IP Address:** Acquiring the host-end USB NIC IP Address<br>
    The last byte of host-end USB NIC IPv4 address is the last byte of BMC-end USB NIC IPv4 address minus 1.

          **_For example:_**<br>
          BMC-end USB NIC IPv4 address: 165.10.0.10<br>
          Host-end USB NIC IPv4 address: 165.10.0.9

    *  **Other Network Properties**: <br>
    Due to the host-end USB NIC and BMC-end USB NIC is a link-local network. Both of them have the same
    network properties such as subnet mask and gateway. RedfishPlatformHostInterfaceLib issues the NetFn
    Transport IPMI command to get the network properties of BMC-end USB NIC and apply it on host-end USB
    NIC.

  * **IPMI Commands that Used to Build up Redfish Host Interface**

    Standard IPMI commands those are used to build up the Redfish Host Interface for BMC-exposed USB NIC.
    The USB NIC exposed by BMC must be reported as one of the message channels as 802.3 LAN/IPMB 1.0 message
    channel.

    | IPMI NetFn          | IPMI Command                              | Purpose                                                                                                               | Corresponding Host Interface Field                                                                                                          |
    | ------------------- | ----------------------------------------- | --------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
    | App<br>(0x06)       | 0x42                                      | Check the message channel's medium type and protocol.<br>Medium: 802.3 LAN<br>Protocol: IPMB 1.0                      | None                                                                                                                                        |
    | Transport<br>(0x0C) | 0x02                                      | Get MAC address of message channel. Used to match with the MAC address populated in EFI Device Path of network device | None                                                                                                                                        |
    | Group Ext<br>(0x2C) | Group Extension ID: 0x52<br>Command: 0x02 | Check if Redfish bootstrap credential is supported or not.                                                            | In Device Descriptor Data, Credential Bootstrapping Handle                                                                                  |
    | Transport<br>(0x0C) | Command: 0x02<br>Parameter: 0x04          | Get BMC-end message channel IP address source                                                                         | In Protocol Specific Record Data<br>- Host IP Assignment Type<br>- Redfish Service IP Discovery Type<br>- Generate the Host-side IP address |
    | Transport<br>(0x0C) | Command: 0x02<br>Parameter: 0x03          | Get BMC-end message channel IPv4 address                                                                              | In Protocol Specific Record Data<br>- Host IP Address Format<br>- Host IP Address                                                           |
    | Transport<br>(0x0C) | Command: 0x02<br>Parameter: 0x06          | Get BMC-end message channel IPv4 subnet mask                                                                          | In Protocol Specific Record Data<br>- Host IP Mask<br>- Redfish Service IP Mask                                                             |
    | Transport<br>(0x0C) | Command: 0x02<br>Parameter: 0x12          | Get BMC-end message channel gateway IP address                                                                        | None, used to configure edk2 network configuration                                                                                          |
    | Transport<br>(0x0C) | Command: 0x02<br>Parameter: 0x14          | Get BMC-end message channel VLAN ID                                                                                   | In Protocol Specific Record Data<br> Redfish Service VLAN ID                                                                                |

  **__NOTE__**
```
Current RedfishPlatformHostInterfaceLib implementation of BMC-exposed USB NIC can only support IPv4 address format.
```

## Miscellaneous:

   * **EFI Shell Application**
   RedfishPlatformConfig.exe is an EFI Shell application used to set up the Redfish service information for the EDK2 Emulator platform. The information such as IP address, subnet, and port.
   ```C
    For example, run shell command "RedfishPlatformConfig.efi -s 192.168.10.101 255.255.255.0 192.168.10.123 255.255.255.0", which means
      the source IP address is 192.168.10.101, and the Redfish Server IP address is 192.168.10.123.
   ```
   * **Redfish Profile Simulator**
   Refer to [Redfish Profile Simulator](https://github.com/DMTF/Redfish-Profile-Simulator) to set up the Redfish service.
   We are also in the progress to contribute bug fixes and enhancements to the mainstream Redfish Profile Simulator in order to incorporate with EDK2 Redfish solution.

## Connect to Redfish Service on EDK2 Emulator Platform
   1. Install the WinpCap and copy [SnpNt32Io.dll](https://github.com/tianocore/edk2-NetNt32Io) to the building directory of the Emulator platform. This is the emulated network interface for EDK2 Emulator Platform.
   ```C
   e.g. %WORKSPACE%/Build/EmulatorX64/DEBUG_VS2015x86/X64
   ```

   2. Enable below macros in EmulatorPkg.dsc
   ```C
  NETWORK_SNP_ENABLE = TRUE
  NETWORK_HTTP_ENABLE = TRUE
  NETWORK_IP6_ENABLE = TRUE
  SECURE_BOOT_ENABLE = TRUE
  REDFISH_ENABLE = TRUE
   ```

   3. Allow HTTP connection
   Enable below macro to allow HTTP connection on EDK2 network stack for connecting to [Redfish Profile Simulator](https://github.com/DMTF/Redfish-Profile-Simulator) becasue Redfish Profile Simulator doesn't support HTTPS.
   ```C
   NETWORK_ALLOW_HTTP_CONNECTIONS = TRUE
   ```

   4. Assign the correct MAC Address
   Assign the correct MAC address of the network interface card emulated by WinpCap.
   - Rebuild EmulatorPkg and boot to EFI shell once SnpNt32Io.dll is copied to the building directory and the macros mentioned in #2 are all set to TURE.
   - Execute the EFI shell command "ifconfig -l" under EFI shell and look for MAC address information, then assign the MAC address to below PCD.
   ```c
   gEfiRedfishPkgTokenSpaceGuid.PcdRedfishRestExServiceDevicePath.DevicePath|{DEVICE_PATH("MAC(000000000000,0x1)")}
   ```

   - Assign the network adapter instaleld on the host (working machine) that will be emulated as the network interface in edk2 Emulator.

   ```c
    #
    # For Windows based host, use a number to refer to network adapter
    #
    gEmulatorPkgTokenSpaceGuid.PcdEmuNetworkInterface|L"1"
    or
    #
    # For Linux based host, use the device name of network adapter
    #
    gEmulatorPkgTokenSpaceGuid.PcdEmuNetworkInterface|L"en0"
   ```

   5. Configure the Redfish service on the EDK2 Emulator platform

   Execute RedfishPlatformConfig.efi under EFI shell to configure the Redfish service information. The EFI variables are created for storing Redfish service information and is  consumed by RedfishPlatformHostInterfaceLib under EmulatorPkg.

## Related Materials
1. [DSP0270](https://www.dmtf.org/sites/default/files/standards/documents/DSP0270_1.3.0.pdf) - Redfish Host Interface Specification, 1.3.0
2. [DSP0266](https://www.dmtf.org/sites/default/files/standards/documents/DSP0266_1.12.0.pdf) - Redfish Specification, 1.12.0
3. Redfish Schemas - https://redfish.dmtf.org/schemas/v1/
4. SMBIOS - https://www.dmtf.org/sites/default/files/standards/documents/DSP0134_3.6.0.pdf
5. USB CDC - https://www.usb.org/document-library/class-definitions-communication-devices-12
6. UEFI Specification - http://uefi.org/specifications

## The Contributors
Thanks to the below predecessors who contributed to the UEFI EDK2 Redfish Prove of Concept code.\
Fu, Siyuan <siyuan.fu@intel.com>\
Ye, Ting <ting.ye@intel.com>\
Wang, Fan <fan.wang@intel.com>\
Wu, Jiaxin <jiaxin.wu@intel.com>\
Yao, Jiewen <jiewen.yao@intel.com>\
Shia, Cinnamon <cinnamon.shia@hpe.com>
