# Feature Name
Port Signal Integrity by Speed </br></br>
# High Level Design Document
#### Rev 0.2

</br>

# Table of Contents
  * [General Information](#general-information)
    * [Revision](#revision)
    * [About This Manual](#about-this-manual)
    * [Definitions/Abbreviations](#definitionsabbreviations)
    * [Reference](#reference)
  * [Feature Motivation](#feature-motivation)
  * [Design](#design)
    * [New SERDES parameters (data)](#new-serdes-parameters-data)
      * [Application DB Enhancements](#application-db-enhancements)
      * [The input](#the-input)
      * [How are we going to use this json?](#how-are-we-going-to-use-this-json)
    * [Port SI configuration (flows)](#port-si-configuration-flows)
  * [Changes to support the new mode](#changes-to-support-the-new-mode)
  * [Unit Test](#unit-test)

</br></br>

# General Information 

## Revision
| Rev |     Date    |       Author       | Change Description                                       |
|:---:|:-----------:|:------------------:|-----------------------------------                       |
| 0.1 | 08/28/2023  | Tomer Shalvi       | Base version                                             |

## About this Manual
This document is the high level design of Independent Module feature for SONiC Nvidia.

## Definitions/Abbreviations
| Term     |     Description                                   |
|:--------:|:-------------------------------------------------:|
| SAI      |     Switch Abstraction Interface                  |
| SONiC    |     Software for Open Networking in the Cloud     |
| CMIS     |     Common Management Interface Specification     |
| MDF      |     Module Detection Flow                         |

## Reference
| Document                                                                                                                                       |     Description                                                  |
|:----------------------------------------------------------------------------------------------------------------------------------------------:|:----------------------------------------------------------------:|
| [Independent module management - SONIC/SAI arch specification](https://confluence.nvidia.com/pages/viewpage.action?pageId=1822392761)          |  SONiC & SAI Independent Module architecture document by Eddy K. |
| [Module management architecture](https://confluence.nvidia.com/pages/viewpage.action?spaceKey=NSWARCH&title=Module+management+architecture)    |     Independent Module Management architecture by Ilya V.        |
| [cmis-init.md](https://github.com/sonic-net/SONiC/blob/master/doc/sfp-cmis/cmis-init.md)                                                       |     CMIS initialization HLD.                                     |
| [Media based port settings in SONiC](https://github.com/sonic-net/SONiC/blob/master/doc/media-settings/Media-based-Port-settings.md)           |     Media based port settings HLD.                               |

</br></br></br></br>


# Feature Motivation


Customers want to have complete flexibility and independence between switch vendors and optical cables.
However, today, if our customers want to have our switches support some module of other vendors, they have to bear long waiting times for a FW release to come out. 
So the goal of this feature is to allow them to take our switches and easily use them interchangeably with other vendors' modules, and vice versa.
Hence, a strategic decision has been made to transition the management of CMIS modules from FW to SW.


</br></br></br></br>



# Design

Currently, the configuration of SerDes SI is done entirely by FW without any involvement of SONiC. However, once we transition to independent mode, SONiC will take over the responsibility of configuring the SerDes. It will achieve this by sending the relevant port SI parameters to FW.</br>
This flow mostly existed before, but was not put into practice because FW was responsible for configuring the SerDes. In the new mode, this flow will be expanded to support the new SI parameters sent to FW and to support the ability to send them based on the relevant lane speed. 
Additionally, as part of working on this entire feature (outside the context of port SI per speed), the synchronization between the configuration of the ASIC and the configuration of the modules will be improved. In the new mode, once the ASIC configuration is complete, a signal will be sent from the FW to indicate that the module configuration can begin.
 </br></br>
 

# New SERDES parameters (data)

## Application DB Enhancements

6 new fields **ob_m2lp**, **ob_alev_out**, **obplev**, **obnlev**, **regn_bfm1p**, **regn_bfm1n** will be added to **PORT_TABLE**:

```
ob_m2lp             = 1*8HEXDIG *( "," 1*8HEXDIG) ; list of hex values, one per lane              ; ratio between the central eye to the upper and lower eyes (for PAM4 only)
ob_alev_out         = 1*8HEXDIG *( "," 1*8HEXDIG) ; list of hex values, one per lane              ; output common mode
obplev              = 1*8HEXDIG *( "," 1*8HEXDIG) ; list of hex values, one per lane              ; output buffers input to Common mode PMOS side
obnlev              = 1*8HEXDIG *( "," 1*8HEXDIG) ; list of hex values, one per lane              ; output buffers input to Common mode NMOS side
regn_bfm1p          = 1*8HEXDIG *( "," 1*8HEXDIG) ; list of hex values, one per lane              ; voltage regulator to pre output buffer PMOS side
regn_bfm1n          = 1*8HEXDIG *( "," 1*8HEXDIG) ; list of hex values, one per lane              ; voltage regulator to pre output buffer NMOS side
```

Here is the table to map the fields and SAI attributes:

| Parameter     |              sai_port_attr_t                      |
|:-------------:|:-------------------------------------------------:|
| ob_m2lp	      |     SAI_PORT_SERDES_ATTR_TX_PAM4_RATIO            |
| ob_alev_out   |     SAI_PORT_SERDES_ATTR_TX_OUT_COMMON_MODE       |
| obplev        |     SAI_PORT_SERDES_ATTR_TX_PMOS_COMMON_MODE      |
| obnlev        |     SAI_PORT_SERDES_ATTR_TX_NMOS_COMMON_MODE      |
| regn_bfm1p    |     SAI_PORT_SERDES_ATTR_TX_PMOS VLTG_REG         |
| regn_bfm1n    |     SAI_PORT_SERDES_ATTR_TX_NMOS VLTG_REG         |


## The input

To ensure that this SI values are transmitted properly to SDK, we are going to use a JSON file, called media_settings.json.</br>
The media_settings.json file will be located at '/usr/share/sonic/device/[device_type]/media_settings.json' and will follow this format:

![Alt text](<media_settings_template.png>)


Within the "PORT_MEDIA_SETTINGS" section, the SI values for each individual port are organized into four levels of hierarchy:

* The first level relates to the vendor_key/media_key level (which will be elaborated upon shortly).
* The second hierarchy level is the lane_speed_key level, which specifies the port speed and lane count.
* On the third level, we encounter the names of the SI fields.
* Finally, at the last hierarchy level, the corresponding values for these fields are presented.



## How are we going to use this json?

The flow of using of this json will be referred to as the **_Notify-Media-setting-Process_**:

![Notify Media Setting Process](Notify-Media-Setting-Process.drawio.svg)


During the **_Notify-Media-setting-Process_** two things occur:

1. STATE_DB is updated with sfp, dom and pm data.
2. The APP_DB is updated with the connected module SI parameters. This is achieved through the notify_media_settings() function, which uses the pre-parsed media_settings.json file to write its contents to APP_DB: First, a key is composed for performing the JSON lookup. Then, the lookup is performed, and the relevant data is extracted from the JSON and stored in APP_DB.

    
    - A file named sai.profile will store a new parameter, **SAI_INDEPENDENT_MODULE_MODE=False**,  indicating whether independent mode is enabled for the platform or not. In order for this functionality to be active, it is not sufficient to simply have a CMIS module plugged in; it is also necessary for the independent module feature to be supported in sai.profile.

    - Before transitioning to independent mode, SERDES SI parameters were never applied because the media_settings.json file was not found on the platform. In the new mode, media_settings.json will be present on every platform that supports the independent mode, but it will only be utilized when it contains an entry for a plugged-in CMIS module (QSFP-DD, OSFP). If it does not have a suitable entry for the connected module, nothing will be written to APP_DB to maintain backward compatibility with the legacy flow. 
    If a platform does not support independent module management, it is better to not have the media_settings.json file on the platform at all, just as it is today.


    </br>Ports OrchAgent listens to changes in APP_DB, so when the APP_DB is updated with SI parameters, PortsOrchagent is triggered. Based on the data found in APP_DB, PortsOA creates a vector (a PortConfig object) that contains the SI values for a certain port and passes it as a whole to the SAI. Eventually, they will write it to the SLTP register in PHY upon receiving ADMIN_UP.  
  </br></br></br>


## Port SI configuration (flows)

![Port SI Flows](<Port_SI_Flows.drawio.svg>) 

As described above, passing the port SI parameters is carried out in 3 scenarios:

- Initialization phase: When the switch is going up, the **_Notify-Media-setting-Process_** is carried out for each of the current interfaces. 
- Module Plug-in event
- Port speed change event: When there is a speed change (port-breakout for example), it is going to affect the speed per lane </br></br></br></br>


## Fastboot / Warmboot

*** Risk: Currently the loading of media_settings.json is skipped in case fast-reboot is enabled. There's an open issue that is under discussion with Microsoft to revert this commit: https://github.com/sonic-net/sonic-platform-daemons/pull/221. 



# Changes to support the new mode





1. Changes in SfpStateUpdateTask thread:
  - In independent mode, as mentioned before, we rely on the lane speed when we lookup in media_settings,json. Hence, this flow has to be triggered not only by insertion/removal of modules, but by configuration changes as well (port breakout for example).
      This thread is going to listen to STATE_DB.PORT_TABLE, and when there is a change in port speed, notify_media_settings() will be triggered.
  - We need the thread to have a new member variable to store the speed and number of lanes for each port: port_dict.

  ```python
  class SfpStateUpdateTask(threading.Thread):
 
   def __init__(self, namespaces, port_mapping, main_thread_stop_event, sfp_error_event):  
      self.port_dict = {}
 
   def task_worker(self):
      sel, asic_context = port_mapping.subscribe_port_update_event(self.namespaces, helper_logger)
      while not self.task_stopping_event.is_set():
         port_mapping.handle_port_update_event(sel,
                                               asic_context,
                                               self.task_stopping_event,
                                               helper_logger,
                                               self.on_port_update_event)
      ...
 
 
   def on_port_update_event(self, port_change_event):
      if port_change_event.event_type == port_change_event.PORT_SET:
         if 'speed' in port_change_event.port_dict and port_change_event.port_dict['speed'] != 'N/A':
             self.port_dict[lport]['speed'] = port_change_event.port_dict['speed']
         if 'lanes' in port_change_event.port_dict:
             self.port_dict[lport]['lanes'] = port_change_event.port_dict['lanes']
         notify_media_setting()
  ```









2. The XCVRD::Notifty_media_settings() function should be modified to support the new SI parameters and the format of media_settings_json:

   - The method get_media_settings_key() should be extended:

      We need to extend the key used for lookup in the 'media_settings.json' file to consider lane speed. Currently, there are two types of keys: 'vendor_key' (vendor name + vendor part number, for example: 'AMPHENOL-1234') and 'media_key' (media type + media_compliance_code + media length, for example: 'QSFP28-40GBASE-CR4-1M'). </br>
      In the new format of 'media_settings.json', the 'get_media_settings_key()' method will return three values instead of the two values described above. The additional value returned from this method will be the 'lane_speed_key', for example: '400GAUI-8' (where '400' refers to the port speed and '8' refers to the lane count). </br></br>

      How will the 'lane_speed_key' be calculated? </br>
      Each module contains a list of supported Application Advertisements in its EEPROM. For example:
      
      ```
      Application Advertisement: 
        400GAUI-8 C2M (Annex 120E) - Host Assign (0x1) - Active Cable assembly with BER < 2.6x10^-4 - Media Assign (0x1)
        IB EDR (Arch.Spec.Vol.2) - Host Assign (0x11) - Active Cable assembly with BER < 10^-12 - Media Assign (0x11)
        200GAUI-4 C2M (Annex 120E) - Host Assign (0x11) - Active Cable assembly with BER < 2.6x10^-4 - Media Assign (0x11)
        CAUI-4 C2M (Annex 83E) without FEC - Host Assign (0x11) - Active Cable assembly with BER < 10^-12 - Media Assign (0x11)
      ```

      
      We will use this list to derive the 'lane_speed' key. We will iterate over this list and return the item whose port speed and lane count match the values for the corresponding port. These values are stored in 'port_dict', which is a mapping between ports and their speeds and lane counts as described earlier. The existing method 'get_cmis_application_desired()' performs exactly this task, so we will use it to calculate the new key.



```python
      def get_media_settings_key(physical_port, transceiver_dict, port_speed, lane_count):
        ...
        vendor_key = vendor_name_str.upper() + '-' + vendor_pn_str
        media_key = media_type + '-' +  media_compliance_code + '-' + str(media_len)
        lane_speed_key = get_lane_speed_key()
        return [vendor_key, media_key, lane_speed_key]


      def get_ lane_speed_key (physical_port, port_speed, lane_count):
        sfp = platform_chassis.get_sfp(physical_port)
        api = sfp.get_xcvr_api()
        speed_index = get_cmis_application_desired(api, int(lane_count), int(port_speed))

        appl_dict = None
        if type(api) == CmisApi:
          appl_dict = api.get_application_advertisement()

          if speed_index is None:
            lane_speed_key= None
          else:
            lane_speed_key = "speed:" + (appl_dict[speed_index].get('host_electrical_interface_id')).split()[0]
        else:
          lane_speed_key = None
        return lane_speed_key
```
    </br>


   - The method get_media_settings_value() needs to be modified to enable lookup in both the extended format JSON and the current one:

      Once the key has been calculated, the lookup in the 'media_settings.json' file will be performed. Currently, the Vendor key (e.g. AMPHENOL-1234) is looked up first, and if there is an exact match, then those values are fetched and returned. If the vendor key doesn't match, then the media key (e.g. QSFP28-40GBASE-CR4-1M) is looked up, and if there is a match, then those values are fetched and returned.

      Upon transitioning to the Independent mode, the lookup will have to support the new 'lane_speed_key' to accommodate per-speed SI parameters. Hence, instead of just fetching the values in the case of an exact match for the vendor key or media_key, we will first check whether the JSON file supports SI parameters per speed or not. In cases where the JSON file does not provide such support, the values will be obtained from the existing hierarchy level, just as it is done currently.

      However, if the 'media_settings.json' file does support SI parameters per speed, we will proceed to search for the 'lane_speed_key' in the subsequent hierarchy level and return the corresponding values accordingly. If no relevant values are found, an empty dictionary will be returned.

      Determining whether the JSON file supports per-speed SI parameters or not will be done by searching for the presence of the string "speed:" in the relevant hierarchy level, which is the prefix of each 'lane_speed_key'. This determination is essential to ensure the compatibility of the code with vendors whose 'media_settings.json' does not include per-speed SI parameters.

 
      ```python
      def get_media_settings_value(physical_port, key):

        if PORT_MEDIA_SETTINGS_KEY in g_dict:
          for keys in g_dict[PORT_MEDIA_SETTINGS_KEY]:
              if int(keys) == physical_port:
                  media_dict = g_dict[PORT_MEDIA_SETTINGS_KEY][keys]
                  break

          if vendor_key in media_dict:
            if is_si_per_speed_supported(media_dict[vendor_key]):
                if lane_speed_key in media_dict[vendor_key]:
                    return media_dict[vendor_key][lane_speed_key]
                else:
                    return {}
            else:
                return media_dict[vendor_key]
          elif media_key in media_dict:
            if is_si_per_speed_supported(media_dict[media_key]):
                if lane_speed_key in media_dict[media_key]:
                    return media_dict[media_key][lane_speed_key]
                else:
                    return {}
            else:
                return media_dict[media_key]
          elif DEFAULT_KEY in media_dict:
              return media_dict[DEFAULT_KEY]
          elif len(default_dict) != 0:
              return default_dict

        return {}
      ```

</br>





3. Ports Orchagent Additions: To support the new SI parameters being transmitted to SAI, it is necessary to integrate them into the existing data flow between APP_DB and SAI. This process takes place within Ports Orchagent: it begins with an operation known as "doPortTask(Consumer&)," a function that is executed only upon the occurrence of specific events. In our case, the PortOrch::doTask(consumer&) function monitors changes in APP_DB and is activated only after APP_DB is updated (during the execution of notify_media_settings()). After that, Ports Orchagent iterates through all the key-value pairs written to APP_DB and uses a function called "ParsePortSerdes()" to create a PortConfig object, responsible for storing all SI values, along with other data. Then, Ports Orchagent invokes the "getPortSerdesAttr()" method, which creates a mapping between the SI values, written to the PortConfig object, to the corresponding SAI attributes. Eventually, this mapping is transmitted to the "setPortSerdesAttribute()" method, that sends it to SAI through the SAI_PORT_API.
These are the changes necessary to enable this flow to support the new SI parameters:

    
orchagent/port/porthlpr.cpp:
  ```cpp
bool PortHelper::parsePortConfig(PortConfig &port) const
{
    for (const auto &cit : port.fieldValueMap)
    {
        const auto &field = cit.first;
        const auto &value = cit.second;

        else if (field == PORT_PREEMPHASIS)
            if (!this->parsePortSerdes(port.serdes.preemphasis, field, value))
                return false;
        
		    ...


        else if (field == PORT_OB_M2LP)
            if (!this->parsePortSerdes(port.serdes.ob_m2lp, field, value))
                return false;

        else if (field == PORT_OB_ALEV_OUT)
            if (!this->parsePortSerdes(port.serdes.ob_alev_out, field, value))
                return false;

        else if (field == PORT_OBPLEV)
            if (!this->parsePortSerdes(port.serdes.obplev, field, value))
                return false;
			
        else if (field == PORT_OBNLEV)
            if (!this->parsePortSerdes(port.serdes.obnlev, field, value))
                return false;


        else if (field == PORT_REGN_BFM1P)
            if (!this->parsePortSerdes(port.serdes.regn_bfm1p, field, value))
                return false;

        else if (field == PORT_REGN_BFM1N)
            if (!this->parsePortSerdes(port.serdes.regn_bfm1n, field, value))
                return false;

		    ...


        else
            SWSS_LOG_WARN("Unknown field(%s): skipping ...", field.c_str());
    }
    return this->validatePortConfig(port);
}

template bool PortHelper::parsePortSerdes(decltype(PortSerdes_t::preemphasis) &serdes, const std::string &field, const std::string &value) const;
...
template bool PortHelper::parsePortSerdes(decltype(PortSerdes_t::ob_m2lp) &serdes, const std::string &field, const std::string &value) const;
template bool PortHelper::parsePortSerdes(decltype(PortSerdes_t::ob_alev_out) &serdes, const std::string &field, const std::string &value) const;
template bool PortHelper::parsePortSerdes(decltype(PortSerdes_t::obplev) &serdes, const std::string &field, const std::string &value) const;
template bool PortHelper::parsePortSerdes(decltype(PortSerdes_t::obnlev) &serdes, const std::string &field, const std::string &value) const;
template bool PortHelper::parsePortSerdes(decltype(PortSerdes_t::regn_bfm1p) &serdes, const std::string &field, const std::string &value) const;
template bool PortHelper::parsePortSerdes(decltype(PortSerdes_t::regn_bfm1n) &serdes, const std::string &field, const std::string &value) const;
  ```



orchagent/portsorch.cpp:
```cpp
static void getPortSerdesAttr(PortSerdesAttrMap_t &map, const PortConfig &port)
{
    if (port.serdes.preemphasis.is_set)
    {
        map[SAI_PORT_SERDES_ATTR_PREEMPHASIS] = port.serdes.preemphasis.value;
    }

	  ...
	
	
    if (port.serdes.ob_m2lp.is_set)
    {
    
        map[SAI_PORT_SERDES_ATTR_TX_PAM4_RATIO] = port.serdes.ob_m2lp.value;
    }

    if (port.serdes.ob_alev_out.is_set)
    {
        map[SAI_PORT_SERDES_ATTR_TX_OUT_COMMON_MODE] = port.serdes.ob_alev_out.value;
    }

    if (port.serdes.obplev.is_set)
    {
        map[SAI_PORT_SERDES_ATTR_TX_PMOS_COMMON_MODE] = port.serdes.obplev.value;
    }

    if (port.serdes.obnlev.is_set)
    {
        map[SAI_PORT_SERDES_ATTR_TX_NMOS_COMMON_MODE] = port.serdes.obnlev.value;
    }

    if (port.serdes.regn_bfm1p.is_set)
    {
        map[SAI_PORT_SERDES_ATTR_TX_PMOS_VLTG_REG] = port.serdes.regn_bfm1p.value;
    }

    if (port.serdes.regn_bfm1n.is_set)
    {
        map[SAI_PORT_SERDES_ATTR_TX_NMOS_VLTG_REG] = port.serdes.regn_bfm1n.value;
    }
}
  ```


orchagent/port/portschema.h:
```cpp
#define PORT_OB_M2LP             "ob_m2lp"
#define PORT_OB_ALEV_OUT         "ob_alev_out"
#define PORT_OBPLEV              "obplev"
#define PORT_OBNLEV              "obnlev"
#define PORT_REGN_BFM1P          "regn_bfm1p"
#define PORT_REGN_BFM1N          "regn_bfm1n"
  ```



orchagent/port/portcnt.h:
```cpp
class PortConfig final
{
public:
    PortConfig() = default;
    ~PortConfig() = default;

    struct {

        struct {
            std::vector<std::uint32_t> value;
            bool is_set = false;
        } preemphasis; // Port serdes pre-emphasis

		...

        struct {
            std::vector<std::uint32_t> value;
            bool is_set = false;
        } ob_m2lp; // Port serdes ob_m2lp

        struct {
            std::vector<std::uint32_t> value;
            bool is_set = false;
        } ob_alev_out; // Port serdes ob_alev_out

        struct {
            std::vector<std::uint32_t> value;
            bool is_set = false;
        } obplev; // Port serdes obplev

        struct {
            std::vector<std::uint32_t> value;
            bool is_set = false;
        } obnlev; // Port serdes obnlev

        struct {
            std::vector<std::uint32_t> value;
            bool is_set = false;
        } regn_bfm1p; // Port serdes regn_bfm1p

        struct {
            std::vector<std::uint32_t> value;
            bool is_set = false;
        } regn_bfm1n; // Port serdes regn_bfm1n


    } serdes; // Port serdes
}
  ```


# Unit Test
- Generation of keys in the new format: Expand the test_get_media_settings_key(self) method to create a dictionary that contains a mapping between a port and its port speed and lane count. Then call get_media_settings_key() with that dictionary and assert that the lane speed was concatenated properly to the key in the current format and composed a valid key in the new format.
- Lookup works with both new format and legacy format keys: Create a new test, test_get_media_settings_value(), that gets a key in the new format and looks it up in two different instances of media_settings.json. If the key cannot be found in the new format, perform a second lookup without the lane_speed. The first lookup will be performed in a JSON file that contains the key in the new format, ensuring a match. The second lookup should be performed on a JSON file that only has data for the trimmed version of the key. There should still be a match, as the functionality of get_media_settings_value() trims the lane_speed from the key if there is no match when using the full key. This test ensures that the new format does not cause any issues for other vendors.
- PortsOrchagent tests:
Verify the SAI object is created properly with the new SI parameters: Create an instance of media_settings.json that contains all new SI parameters for a certain module, call to notify_media_setting() and ensure PortOrchagent creates SAI object that contains all new parameters (reference: https://github.com/sonic-net/sonic-swss/blob/ffce92658da01e8a8f613dc1d89f9b439509d1be/tests/mock_tests/portsorch_ut.cpp#L68C5-L68C50)
