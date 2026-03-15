Required Changes:

**1. platform.json**  
Combining the changes proposed in the SONiC arch spec and the community HLD results in the following updates for *platfor.json*:

```json
{
    "cpos": {
        "cpo_0": {
            "lanes": {
                "0": {
                    "OE": "0",
                    "ELS": "0",
                    "laser": "0"
                },
                "1": {
                    "OE": "0",
                    "ELS": "0",
                    "laser": "0"
                },
                "31": {
                    "OE": "0",
                    "ELS": "0",
                    "laser": "7"
                }
            }
        },
        "cpo_1": {
            "lanes": {
                "0": {
                    "OE": "1",
                    "ELS": "1",
                    "laser": "0"
                },
                "1": {
                    "OE": "1",
                    "ELS": "1",
                    "laser": "0"
                },
                "31": {
                    "OE": "1",
                    "ELS": "1",
                    "laser": "7"
                }
            }
        },
        ...
        "cpo_15": {
            "lanes": {
                "0": {
                    "OE": "15",
                    "ELS": "15",
                    "laser": "0"
                },
                "1": {
                    "OE": "15",
                    "ELS": "15",
                    "laser": "0"
                },
                "31": {
                    "OE": "15",
                    "ELS": "15",
                    "laser": "7"
                }
            }
        }
    },
    "interfaces": {
        "Ethernet0": {
            "index": "1,1,1,1,1,1,1,1",
            "bank": 0,
            "lanes": "0,1,2,3,4,5,6,7",
            "breakout_modes": {
                "2x400G[200G]": ["etp1", "etp2"],
                "4x200G": ["etp1a", "etp1b", "etp2a", "etp2b"],
                "8x100G": ["etp1a", "etp1b", "etp1c", "etp1d", "etp2a", "etp2b", "etp2c", "etp2d"]
            }
        },
        "Ethernet8": {
            "index": "2,2,2,2,2,2,2,2",
            "bank": 1,
            "lanes": "8,9,10,11,12,13,14,15",
            "breakout_modes": {
                "2x400G[200G]": ["etp3", "etp4"],
                "4x200G": ["etp3a", "etp3b", "etp4a", "etp4b"],
                "8x100G": ["etp3a", "etp3b", "etp3c", "etp3d", "etp4a", "etp4b", "etp4c", "etp4d"]
            }
        },
        "Ethernet16": {
            "index": "3,3,3,3,3,3,3,3",
            "bank": 2,
            "lanes": "16,17,18,19,20,21,22,23",
            "breakout_modes": {
                "2x400G[200G]": ["etp5", "etp6"],
                "4x200G": ["etp5a", "etp5b", "etp6a", "etp6b"],
                "8x100G": ["etp5a", "etp5b", "etp5c", "etp5d", "etp6a", "etp6b", "etp6c", "etp6d"]
            }
        },
        "Ethernet24": {
            "index": "4,4,4,4,4,4,4,4",
            "bank": 3,
            "lanes": "24,25,26,27,28,29,30,31",
            "breakout_modes": {
                "2x400G[200G]": ["etp7", "etp8"],
                "4x200G": ["etp7a", "etp7b", "etp8a", "etp8b"],
                "8x100G": ["etp7a", "etp7b", "etp7c", "etp7d", "etp8a", "etp8b", "etp8c", "etp8d"]
            }
        },
        "Ethernet32": {
            "index": "5,5,5,5,5,5,5,5",
            "bank": 0,
            "lanes": "32,33,34,35,36,37,38,39",
            "breakout_modes": {
                "2x400G[200G]": ["etp9", "etp10"],
                "4x200G": ["etp9a", "etp9b", "etp10a", "etp10b"],
                "8x100G": ["etp9a", "etp9b", "etp9c", "etp9d", "etp10a", "etp10b", "etp10c", "etp10d"]
            }
        },
        "Ethernet40": {
            "index": "6,6,6,6,6,6,6,6",
            "bank": 1,
            "lanes": "40,41,42,43,44,45,46,47",
            "breakout_modes": {
                "2x400G[200G]": ["etp11", "etp12"],
                "4x200G": ["etp11a", "etp11b", "etp12a", "etp12b"],
                "8x100G": ["etp11a", "etp11b", "etp11c", "etp11d", "etp12a", "etp12b", "etp12c", "etp12d"]
            }
        },
        "Ethernet48": {
            "index": "7,7,7,7,7,7,7,7",
            "bank": 2,
            "lanes": "48,49,50,51,52,53,54,55",
            "breakout_modes": {
                "2x400G[200G]": ["etp13", "etp14"],
                "4x200G": ["etp13a", "etp13b", "etp14a", "etp14b"],
                "8x100G": ["etp13a", "etp13b", "etp13c", "etp13d", "etp14a", "etp14b", "etp14c", "etp14d"]
            }
        },
        "Ethernet56": {
            "index": "8,8,8,8,8,8,8,8",
            "bank": 3,
            "lanes": "56,57,58,59,60,61,62,63",
            "breakout_modes": {
                "2x400G[200G]": ["etp15", "etp16"],
                "4x200G": ["etp15a", "etp15b", "etp16a", "etp16b"],
                "8x100G": ["etp15a", "etp15b", "etp15c", "etp15d", "etp16a", "etp16b", "etp16c", "etp16d"]
            }
        },
        ...
        "Ethernet488": {
            "index": "62,62,62,62,62,62,62,62",
            "bank": 0,
            "lanes": "48,49,50,51,52,53,54,55",
            "breakout_modes": {
                "2x400G[200G]": ["etp13", "etp14"],
                "4x200G": ["etp13a", "etp13b", "etp14a", "etp14b"],
                "8x100G": ["etp13a", "etp13b", "etp13c", "etp13d", "etp14a", "etp14b", "etp14c", "etp14d"]
            }
        },
        "Ethernet496": {
            "index": "63,63,63,63,63,63,63,63",
            "bank": 1,
            "lanes": "48,49,50,51,52,53,54,55",
            "breakout_modes": {
                "2x400G[200G]": ["etp13", "etp14"],
                "4x200G": ["etp13a", "etp13b", "etp14a", "etp14b"],
                "8x100G": ["etp13a", "etp13b", "etp13c", "etp13d", "etp14a", "etp14b", "etp14c", "etp14d"]
            }
        },
        "Ethernet504": {
            "index": "64,64,64,64,64,64,64,64",
            "bank": 2,
            "lanes": "48,49,50,51,52,53,54,55",
            "breakout_modes": {
                "2x400G[200G]": ["etp13", "etp14"],
                "4x200G": ["etp13a", "etp13b", "etp14a", "etp14b"],
                "8x100G": ["etp13a", "etp13b", "etp13c", "etp13d", "etp14a", "etp14b", "etp14c", "etp14d"]
            }
        },
        "Ethernet512": {
            "index": "65,65,65,65,65,65,65,65",
            "bank": 3,
            "lanes": "48,49,50,51,52,53,54,55",
            "breakout_modes": {
                "2x400G[200G]": ["etp13", "etp14"],
                "4x200G": ["etp13a", "etp13b", "etp14a", "etp14b"],
                "8x100G": ["etp13a", "etp13b", "etp13c", "etp13d", "etp14a", "etp14b", "etp14c", "etp14d"]
            }
        }
    }
}
```
  
