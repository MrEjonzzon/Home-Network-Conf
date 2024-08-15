# Home-Network-Conf
Confs for hosts, ssh on my home network

Should probably fix NetBox, but ip range for now:

```
192.168.1.101  pve01 <--- Old

192.168.1.102 - 192.168.1.109 VPN Stuff

192.168.1.110-119  omv stuff

192.168.1.120-129 windows stuff

192.168.1.130-139 kub stuff
```

## PiVPN
All hosts run PiVPN so they can be accessed remotely

## SMB Share

Windows 11 has some issues, this has fixed the problem:  
```
Use the local security policy approach:  (Group Policy Editor will pop up if you type that in the search box)
Use “Start->Run” and type in “gpedit.msc” in the “Run” dialog box. A “Group Policy” window will open.


Click down to “Local Computer Policy -> Computer Configuration -> Windows Settings -> Security Settings -> Local Policies -> Security Options.

     Scroll the list to find the policy “Network Security: LAN Manager authentication level”.
     Right click on this policy and choose “Properties”.
     Choose “Send NTLMv2 response only/refuse LM & NTLM”  (My original setting was: "Send LM & NTLM - use NTLMv2 session security if negotiated" )
     Click OK and confirm the setting change.
     Close the “Group Policy” window.

```
