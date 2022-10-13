.. _payload-interface:

SBL Payload Interface
=====================

Introduction
------------
Slim Bootloader depends on payloads to perform any post platform initialization functionality including booting an OS.
Slim Bootloader passes platform information for payloads to rely on and utilize as needed. 

The design intention for the Payload Interface is to enable platform agnostic payload development. Platform agnostic payloads \
have the advantage of portability and can run on multiple platforms.

Supported Payload Image Formats
-------------------------------

Slim Bootloader Stage2 transitions to the payload and can support the following formats

* PE32 <check Pe32 or FV>
* ELF

HOBs required by Universal Payload ( rename to hobs passed by SBL? )
----------------------------------

Phase Handoff Information Table (PHIT)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~


.. code-block:: C

    typedef struct {
        EFI_HOB_GENERIC_HEADER  Header;
        UINT32                  Version;
        EFI_BOOT_MODE           BootMode;
        EFI_PHYSICAL_ADDRESS    EfiMemoryTop;
        EFI_PHYSICAL_ADDRESS    EfiMemoryBottom;
        EFI_PHYSICAL_ADDRESS    EfiFreeMemoryTop;
        EFI_PHYSICAL_ADDRESS    EfiFreeMemoryBottom;
        EFI_PHYSICAL_ADDRESS    EfiEndOfHobList;
    } EFI_HOB_HANDOFF_INFO_TABLE;

+---------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Header              | The HOB generic header. ``Header.HobType = EFI_HOB_TYPE_HANDOFF``.                                                                                                                |
+---------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Version             | The version number pertaining to the PHIT HOB definition. This value is four bytes in length to provide an 8-byte aligned entry when it is combined with the 4-byte ``BootMode``. |
+---------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| EfiMemoryTop        | The highest address location of memory that is allocated for use by the HOB producer phase. This address must be 4-KB aligned to meet page restrictions of UEFI.                  |
+---------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| EfiMemoryBottom     | The lowest address location of memory that is allocated for use by the HOB producer phase.                                                                                        |
+---------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| EfiFreeMemoryTop    | The highest address location of free memory that is currently available for use by the HOB producer phase.                                                                        |
+---------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| EfiFreeMemoryBottom | The lowest address location of free memory that is available for use by the HOB producer phase.                                                                                   |
+---------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| EfiEndOfHobList     | The end of the HOB list.                                                                                                                                                          |
+---------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

CPU HOB
~~~~~~~

.. code-block:: C

    typedef struct {
        EFI_HOB_GENERIC_HEADER Header;
        UINT8 SizeOfMemorySpace;
        UINT8 SizeOfIoSpace;
        UINT8 Reserved[6];
    } EFI_HOB_CPU;

+-------------------+-------------------------------------------------------------------------+
| Header            | The HOB generic header. ``Header.HobType = EFI_HOB_TYPE_CPU``.          |
+-------------------+-------------------------------------------------------------------------+
| SizeOfMemorySpace | Identifies the maximum physical memory addressability of the processor. |
+-------------------+-------------------------------------------------------------------------+
| SizeOfIoSpace     | Identifies the maximum physical I/O addressability of the processor.    |
+-------------------+-------------------------------------------------------------------------+
| Reserved          | This field will always be set to zero.                                  |
+-------------------+-------------------------------------------------------------------------+

Resource Descriptor HOBs
~~~~~~~~~~~~~~~~~~~~~~~~

These types of HOBs report system resources to the payload. For example, physical memory should be reported using resource type \
``EFI_RESOURCE_SYSTEM_MEMORY``, and the reserved memory used by the bootloader should be reported by resource type \
``EFI_RESOURCE_MEMORY_RESERVED``. Slim Bootloader reports both of these resources.

.. code-block:: C

    typedef struct {
        EFI_HOB_GENERIC_HEADER Header;
        EFI_GUID Owner;
        EFI_RESOURCE_TYPE ResourceType;
        EFI_RESOURCE_ATTRIBUTE_TYPE ResourceAttribute;
        EFI_PHYSICAL_ADDRESS PhysicalStart;
        UINT64 ResourceLength;
    } EFI_HOB_RESOURCE_DESCRIPTOR;

