1.	WPR -4 mins to predicted time
a.	Command line: WPR -start CPU
b.	Command line: WPR -stop C:\Temp\AutoD_WPR.etl
NOTE: *This could take 20 mins to stop
NOTE2:*Will need to update the ETL and the folder ending in “.NGENPDB”

2.	Run ExPerfWiz -4 mins to predicted time (1 second interval)

3.	Run this PowerShell script

Start-Transcript
$expectedMessage = "The remote server returned an error: (401) Unauthorized."
$timeoutMessage = "The operation has timed out."
while ($true) {
  try {
    Invoke-WebRequest "https://<Affected_Exchange_Server_Hostname>/autodiscover/autodiscover.xml" -TimeoutSec 1
  } catch {
    if ($_.Exception.Message -ne $expectedMessage) {
      Write-Host
      if ($_.Exception.Message -eq $timeoutMessage) {
        Write-Host (Get-Date) "Timeout"
      } else {
        Write-Host (Get-Date) "Unexpected error: " ($_.Exception.Message)
      }
    } else {
      Write-Host "." -NoNewLine
    }
    Start-Sleep -Seconds 1
  }
}
Stop-Transcript
