# Embedded Event Manager \(EEM\)

I hope that this repository saves you lots of time.    
Because, I found EEM documentation to be a special kind of awful.    
Hand's down, the best resource is piecing things together from Cisco Community Forum posts.    
I will provide links at the end to the resources that were most beneficial to me on this journey.   

:warning:<ins>Caveats & Warnings</ins>:warning:     

EEM's and AAA can be made to work together, but your `aaa new-model` is likely *not* configured for it.     
> The guaranteed method is to configure your applets with `authorization bypass`.     

If you are using [IOS.sh](https://github.com/plmcdowe/Cisco-and-Bash):    
> ***:exclamation: Do not attempt to run a function from within an EEM applet. :exclamation:***     
>> There's methods for triggering shell functions from EEM detectors, but ***`action cli` is not it.***    
>>  
> EEM built-in variables, such as `$_cli_username`, must have the *$* escaped: `\$_cli_username`.    
    
If your device's IOS version is >= 17, then your EEM version should be the latest (4.0).    
> You can confirm with: `sh event manager version`        
---    

<ins>Additional notes before examples</ins>:     

EEM ranges from version 1.0 - 4.0, with sub-releases.    
The specifics of each release are too much to cover now, but can be read in entirety [here](https://www.cisco.com/c/en/us/td/docs/routers/ios/config/17-x/syst-mgmt/b-system-management/m_eem-overview.html).     

Key terms involved: *"EEM Applet"* and *"EEM Policy"*, *"TCL.sh"* and *"IOS.sh"*, **and** *"TCL in EEM Applets"*.     
> <sub>If you aren't familiar, TCL is Tool Control Language, and is deserving of its own ReadMe.</sub>    
> <sup>I will cover some TCL basics later and provide references at the end.</sup>    

**EEM Applet** `(config)#event manager applet AppletName`**:**    
> Applets are the focus of this ReadMe.    
> EEM 4.0 Applets use a TCL patch 8.3.4. You can confirm from CLI within the TCL shell:    
>> ```
>> #tclsh
>> (tcl)#info patchlevel
>> ```
>>
> *:exclamation: Using TCL in EEM 4.0 is recommended over TCL.sh scripts, which uses no longer updated TCL versions.*    
>> It is possible to still use TCL.sh safely and effectively,     
>> just ensure that you reference sufficiently old documentation.    
> 
> TCL is written directly in an applet under `action` steps.    

**EEM Policy** `(config)#event manager policy PolicyFile.tcl`**:**    
> Unlike applets, policies reference a TCL file stored in the device.    
> You have to register the user policy directory that contains `.tcl` policy files with event manager server.     
> Then, configure your event manager policy much like an applet.    
---    

<ins>When building out an applet, there's a couple of really useful tools</ins>:   

> **Tool 1 `sh event manager detector <...> detailed`**   
>> I have included the output of `sh event manager detector all detailed` in the file [sh_detector_all_detailed.txt](https://github.com/plmcdowe/Cisco-and-EEM/blob/706ef6531086a391ff9773cd9d2e0275cb52cf44/sh_detector_all_detailed.txt).   
>> But, you'll likely be most interested in the output of the following detectors:   
>>> ```
>>> cli                 CLI event detector
>>> config              Config event detector
>>> env                 Environmental event detector
>>> generic             Generic event detector
>>> gold                GOLD event detector
>>> identity            Identity event detector
>>> interface           Interface event detector
>>> ioswdsysmon         Ioswdsysmon event detector
>>> ipsla               IPSLA event detector
>>> mat                 mac-address-table event detector
>>> neighbor-discovery  neighbor discovery event detector
>>> nhrp                NHRP event detector
>>> routing             Routing event detector
>>> snmp                Snmp event detector
>>> snmp-notification   Snmp notification event detector
>>> snmp-object         Snmp Object event detector
>>> syslog              Syslog event detector
>>> ```
>>>   
>> And, as a relevant reference for the first example, here's the output of `sh event manager detector neighbor-discovery detailed`:   
>>> ```
>>> No.  Name                Version   Node        Type    
>>> 1    neighbor-discovery  01.00     node0/0     RP      
>>> 
>>>         Tcl Configuration Syntax: 
>>>         ::cisco::eem::event_register_neighbor_discovery 
>>>                  [tag <tag-val>] 
>>>                  interface <interface-pattern> 
>>>                  [cdp {add | update | delete | all}] 
>>>                  [lldp {add | update | delete | all}]
>>>                  [link-event {down|goingdown|init|testing|up|reset|admindown|deleted|all}] 
>>>                  [line-event {up|down|all}] 
>>>                  [queue_priority {normal | low | high | last}] 
>>>                  [maxrun <sec.msec>]
>>>                  [ratelimit <sec.msec>]
>>>                  [nice {0 | 1}]
>>> 
>>>         Tcl event_reqinfo Array Names: 
>>>         event_id
>>>         job_id
>>>         event_type
>>>         event_type_string
>>>         event_pub_time
>>>         event_pub_sec
>>>         event_pub_msec
>>>         event_trigger_num
>>>         event_severity
>>>         COMMON VARIABLES: 
>>>         notification 
>>>         intf_linkstatus 
>>>         intf_linestatus 
>>>         local_intf_name 
>>>         short_local_intf_name 
>>>         port_id 
>>>         CDP EVENT VARIABLES: 
>>>         protocol 
>>>         proto_notif 
>>>         proto_new_entry 
>>>         cdp_entry_name 
>>>         cdp_hold_time 
>>>         cdp_mgmt_domain 
>>>         cdp_platform 
>>>         cdp_version 
>>>         cdp_capabilities_string 
>>>         cdp_capabilities_bits 
>>>         cdp_capabilities_bits_[0-31] 
>>>         LLDP EVENT VARIABLES: 
>>>         protocol 
>>>         proto_notif 
>>>         proto_new_entry 
>>>         lldp_chassis_id 
>>>         lldp_system_name 
>>>         lldp_system_description 
>>>         lldp_ttl 
>>>         lldp_port_description 
>>>         lldp_system_capabilities_string 
>>>         lldp_enabled_capabilities_string 
>>>         lldp_system_capabilities_bits 
>>>         lldp_enabled_capabilities_bits 
>>>         lldp_capabilities_bits 
>>>         lldp_capabilities_bit_[0-31] 
>>> 
>>>         Applet Configuration Syntax: 
>>>         [ no ] event [tag <tag-val>] neighbor-discovery 
>>>                  interface {<interface> | regexp <interface-pattern>} 
>>>                  [cdp {add | update | delete | all}] 
>>>                  [lldp {add | update | delete | all}] 
>>>                  [link-event {down|goingdown|init|testing|up|reset|admindown|deleted|all}] 
>>>                  [line-event {up|down|all}] 
>>>                  [maxrun <sec.msec>] 
>>>                  [ratelimit <sec.msec>] 
>>> 
>>>         Applet Built-in Environment Variables: 
>>>         $_event_id
>>>         $_job_id
>>>         $_event_type
>>>         $_event_type_string
>>>         $_event_pub_time
>>>         $_event_pub_sec
>>>         $_event_pub_msec
>>>         $_event_severity
>>>         COMMON VARIABLES: 
>>>         $_nd_notification 
>>>         $_nd_intf_linkstatus 
>>>         $_nd_intf_linestatus 
>>>         $_nd_local_intf_name 
>>>         $_nd_short_local_intf_name 
>>>         $_nd_port_id 
>>>         CDP EVENT VARIABLES: 
>>>         $_nd_protocol 
>>>         $_nd_proto_notif 
>>>         $_nd_proto_new_entry 
>>>         $_nd_cdp_entry_name 
>>>         $_nd_cdp_hold_time 
>>>         $_nd_cdp_mgmt_domain 
>>>         $_nd_cdp_platform 
>>>         $_nd_cdp_version 
>>>         $_nd_cdp_capabilities_string 
>>>         $_nd_cdp_capabilities_bits 
>>>         $_nd_cdp_capabilities_bits_[0-31] 
>>>         LLDP EVENT VARIABLES: 
>>>         $_nd_protocol 
>>>         $_nd_proto_notif 
>>>         $_nd_proto_new_entry 
>>>         $_nd_lldp_chassis_id 
>>>         $_nd_lldp_system_name 
>>>         $_nd_lldp_system_description 
>>>         $_nd_lldp_ttl 
>>>         $_nd_lldp_port_description 
>>>         $_nd_lldp_system_capabilities_string 
>>>         $_nd_lldp_enabled_capabilities_string 
>>>         $_nd_lldp_system_capabilities_bits 
>>>         $_nd_lldp_enabled_capabilities_bits 
>>>         $_nd_lldp_capabilities_bits 
>>>         $_nd_lldp_capabilities_bit_[0-31] 
>>> ```
>>    
> **Tool 2: `debug event manager [ action cli | detector <...> ]`**   
> 

## [ 2 ] **Examples**     
> ### ↘️[ 2.1 ] <ins>CDP Neighbor</ins>:    
>> **I extended the eem applet `cdp-update` from a Cisco Community post found [here](https://community.cisco.com/t5/networking-knowledge-base/automatically-set-port-descriptions/tac-p/3345176/highlight/true#M5362).**   
>> <b></b>
>> <b></b>
>> <b></b>
>> ```
>> event manager session cli username LocalEEM privilege 15
>> event manager applet cdp-update authorization bypass
>> event neighbor-discovery interface regexp .*net.*/.* cdp add
>> action 00.00 regexp "(Switch|Router)" \$_nd_cdp_capabilities_string
>> action 00.01 if \$_regexp_result eq 1
>> action 01.00  cli command "enable" 
>> action 01.01  cli command "show cdp neighbor \$_nd_local_intf_name detail | i MAC"
>> action 01.02  regexp "Peer Source MAC: ....\\.....\\.(....)" "\$_cli_result" match MAC 
>> action 01.03  regexp "(.*C9120.*|.*9115.*)" \$_nd_cdp_platform
>> action 01.04  if \$_regexp_result eq 1
>> action 01.05   regexp "^(.*)" \$_nd_cdp_entry_name match AP
>> action 01.06   cli command "config t"
>> action 01.07   cli command "do show interface \$_nd_local_intf_name | i Description"
>> action 01.08   set olddesc "<none>"
>> action 01.09   set olddesc_sub1 "<none>"
>> action 01.10   regexp "Description: (.*)" "\$_cli_result" olddesc olddesc_sub1
>> action 01.11   if "\$olddesc_sub1" eq "\$AP,\$MAC"
>> action 01.12    exit 10
>> action 01.13   end
>> action 01.14   cli command "interface  \$_nd_local_intf_name"
>> action 01.15   cli command "description \$AP,\$MAC"
>> action 01.16   cli command "do write"
>> action 01.17   exit 10
>> action 01.18  end
>> action 02.00  regexp "^(.*)\\.abc" \$_nd_cdp_entry_name match HOST
>> action 02.01  regexp "cisco (.*)" "\$_nd_cdp_platform" match MOD
>> action 02.02  string first "Ethernet" "\$_nd_port_id"
>> action 02.03  if "\$_string_result" eq 7
>> action 02.04   string replace "\$_nd_port_id" 0 14 "Gi"
>> action 02.05  elseif "\$_string_result" eq 10
>> action 02.06   string replace "\$_nd_port_id" 0 17 "Te"
>> action 02.07  elseif "\$_string_result" eq 4
>> action 02.08   string replace "\$_nd_port_id" 0 11 "Fa"
>> action 02.09  end
>> action 02.10  set INT "\$_string_result"
>> action 02.11  cli command "config t"
>> action 02.12  cli command "do show cdp neighbor \$_nd_local_intf_name detail | i ^  IP"
>> action 02.13  regexp "  IP address: 130\\.42\\.([8|9][0-9]\\.[0-9]+)" "\$_cli_result" match IP
>> action 02.14  if \$_regexp_result eq 1
>> action 02.15   cli command "do show interface \$_nd_local_intf_name | i Description"
>> action 02.16   set olddesc "<none>"
>> action 02.17   set olddesc_sub1 "<none>"
>> action 02.18   regexp "Description: (.*)" "\$_cli_result" olddesc olddesc_sub1
>> action 02.19   if "\$olddesc_sub1" eq "\$IP,\$INT,\$MOD,\$MAC"
>> action 02.20    exit 10
>> action 02.21   end
>> action 02.22   regexp "Po[1-9]+,\$INT,\$MOD,\$MAC" \$olddesc_sub1
>> action 02.23   if \$_regexp_result eq 1
>> action 02.24    exit 10
>> action 02.25   end
>> action 02.26   cli command "interface \$_nd_local_intf_name"
>> action 02.27   cli command "description \$IP,\$INT,\$MOD,\$MAC"
>> action 02.28  else
>> action 02.29   cli command "do show interface \$_nd_local_intf_name | i Description"
>> action 02.30   set olddesc "<none>"
>> action 02.31   set olddesc_sub1 "<none>"
>> action 02.32   regexp "Description: (.*)" "\$_cli_result" olddesc olddesc_sub1
>> action 02.33   if "\$olddesc_sub1" eq "\$HOST,\$INT,\$MOD,\$MAC"
>> action 02.34    exit 10
>> action 02.35   end
>> action 02.36   regexp "Po[1-9]+,\$INT,\$MOD,\$MAC" \$olddesc_sub1
>> action 02.37   if \$_regexp_result eq 1
>> action 02.38    exit 10
>> action 02.39   end 
>> action 02.40   cli command "interface \$_nd_local_intf_name"
>> action 02.41   cli command "description \$HOST,\$INT,\$MOD,\$MAC"
>> action 02.42  end
>> action 02.43 cli command "do write"
>> action 02.44 end
>> ```
