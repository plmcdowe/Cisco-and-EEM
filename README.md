# Embedded Event Manager \(EEM\)

I hope that this repository saves you lot's of time.    
Because, I found EEM documentation to be a special kind of awful.    
Hand's down, the best resource is piecing things together from Cisco Community Forum posts.    
I will provide links at the end to the resources that were most beneficial to me on this journey.   

:warning:<ins>Caveats & Warnings</ins>:warning:     
> EEM's and AAA can be made to work together, but your `aaa new-model` is likely *not* configured for it.     
>> The guaranteed method is to configure your applets with `authorization bypass`.     
> If you are using [IOS.sh](https://github.com/plmcdowe/Cisco-and-Bash):    
>> ***:exclamation: Do not attempt to run a function from within an EEM applet. :exclamation:***     
>>> There's methods for triggering shell functions from EEM detectors, but ***`action cli` is not it.***    
>>>
>> EEM built-in variables, such as `$_cli_username`, must have the *$* escaped: `\$_cli_username`.    
> If your device's IOS version is >= 17, then your EEM version should be the latest (4.0).
>> You can confirm with: `sh event manager version`    


<ins>Additional notes before examples</ins>:     
EEM ranges from version 1.0 - 4.0, with sub-releases.    
The specifics of each release are too much to cover now, but can be read in entirety [here](https://www.cisco.com/c/en/us/td/docs/routers/ios/config/17-x/syst-mgmt/b-system-management/m_eem-overview.html).     
*"EEM Applet"* and *"EEM Policy"*, *"TCL.sh"* and *"IOS.sh"*, **and** *"TCL in EEM Applets"*. They're different, but related..    
> <sub>If you aren't familiar, TCL is Tool Control Language, and is deserving of its own ReadMe.</sub>    
> <sup>But, I will cover some basics later and provide references at the end.</sup>    

- **EEM Applet** `(config)#event manager applet AppletName`**:**    
   - Applets are the focus of this ReadMe.    
   - EEM 4.0 Applets use a TCL patch 8.3.4. You can confirm from CLI within the TCL shell:    
      ```
      #tclsh
      (tcl)#info patchlevel
      ```
   - *:exclamation: Using TCL in EEM 4.0 is recommended over TCL.sh scripts, which uses no longer updated TCL versions.*    
      It is possible to still use TCL.sh safely and effectively,     
      you just need to ensure that you reference sufficiently old documentation.    
   - TCL is written directly in an applet under `action` steps.    

- **EEM Policy** `(config)#event manager policy PolicyFile.tcl`**:**    
   - Unlike applets, policies reference a TCL file stored in the device.    
   - You have to register the user policy directory that contains `.tcl` policy files with event manager server.     
   - Then, configure your event manager policy much like an applet.    


