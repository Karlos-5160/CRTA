To see the services running on your computer using the Command Prompt (CMD), you can use a few different commands depending on how much detail you need.
## Method 1: The Quickest Way (Simple List)
The fastest way to see a simple text list of only the services currently running is the net start command: [1, 2] 

   1. Open the Start menu, type cmd, and press Enter.
   2. Type the following command and press Enter:
   
   net start
   
   
This will quickly output a clean list of the display names of all active services. [1] 
## Method 2: The Detailed Way (Service Control)
If you want to see more technical information (like the service's short name, type, and exit codes), use the sc query command. [3] 

   1. Open the Start menu, type cmd, right-click Command Prompt, and select Run as administrator. [4] 
   2. To see only active/running services, type:
   
   sc query type= service
   
   3. To see all services (both running and stopped), type: [3, 4] 
   
   sc query state= all
   
   
(Note: The space after the = sign in type= and state= is mandatory for the command to work properly).
## Method 3: View the Graphical Interface via CMD
If you prefer looking at the standard Windows Services manager window but want to launch it using the command prompt, type the following and hit Enter: [5, 6, 7] 

services.msc

If you are open to using PowerShell instead of CMD, you can get a cleaner, filterable table using Get-Service | Where-Object {$_.Status -eq "Running"}. [8, 9] 
If you'd like, let me know:

* If you need to start or stop a specific service using CMD. [2] 
* If you are looking for a specific service name and want to filter the results. [5] 
* Whether you want to output this list to a text file to save it.


[1] [https://4it.com.au](https://4it.com.au/kb/article/how-to-list-services-running-using-the-command-prompt/)
[2] [https://www.instagram.com](https://www.instagram.com/reel/DaijxGGMl-F/)
[3] [https://www.youtube.com](https://www.youtube.com/watch?v=lLdsbOP9HUs)
[4] [https://m.youtube.com](https://m.youtube.com/watch?v=3ryGvBWXm2g)
[5] [https://www.ryadel.com](https://www.ryadel.com/en/windows-list-services-cmd-command-prompt-powershell-system/)
[6] [https://www.youtube.com](https://www.youtube.com/watch?v=5z8laTRRvVw&t=7)
[7] [https://www.wikihow.com](https://www.wikihow.com/Open-Windows-Services)
[8] [https://learn.microsoft.com](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.management/get-service?view=powershell-7.6)
[9] [https://www.youtube.com](https://www.youtube.com/shorts/AfN0eo6Or5c)
