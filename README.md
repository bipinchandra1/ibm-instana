Troubleshooting Power HMC Instana Sensor problems.


Problem: 
You configured the Power HMC Instana Sensor correctly but the sensor is not getting activated successfully.
Or
Sensor is activated successfully but getting terminated for some unknown reasons.


Resolution:
This usually indicates that the sensor is not configured properly. 
Validate the configuration.yaml file 
The user should have api access to retrieve the performance metrics for the resources. 
assign 'hmcviewer' role to the user, and enable the 'Performance and Capacity Monitoring' preferences for the managed systems.
The following flags also need to be enabled:
LongTermMonitorEnabled: Long term monitoring configuration value
AggregationEnabled: Utilization data aggregation configuration value
EnergyMonitorEnabled: Energy monitoring status



Problem: 
There is a need to validate if the Power HMC API is functioning properly or not.
Or
There are frequent gaps in the metrics data in the Instana dashboard UI.
Or
Following exception is seen in the log:
2022-07-27T05:42:47.536-04:00 | DEBUG | powerhmc-service-pool-thread-2   | PowerHMC         | com.instana.sensor-power-hmc - 1.0.6 | Stacktrace: 
java.net.SocketTimeoutException: Read timed out
        at java.net.SocketInputStream.socketRead0(Native Method) ~[?:1.8.0]
        at java.net.SocketInputStream.socketRead(SocketInputStream.java:127) ~[?:1.8.0]
        at java.net.SocketInputStream.read(SocketInputStream.java:182) ~[?:1.8.0]
        at java.net.SocketInputStream.read(SocketInputStream.java:152) ~[?:1.8.0]


Resolution:
Prerequisite:
1) I used RHEL instead of mac.

STEP 1
Login:
Request method: PUT

Create file login.xml. See the attachment.
Put valid userid and password.

$ curl -k -i -X PUT -H "Content-Type: application/vnd.ibm.powervm.web+xml; type=LogonRequest" -H "Accept: application/vnd.ibm.powervm.web+xml; type=LogonResponse" -H "X-Audit-Memento: hmc_test" -d @login.xml https://9.3.147.20:12443/rest/api/web/Logon

Response:
HTTP/2 200 
date: Tue, 10 May 2022 05:26:37 GMT
server: Server Hardware Management Console
x-content-type-options: nosniff
strict-transport-security: max-age=31536000; includeSubDomains
referrer-policy: no-referrer
x-frame-options: SAMEORIGIN
x-transaction-id: XT10066672
content-type: application/vnd.ibm.powervm.web+xml; type=LogonResponse
set-cookie: JSESSIONID=684CEB2C0244BE4274025F0A237A295B; Path=/rest; Secure; HttpOnly
set-cookie: CCFWSESSION=DCB4CCFBE1C68327D4950873FAD985F3; Path=/; Secure; HttpOnly
content-security-policy: default-src https:; script-src https: 'unsafe-inline' 'unsafe-eval'; style-src https: 'unsafe-inline' 'unsafe-eval'; connect-src 'self'
x-xss-protection: 1; mode=block
vary: User-Agent

<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<LogonResponse xmlns="http://www.ibm.com/xmlns/systems/power/firmware/web/mc/2012_10/" xmlns:ns2="http://www.w3.org/XML/1998/namespace/k2" schemaVersion="V1_0">
    <Metadata>
        <Atom/>
    </Metadata>
    <X-API-Session kb="ROR" kxe="false">Ivq2wxwdH42qfYLSurUMgkaZZuRrYW4Rr8dN4WWufIT1UIKqZqh1DaR_DsVBnWCQGw6ffgTbEv17UwrUQHsr_XNbBs3WMvSfRK58D4tiYFcPPk-ekP5QCfc7dnMSJ3sJT8fcPCLO_QwxmjhGY-E7YjiW9q764nlg-wtFsVMrjRBBqLvvHfQyh8W1hxSFDSxDxCXFzcGjF-PRt4M6VHXW5_8V6Kmb68gJwnSDbI367C4=</X-API-Session>
</LogonResponse>


Copy the X-API-Session id from above.

STEP 2
GET current PCM attributes:

