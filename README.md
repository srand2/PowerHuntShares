# PowerHuntShares
PowerHuntShares is design to automatically inventory, analyze, and report excessive privilege assigned to SMB shares on Active Directory domain joined computers.  
It is intented to help IAM and other blue teams gain a better understand of their SMB Share attack surface and provides data insights to help naturally group related share to help stream line remediation efforts at scale.

It supports functionality to:
* <strong>Authenticate</strong> using the current user context, a credential, or clear text user/password.
* <strong>Discover</strong> accessible systems associated with an Active Directory domain automatically. It will also filter Active Directory computers based on available open ports.
* <strong>Target</strong> a single computer, list of computers, or discovered Active Directory computers (default).
* <strong>Collect</strong> SMB share ACL information from target computers using PowerShell.
* <strong>Analyze</strong> collected Share ACL data.
* <strong>Report</strong> summary reports and excessive privilege details in HTML and CSV file formats.

Excessive SMB share ACLs are a systemic problem and an attack surface that all organizations struggle with.  The goal of this project is to provide a proof concept that will work towards building a better share collection and data insight engine that can help inform and priorititize remediation efforts.

# Vocabulary
PowerHuntShares will inventory SMB share ACLs configured with "excessive privileges" and highlight "high risk" ACLs.  Below is how those are defined in this context.

<strong>Excessive Privileges</strong><br>
Excessive read and write share permissions have been defined as any network share ACL containing an explicit ACE (Access Control Entry) for the "Everyone", "Authenticated Users", "BUILTIN\Users", "Domain Users", or "Domain Computers" groups. All provide domain users access to the affected shares due to privilege inheritance issues.  Note there is a parameter that allow operators to add their own target groups.<br>
Below is some additional background:<br>
* Everyone is a direct reference that applies to both unauthenticated and authenticated users.  Typically only a null session is required to access those resources.
* BUILTIN\Users contains Authenticated Users
* Authenticated Users contains Domain Users on domain joined systems. That's why Domain Users can access a share when the share permissions have been assigned to "BUILTIN\Users".
* Domain Users is a direct reference
* Domain Users can also create up to 10 computer accounts by default that get placed in the Domain Computers group
* Domain Users that have local administrative access to a domain joined computer can also impersonate the computer account. 

Please Note: Share permissions can be overruled by NTFS permissions. Also, be aware that testing excluded share names containing the following keywords: "print$", "prnproc$", "printer", "netlogon",and "sysvol".

<strong>High Risk Shares</strong><br>
In the context of this report, high risk shares have been defined as shares that provide unauthorized remote access to a system or application. By default, that includes wwwroot, inetpub, c$, and admin$ shares. However, additional exposures may exist that are not called out beyond that.

# Example Commands
Important Note: All commands should be run as an unprivileged domain user.
<pre>
.EXAMPLE 1: Run from a domain computer. Performs Active Directory computer discovery by default.
PS C:\temp\test> Invoke-HuntSMBShares -Threads 100 -OutputDirectory c:\temp\test 

.EXAMPLE 2: Run from a domain computer with alternative domain credentials. Performs Active Directory computer discovery by default.
PS C:\temp\test> Invoke-HuntSMBShares -Threads 100 -OutputDirectory c:\temp\test -Credentials domain\user

.EXAMPLE 3: Run from a domain computer as current user. Target hosts in a file. One per line.
PS C:\temp\test> Invoke-HuntSMBShares -Threads 100 -OutputDirectory c:\temp\test  -HostList c:\temp\hosts.txt      

.EXAMPLE 4: Run from a non-domain computer with credential. Performs Active Directory computer discovery by default.
C:\temp\test> runas /netonly /user:domain\user PowerShell.exe
PS C:\temp\test> Import-Module Invoke-HuntSMBShares.ps1
PS C:\temp\test> Invoke-HuntSMBShares -Threads 100 -RunSpaceTimeOut 10 -OutputDirectory c:\folder\ -DomainController 10.1.1.1 -Credential domain\user 

