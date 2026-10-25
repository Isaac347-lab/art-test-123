Atomic Test Take 1, 2, 3...
Overview and Introduction
In this blog post I will be covering my efforts to set up a new endpoint in a Wazuh SIEM and forward its logs with Sysmon. A few different principles were reinforced upon me in the process that will be helpful in the projects yet to come.

I have been in the construction and manufacturing field for the last few years. I found Triple Ten and thought it would be a great career switch, and I would be able to rekindle my interest with computers. As I was perusing the different courses offered, I was thinking about my life experiences thus far and what aspects of different jobs I had enjoyed. In my most recent role in manufacturing I performed root cause analysis for microprocessor prosthetic knees. I really enjoy the aspect of having a different problem to solve and a chance to stretch mentally. This attracted me initially to cybersecurity, and as I researched more about what it was, other factors grasped my attention. I have really enjoyed what I have done so far in my learning regarding penetration testing. To me, security and testing echo throughout the ages of humanity in all forms; cybersecurity is the next step that ties into human history.

Setup
In this setup I already had access to several VMs, including a Wazuh server. I made two modifications to the testing environment.

I started with adding an agent to another workstation called ART Workstation. This is the endpoint that had the Atomic Red Team tests installed. I followed the simple instructions in the Wazuh SIEM under agents management to add a new endpoint. I ran the commands provided on ART Workstation.

To install Sysmon on the endpoint, I started by downloading Sysmon from Microsoft. I then proceeded by looking up a basic Sysmon configuration XML file. I downloaded the Neo23x0 config file from GitHub. Next I extracted the zip file from the Sysmon download. Then I installed Sysmon via PowerShell using this command: { sysmon64.exe -accepteula -i sysmonconfig-export.xml }. I then verified the installation with Event Viewer.

Though relatively simple, I made a significant mistake in this setup. I forgot to restart the Wazuh service after installing Sysmon. This mistake was not caught during this process and therefore caused quite a bit of confusion when it came time to run the experiments.

Expirements
Atomic Test T1547.001
The purpose of this experiment was to test the log recording for an attack involving persistence. I installed the prerequisites for Atomic Test T1547.001 and then proceeded to run the actual test without much thought. I received an error when I attempted the test, however, and had received the same error with the request to get the prerequisites. When I researched the error, I found that I had forgotten to locate ART before running the tests. I ran { Import-Module "C:\AtomicRedTeam\invoke-atomicredteam\Invoke-AtomicRedTeam.psd1" -Force } to locate the tests. After I had done this, the test successfully ran. Upon examination of the logs, I saw some of the indicators I was expecting. I had, however, forgotten to filter for only the Sysmon logs to test the modification I had made to the environment. After completing the second experiment, I realized this and ran it again with the log forwarding in place to ensure Sysmon logs were being recorded in the SIEM when I ran T1547.001.

Atomic Test T1046
This experiment was to test the Sysmon logs for network connection activity. Similarly to the last test, I started by getting the prerequisites installed. I then ran the Atomic Test T1046. The test ran successfully. When I went to check the logs in Wazuh, I expected to see many event ID 3 Sysmon logs. There were no such logs. I researched and found that potentially Sysmon would not log event ID 3 for Nmap scans like the ones being performed in the test. I then proceeded to run T1016.001 instead to see if I could induce event ID 3. The test ran successfully again, but there were still no logs. From there, in my research it seemed like it could be blocked by my configuration file. I downloaded a different config file on GitHub from SwiftOnSecurity. I removed and reinstalled Sysmon. During the reinstallation I remembered that I had not added the log forwarding to the ossec.conf file. I added the necessary changes and ran the test again. This time I was getting event ID 1 and 11 logs sent to the SIEM. I ran { Invoke-AtomicTest T1046 -TestNumbers 10 -InputArgs @{target_host="192.168.1.1"} } to attempt to create event ID 3 logs. I verified that event ID 3 logs were being created in the event viewer. Next, I made an adjustment from no to yes in the Wazuh server ossec.conf file { <logall_json>yes</logall_json> }. I restarted the Wazuh server and tried the test again. After many different tries, I had finally logged event ID 3 in the SIEM.

Atomic Test T1055
I ran Atomic Test T1055 to test the logs for process injection. The expectation I had was to see the source image reflect the ART test executable and event ID 10 generated. I ran the test, and it caused many instances of calculator and PowerShell to pop up on the workstation. Event ID 10 was logged by Sysmon, and the images reflected what was occurring during the test.

Conclusion
Overall, I am pleased that my modifications were successful with a few tweaks applied during the experiments phase. I saw the appropriate logs and indicators.

If I were to go back in time to give myself advice to start this project, I would say to double-check the details. I would advise myself to think about the goals I have as a whole and every step of the process I am trying to accomplish. If I had been more aware of exactly how I see different events in my SIEM, I could have caught my mistake earlier and saved myself quite a bit of time.

Final Thoughts
The coolest thing I learned
I think it is cool how interdependent everything is in the ecosystem of logs and event management. I missed a couple of things in my setup, and even if I had only missed one, it would have had major consequences for how I would, or if I would, receive logs in the SIEM.

One piece of advice
Walk through the process step by step to make sure the workflow is complete. One broken link in the chain can cause complete failure for your monitoring goals.

My favorite resource
GitHub has been a very useful resource to me during this project.

Appreciation
I really appreciate the availability of so many different config files for Sysmon. Being able to peruse a few of them and study the differences was really. I am especially grateful to Neo23x0 (Florian Roth) for their config file that I used in my setup.

References
1. Wazuh SIEM Website
wazuh-siem.megaquagga.local
This was used to set up the Wazuh agent on the ART Workstation.

2. Reference for Sysmon Deployment
“Deploying Sysmon: Windows” TripleTen Sprint 10 Chapter 6
I used this to duplicate the same work process that had already been done to set up another workstation in the environment.

3. Sysmon Download
Sysmon Download Page - Microsoft.com
By Mark Russinovich and Thomas Garnier
Published 09/10/2026
I downloaded the zip file for Sysmon here.

4. Sysmon Config File
https://github.com/Neo23x0/sysmon-config/blob/master/sysmonconfig-export.xml
By Florian Roth (Neo23x0)
I used this config file for my Sysmon installation.

5. ART T1547.001
AtomicTest T1547.001
Atomic Red Team
I used this to test for persistence in the logs.

6. ART T1046
AtomicTest T1046
Atomic Red Team
I used this to test for network connections in the logs.

7. ART T1016.001
AtomicTest T1016.001
Atomic Red Team
I used this as an alternative test for network connections.

8. ART T1055
AtomicTest T1055
Atomic Red Team
I used this to test process injection in the logs.

9. Ossec XML
Log forwarding xml: ```

  <localfile>
    <location>Microsoft-Windows-Sysmon/Operational</location>
    <log_format>eventchannel</log_format>
  </localfile>
I used this to forward the logs from Sysmon to the Wazuh SIEM.