curl -k -i -H "Content-Type: application/xml" -H "X-API-Session: Ivq2wxwdH42qfYLSurUMgkaZZuRrYW4Rr8dN4WWufIT1UIKqZqh1DaR_DsVBnWCQGw6ffgTbEv17UwrUQHsr_XNbBs3WMvSfRK58D4tiYFcPPk-ekP5QCfc7dnMSJ3sJT8fcPCLO_QwxmjhGY-E7YjiW9q764nlg-wtFsVMrjRBBqLvvHfQyh8W1hxSFDSxDxCXFzcGjF-PRt4M6VHXW5_8V6Kmb68gJwnSDbI367C4=" -X GET https://9.3.147.20:12443/rest/api/pcm/preferences

Response:
HTTP/2 200 
date: Tue, 10 May 2022 05:26:37 GMT
Server: Server Hardware Management Console PCM
X-Content-Type-Options: nosniff
Strict-Transport-Security: max-age=31536000; includeSubDomains
X-Frame-Options: SAMEORIGIN
Referrer-Policy: no-referrer
ETag: -1049443091
Last-Modified: Fri, 23 Sep 2022 06:04:12 GMT
X-HMC-Schema-Version: V1_11_1
Content-Type: application/atom+xml
Vary: Accept-Encoding,User-Agent
Content-Security-Policy: default-src https:; script-src https: 'unsafe-inline' 'unsafe-eval'; style-src https: 'unsafe-inline' 'unsafe-eval'; connect-src 'self'
X-XSS-Protection: 1; mode=block
Transfer-Encoding: chunked


<feed xmlns="http://www.w3.org/2005/Atom" xmlns:ns2="http://a9.com/-/spec/opensearch/1.1/" xmlns:ns3="http://www.w3.org/1999/xhtml">
    <id>c6fab59d-4ace-4614-825f-a22a6c0bc722</id>
    <title type="text">Performance and Capacity Monitoring Preferences</title>
    <subtitle type="text"/>
    <link rel="self" href="https://9.212.142.9/rest/api/pcm/preferences"/>
    <generator uri="IBM Power Systems Management Console" version="1"/>
    <entry>
        <id>533b1454-73f0-401c-a5dc-2aebcdef2f70</id>
        <updated>2022-09-23T06:04:12.400Z</updated>
        <title type="text">Performance and Capacity Monitoring Preferences</title>
        <published>2022-09-23T06:04:12.400Z</published>
        <author>
            <name>IBM Power Systems Management Console</name>
        </author>
        <content type="application/xml">
            <ManagementConsolePcmPreference:ManagementConsolePcmPreference xmlns:ManagementConsolePcmPreference="http://www.ibm.com/xmlns/systems/power/firmware/pcm/mc/2012_10/" xmlns="http://www.ibm.com/xmlns/systems/power/firmware/pcm/mc/2012_10/" xmlns:ns2="http://www.w3.org/XML/1998/namespace/k2" schemaVersion="V1_0">
    <Metadata>
        <Atom>
            <AtomID>2e587e15-e815-39ef-9ec6-22e365dd043f </AtomID>
            <AtomCreated>1663913052394</AtomCreated>
        </Atom>
    </Metadata>
    <MaximumManagedSystemsForLongTermMonitor kxe="false" kb="ROR">48</MaximumManagedSystemsForLongTermMonitor>
    <MaximumManagedSystemsForComputeLTM kb="ROR" kxe="false">60</MaximumManagedSystemsForComputeLTM>
    <MaximumManagedSystemsForAggregation kxe="false" kb="ROR">48</MaximumManagedSystemsForAggregation>
    <MaximumManagedSystemsForShortTermMonitor kxe="false" kb="ROR">5</MaximumManagedSystemsForShortTermMonitor>
    <MaximumManagedSystemsForEnergyMonitor kb="ROR" kxe="false">48</MaximumManagedSystemsForEnergyMonitor>
    <AggregatedMetricsStorageDuration kb="UOD" kxe="false">3888000</AggregatedMetricsStorageDuration>
    <ManagedSystemPcmPreference kb="UOD" kxe="false" schemaVersion="V1_0">
        <Metadata>
            <Atom>
                <AtomID>ce383435-2e1e-3fbb-beee-f8851bbe008f</AtomID>
                <AtomCreated>1663913052303</AtomCreated>
            </Atom>
        </Metadata>
        <SystemName kb="ROR" kxe="false">BETA</SystemName>
        <MachineTypeModelSerialNumber kb="ROR" kxe="false" schemaVersion="V1_0">
            <Metadata>
                <Atom/>
            </Metadata>
            <MachineType kxe="false" kb="ROR">9009</MachineType>
            <Model kxe="false" kb="ROR">22A</Model>
            <SerialNumber kb="ROR" kxe="false">78DC780</SerialNumber>
        </MachineTypeModelSerialNumber>
        <EnergyMonitoringCapable kb="ROO" kxe="false">true</EnergyMonitoringCapable>
        <LongTermMonitorEnabled kb="UOD" kxe="false">true</LongTermMonitorEnabled>
        <AggregationEnabled kxe="false" kb="UOD">true</AggregationEnabled>
        <ShortTermMonitorEnabled kxe="false" kb="UOD">false</ShortTermMonitorEnabled>
        <ComputeLTMEnabled ksv="V1_1_0" kb="UOD" kxe="false">false</ComputeLTMEnabled>
        <EnergyMonitorEnabled kb="UOD" kxe="false">true</EnergyMonitorEnabled>
        <AssociatedManagedSystem kb="ROO" kxe="false" href="https://9.3.147.20:12443/rest/api/uom/ManagedSystem/ce383435-2e1e-3fbb-beee-f8851bbe008f" rel="related"/>
    </ManagedSystemPcmPreference>
    <ManagedSystemPcmPreference kb="UOD" kxe="false" schemaVersion="V1_0">
        <Metadata>
            <Atom>
                <AtomID>d410c86b-812d-3d14-b872-ba933a9f337f</AtomID>
                <AtomCreated>1663913052303</AtomCreated>
            </Atom>
        </Metadata>
        <SystemName kb="ROR" kxe="false">EPSILON</SystemName>
        <MachineTypeModelSerialNumber kb="ROR" kxe="false" schemaVersion="V1_0">
            <Metadata>
                <Atom/>
            </Metadata>
            <MachineType kxe="false" kb="ROR">9009</MachineType>
            <Model kxe="false" kb="ROR">22G</Model>
            <SerialNumber kb="ROR" kxe="false">78F3B40</SerialNumber>
        </MachineTypeModelSerialNumber>
        <EnergyMonitoringCapable kb="ROO" kxe="false">true</EnergyMonitoringCapable>
        <LongTermMonitorEnabled kb="UOD" kxe="false">true</LongTermMonitorEnabled>
        <AggregationEnabled kxe="false" kb="UOD">true</AggregationEnabled>
        <ShortTermMonitorEnabled kxe="false" kb="UOD">false</ShortTermMonitorEnabled>
        <ComputeLTMEnabled ksv="V1_1_0" kb="UOD" kxe="false">false</ComputeLTMEnabled>
        <EnergyMonitorEnabled kb="UOD" kxe="false">true</EnergyMonitorEnabled>
        <AssociatedManagedSystem kb="ROO" kxe="false" href="https://9.3.147.20:12443/rest/api/uom/ManagedSystem/d410c86b-812d-3d14-b872-ba933a9f337f" rel="related"/>
    </ManagedSystemPcmPreference>