+-------------------+------------------------------------------------------------------------------------------------------------------------------------------------+
| Header            | The HOB generic header. ``Header.HobType = EFI_HOB_TYPE_RESOURCE_DESCRIPTOR``.                                                                 |
+-------------------+------------------------------------------------------------------------------------------------------------------------------------------------+
| Owner             | A GUID representing the owner of the resource. This GUID is used by HOB consumer phase components to correlate device ownership of a resource. |
+-------------------+------------------------------------------------------------------------------------------------------------------------------------------------+
| ResourceType      | The resource type enumeration as defined by ``EFI_RESOURCE_TYPE``                                                                              |
+-------------------+------------------------------------------------------------------------------------------------------------------------------------------------+
| ResourceAttribute | Resource attributes as defined by ``EFI_RESOURCE_ATTRIBUTE_TYPE``                                                                              |
+-------------------+------------------------------------------------------------------------------------------------------------------------------------------------+
| PhysicalStart     | The physical start address of the resource region.                                                                                             |
+-------------------+------------------------------------------------------------------------------------------------------------------------------------------------+
| ResourceLength    | The number of bytes of the resource region.                                                                                                    |
+-------------------+------------------------------------------------------------------------------------------------------------------------------------------------+

Memory Allocation HOB
~~~~~~~~~~~~~~~~~~~~~

This HOB should report the memory usages that exist outside of the HOB
list.

.. code-block:: C

    typedef struct {
        EFI_HOB_GENERIC_HEADER Header;
        EFI_HOB_MEMORY_ALLOCATION_HEADER AllocDescriptor;
    } EFI_HOB_MEMORY_ALLOCATION;

+-----------------+----------------------------------------------------------------------------------------+
| Header          | The HOB generic header. Header.HobType = ``EFI_HOB_TYPE_MEMORY_ALLOCATION``.           |
+-----------------+----------------------------------------------------------------------------------------+
| AllocDescriptor | An instance of the ``EFI_HOB_MEMORY_ALLOCATION_HEADER`` that describes the allocation. |
+-----------------+----------------------------------------------------------------------------------------+

Graphics Information HOBs
~~~~~~~~~~~~~~~~~~~~~~~~~

Graphics Mode and Framebuffer Information HOB
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: C

    #define EFI_PEI_GRAPHICS_INFO_HOB_GUID \
    { \
        0x39f62cce, 0x6825, 0x4669, { 0xbb, 0x56, 0x54, 0x1a, 0xba, 0x75, 0x3a, 0x07 } \
    }

    typedef struct {
        EFI_PHYSICAL_ADDRESS FrameBufferBase;
        UINT32 FrameBufferSize;
        EFI_GRAPHICS_OUTPUT_MODE_INFORMATION GraphicsMode;
    } EFI_PEI_GRAPHICS_INFO_HOB;

+-----------------+----+
| FrameBufferBase | ?? |
+-----------------+----+
| FrameBufferSize | ?? |
+-----------------+----+
| GraphicsMode    | ?? |
+-----------------+----+

Graphics Hardware Information HOB
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: C

    #define EFI_PEI_GRAPHICS_DEVICE_INFO_HOB_GUID \
    { \
    0xe5cb2ac9, 0xd35d, 0x4430, { 0x93, 0x6e, 0x1d, 0xe3, 0x32, 0x47, 0x8d,
    0xe7 } \
    }

    typedef struct {
        UINT16 VendorId;
        UINT16 DeviceId;
        UINT16 SubsystemVendorId;
        UINT16 SubsystemId;
        UINT8 RevisionId;
        UINT8 BarIndex;
    } EFI_PEI_GRAPHICS_DEVICE_INFO_HOB;

+-------------------+-------------------------------+
| VendorId          | Ignore if the value is 0xFFFF |
+-------------------+-------------------------------+
| DeviceId          | Ignore if the value is 0xFFFF |
+-------------------+-------------------------------+
| SubsystemVendorId | Ignore if the value is 0xFFFF |
+-------------------+-------------------------------+
| SubsystemId       | Ignore if the value is 0xFFFF |
+-------------------+-------------------------------+
| RevisionId        | Ignore if the value is 0xFF   |
+-------------------+-------------------------------+
| BarIndex          | Ignore if the value is 0xFF   |
+-------------------+-------------------------------+


ACPI Table
~~~~~~~~~~

.. code-block:: C

    gUniversalPayloadAcpiTableGuid = { 0x9f9a9506, 0x5597, 0x4515, { 0xba, 0xb6, 0x8b, 0xcd, 0xe7, 0x84, 0xba, 0x87 } }

    typedef struct {
        UNIVERSAL_PAYLOAD_GENERIC_HEADER Header;
        EFI_PHYSICAL_ADDRESS Rsdp;
    } UNIVERSAL_PAYLOAD_ACPI_TABLE;

