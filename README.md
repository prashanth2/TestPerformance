﻿# Introduction
A 4.7.2 .NET web application with EntityFramework 6.2.0 running on-premise completes http requests in less than 1.2 seconds. 
The same application deployed to azure using Service Fabric Windows 1803 with Containers and azure sql database premium, the performance drops to some requests completing in less than 4 seconds, others under 30 seconds, and a few under 90 seconds.
Looking through the logs, we see errors and delays opening sql connections and running sql queries.

## Repro in a machine just created
```
powershell
Get-NetIPInterface -AddressFamily IPv4
c:\test\test.ps1 -tag vm_mtu1500
c:\test\docker-compose.exe up -d
docker exec -it test_webapp_1 powershell -Command "c:\test\test.ps1 -tag container_mtu1500"
c:\test\docker-compose.exe down
```

## Repro setting MTU=1440
```
Get-NetIPInterface -AddressFamily IPv4 -NlMtuBytes 1500 | Set-NetIPInterface -NlMtuBytes 1440
Get-NetIPInterface -AddressFamily IPv4
Restart-Computer
powershell
Get-NetIPInterface -AddressFamily IPv4
c:\test\test.ps1 -tag vm_mtu1440
c:\test\docker-compose.exe up -d
docker exec -it test_webapp_1 powershell -Command "c:\test\test.ps1 -tag container_mtu1440"
c:\test\docker-compose.exe down
```