</ManagementConsolePcmPreference:ManagementConsolePcmPreference>
        </content>
    </entry>
</feed>

Copy one of the managed System AtomID.

STEP 3
GET current PCM attributes:

curl -k -i -H "Content-Type: application/xml" -H "X-API-Session: Ivq2wxwdH42qfYLSurUMgkaZZuRrYW4Rr8dN4WWufIT1UIKqZqh1DaR_DsVBnWCQGw6ffgTbEv17UwrUQHsr_XNbBs3WMvSfRK58D4tiYFcPPk-ekP5QCfc7dnMSJ3sJT8fcPCLO_QwxmjhGY-E7YjiW9q764nlg-wtFsVMrjRBBqLvvHfQyh8W1hxSFDSxDxCXFzcGjF-PRt4M6VHXW5_8V6Kmb68gJwnSDbI367C4=" -X GET https://9.3.147.20:12443/rest/api/pcm/ManagedSystem/2e587e15-e815-39ef-9ec6-22e365dd043f/AggregatedMetrics

Response:
HTTP/2 200 


<feed xmlns="http://www.w3.org/2005/Atom" xmlns:ns2="http://a9.com/-/spec/opensearch/1.1/" xmlns:ns3="http://www.w3.org/1999/xhtml">
    <id>06751d27-5953-3f7f-bfaa-44382f516fea</id>
    <updated>2022-09-23T06:05:00.000Z</updated>
    <title type="text">AggregatedMetrics</title>
    <subtitle type="text">ManagedSystem 06751d27-5953-3f7f-bfaa-44382f516fea</subtitle>
    <link rel="self" href="https://9.212.142.9/rest/api/pcm/ManagedSystem/06751d27-5953-3f7f-bfaa-44382f516fea/AggregatedMetrics"/>
    <generator uri="IBM Power Systems Management Console" version="1"/>
    <entry>
        <id>ddafdf9a-01f0-472d-b13d-b6e046940e4e</id>
        <updated>2022-09-23T00:00:00.000Z</updated>
        <title type="text">ManagedSystem_06751d27-5953-3f7f-bfaa-44382f516fea_20220810T000000+0000_20220923T000000+0000_86400.json</title>
        <published>2022-08-10T00:00:00.000Z</published>
        <link type="application/json" href="https://9.3.147.20:12443/rest/api/pcm/AggregatedMetrics/ManagedSystem_06751d27-5953-3f7f-bfaa-44382f516fea_20220810T000000+0000_20220923T000000+0000_86400.json"/>
        <author>
            <name>IBM Power Systems Management Console</name>
        </author>
        <category term="ManagedSystem" frequency="86400"/>
    </entry>