+--------+---------------------------------------------------------------------------------------------------+
| Header | Header.Revision is 1                                                                              |
|        |                                                                                                   |
|        | Header.Length is 12                                                                               |
+--------+---------------------------------------------------------------------------------------------------+
| Rdsp   | Point to the ACPI RSDP table. The ACPI table need follow ACPI specification version 2.0 or above. |
+--------+---------------------------------------------------------------------------------------------------+

SMBIOS Table
~~~~~~~~~~~~

.. code-block:: C

    gUniversalPayloadSmbios3TableGuid = { 0x92b7896c, 0x3362, 0x46ce, { 0x99, 0xb3, 0x4f, 0x5e, 0x3c, 0x34, 0xeb, 0x42 } }

    gUniversalPayloadSmbiosTableGuid = { 0x590a0d26, 0x06e5, 0x4d20, { 0x8a, 0x82, 0x59, 0xea, 0x1b, 0x34, 0x98, 0x2d } }

    typedef struct {
        UNIVERSAL_PAYLOAD_GENERIC_HEADER Header;
        EFI_PHYSICAL_ADDRESS SmBiosEntryPoint;
    } UNIVERSAL_PAYLOAD_SMBIOS_TABLE;

+------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Header           | Header.Revision is 1                                                                                                                                                                               |
|                  |                                                                                                                                                                                                    |
|                  | Header.Length is 12                                                                                                                                                                                |
+------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| SmBiosEntryPoint | Points to the SMBIOS table in SMBIOS 3.0+ format if GUID is ``gUniversalPayloadSmbios3TableGuid``.Points to the SMBIOS table in SMBIOS 2.x format if GUID is ``gUniversalPayloadSmbiosTableGuid``. |
+------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+


Serial Port Information
~~~~~~~~~~~~~~~~~~~~~~~

If the debug device type and subtype are specified in DBG2, the
bootloader should pass 16550 compatible serial debug port information to
payload.

.. code-block:: C

    gUniversalPayloadSerialPortInfoGuid = {0xaa7e190d, 0xbe21, 0x4409, {0x8e, 0x67, 0xa2, 0xcd, 0xf, 0x61, 0xe1, 0x70 }}

    typedef struct {
        UNIVERSAL_PAYLOAD_GENERIC_HEADER Header;
        BOOLEAN UseMmio;
        UINT8 RegisterStride;
        UINT32 BaudRate;
        EFI_PHYSICAL_ADDRESS RegisterBase;
    } UNIVERSAL_PAYLOAD_SERIAL_PORT_INFO;

+----------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Header         | Header.Revision is 1.                                                                                                                                                        |
|                |                                                                                                                                                                              |
|                | Header.Length is 18.                                                                                                                                                         |
+----------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| UseMmio        | Indicates the 16550 serial port registers in MMIO space or in I/O space                                                                                                      |
+----------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| RegisterStride | Number of bytes between registers                                                                                                                                            |
+----------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| BaudRate       | Baudrate for the serial port. Could be 921600, 460800, 230400, 115200, 57600, 38400, 19200, 9600, 7200, 4800, 3600, 2400, 2000, 1800, 1200, 600, 300, 150, 134, 110, 75, 50. |
|                |                                                                                                                                                                              |
|                | Set this to 0 to use default baud rate of 115200.                                                                                                                            |
+----------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| RegisterBase   | Base address of 16550 serial port registers in MMIO or I/O space.                                                                                                            |
+----------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+


PCI Root Bridge HOBs
~~~~~~~~~~~~~~~~~~~~