===============================================================
INVOKE-HUNTSMBSHARES 
===============================================================
 This function automates the following tasks:     

 o Determine current computer's domain
 o Enumerate domain computers        
 o Filter for computers that respond to ping reqeusts          
 o Filter for computers that have TCP 445 open and accessible  
 o Enumerate SMB shares 
 o Enumerate SMB share permissions   
 o Identify shares with potentially excessive privielges       
 o Identify shares that provide reads & write access           
 o Identify shares thare are high risk
 o Identify common share owners, names, & directory listings   
 o Generate creation, last written, & last accessed timelines
 o Generate html summary report and detailed csv files         

 Note: This can take hours to run in large environments.       
---------------------------------------------------------------
|||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
---------------------------------------------------------------
SHARE DISCOVERY      
---------------------------------------------------------------
[*][03/01/2021 09:35] Scan Start
[*][03/01/2021 09:35] Output Directory: c:\temp\smbshares\SmbShareHunt-03012021093504
[*][03/01/2021 09:35] Successful connection to domain controller: dc1.demo.local
[*][03/01/2021 09:35] Performing LDAP query for computers associated with the demo.local domain
[*][03/01/2021 09:35] - 245 computers found
[*][03/01/2021 09:35] Pinging 245 computers
[*][03/01/2021 09:35] - 55 computers responded to ping requests.
[*][03/01/2021 09:35] Checking if TCP Port 445 is open on 55 computers
[*][03/01/2021 09:36] - 49 computers have TCP port 445 open.
[*][03/01/2021 09:36] Getting a list of SMB shares from 49 computers
[*][03/01/2021 09:36] - 217 SMB shares were found.
[*][03/01/2021 09:36] Getting share permissions from 217 SMB shares
[*][03/01/2021 09:37] - 374 share permissions were enumerated.
[*][03/01/2021 09:37] Getting directory listings from 33 SMB shares
[*][03/01/2021 09:37] - Targeting up to 3 nested directory levels
[*][03/01/2021 09:37] - 563 files and folders were enumerated.
[*][03/01/2021 09:37] Identifying potentially excessive share permissions
[*][03/01/2021 09:37] - 33 potentially excessive privileges were found across 12 systems..
[*][03/01/2021 09:37] Scan Complete
---------------------------------------------------------------
SHARE ANALYSIS      
---------------------------------------------------------------
[*][03/01/2021 09:37] Analysis Start
[*][03/01/2021 09:37] - 14 shares can be read across 12 systems.
[*][03/01/2021 09:37] - 1 shares can be written to across 1 systems.
[*][03/01/2021 09:37] - 46 shares are considered non-default across 32 systems.
[*][03/01/2021 09:37] - 0 shares are considered high risk across 0 systems
[*][03/01/2021 09:37] - Identified top 5 owners of excessive shares.
[*][03/01/2021 09:37] - Identified top 5 share groups.
[*][03/01/2021 09:37] - Identified top 5 share names.
[*][03/01/2021 09:37] - Identified shares created in last 90 days.
[*][03/01/2021 09:37] - Identified shares accessed in last 90 days.
[*][03/01/2021 09:37] - Identified shares modified in last 90 days.
[*][03/01/2021 09:37] Analysis Complete
---------------------------------------------------------------
SHARE REPORT SUMMARY      
---------------------------------------------------------------
[*][03/01/2021 09:37] Domain: demo.local
[*][03/01/2021 09:37] Start time: 03/01/2021 09:35:04
[*][03/01/2021 09:37] End time: 03/01/2021 09:37:27
[*][03/01/2021 09:37] Run time: 00:02:23.2759086
[*][03/01/2021 09:37] 
[*][03/01/2021 09:37] COMPUTER SUMMARY
[*][03/01/2021 09:37] - 245 domain computers found.
[*][03/01/2021 09:37] - 55 (22.45%) domain computers responded to ping.
[*][03/01/2021 09:37] - 49 (20.00%) domain computers had TCP port 445 accessible.
[*][03/01/2021 09:37] - 32 (13.06%) domain computers had shares that were non-default.
[*][03/01/2021 09:37] - 12 (4.90%) domain computers had shares with potentially excessive privileges.
[*][03/01/2021 09:37] - 12 (4.90%) domain computers had shares that allowed READ access.
[*][03/01/2021 09:37] - 1 (0.41%) domain computers had shares that allowed WRITE access.
[*][03/01/2021 09:37] - 0 (0.00%) domain computers had shares that are HIGH RISK.
[*][03/01/2021 09:37] 
[*][03/01/2021 09:37] SHARE SUMMARY
[*][03/01/2021 09:37] - 217 shares were found. We expect a minimum of 98 shares
[*][03/01/2021 09:37]   because 49 systems had open ports and there are typically two default shares.
[*][03/01/2021 09:37] - 46 (21.20%) shares across 32 systems were non-default.
[*][03/01/2021 09:37] - 14 (6.45%) shares across 12 systems are configured with 33 potentially excessive ACLs.
[*][03/01/2021 09:37] - 14 (6.45%) shares across 12 systems allowed READ access.
[*][03/01/2021 09:37] - 1 (0.46%) shares across 1 systems allowed WRITE access.
[*][03/01/2021 09:37] - 0 (0.00%) shares across 0 systems are considered HIGH RISK.
[*][03/01/2021 09:37] 
[*][03/01/2021 09:37] SHARE ACL SUMMARY
[*][03/01/2021 09:37] - 374 ACLs were found.
[*][03/01/2021 09:37] - 374 (100.00%) ACLs were associated with non-default shares.
[*][03/01/2021 09:37] - 33 (8.82%) ACLs were found to be potentially excessive.
[*][03/01/2021 09:37] - 32 (8.56%) ACLs were found that allowed READ access.
[*][03/01/2021 09:37] - 1 (0.27%) ACLs were found that allowed WRITE access.
[*][03/01/2021 09:37] - 0 (0.00%) ACLs were found that are associated with HIGH RISK share names.
[*][03/01/2021 09:37] 
[*][03/01/2021 09:37] - The 5 most common share names are:
[*][03/01/2021 09:37] - 9 of 14 (64.29%) discovered shares are associated with the top 5 share names.
[*][03/01/2021 09:37]   - 4 backup
[*][03/01/2021 09:37]   - 2 ssms
[*][03/01/2021 09:37]   - 1 test2
[*][03/01/2021 09:37]   - 1 test1
[*][03/01/2021 09:37]   - 1 users
[*] -----------------------------------------------
</pre>

