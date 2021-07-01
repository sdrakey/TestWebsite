cls

$Path = [string]$args[0]

#$Path = "C:\scripts\SFTP_DOWNLOADER\SFTP_DOWNLOAD_INFO.XML"

# load it into an XML object:
$xml = New-Object -TypeName XML
$xml.Load($Path)

foreach ($c in $xml.SFTP.SFTP_UPLOADS.SFTP_UPLOAD)
{
	$ENABLED = $c.ENABLED
    $HOSTNAME = $c.HOSTNAME
	$USERNAME = $c.USERNAME
    $PASSWORD = $c.PASSWORD
    $REMOTEPATH = $c.REMOTEPATH
    $LOCALPATH = $c.LOCALPATH
    $WILDCARD = $c.WILDCARD
    $recipients = @($c.EMAIL)
    $recipient = $recipients.split(",")
    $SMTP_GATEWAY = $c.SMTP_GATEWAY	
    $SSHKEYFRINGERPRINT = $c.SSHKEYFRINGERPRINT

    $ArrayEmailBody = [System.Collections.ArrayList]@()

    if($ENABLED -match "YES"){
                    try
                    {
                        # Load WinSCP .NET assembly
                        Add-Type -Path "WinSCPnet.dll"
 
                        # Setup session options
                        $sessionOptions = New-Object WinSCP.SessionOptions -Property @{
                            Protocol = [WinSCP.Protocol]::Sftp
                            HostName = $HOSTNAME
                            UserName = $USERNAME
                            Password = $PASSWORD
                            SshHostKeyFingerprint = $SSHKEYFRINGERPRINT
                        }
 
                        $session = New-Object WinSCP.Session
 
                        try
                        {
                            # Connect
                            $session.Open($sessionOptions)
 
                            # Get list of matching files in local directory
                            $files = Get-ChildItem $LOCALPATH -Filter $wildcard
 
                            # Any file matched?
                            if ($files.Count -gt 0)
                            {
                                foreach ($fileInfo in $files)
                                {
            
                                if ($($fileInfo.Length) -gt 0kb){
                                                                 #write-host $($fileInfo.Name)  "is larger than 0kb"
                                                                 # Download files
                                                                 $transferOptions = New-Object WinSCP.TransferOptions
                                                                 $transferOptions.TransferMode = [WinSCP.TransferMode]::Binary
                                                                 $filetoupload = $localPath + $($fileInfo.Name)
                                                                 $remoteuploadfile = $REMOTEPATH + $($fileInfo.Name)
                                                                 #write-host "Uploading " $filetodownload 
 
                                                                 $transferResult = $session.PutFiles($filetoupload, $remoteuploadfile, $True, $transferOptions)
                                                                
 
                                                                 # Throw on any error
                                                                 $transferResult.Check()

                                                                 # add files to email body array
                                                                 $line =  $($fileInfo.Name) + " uploaded to " + $remoteuploadfile
                                                                 [void]$ArrayEmailBody.Add($line)                                                                                                                                                                                                                                                                                                    
                                                                 }   


                                else                             {
                                                                  #write-host $($fileInfo.Name)  "is less than 0kb"
                                                                 }
                                }
                            }
                            else
                            {
                                Write-Host "No files matching $wildcard found"
                            }
                        }
                        finally
                        {
                            # Disconnect, clean up
                            $session.Dispose()

                            #if emailbody has content then email
                            if ($ArrayEmailBody.Count -gt 0)
                                                       {
                                                        $subject = "SFTP files Uploaded "                                                  
                                                        $emailbody = "The SFTP upload service has uploaded the following files from "+ $localPath + " for the user " +$USERNAME +"<br><br>"

                                                        foreach ($line in $ArrayEmailBody)
                                                                                         {
                                                                                          $emailbody = $emailbody + $line
                                                                                          $emailbody = $emailbody + "<br>"
                                                                                         }                                                                                                               

                                                        $fromaddress = "sftpfileuploads@tpt.org.uk"
 

                                                        foreach ($emailTo in $recipient)
                                                                                        {
                                                                                        
                                                                                         Send-MailMessage -To $emailTo -From $fromaddress -Subject "$subject" -BodyAsHtml $emailbody -SmtpServer $SMTP_GATEWAY  
                                                                                        }
                                                        
                                                                                                                                         
                                                        $ArrayEmailBody = $null
                                                        $emailbody = $null
                                                       }
                            
                        }
                    }
                    catch
                    {
                        Write-Host "Error: $($_.Exception.Message)"
                        exit 1
                    }
                      }
}