.. code-block:: C

    gUniversalPayloadPciRootBridgeInfoGuid = { 0xec4ebacb, 0x2638, 0x416e, { 0xbe, 0x80, 0xe5, 0xfa, 0x4b, 0x51, 0x19, 0x01 }}

    typedef struct {
        UNIVERSAL_PAYLOAD_GENERIC_HEADER Header;
        BOOLEAN ResourceAssigned;
        UINT8 Count;
        UNIVERSAL_PAYLOAD_PCI_ROOT_BRIDGE RootBridge[0];
    } UNIVERSAL_PAYLOAD_PCI_ROOT_BRIDGES;

    typedef struct {
        UINT32 Segment;
        UINT64 Supports;
        UINT64 Attributes;
        BOOLEAN DmaAbove4G;
        BOOLEAN NoExtendedConfigSpace;
        UINT64 AllocationAttributes;
        UNIVERSAL_PAYLOAD_PCI_ROOT_BRIDGE_APERTURE Bus;
        UNIVERSAL_PAYLOAD_PCI_ROOT_BRIDGE_APERTURE Io;
        UNIVERSAL_PAYLOAD_PCI_ROOT_BRIDGE_APERTURE Mem;
        UNIVERSAL_PAYLOAD_PCI_ROOT_BRIDGE_APERTURE MemAbove4G;
        UNIVERSAL_PAYLOAD_PCI_ROOT_BRIDGE_APERTURE PMem;
        UNIVERSAL_PAYLOAD_PCI_ROOT_BRIDGE_APERTURE PMemAbove4G;
        UINT32 HID;
        UINT32 UID;
    } UNIVERSAL_PAYLOAD_PCI_ROOT_BRIDGE;

    typedef struct {
        UINT64 Base;
        UINT64 Limit;
        UINT64 Translation;
    } UNIVERSAL_PAYLOAD_PCI_ROOT_BRIDGE_APERTURE;

+------------------+-----------------------------------------------------------------------------------+
| Header           | Header.Revision is 1.                                                             |
|                  |                                                                                   |
|                  | Header.Length is ``6 + Count * sizeof (UNIVERSAL_PAYLOAD_PCI_ROOT_BRIDGE)``.      |
+------------------+-----------------------------------------------------------------------------------+
| ResourceAssigned | Bus/IO/MMIO resources for all root bridges have been assigned when it’s ``TRUE``. |
+------------------+-----------------------------------------------------------------------------------+
| Count            | Count of root bridges. Number of elements in ``RootBridge`` array.                |
+------------------+-----------------------------------------------------------------------------------+

Details of elements inside the RootBridge Structure:

+-----------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Segment               | Segment number                                                                                                                                                                                                                          |
+-----------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Supports              | Supported attributes. Refer to EFI_PCI_ATTRIBUTE_xxx used by GetAttributes() and SetAttributes() in EFI_PCI_ROOT_BRIDGE_IO_PROTOCOL defined in PI Specification.                                                                        |
+-----------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Attributes            | Initial attributes. Refer to ``EFI_PCI_ATTRIBUTE_xxx`` used by ``GetAttributes()`` and ``SetAttributes()`` in ``EFI_PCI_ROOT_BRIDGE_IO_PROTOCOL`` defined in PI Specification.                                                          |
+-----------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| DmaAbove4G            | Root bridge supports DMA above 4GB memory when it’s ``TRUE``.                                                                                                                                                                           |
+-----------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| NoExtendedConfigSpace | Root bridge supports 256-byte configuration space only when it’s ``TRUE``. Root bridge supports 4K-byte configuration space when it’s ``FALSE``.                                                                                        |
+-----------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| AllocationAttributes  | Allocation attributes. Refer to ``EFI_PCI_HOST_BRIDGE_COMBINE_MEM_PMEM`` and ``EFI_PCI_HOST_BRIDGE_MEM64_DECODE`` used by ``GetAllocAttributes()`` in ``EFI_PCI_HOST_BRIDGE_RESOURCE_ALLOCATION_PROTOCOL`` defined in PI Specification. |
+-----------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Bus                   | Bus aperture for the root bridge.                                                                                                                                                                                                       |
+-----------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Io                    | IO aperture for the root bridge.                                                                                                                                                                                                        |
+-----------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Mem                   | MMIO aperture below 4GB for the root bridge.                                                                                                                                                                                            |
+-----------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| MemAbove4G            | MMIO aperture above 4GB for the root bridge.                                                                                                                                                                                            |
+-----------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| PMem                  | Prefetchable MMIO aperture below 4GB for the root bridge.                                                                                                                                                                               |
+-----------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| PMemAbove4G           | Prefetchable MMIO aperture above 4GB for the root bridge.                                                                                                                                                                               |
+-----------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| HID                   | PnP hardware ID of the root bridge. This value must match the corresponding HID in the ACPI name space.                                                                                                                                 |
+-----------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| UID                   | Unique ID that is required by ACPI if two devices have the same HID. This value must also match the corresponding UID/HID pair in the ACPI name space.                                                                                  |
+-----------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+


Additional HOBs
---------------