# HTML Report Examples
![HtmlReport1](https://raw.githubusercontent.com/NetSPI/PowerHuntShares/main/Sample1.png) 

# Credits
<strong>Author</strong><Br>
Scott Sutherland (@_nullbind)<Br>
           
<strong>Open-Source Code Used</strong> <Br>
These individuals wrote open source code that was used as part of this project. A big thank you goes out them and their work!<br>
|Name|Site|
|:--------------------------------|:-----------|
|Will Schroeder (@harmj0y)|https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1
|Warren F (@pscookiemonster)|https://github.com/RamblingCookieMonster/Invoke-Parallel

<strong>License</strong><Br>
BSD 3-Clause

Primary Todo
--
**Pending Fixes/Bugs**
* Directory listings on data insight pages
* when we run as a DA, are we getting ntfs privs instead of share privs? check share write, and share acl write - they were a 1-1 on the last scan
* need defintions to provide an overview of when create lastmodified and lastaccess dates get set on shares (they seem too closely correlated to the scan date) 
* update code to avoid defender
* fix owner sid resolution
* grab system os version
 
**Features**
* Complete file type search
* Add ability to specific additional groups
* Add DontExcludePrintShares option
* Add ability to target any domain and any DC in any user context
* Add collection of computer os + charts 
* Add file content search; snaffler like
* Add an options to add more computers from a file, in case they are not domain joined.
* Add auto targeting of groups that contain a large % of the user population; over 70% (make configurable) 
* netlogon and sysvol you may get access denied when using windows 10 unless the setting below is configured. Automat a check for this, and attempt to modify if privs are at correct level. gpedit.msc, go to Computer -> Administrative Templates -> Network -> Network Provider -> Hardened UNC Paths, enable the policy and click "Show" button. Enter your server name (* for all servers) into "Value name" and enter the folowing text "RequireMutualAuthentication=0,RequireIntegrity=0,RequirePrivacy=0" wihtout quotes into the "Value" field. 
* add an interesting shares insight to the csv/html reports - interesting shares - sql, backup, password, etc 
* add download details links to all data insight pages
* fix date format on scanner summary - home page
* grab active sessions to help identify owners/users of share
* pull spns and computer description/spn account descriptions to help identify owner/business unit
 
**Questions**
* under what conditions are Creation time, "LastAccessTime" and "LastWriteTime" set? CreationTime is the time that the file was created on a disk partition;  Windows doesn't keep track of the last access times for directories since win7?;In general adding, renaming or deleting a file or folder will change both LastAccessTime and LastWriteTime.;last accessed timestamp is static unless the feature is enabled; fsutil behavior set disablelastaccess 0 (HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\FileSystem\NtfsDisableLastAccessUpdate);Registry - default disabled setting: dword:80000003;However, if you move a file to a different partition/disk on your computer, the CreationTime will be updated, but because the content hasn't changed, the LastWriteTime won't be. 
So you end up in a situation where your CreationTime is later than your LastWriteTime. 
* what does share owner mean when system, vs trustedinstaller vs administrators vs network service - what can we infer that would be meaningful
* what are some of the most common shares, can we automat profile them and highlight "known" application shars in the data insights?
* can we predict file path with enough collect data to analyze?
* are their faster ways pull and thread all this info?
* what other visualisations and analysis techniques should be built in .. searching, grouping, clusting, etc
 
 **References**
 * Get-SmbShareAccess
 * https://docs.microsoft.com/en-us/powershell/module/smbshare/get-smbshareaccess?view=windowsserver2022-ps
 * Get-ACL
 *  https://social.technet.microsoft.com/wiki/contents/articles/52329.powershell-how-to-get-acl-share-permissions-for-folder.aspx
 * https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.security/get-acl?view=powershell-7.2
 * https://docs.microsoft.com/en-us/dotnet/api/microsoft.powershell.security.activities.getacl?view=powershellsdk-1.1.0
 * TimeStamps
 * http://learningpcs.blogspot.com/2011/08/powershell-forensics-analysis-of.html
 * https://datadobi.com/blog/the-impact-of-timestamps/ 
 * https://www.forensixchange.com/posts/19_04_22_win10_ntfs_time_rules/
 * https://docs.microsoft.com/en-us/dotnet/api/system.io.filesysteminfo.lastwritetime?view=net-6.0
 * https://social.technet.microsoft.com/Forums/en-US/e90d8a90-9102-46a7-b5b0-d0a591719c23/getchilditem-differences-between-lastwritetime-and-lastaccesstime?forum=winserverpowershell
 
 **CSS Things**
<pre>
https://css-tricks.com/pure-css-horizontal-scrolling/
https://uxdesign.cc/creating-horizontal-scrolling-containers-the-right-way-css-grid-c256f64fc585
https://www.freecodecamp.org/news/create-gantt-chart-using-css-grid/
http://www.coding-dude.com/wp/html5/bar-chart-html/
https://thomaswilburn.github.io/viz-book/css-positioning.html
https://canvasjs.com/asp-net-mvc-charts/scatter-point-chart/
https://www.educative.io/edpresso/how-to-create-a-scatter-plot-using-d3
https://developers.google.com/chart/interactive/docs/gallery/scatterchart
https://codepen.io/sandeepguggu/pen/IoFqJ
https://www.cssscript.com/css-bar-scatter-plot-graphs/
https://jessekorzan.github.io/scatter-plot/
https://www.w3schools.com/ai/ai_scatter_plots.asp
https://chartscss.org/charts/area/#multiple-datasets
https://chartscss.org/charts/line/
https://chartscss.org/charts/mixed/  
https://www.w3schools.com/ai/tryit.asp?filename=tryai_plotly_scatter
https://www.web-workers.ch/index.php/2017/05/16/domain-joined-windows-10-network-access-is-denied-on-sysvol-and-netlogon/
https://github.com/SnaffCon/Snaffler 
https://shellgeek.com/powershell-how-to-get-file-creation-date/#:~:text=To%20find%20files%20creation%20date,filter%20to%20select%20file%20only.&text=In%20the%20above%20PowerShell%20Get,output%20to%20the%20second%20command
https://www.groovypost.com/howto/howto/enable-last-access-time-stamp-to-files-folder-windows-7/
http://woshub.com/manage-ntfs-permissions-powershell/
</pre>
 


  








