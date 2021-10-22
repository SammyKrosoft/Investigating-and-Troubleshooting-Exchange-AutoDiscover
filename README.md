# Investigating-and-Troubleshooting-Exchange-AutoDiscover

The logs we use on Exchange servers would be:

- IIS Logs (all covering the same timeframe)

-- C:\inetpub\logs\LogFiles\W3SVC1 (that's for the front end part, which corresponds to theinitial client connection logs, before it's sent on the back end for server processing)

-- C:\inetpub\logs\LogFiles\W3SVC2 (that's for the back end part, client requests proxied by the front end part from other servers or the same one, it's random unless we force the client to connect first to a specific server with Hosts file entries for example)

-- C:\Windows\System32\LogFiles\HTTPERR (that's for the HTTP errors encountered)

- Autodiscover logs

-- V15\Logging\Autodiscover (back end processing of Autodiscover requests)

-- V15\Logging\HttpProxy\Autodiscover (front end processing of Autodiscover requests, in other words, initial client requests for autodiscover information)

# Going further with dumps

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
