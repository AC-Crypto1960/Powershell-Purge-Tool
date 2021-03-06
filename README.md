# Office 365 Mail Purge

A Tool to Purge Mail from O365. The automated ps1 file will ask you these via a series of windows.forms boxes.

# An automated version of this script is avalible
https://github.com/joshuaebmcguire1/Powershell-Purge-Tool/blob/main/RunAutoRemove.ps1

# Variable Translations
 This tool will ask you to specify;
 
 Sender of the email to be purged = X (This allows us to narrow down the search from a specific sender)
 
 Your SMTP server address = S (This will be used to send you a notification email of completion of the purge.)
 
 Subject of email to be removed = U (This allows us to narrow down the search as it is limited to 10 items per mailbox)
 
 You're email address = Y (This is used to autenitcate against Exchange Powershell online and also to send you a notification email of completion of the purge.)
 
 Recieved Starting From = Z (Enter in M/D/YYYY Format)
 
 Recieved Ending From = W (Enter in M/D/YYYY Format)

# Connect using Online Exchange Powershell Module

Connect-IPPSSession -Credential $y

# Begin a new compliance search

New-ComplianceSearch -Name "Remove Phishing Message" -ExchangeLocation All -ContentMatchQuery "(Received:$z..$w) AND (From:$x) AND (SUBJECT:'$u')"

# Start the search.

Start-ComplianceSearch -Identity "Remove Phishing Message"

# View Results

$Stats = Get-ComplianceSearch -Identity "Remove Phishing Message" | Select -Expand SearchStatistics | Convertfrom-JSON
   $Data = $Stats.ExchangeBinding.Sources |?{$_.ContentItems -gt 0}
   Write-Host ""
   Write-Host "Total Items found matching query:" $ItemsFound 
   Write-Host ""
   Write-Host "Items found in the following mailboxes"
   Write-Host "--------------------------------------"
   Foreach ($D in $Data)  {Write-Host $D.Name "has" $D.ContentItems "items of size" $D.ContentSize }
   
# Hard Delete

New-ComplianceSearchAction -SearchName "Remove Phishing Message" -Purge -PurgeType HardDelete -Confirm:$false

# Get Status of the Delete

Get-ComplianceSearchAction

# Mails a completion report

$SmtpClient = new-object system.net.mail.smtpClient

$SmtpClient.EnableSsl = $true

$MailMessage = New-Object system.net.mail.mailmessage

$SmtpClient.Host = "$s"

$mailmessage.from = new-object System.Net.Mail.MailAddress("$y"PowerShell Alerts")

$mailmessage.To.add("$y")

$mailmessage.Subject = “PowerShell O365 Purge Has Completed”

$mailmessage.Body = "PowerShell has been instructed by $y to purge all emails from $x between the dates $z - $w and with the subject: $u. The reason has been given as $v. This Job has completed and the results are as follows; $Data"

$smtpclient.Send($mailmessage)

# Remove the Case
Remove-ComplianceSearch -Identity "Remove Phishing Message" -Confirm:$false

# Disconnects after complete

Disconnect-ExchangeOnline -Confirm:$false
