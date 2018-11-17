# Introduction
A 4.7.2 .NET web application with EntityFramework 6.2.0 and SqlAzureExecutionStrategy running on-premise completes http requests in less than 1.2 seconds. The same application deployed to azure Service Fabric Windows 1803 with Containers and azure sql database premium, the performance drops as some requests complete in less than 4 seconds, others under 30 seconds, and a few under 90 seconds. Looking through the application logs, we see errors, retries, and delays opening sql connections and running sql queries.

## About the repro test utilities
* TestEntityFramework472 creates the database schema, tables, and rows
* TestSqlDatabase472 opens sql connections and execute queries without a retry strategy. It is expected that every operation completes or fails in less than 1 second. Otherwise, the retry strategy will add compund delays and a simple operation that usually completes in less than 10 milliseconds would complete in a worst scenario of 2 minutes and 30 seconds.
* TestHttpPerformance472 is used to measure the performance of the full application, not included in this repository.

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
