# Using IIS logs along with FREB (Failed Requests Events Buffering) logs
  
## Get IIS and Autodiscover logs

<details>
<summary> Expand/Collapse </summary>
  
```output
  
## Front-End IIS:
C:\inetpub\logs\LogFiles\W3SVC1
## Back-End IIS:
C:\inetpub\logs\LogFiles\W3SVC2
  
## Front-End Autodiscover:
C:\Program Files\Microsoft\Exchange Server\V15\Logging\HttpProxy\Autodiscover
## Back-End Autodiscover:
C:\Program Files\Microsoft\Exchange Server\V15\Logging\Autodiscover
  
```

</details>

## Set, Start, Stop, and Get the FREB logs for the Front-End and Back-End autodiscover Virtual Directory

  ### Set IIS autodiscover logging content and status codes
  
<details>
  <summary> Expand/Collapse </summary>

#### On the IIS "Default Web site" object

> IIS -> Sites - > Default Website -> AutoDiscover -> Failed Request Tracing Rules
  
![image](https://user-images.githubusercontent.com/33433229/142072783-a22481ff-4e9d-44a7-bb39-fbe65be2607b.png)

> Select Add - > All content -> Status codes 100-999 -> Next -> Finish
  
![image](https://user-images.githubusercontent.com/33433229/142071549-4c54a72a-78af-4e32-8960-8c3439aa9cce.png)
  
![image](https://user-images.githubusercontent.com/33433229/142072445-f80279f0-3293-4328-bc08-e39758284916.png)

![image](https://user-images.githubusercontent.com/33433229/142072505-4bd70b88-4d37-4ba9-9856-7c834235ee27.png)

![image](https://user-images.githubusercontent.com/33433229/142072532-3ccfbaca-ef50-430c-aeee-983f3cac7338.png)

![image](https://user-images.githubusercontent.com/33433229/142072591-0e9c5cd5-7158-461b-a7a7-f1db15fbb7f2.png)


#### Same thing on the IIS "Exchange Back End" object
  
> IIS -> Sites - > Exchange Back End -> AutoDiscover -> Failed Request Tracing Rules
> Select Add - > All content -> Status codes 100-999 -> Next -> Finish

  </details>
  
  ### Start FREB on both Front-End and Back-End IIS sites
  
<details>
  <summary> Expand/Collapse </summary>
  
> IIS -> Sites - > Default Website -> (in the actions pane) 
> select Failed Request Tracing -> Check Enabled + set the Max number of files to 10,000 -> [Ok]

![image](https://user-images.githubusercontent.com/33433229/142069277-018fe643-fd4b-47b4-b094-ff87342eaf69.png)
  
> IIS -> Sites - > Exchange Back End -> (in the actions pane) 
> select Failed Request Tracing -> Check Enabled + set the Max number of files to 10,000 -> [Ok]

</details>
  
  ### Once we have captured the issue, stop FREB (uncheck "Enabled" check box)

<details>
  <summary>Expand/Collapse</summary>

> IIS -> Sites - > Default Website -> (in the actions pane)
>
> select Failed Request Tracing -> uncheck Enable -> [Ok]
  
![image](https://user-images.githubusercontent.com/33433229/142069377-ffe25929-0dd1-4851-966e-c6ff20d2b00b.png)
  
> IIS -> Sites - > Exchange Back End -> (in the actions pane)
>
> select Failed Request Tracing -> uncheck Enable -> [Ok]

</details>
  
### Collect both folders in ```%SystemDrive%\inetpub\logs\FailedReqLogFiles```

  > NOTE: this folder is the default location for FREB logs, you can specify any other folder on a disk where you have space when configuring FREB (see a couple of sections above)
  


# Logs to collect to generally investigate-and-Troubleshoot-Exchange-AutoDiscover

<details>
  <summary>Expand/Collapse</summary>
  
  
The logs we use on Exchange servers would be:

- IIS Logs (all covering the same timeframe)

  - C:\inetpub\logs\LogFiles\W3SVC1 (that's for the front end part, which corresponds to theinitial client connection logs, before it's sent on the back end for server processing)

  - C:\inetpub\logs\LogFiles\W3SVC2 (that's for the back end part, client requests proxied by the front end part from other servers or the same one, it's random unless we force the client to connect first to a specific server with Hosts file entries for example)

  - C:\Windows\System32\LogFiles\HTTPERR (that's for the HTTP errors encountered)

- Autodiscover logs (also covering the same timeframe as the above IIS logs)

  - C:\Program Files\Microsoft\Exchange Server\V15\Logging\Autodiscover (back end processing of Autodiscover requests)

  - C:\Program Files\Microsoft\Exchange Server\V15\Logging\HttpProxy\Autodiscover (front end processing of Autodiscover requests, in other words, initial client requests for autodiscover information)

</details>
  
# Going further with dumps

<details>
  <summary> Expand/Collapse </summary>

If we want to dump the Autodiscover application pool, we need first to get the process ID (PID) of the process holding the Autodiscover tasks, then we can dump that process once identified.

## Get the PID we want to dump

Use ```cmd.exe``` command console to run the below

```powershell
C:\Windows\System32\inetsrv\appcmd.exe list wp
```

The output will contain all the IIS App pool PIDs, in that specific Autodiscover example we need to find the ```output MSExchangeAutodiscoverAppPool``` in the list.

## Dump the process with ProcDump

Like the above section, use ```cmd.exe``` command console to run the below as well.

Below we setup ProcDUmp to run when the CPU gets high (above 80%).Type the following on your cmd console:

```powershell
procdump -ma -s 10 -n 3 <PID of the AutoD App Pool> -p "\Processor(_Total)\%Processor Time" 80
```

This will take:
- 3 dumps (```-n 3```) 
- every 10 seconds (```-s 10```)
- when the processor will be above 80% (```-p "<processor counter>" 80```)

> Running Procdump will generally slow the server down during the capture.

</details>