<br>
              
**2. Parsing platfom.json to create an SFP object per bank**  
To minimize changes to the CMIS state machine and maintain alignment with the community implementation, the SFP object model will be updated. Instead of creating a single SFP object per module, the system will create one SFP object per bank. The bank index will be stored as a class member of the SFP object.
As today, all SFP objects are stored in the _sfp_list. With the new approach, the list will be structured as follows:  
**_sfp_list ← [Sfp1(index=1,bank=0), Sfp2(index=2,bank=1), Sfp3(index=3,bank=2), Sfp4(index=4,bank=3), Sfp5(index=5,bank=0), Sfp6(index=6,bank=1), Sfp7(index=7,bank=2), Sfp8(index=8,bank=3) ... , Sfp61(index=61,bank=0), Sfp62(index=62,bank=1), Sfp63(index=63,bank=2), Sfp64(index=64,bank=3)]**

- Determine the number of CPO ports based on the new "cpos" section in platform.json. 
```python
    def extract_cpo_ports_index(port_type, num_of_asics=1):
        platform_file = os.path.join(device_info.get_path_to_platform_dir(), device_info.PLATFORM_JSON_FILE)
        if not os.path.exists(platform_file):
            return None
        cpo_ports = load_json_file(platform_file)['cpos']
        cpo_ports_count = len(cpo_ports.keys())
        return range(cpo_ports_count * NUMBER_OF_BANKS)
```