Loader Library Data
~~~~~~~~~~~~~~~~~~~

.. code-block:: C

    typedef struct {
    UINT8 Revision;
    UINT8 Reserved0[3];
    UINT16 Count;
    UINT16 Flags;
    VOID *Data;
    } LOADER_LIBRARY_DATA;

System Table Info
~~~~~~~~~~~~~~~~~

.. code-block:: C

    typedef struct {
        UINT8 Revision;
        UINT8 Reserved0[3];
        UINT64 AcpiTableBase;
        UINT32 AcpiTableSize;
        UINT64 SmbiosTableBase;
        UINT32 SmbiosTableSize;
    } SYSTEM_TABLE_INFO;

Loader Platform Data
~~~~~~~~~~~~~~~~~~~~

.. code-block:: C

    typedef struct {
        UINT8 Revision;
        UINT8 Reserved0[3];
        VOID *DebugLogBuffer;
        VOID *ConfigDataPtr;
        VOID *ContainerList;
        VOID *DmaBufferPtr;
    } LOADER_PLATFORM_DATA;


Loader Platform Data
~~~~~~~~~~~~~~~~~~~~

.. code-block:: C

    typedef struct {
        UINT8 Revision;
        UINT8 Reserved0[3];
        VOID *DebugLogBuffer;
        VOID *ConfigDataPtr;
        VOID *ContainerList;
        VOID *DmaBufferPtr;
    } LOADER_PLATFORM_DATA;

OS Boot Option List
~~~~~~~~~~~~~~~~~~~

.. code-block:: C

    typedef struct {
        UINT8 Revision;
        UINT8 ResetReason;
        UINT8 BootOptionReset:1;
        UINT8 BootToShell:1;
        UINT8 RestrictedBoot:1;
        UINT8 CurrentBoot:5;
        UINT8 OsBootOptionCount;
        OS_BOOT_OPTION OsBootOption[0];
    } OS_BOOT_OPTION_LIST;

Bootloader Services List
~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: C

    typedef struct {
        UINT8 Revision;
        UINT8 Reserved[3];
        SERVICES_LIST ServiceList;
    } BOOT_LOADER_SERVICES_LIST;

PLT Device Table
~~~~~~~~~~~~~~~~

.. code-block:: C

    typedef struct {
        UINT16 DeviceNumber;
        UINT16 Reserved;
        PLT_DEVICE Device[0];
    } PLT_DEVICE_TABLE;

Loader SMM Info
~~~~~~~~~~~~~~~

.. code-block:: C

    typedef struct {
        UINT8 Revision;
        UINT8 Flags;
        UINT8 Reserved[2];
        UINT32 SmmBase;
        UINT32 SmmSize;
        SMI_CTRL_REG SmiCtrlReg;
        SMI_STS_REG SmiStsReg;
        SMI_LOCK_REG SmiLockReg;
    } LDR_SMM_INFO;

Performance Info
~~~~~~~~~~~~~~~~

.. code-block:: C

    typedef struct {
        UINT8 Revision;
        UINT8 Reserved0[3];
        UINT16 Count;
        UINT16 Flags;
        UINT32 Frequency;
        UINT64 TimeStamp[0];
    } PERFORMANCE_INFO;

CSME Performance Info
~~~~~~~~~~~~~~~~~~~~~

.. code-block:: C

    typedef struct {
        UINT8 Revision;
        UINT8 Reserved0[3];
        UINT32 BootDataVersion;
        UINT32 BootDataLength;
        UINT32 BootPerformanceData[];
    } CSME_PERFORMANCE_INFO;

Loader Platform Info
~~~~~~~~~~~~~~~~~~~~

.. code-block:: C

    typedef struct {
        UINT8 Revision;
        UINT8 Reserved[3];
        UINT8 BootPartition;
        UINT8 BootMode;
        UINT16 PlatformId;
        UINT32 CpuCount;
        UINT16 HwState;
        UINT16 Flags;
        UINT32 LdrFeatures;
        CHAR8 SerialNumber[MAX_SERIAL_NUMBER_LENGTH];
        UINT8 TpmType;
    } LOADER_PLATFORM_INFO;

System CPU Task HOBs
~~~~~~~~~~~~~~~~~~~~

.. code-block:: C

    typedef struct {
        EFI_PHYSICAL_ADDRESS SysCpuTask;
        EFI_PHYSICAL_ADDRESS SysCpuInfo;
    } SYS_CPU_TASK_HOB;

