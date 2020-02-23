# Apache/Nginx Section

```
Return number of failed 404 requests and the corresponding page
grep " 404 " access.log | cut -d ' ' -f 7 | sort | uniq -c | sort -nr | head -20


Return number of requests by user agent
cat access.log | cut -d '"' -f 6 | sort | uniq -c | sort -nr | head -20

Return user agents ordered by the number of hits
awk -F\" '{print $6}' access.log | sort | uniq -c | sort -fr | head -20

Return number of status codes (userful to see how many 404 hits and 403 hits)
awk '{print $9}' access.log | sort | uniq -c | sort -nr | head -20

Unique Request IP Addresses
cat access.log | awk '{ print $1 }' | sort | uniq -c | sort -rn | head -n 25


Unique Request IP Addresses – Resolve country
apt-get install geoip-bin
cat access.log | awk '{ print $1 }' | sort | uniq -c | sort -rn | head -n 25 | awk '{ printf("%5d\t%-15s\t", $1, $2); system("geoiplookup " $2 " | cut -d \\: -f2 ") }'


Reurns which IP are requesting with Blank user agents
awk -F\" '($6 ~ /^-?$/)' access.log | awk '{print $1}' | sort | uniq


Returns top 10 IPs
cat access.log | awk '{ print $1 ; }' | sort | uniq -c | sort -n -r | head -n 10


Returns top 10 user agents
cat access.log | awk -F\" ' { print $6 } ' | sort | uniq -c | sort -rn | head -n 10

Unique IPs today
cat access.log | grep `date '+%e/%b/%G'` | awk '{print $1}' | sort | uniq -c | wc -l

unique IPs this month
cat access.* | grep `date '+%b/%G'` | awk '{print $1}' | sort | uniq -c | wc -l

Most popular URLs
cat access.log | awk '{ print $7 }' | sort | uniq -c | sort -rn | head -n 25


```


# IIS Section

```
List Number of user agents (Log file)
logparser.exe "SELECT cs(User-Agent), count(*) as count FROM "\Logs\iis\u_ex200222.log" GROUP BY cs(User-Agent) "

How many hits from each IP
logparser "select s-ip as IP, count(*) as hits  from "\Logs\iis\u_ex200222.log" group by ip"

What Path is getting the most hits
logparser "select cs-uri-stem as path, count(*) as hits  from "\Logs\iis\u_ex200222.log" group by path"

Returns a listing of query parameters passed to pages, with the number of times such requests were made
logparser "SELECT cs-uri-query, COUNT(*) AS [Requests]  FROM u_ex200222.log WHERE cs-uri-query IS NOT null GROUP BY cs-uri-query ORDER BY cs-uri-query"

Return the most noisy IP addrsses
logparser  "SELECT c-ip, COUNT(*) AS [Requests] FROM u_ex200222.log GROUP BY c-ip ORDER BY [Requests] DESC"

Returns requests per day (how many requests in a day)
logparser "SELECT TO_STRING(TO_LOCALTIME(TO_TIMESTAMP(date, time)), 'yyyy-MM-dd') AS [Day], COUNT(*) AS [Requests]  FROM u_ex200222.log GROUP BY [Day] ORDER BY [Day]"



Returns all requests by method, excluding common GETs and POSTs. Returns ip address and user agents, with count for each, for determining whether any are testing vulnerabilities
logparser "SELECT cs-method, c-ip, cs(User-Agent), COUNT(*) AS [Requests] FROM  u_ex200222.log WHERE cs-method NOT IN ('GET';'POST') GROUP BY cs-method, c-ip, cs(User-Agent) ORDER BY cs-method, Requests"

Return requests by user agent or browser
logparser "SELECT cs(User-Agent) AS Browser, COUNT(*) AS Requests  FROM u_ex*.log GROUP BY Browser ORDER BY Requests DESC"

returns number of requests for each uri path
logparser "SELECT COUNT(*) AS [Requests], EXTRACT_PATH(cs-uri-stem) AS [Path Requested]  FROM u_ex*.log GROUP BY [Path Requested] ORDER BY [Requests] DESC"


```




# Windows Events Section
### On Desktop systems

You Need to enable logging for the following items before you can use your event logs.
**TODO**
How to enable logging for ...

```
*Important event IDs*

512 / 4608  STARTUP
513 / 4609  SHUTDOWN
528 / 4624  LOGON
538 / 4634  LOGOFF
551 / 4647  BEGIN_LOGOFF
N/A / 4778  SESSION_RECONNECTED
N/A / 4779  SESSION_DISCONNECTED
N/A / 4800  WORKSTATION_LOCKED
* /    4801  WORKSTATION_UNLOCKED
N/A / 4802  SCREENSAVER_INVOKED
N/A / 4803  SCREENSAVER_DISMISSED
4525 / 4625 LOGIN FAILED

Successful logon
528, 540
failed logon
529-537, 539; 538, 551,

Accounts
Created 624;
enabled 626;
changed 642;
disabled 629;
deleted 630

Service Started or Stopped
7035, 7036


```
### On ADDC environment


#### Using LogParser.exe
```
Total Application Errors by day (Live logs)
LogParser "SELECT QUANTIZE(TimeGenerated, 86400) AS Day, COUNT(*) AS [Total Errors] FROM Application WHERE EventType = 1 OR EventType = 2 GROUP BY Day ORDER BY Day ASC"



```


#### Using Poweshell
```
List Service Errors and Installation info
Get-WinEvent -FilterHashtable @{Path="system.evtx"; ID=7030,7045}

same as above but use live system logs
Get-WinEvent -FilterHashtable @{logname="system"; ID=7030,7045}

Get USB info from event logs
Get-WinEvent -FilterHashtable @{logname="system"} | Where {$_.Message -like "*USB*"}

from offline log
Get-WinEvent -FilterHashtable @{path="system.evtx"} | Where {$_.Message -like "*USB*"}



```

## Sysmon Section

Event ID |   Descriptoin
-----|-----
 1 | Process Created
 2 | A process changed a file creation time
 3 | Network Connection
 4 | Sysmon service state change
 5 | Process Terminated
 6 | Driver Loaded
 7 | Image Loaded
 8 | CreateRemoteThread
 9 | RawAccessRead
10 | ProcessAccess
11 | FileCreate
12 | RegistryEvent (Object create and delete)
13 | RegistryEvent (Value Set)
14 | RegistryEvent (Key and Value Rename)
15 | FileCreateStreamHash
16 |
17 | PipeEvent (Pipe Created)
18 | PipeEvent (Pipe Connected)
19 | WmiEvent (WmiEventFilter activity detected)
20 | WmiEvent (WmiEventConsumer activity detected)
21 |  WmiEvent (WmiEventConsumerToFilter activity detected)
22 | DNSEvent (DNS query)
255 | Error



```
Get all DNS events without info
Get-WinEvent -FilterHashtable @{ path="sysmon.evtx"; id=22}


```