- Populate _sfp_list by creating a CpoPort object with the appropriate bank_index.
```python
    def initialize_sfp(self):
        sfp_count = self.get_num_sfps()
        ...
        for index in range(sfp_count):
            asic_id = self._get_asic_id_by_sfp_index(index)
            if self.RJ45_port_list and index in self.RJ45_port_list:
                sfp_object = sfp_module.RJ45Port(index, bank_index=0, asic_id=asic_id)
            elif self.cpo_port_list and index in self.cpo_port_list:
                sfp_object = sfp_module.CpoPort(sfp_index=index, bank_index=index % NUMBER_OF_BANKS, asic_id=asic_id)  # Added bank index
            else:
                sfp_object = sfp_module.SFP(sfp_index=index, bank_index=0, asic_id=asic_id)
            self._sfp_list.append(sfp_object)
        self.sfp_initialized_count = sfp_count
```
    
<br>

**3. Plug-in/Plug out event handling: chassis.get_change_event() changes:**  
Since the new design introduces one SFP object per bank (rather than per module), special handling is required for plug-in / plug-out events.

- Shared Sysfs Handling for CPO Modules: Although multiple SFP objects exist for each vModule, sysfs entries are shared across all banks of the same vModule (with the exception of EEPROM sysfs paths, which remain bank-aware). Therefore, a sysfs update affecting one SFP object is relevant to all four banks of the same vModule.  
To avoid accessing non-exist sysfs files, the system will listen to sysfs events from only one SFP object per vModule, which represents the entire vModule.  
For example, Ethernet0, Ethernet8, Ethernet16, and Ethernet24 all correspond to the same module and will therefore monitor the same sysfs entries for plug-in/plug-out events.  
Accordingly, when registering sysfs file descriptors, the system will register them once per module rather than once per SFP object (bank).


```python
    def is_cpo(self):
        return bool(self.cpo_port_list)
```

```python
    def get_sysfs_module_index(self):
        """Module index for sysfs paths. On CPO, multiple SFP objects share one module; on non-CPO, 1:1."""
        return self.sdk_index // NUMBER_OF_BANKS if is_cpo() else self.sdk_index
```

- Polling Only One SFP per Module: During event polling initialization, only one SFP object per module is registered for sysfs event monitoring.

```python
    def get_unique_sfps_by_module_index(sfp_list):
        unique_sfps = []
        seen_indexes = set()
        for sfp in sfp_list:
            module_index = sfp.get_sysfs_module_index()
            if module_index not in seen_indexes:
                seen_indexes.add(module_index)
                unique_sfps.append(sfp)
        return unique_sfps
        
  def get_change_event_for_module_host_management_mode(self, timeout):
        if not self.poll_obj:
            self.poll_obj = select.poll()
            self.registered_fds = {}
            unique_sfp_list  = get_unique_sfps_by_module_index(self._sfp_list)
            for s in unique_sfp_list :
                for fd_type, fd in s.get_fds_for_poling().items():
                    if fd is None: ...
                    self.poll_obj.register(fd, select.POLLERR | select.POLLPRI)
                    self.registered_fds[fd.fileno()] = (s.sdk_index, fd, fd_type)
        ...
```

- Updating Sysfs Access: Whenever a sysfs path references a module index, the CPO-aware module index must be used via get_sysfs_module_index() instead of self.sdk_index. Example:

```python
    def get_fd(self, fd_type):
        try:
            return open(f'/sys/module/sx_core/asic0/module{self.get_sysfs_module_index()}/{fd_type}')
        except FileNotFoundError as e:
            logger.log_warning(f'Trying to access /sys/module/sx_core/asic0/module{self.get_sysfs_module_index()}/{fd_type} file which does not exist')
            return None
```



- Propagating Events to xcvrd: To allow xcvrd to correctly process plug-in / plug-out events, the chassis must report four events per module (one per bank), rather than a single event.
This ensures that xcvrd can correctly map the event to the corresponding logical ports (get_physical_to_logical(int(key))_):

