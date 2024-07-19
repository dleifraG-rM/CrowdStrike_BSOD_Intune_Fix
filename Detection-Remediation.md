### Crowdstrike BSOD Workaround: Safe Mode & Intune Remediation scripts

As long as you can instruct your users on how to enter safe mode with networking, you can fix the Crowdstrike BSOD issue remotely. Here’s how:

1. **Add The Below Detection Script To Intune**
   
    - Go to devices -> Scripts and Remediations -> Create
    - Right-click on the appropriate Organizational Unit (OU) and select `Create a GPO in this domain, and Link it here...`.
    - Save the below as a .ps1 file named "Detection.ps1" and upload the detection script under "Detection script file"
    
```powershell
    
#CS_Intune_Detection.ps1
$filePath = "C:\Windows\System32\drivers\CrowdStrike\C-00000291*.sys" 
$dateToCompare = Get-Date "07/19/2024 5:27"
$files = Get-ChildItem -Path $filePath -Recurse -Force

if ($files) {
    foreach ($file in $files) {

        if ($file.LastWriteTimeUtc -lt $dateToCompare) {
            Write-Output "BAD FILE FOUND. C-00000291*.sys found."
            exit 1 
            }
            
            else {
            Write-Output "GOOD FILE FOUND. C-00000291*.sys found."
            exit 0
        }
    }
} else {
    Write-Output "C-00000291*.sys not found."
    exit 0
}
    
```
    
2. **Add The Below Remediation Script To Intune**
   
    - Save the below as a .ps1 file named "Detection.ps1" and upload the detection script under "Detection script file"
```powershell
    
#CS_Intune_Remediation.ps1
$filePath = "C:\Windows\System32\drivers\CrowdStrike\C-00000291*.sys" 
$files = Get-ChildItem -Path $filePath -Recurse -Force

if ($files) {
    foreach ($file in $files) {

        if ($file.LastWriteTime -lt $dateToCompare) {
            Remove-Item $file -Force
            Restart-Computer
            exit 0 
            }
    }
} 
exit 0
    
```
3. **Ensure The Below Settings Are Selected Before Continuing**

![image](https://github.com/user-attachments/assets/902fee14-88a0-4728-badf-2c441bbafef9)

4. **Add The Group Of Affected Devices To The Policy, Set To Run Hourly**
   
![image](https://github.com/user-attachments/assets/11638a08-201b-41ae-92fa-85bcf0eabeeb)
    
6. **Deploy The Script**
   
    - After the script is deployed, instruct the users on how to boot into safe mode with networking.
    - Once there, all they have to do is wait for the machine to get the script from intune.
    - They will know their machine has recieved the script once it restarts.
    - Congratulations!
