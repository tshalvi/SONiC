# Feature Name
CPO Phase 1 - Single Asic with FW controlled module management
# High Level Design Document
#### Rev 0.2

# Table of Contents
  * [General Information](#general-information)
    * [Revision](#revision)
    * [About This Manual](#about-this-manual)
    * [Definitions/Abbreviations](#definitionsabbreviations)
    * [Reference](#reference)
  * [Feature Overview](#feature-overview)
    * [Motivation](#motivation)
    * [System Overview](#system-overview)
    * [Modules in CPO](#modules-in-cpo)
    * [Linkup Flow in CPO](#linkup-flow-in-cpo)
    * [Design Scope And Assumptions](#design-scope-and-assumptions)
  * [Changes to support the CPO platform](#changes-to-support-the-cpo-platform)
  * [Testing](#testing)

</br></br>

# General Information 

## Revision
| Rev |     Date    |       Author       | Change Description                                       |
|:---:|:-----------:|:------------------:|-----------------------------------                       |
| 0.1 | 20/05/2025  | Tomer Shalvi       | Base version                                             |

## About this Manual
This document is the high level design of Single ASIC FW controlled CPO.

## Definitions/Abbreviations
| Term     |     Description                                   |
|:--------:|:-------------------------------------------------:|
| CPO      |     Co-Packaged Optics                            |
| DSP      |     Digital Signal Processor                      |
| OE       |     Optical Engine                                |
| ELS      |     External Laser Source                         |
| cModule  |     Combined Module                               |
| SONiC    |     Software for Open Networking in the Cloud     |
| SAI      |     Switch Abstraction Interface                  |


## Reference
| Document                                                                                                                                                                   |     Description                                                  |
|:----------------------------------------------------------------------------------------------------------------------------------------------:|:--------------------------------------------------------------------------------------------:|
| [SPC5/SPC6 CPO/Multi-ASIC SONIC architecture](https://confluence.nvidia.com/pages/viewpage.action?pageId=3763441652#SPC5/SPC6CPO/MultiASICSONICarchitecture-Introduction)  |  Architecture document for CPO by Eddy K.                        |

</br></br>

# Feature Overview


## Motivation

In traditional optical transceivers, a DSP (Digital Signal Processor) converts the electrical signal from the ASIC into an optical signal and compensates for signal loss along the PCB trace. An Externally Modulated Laser provides the light carries data over the fiber.  
This architecture consumes ~30W per module and might introduce signal integrity loss due to the distance between the ASIC and the optics.  

![Alt text](<Traditional module.png>)

As module counts grow, overall power consumption becomes a major limitation, making power reduction essential.

The new CPO (Co-Packaged Optics) platform changes this by:

- Moving the DSP into the switch package (OE), placing it directly next to the ASIC.  
- Decoupling the laser into a centralized External Laser Source (ELS), shared across multiple ports.  

By placing the optics next to the ASIC, CPO eliminates the need for DSP compensation and significantly reduces power per module.  

![Alt text](<Transition to CPO.png>)
</br></br>

## System Overview

![Alt text](<system_overview.png>)

The system includes two main hardware components: Optical Engines (OEs) and External Laser Sources (ELS).  
 - OEs are responsible for converting electrical signals to optical signals and vice versa.  
 - ELS units provide continuous laser light, used by the OEs for sending data.  

There are 32 OEs located near the ASIC and 16 ELS units:  
 Each ELS supplies laser light for 8 lanes, shared equally between 2 OEs (4 lanes each).  
 Each OE uses its 4 laser sources to create 4  lanes for 4 MPO connectors, resulting in 4 x 400G ports.

** The MPOs are connected with passive cables, and the ELS is the only pluggable component in the system. 
</br></br>

## Modules In CPO

In traditional platforms, pluggable transceiver modules (e.g., QSFPs) serve as the fundamental building blocks for port and module management. However, in CPO systems, there are no physical pluggable modules — the optics are integrated directly into the switch system. This presents a challenge for existing software like SONiC, which assumes the presence of such modules for its port and module management architecture. e.g with sysfs-related module operations (reset, presence, etc).
To address this, a new conecept is interoduced - the "Combined Module", aka the cModule.  
The cModule is a 32-lanes module representation, created by the FW, to simulate the existence of modules in a CPO-based platform.
This allows SONiC and SAI to continue operating with minimal changes, preserving their existing port/module management logic.

![Alt text](<cModule_abstration.png>)

As shown in the diagram above, SONiC and SAI interact only with logical ports and cModule identifiers. The details of the underlying optical hardware — including OEs and ELSs — are hidden by the firmware and SDK.
These components are internal to the FW/SDK and are not exposed to the operating system, but the full mapping is provided below for reference:


![Alt text](<cModules.png>)
</br></br>

## Linkup Flow in CPO

![Alt text](<linkup_flow.png>)

In CPO systems, SONiC continues to manage port creation as it does today, using logical port names and lane numbers in PORT_TABLE. The underlying optical complexity is hidden by the firmware and SDK through the cModule abstraction.

The flow is as follows:
- SONiC defines a port in PORT_TABLE using a name (e.g., Ethernet0) and a list of lanes.
- Ports OA calls sai_create_port() with the corresponding lane list.
- SAI converts this list into the following triplet used by SDK: (cModule number, submodule, lane numbers within the sub-module)
- SAI sends this triplet to the SDK to create the port, using: sx_mgmt_phy_module_split_get() and sx_api_port_mapping_set()

This allows SONiC and SAI to remain unchanged, while the SDK handles all hardware-specific mapping internally.

Port breakout is fully supported for single ASIC CPO. Supported number of lanes per physical port: 4/2/1.
</br></br>

## Design Scope And Assumptions

In the first phase of this feature, the changes described below aim to support the required functionality for the CPO platform—limited to Single ASIC systems and only for ports managed by the firmware. The NOS (SONiC) will set the port's admin status to UP, and the remaining link-up sequence will be handled by the SDK and firmware.

Assumptions:
1. No changes are expected in the sysfs files provided by the SDK. The existing sysfs structure (e.g., present, reset, lpm, etc.) will continue to function as before. These files will be accessed under the same path: */sys/module/sx_core/$asic/$module/*. The only difference is that the module ID will correspond to the cModule ID.
2. The lower layers will emulate the EEPROM at the offsets required by SONiC (mostly offsets on page 0).
</br></br>

# Changes to support the CPO platform

1. New Platform and SKU for SN5810-LD:  
To support the CPO platform, a new platform folder `x86_64-nvidia_sn5810-ld-r0` and SKU folder `Mellanox-SN5810-LD-O128A2` will be created.
These will be based on the existing SN5640 platform folder and C512S2 SKU folder, with the modifications described below.

  - paltform.json: This file defines all platform components and port breakout configurations. It will be based on the Bison platform.json, with two main differences:  
&nbsp;&nbsp;&nbsp;&nbsp; a. The sfps section will only include the two service ports.  
&nbsp;&nbsp;&nbsp;&nbsp; b. The interfaces section will define the supported breakout options for all ports. Since each CPO port supports up to 4 lanes, it can be split into two-lane and single-lane ports, as shown below:


```json
],
        "sfps": [
            {
                "name": "sfp1",
                "thermals": [
                    {
                        "name": "xSFP module 1 Temp"
                    }
                ]
            },
            {
                "name": "sfp2",
                "thermals": [
                    {
                        "name": "xSFP module 2 Temp"
                    }
                ]
            }
        ]
    },
    "interfaces": {
        "Ethernet0": {
            "index": "1,1,1,1,1,1,1,1",
            "lanes": "0,1,2,3,4,5,6,7",
            "breakout_modes": {
                "2x400G[200G]": ["etp1a", "etp1b"],
                "4x200G": ["etp1a", "etp1b", "etp1c", "etp1d"],
                "8x100G": ["etp1a", "etp1b", "etp1c", "etp1d", "etp1e", "etp1f", "etp1g", "etp1h"]
            }
        },
    
    ...
```

  - hwsku.json: This file specifies the default breakout mode for each port and its port type (explained later). Each CPO port will be split into two 400G ports, resulting in 128 400G ports. The two service ports will remain as 25G:

```json
{
    "interfaces": {
        "Ethernet0": {
            "default_brkout_mode": "2x400G[200G]",
            "port_type": "CPO"
        },
        "Ethernet8": {
            "default_brkout_mode": "2x400G[200G]",
            "port_type": "CPO"
        },
        
        ...
        
        "Ethernet504": {
            "default_brkout_mode": "2x400G[200G]",
            "port_type": "CPO"
        },
        "Ethernet512": {
            "default_brkout_mode": "1x25G[10G]"
        },
        "Ethernet513": {
            "default_brkout_mode": "1x25G[10G]"
        }
    }
}
```

  - Independent Module Configutaion: Disabled in this phase, as ports are managed by firmware:
    - Omit the Independent Module flag in sai.profile.
    - Set skip_xcvrd_cmis_mgr=true in pmon_daemon_control.json.
    - Exclude SI-related JSONs.
    - Firmware will also disable independent module behavior via its INI file.


All platform-related changes are available in this commit: https://github.com/nvidia-sonic/sonic-buildimage/commit/120816ecb98b9fe02c02a77411c9fd249f58aa9f     

</br>
  

2. Introduce a new type of module to represent the CPO module: 

  - New module object for the CPO: In xcvrd code, some module APIs are called to post transeceiver info to database, check module presence, etc. The implementation of these API calls depends on the type of module being used. XcvrApiFactory.create_xcvr_api() is the function that retrieves the module type id from the module's EEPROM, and according to this id it creates a transceiver object, that is used to call the module API's. Currently, Sonic does not know how to create such object for a CPO module. This is why we need to introduce this module type to Sonic by adding its module type id to the known types list.

```python
    def create_xcvr_api(self):
        id = self._get_id()

        # Instantiate various Optics implementation based upon their respective ID as per SFF8024
        id_mapping = {
            0x03: (self._create_api, (Sff8472Codes, Sff8472MemMap, Sff8472Api)),
            0x0D: (self._create_qsfp_api, ()),
            0x11: (self._create_api, (Sff8636Codes, Sff8636MemMap, Sff8636Api)),
            0x18: (self._create_cmis_api, ()),
            0x19: (self._create_cmis_api, ()),
            0x1b: (self._create_cmis_api, ()),
            0x1e: (self._create_cmis_api, ()),
            0x7e: (self._create_api, (AmphBackplaneCodes,
                                     AmphBackplaneMemMap, AmphBackplaneImpl)),

            0x80: (self._create_cmis_api, ())   # support for the new CPO module
        }

        # Check if the ID exists in the mapping
        if id in id_mapping:
            func, args = id_mapping[id]
            if isinstance(args, tuple):
                return func(*args)
        return None
```

Since the CPO module is compatible with CMIS 5.3 and we aim to minimize changes to SONiC, we can add this module type and map it to the existing CMIS memory map. This will allow SONiC to access the required EEPROM fields seamlessly, with the lower layers emulating only the necessary fields—eliminating the need for additional code changes.

  - Update the New Module Type in XCVR_IDENTIFIERS: The XCVR_IDENTIFIERS dictionary maps module types (found at page 0, byte 0 of the module's EEPROM) to strings displayed in various CLI outputs, such as the *show interfaces status* command. To support the CPO module, this dictionary must be updated with the value emulated by the SDK at page 0, byte 0 of the EEPROM: https://github.com/sonic-net/sonic-platform-common/blob/fad0a144e1215c91e6bbfee761b40cdd1615a614/sonic_platform_base/sonic_xcvr/codes/public/sff8024.py#L10

  - The chassis.get_num_sfps() method returns the number of plugged-in modules but currently only considers SFP-type modules. It should be updated to also include CPO ports in the count. Our SKU includes only two SFPs, so without accounting for the CPO ports, we won’t be able to interact with modules whose index > 2.   
  This can be seen here: https://github.com/sonic-net/sonic-buildimage/blob/d02ec51ca00e468d16d627b06e0da8837256650a/platform/mellanox/mlnx-platform-api/sonic_platform/chassis.py#L270 (Without the suggested code change, sfp_count will always be 2, meaning only two modules can be initialized)  
  *** CPO is not an SFP by definition, so get_num_sfps() will be renamed.

  ```python
class Chassis(ChassisBase):

    def __init__(self):
      ...      
      # Build the CPO port list from platform.json and hwsku.json   # new code for CPO
      self._cpo_port_inited = False                                 # new code for CPO
      self._cpo_port_list = None                                    # new code for CPO
      ...
    
    @property                                                       # new code for CPO
    def cpo_port_list(self):                                        # new code for CPO
      if not self._cpo_port_inited:                                 # new code for CPO
        self._cpo_port_list = extract_cpo_ports_index()             # new code for CPO
        self._cpo_port_inited = True                                # new code for CPO
        return self._cpo_port_list                                  # new code for CPO

    def get_num_sfps(self):
          """
          Retrieves the number of sfps available on this chassis

          Returns:
              An integer, the number of sfps available on this chassis
          """
          num_sfps = 0
          if not self._RJ45_port_inited:
              self._RJ45_port_list = extract_RJ45_ports_index()
              self._RJ45_port_inited = True
          
          if not self._cpo_port_inited:                          # new code for CPO
              self._cpo_port_list = extract_cpo_ports_index()    # new code for CPO
              self._cpo_port_inited = True                       # new code for CPO
          
          num_sfps = DeviceDataManager.get_sfp_count()
          if self._RJ45_port_list is not None:
              num_sfps += len(self._RJ45_port_list)
          if self._cpo_port_list is not None:                   # new code for CPO
              num_sfps += len(self._cpo_port_list)              # new code for CPO
          
          return num_sfps



platform/mellanox/mlnx-platform-api/sonic_platform/utils.py:

def extract_cpo_ports_index():
    # Cross check 'platform.json' and 'hwsku.json' to extract the cpo port index if exists.
    hwsku_path = device_info.get_path_to_hwsku_dir()
    hwsku_file = os.path.join(hwsku_path, HWSKU_JSON)
    if not os.path.exists(hwsku_file):
        # Platforms having no hwsku.json do not have cpo port
        return None

    platform_file = device_info.get_path_to_port_config_file()
    platform_dict = load_json_file(platform_file)['interfaces']
    hwsku_dict = load_json_file(hwsku_file)['interfaces']
    port_name_to_index_map_dict = {}
    cpo_port_index_list = []

    # Compose a interface name to index mapping from 'platform.json'
    for i, (key, value) in enumerate(platform_dict.items()):
        if PORT_INDEX_KEY in value:
            index_raw = value[PORT_INDEX_KEY]
            # The index could be "1" or "1, 1, 1, 1"
            index = index_raw.split(',')[0]
            port_name_to_index_map_dict[key] = index

    if not bool(port_name_to_index_map_dict):
        return None

    # Check if "port_type" specified as "CPO", if yes, add the port index to the list.
    for i, (key, value) in enumerate(hwsku_dict.items()):
        if key in port_name_to_index_map_dict and PORT_TYPE_KEY in value and value[PORT_TYPE_KEY] == CPO_PORT_TYPE:
            cpo_port_index_list.append(int(port_name_to_index_map_dict[key])-1)

    return cpo_port_index_list if bool(cpo_port_index_list) else None


```


</br></br>

## Testing

Verify that all ports come up with the correct split as defined in the SKU.
Each port should:

* Be displayed with type "CPO"
* Have its transceiver information present in the TRANSCEIVER_INFO table
* Expose all required EEPROM fields, emulated as expected by the lower layers (Verification tests should align with the list of EEPROM fields the SDK emulates for CPO)
* Respond correctly to all relevant sfputil commands (Already being covered by Verification tests)