```python
    def fill_change_event(self, port_dict):
        """Fill change event data based on current state.

        Args:
            port_dict (dict): {<sfp_index>:<sfp_state>}
        """
        if self.state == STATE_NOT_PRESENT or self.state == STATE_FCP_NOT_PRESENT:
            port_dict[self.sdk_index + 1] = SFP_STATUS_REMOVED
            port_dict[self.sdk_index + 2] = SFP_STATUS_REMOVED
            port_dict[self.sdk_index + 3] = SFP_STATUS_REMOVED
            port_dict[self.sdk_index + 4] = SFP_STATUS_REMOVED
        elif self.state == STATE_SW_CONTROL or self.state == STATE_FW_CONTROL or self.state == STATE_FCP_PRESENT:
            port_dict[self.sdk_index + 1] = SFP_STATUS_INSERTED
            port_dict[self.sdk_index + 2] = SFP_STATUS_INSERTED
            port_dict[self.sdk_index + 3] = SFP_STATUS_INSERTED
            port_dict[self.sdk_index + 4] = SFP_STATUS_INSERTED
        elif self.state == STATE_POWER_BAD or self.state == STATE_POWER_LIMIT_ERROR:
            sfp_state = SFP.SFP_ERROR_BIT_POWER_BUDGET_EXCEEDED | SFP.SFP_STATUS_BIT_INSERTED
            port_dict[self.sdk_index + 1] = str(sfp_state)
            port_dict[self.sdk_index + 2] = str(sfp_state)
            port_dict[self.sdk_index + 3] = str(sfp_state)
            port_dict[self.sdk_index + 4] = str(sfp_state)
```

<br>

**4. Introduction of read_eeprom and write_eeprom with bank_index**  
The SDK will introduce bank-based EEPROM access to support pages that utilize banking (specifically pages 10h–1Fh), while maintaining strict backward compatibility with legacy access methods.

Currently, the EEPROM sysfs is accessed via the following path:  
*/sys/module/sx_core/asic0/moduleX/eeprom/pages*

To support bank selection, the SDK will introduce a new directory structure that includes the bank index (bankY):  
*/sys/module/sx_core/asic0/moduleX/bankY/eeprom/pages*

To ensure seamless backward compatibility, the legacy path will map directly to bank0. The following two paths will function identically and can be used interchangeably:

- /sys/module/sx_core/asic0/moduleX/eeprom/pages

- /sys/module/sx_core/asic0/moduleX/bank0/eeprom/pages


```python
CMIS_banked_pages = [
    0x10, 0x11, 0x12, 0x13, 0x14, 0x15, 0x16, 0x17, 
    0x18, 0x19, 0x1A, 0x1B, 0x1C, 0x1D, 0x1E, 0x1F
]

def _get_eeprom_path_for_page(self, page_num):
        if page_num in CMIS_BANKED_PAGES:
            return (SFP_SDK_MODULE_SYSFS_ROOT_TEMPLATE.format(self.get_sysfs_module_index()) +
                    'bank{}/eeprom/pages'.format(self.bank_index))
        return self._get_eeprom_path()

def _get_page_and_page_offset(self, overall_offset):
        ...
        page_num = (overall_offset - page1h_start) // SFP_UPPER_PAGE_OFFSET + 1
        page = f'{page_num}/data'
        offset = (overall_offset - page1h_start) % SFP_UPPER_PAGE_OFFSET
        
        effective_path = self._get_eeprom_path_for_page(page_num) 
        
        return page_num, os.path.join(effective_path, page), offset
```

<br>

**5. MDF changes**  
No additional MDF changes are required. CPO modules are CMIS 5.3 and are already treated as sw_control like all CMIS modules.

<br>

**6. DOM updates**  
The DOM thread periodically reads transceiver diagnostics from the hardware and writes them to Redis. These diagnostics include DOM, VDM, PM, firmware, and hardware status.  
Existing DOM statistics are keyed by logical port only; therefore, no changes to this thread are required to support the statistics currently published to the databases.  
Additional statistics for ELS monitoring will be introduced.  
***TODO: Investigate how to access the new statistics and publish them to the relevant tables. (Details TBD- Discussion on 12.3).***
    