.
.
.
</feed>

Copy above link :
https://9.3.147.20:12443/rest/api/pcm/AggregatedMetrics/ManagedSystem_06751d27-5953-3f7f-bfaa-44382f516fea_20220810T000000+0000_20220923T000000+0000_86400.json

STEP 3
GET current PCM attributes:

curl -k -i -H "Content-Type: application/xml" -H "X-API-Session: Ivq2wxwdH42qfYLSurUMgkaZZuRrYW4Rr8dN4WWufIT1UIKqZqh1DaR_DsVBnWCQGw6ffgTbEv17UwrUQHsr_XNbBs3WMvSfRK58D4tiYFcPPk-ekP5QCfc7dnMSJ3sJT8fcPCLO_QwxmjhGY-E7YjiW9q764nlg-wtFsVMrjRBBqLvvHfQyh8W1hxSFDSxDxCXFzcGjF-PRt4M6VHXW5_8V6Kmb68gJwnSDbI367C4=" -X GET https://9.3.147.20:12443/rest/api/pcm/AggregatedMetrics/ManagedSystem_06751d27-5953-3f7f-bfaa-44382f516fea_20220810T000000+0000_20220923T000000+0000_86400.json 

Response:
A big json with metrics data.












Note: (Optional)
If you want to run the curl command in a loop, use following command:
watch -n {{seconds}} {{your-command}}

eg. 
watch -n 300 “curl -k -i -H ‘Content-Type: application/xml’ -H ‘X-API-Session: Ivq2wxwdH42qfYLSurUMgkaZZuRrYW4Rr8dN4WWufIT1UIKqZqh1DaR_DsVBnWCQGw6ffgTbEv17UwrUQHsr_XNbBs3WMvSfRK58D4tiYFcPPk-ekP5QCfc7dnMSJ3sJT8fcPCLO_QwxmjhGY-E7YjiW9q764nlg-wtFsVMrjRBBqLvvHfQyh8W1hxSFDSxDxCXFzcGjF-PRt4M6VHXW5_8V6Kmb68gJwnSDbI367C4=’ -X GET https://9.3.147.20:12443/rest/api/pcm/ManagedSystem/2e587e15-e815-39ef-9ec6-22e365dd043f/preferences”

or
while sleep {{seconds}}; do {{your-command}}; done

eg.
while sleep 300; do “curl -k -i -H ‘Content-Type: application/xml’ -H ‘X-API-Session: Ivq2wxwdH42qfYLSurUMgkaZZuRrYW4Rr8dN4WWufIT1UIKqZqh1DaR_DsVBnWCQGw6ffgTbEv17UwrUQHsr_XNbBs3WMvSfRK58D4tiYFcPPk-ekP5QCfc7dnMSJ3sJT8fcPCLO_QwxmjhGY-E7YjiW9q764nlg-wtFsVMrjRBBqLvvHfQyh8W1hxSFDSxDxCXFzcGjF-PRt4M6VHXW5_8V6Kmb68gJwnSDbI367C4=’ -X GET https://9.3.147.20:12443/rest/api/pcm/ManagedSystem/2e587e15-e815-39ef-9ec6-22e365dd043f/preferences”; done




![image](https://user-images.githubusercontent.com/83693053/191935370-0ce4c842-c5cc-4f4f-a1af-acd91ba54c16.png)

