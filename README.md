# MailSniper
MailSniper is a penetration testing tool for searching through email in a Microsoft Exchange environment for specific terms (passwords, insider intel, network architecture information, etc.). It can be used as a non-administrative user to search their own email, or by an Exchange administrator to search the mailboxes of every user in a domain.

For more information about MailSniper check out this [blog post](http://www.blackhillsinfosec.com/?p=5296).

MailSniper also includes additional modules for password spraying, enumerating users/domains, gathering the Global Address List from OWA and EWS, and checking mailbox permissions for every Exchange user at an organization.

For more information about additional MailSniper modules check out these blog posts: 

- [GAL & Password Spraying](http://www.blackhillsinfosec.com/?p=5330)

- [Invoke-OpenInboxFinder](http://www.blackhillsinfosec.com/?p=5871)

## Quick Start Guide
There are two main functions in MailSniper. These two functions are **Invoke-GlobalMailSearch** and **Invoke-SelfSearch**.

**Invoke-GlobalMailSearch** is a module that will connect to a Microsoft Exchange server and grant the "ApplicationImpersonation" role to a specified user. Having the "ApplicationImpersonation" role allows that user to search through all other domain user's mailboxes. After this role has been granted the Invoke-GlobalMailSearch function creates a list of all mailboxes in the Exchange database. It then connects to Exchange Web Services using the impersonation role to gather a number of emails from each mailbox, and ultimately searches through them for specific terms. By default the script searches for "\*password\*","\*creds\*","\*credentials\*"

To search all mailboxes in a domain the following command can be used:

```PowerShell
Invoke-GlobalMailSearch -ImpersonationAccount current-username -ExchHostname Exch01 -OutputCsv global-email-search.csv
```

This command will connect to the Exchange server located at 'Exch01' and prompt for administrative credentials (i.e. member of "Exchange Organization Administrators" or "Organization Management" group). Once administrative credentials have been entered a PS remoting session is setup to the Exchange server where the ApplicationImpersonation role is then granted to the "current-username" user. A list of all email addresses in the domain is then gathered, followed by a connection to Exchange Web Services as "current-username" where by default 100 of the latest emails from each mailbox will be searched through for the terms "\*pass\*","\*creds\*","\*credentials\*" and output to a CSV file called global-email-search.csv.

**Invoke-SelfSearch** is a module that will connect to a Microsoft Exchange server using Exchange Web Services to gather a number of emails from the current user's mailbox. It then searches through them for specific terms. This could potentially assist in privilege escalation after obtaining a user's credentials or assist in locating sensitive data as a non-admin.

To search the current user's mailbox the following command can be used:

```PowerShell
Invoke-SelfSearch -Mailbox current-user@domain.com
```

This command will connect to the Exchange server autodiscovered from the email address entered using Exchange Web Services where by default 100 of the latest emails from the "Mailbox" will be searched through for the terms "\*pass\*","\*creds\*","\*credentials\*".

### Invoke-GlobalMailSearch Options
```
ImpersonationAccount  - This user will be granted the ApplicationImpersonation role on the Exchange server.
ExchHostname          - The hostname of the Exchange server to connect to (If $AutoDiscoverEmail is specified the server will be autodiscovered).
AutoDiscoverEmail     - A valid email address that will be used to autodiscover where the Exchange server is located.
MailsPerUser          - The total number of emails to return for each mailbox.
Terms                 - Certain terms to search through each email subject and body for. By default the script searches for "*password*","*creds*","*credentials*".
OutputCsv             - Outputs the results of the search to a CSV file.
ExchangeVersion       - Specify the version of Exchange server to connect to. By default the script tries Exchange2010.
AdminUserName         - The username of an Exchange administator (i.e. member of the "Exchange Organization Administrators" or "Organization Management" group) including the domain (i.e. domain\adminusername).
AdminPassword         - The password to the Exchange administator (i.e. member of the "Exchange Organization Administrators" or "Organization Management" group) account specified with AdminUserName.
EmailList             - A text file listing email addresses to search (one per line).
Folder                - The folder within each mailbox to search. By default the script only searches the "Inbox" folder. By specifying 'all' for the Folder option all of the folders including subfolders of the specified mailbox will be searched.
Regex                 - The regex parameter allows for the use of regular expressions when doing searches. This will override the -Terms flag.
CheckAttachments      - If the CheckAttachments option is added MailSniper will attempt to search through the contents of email attachements in addition to the default body/subject. These attachments can be downloaded by specifying the -DownloadDir option. It only searches attachments that are of extension .txt, .htm, .pdf, .ps1, .doc, .xls, .bat, and .msg currently.
DownloadDir           - When the CheckAttachments option finds attachments that are matches to the search terms the files can be downloaded to a specific location using the -DownloadDir option. 
```
### Invoke-SelfSearch Options
```
ExchHostname          - The hostname of the Exchange server to connect to (If $Mailbox is specified the server will be autodiscovered).
Mailbox               - Email address of the current user the PowerShell process is running as.
MailsPerUser          - The total number of emails to return.
Terms                 - Certain terms to search through each email subject and body for. By default the script searches for "*password*","*creds*","*credentials*".
OutputCsv             - Outputs the results of the search to a CSV file.
ExchangeVersion       - Specify the version of Exchange server to connect to. By default the script tries Exchange2010.
Remote                - A switch for performing the search remotely across the Internet against a system hosting EWS. Instead of utilizing the current user's credentials if the -Remote option is added a new credential box will pop up for accessing the remote EWS service. 
Folder                - The folder within each mailbox to search. By default the script only searches the "Inbox" folder. By specifying 'all' for the Folder option all of the folders including subfolders of the specified mailbox will be searched.
Regex                 - The regex parameter allows for the use of regular expressions when doing searches. This will override the -Terms flag.
CheckAttachments      - If the CheckAttachments option is added MailSniper will attempt to search through the contents of email attachements in addition to the default body/subject. These attachments can be downloaded by specifying the -DownloadDir option. It only searches attachments that are of extension .txt, .htm, .pdf, .ps1, .doc, .xls, .bat, and .msg currently.
DownloadDir           - When the CheckAttachments option finds attachments that are matches to the search terms the files can be downloaded to a specific location using the -DownloadDir option. 
OtherUserMailbox      - Specify this flag when attempting to read emails from a different user's mailbox 
```
## Additional MailSniper Modules
**Get-GlobalAddressList** is a module that will first attempt to connect to an Outlook Web Access portal and utilize the "FindPeople" method (only available in Exchange2013 and up) of gathering email addresses from the Global Address List. If this does not succeed the script will attempt to connect to Exchange Web Services where it will attempt to gather the Global Address List. 
```PowerShell
Get-GlobalAddressList -ExchHostname mail.domain.com -UserName domain\username -Password Fall2016 -OutFile global-address-list.txt
```
**Get-MailboxFolders** is a module that will connect to a Microsoft Exchange server using Exchange Web Services to gather a list of folders from the current user's mailbox. 
```PowerShell
Get-MailboxFolders -Mailbox current-user@domain.com
```
**Invoke-PasswordSprayOWA** is a module that will attempt to connect to an Outlook Web Access portal and perform a password spraying attack using a userlist and a single password. PLEASE BE CAREFUL NOT TO LOCKOUT ACCOUNTS!
```PowerShell
Invoke-PasswordSprayOWA -ExchHostname mail.domain.com -UserList .\userlist.txt -Password Fall2016 -Threads 15 -OutFile owa-sprayed-creds.txt
```
**Invoke-PasswordSprayEWS** is a module that will attempt to connect to an Exchange Web Services portal and perform a password spraying attack using a userlist and a single password. PLEASE BE CAREFUL NOT TO LOCKOUT ACCOUNTS!
```PowerShell
Invoke-PasswordSprayEWS -ExchHostname mail.domain.com -UserList .\userlist.txt -Password Fall2016 -Threads 15 -OutFile sprayed-ews-creds.txt
```
**Invoke-DomainHarvestOWA** is a module that will attempt to connect to an Outlook Web Access portal and determine a valid domain name for logging into the portal based off of small timing differences in login attempts.
```PowerShell
Invoke-DomainHarvestOWA -ExchHostname mail.domain.com -DomainList .\domainlist.txt -OutFile potentially-valid-domains.txt
```
**Invoke-UsernameHarvestOWA** is a module that will attempt to connect to an Outlook Web Access portal and harvest valid usernames based off of small timing differences in login attempts.
```PowerShell
Invoke-UsernameHarvestOWA -ExchHostname mail.domain.com -UserList .\userlist.txt -Threads 1 -OutFile owa-valid-users.txt
```
**Invoke-OpenInboxFinder** is a module that will attempt to determine if the current user running MailSniper has access to the Inbox of each email address in a list of addresses.
```PowerShell
Invoke-OpenInboxFinder -EmailList email-list.txt
```
**Get-ADUsernameFromEWS** is a module that will attempt to determine the Active Directory username for a single email address or a list of addresses. Use the Get-GlobalAddressList module to harvest a full list of email addresses to use with Get-ADUsernameFromEWS.
```PowerShell
Get-ADUsernameFromEWS -EmailList email-list.txt
```