<br>

**7. Transition to Module Type ID–Based Transceiver API Selection**  
Currently, for FW-controlled CPO, the transceiver API is hard-coded to use the CMIS API:
```
self._xcvr_api = self._xcvr_api_factory._create_api(cmis_codes.CmisCodes, cmis_mem.CmisMemMap, cmis_api.CmisApi)
```
This implementation does not rely on the module type ID stored in the EEPROM. The workaround was originally introduced because the CPO EEPROM was not accessible, and we wanted to avoid exposing a new module type to the community.

For SW-controlled CPO, this behavior will be aligned with other CMIS modules by dynamically selecting the transceiver API based on the module type ID stored in the CPO EEPROM, which is 0x80.


- Remove the get_xcvr_api() implementation from the CpoPort class. This allows CpoPort to inherit the standard SFP class implementation, which already determines the API based on the module type ID.

- Extend the known module types mapping to include the new CPO module ID (0x80).
```python
    def create_xcvr_api(self):
        id = self._get_id()

        id_mapping = {
            0x18: (self._create_cmis_api, ()),
            0x19: (self._create_cmis_api, ()),
            0x80: (self._create_cmis_api, (CmisCpoMemoryMap)),
            ...
        }
```
    
<br>    

**7. Introduce CPO to the known CMIS types**  
For a port to enter the CMIS State Machine (SM), its module type must be recognized as a valid CMIS type. Therefore, 'CPO' must be added to the list of supported module types:

```python
    CMIS_MODULE_TYPES    = ['QSFP-DD', 'QSFP_DD', 'OSFP', 'OSFP-8X', 'QSFP+C', 'CPO']
```

<br>

**8. SerDes SI and Module SI**  
**Blocked on SerDes**
The SerDes team has confirmed that CPO modules require a dedicated configuration within the TX DB. Specifically, they utilize an nLUT (a 255-entry value table), which differs from the standard settings applied to non-CPO modules.  
***TODO: Obtain the new SI (Signal Integrity) parameters for the CPO from the SerDes team. Once received, update the IM JSON files (as well as the JSON generator script itself), and update the SWSS ports OA if new SerDes SI parameters are introduced.***

<br>

**9. CpoCmisMemoryMap**  
To support the new DOM statistics, a dedicated memory map will be introduced for CPO modules: CmisCpoMemoryMap.
CmisCpoMemoryMap will inherit from the existing CmisMemoryMap and extend it to include all ELS fields defined in the architecture specification.  
***TODO: Consult with FW Arch to determine how to access each new fields.***
    
<br>

**10. CLI changes**  
- 'show commands: 
    - *show interfaces transceiver eeprom*: Update to display ELS information.
    - *show interfaces transceiver eeprom -d*: Update to display extended DOM data.
    - *show interfaces transceiver status*: Expand to output lane-specific information for 32 lanes (scaled up from the standard 8).
    - *show interfaces transceiver error-status*: Extend to report CPO-specific error conditions.

- sfputil: Several sfputil commands interface directly with the EEPROM (e.g., sfputil read-eeprom, sfputil write-eeprom, sfputil show eeprom). Because the CPO EEPROM is bank-aware, these CLI commands must be updated to support bank selection and accurately reflect bank-specific data in their outputs.  
Need to update the sfputil utility to accept bank parameters when needed, and ensure the CLI output properly formats and displays bank-level EEPROM data.
    
<br>

**11. Error handling**  
To manage CPO platform faults - such as thermal events and laser power anomalies - a 3-level Protection Schema will be implemented. The following changes are required within SONiC:
- SW-Based Protection: Monitor the 'interrupt' sysfs. Upon receiving an event, parse the relevant EEPROM error pages and report the fault via syslog and gNMI.
- FW-Based Protection: Ingest link status change notifications from SAI and report the state changes via syslog and gNMI.  

***TODO: Investigate and finalize the specific implementation details for both SW and FW protection mechanisms (e.g., exact EEPROM offsets to parse and the specific SAI notification formats).***
    
<br>

**12. FW upgrade**  
ELS FW upgrade and vModule FW Upgrade.  
***TODO: Verify sonic work is needed here (will be done with mft or sfputil? In case done with sfputil, need to investigate how to do it correctly).***
    
<br>