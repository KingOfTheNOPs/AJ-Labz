---
description: Little becon detection analytic that the Ricks created
---

# Beacon Detection

### Primary Question

Using the analytics discussed in this paper: Is the \(Detecting Cobalt Strike beacons in NetFlow data\) whitepaper does there appear to be Cobalt Strike BEACON behavior occurring within the environment?

### Data Collection Technique

Port Mirror/SPAN to Network Security Monitor\(ZEEK\)

### Data Sources

**UPSTREAM DEPENDENCIES**: None

**DATA SOURCES**:

* Zeek::conn.log

### Input Data

Source IP, Destination IP, responding bytes, destination port, protocol, time

### Analytic / Algorithm / Process

Search for connections with a value greater than 10, a standard deviation for responding bytes less than 100, and identify flows with a periodicity similar to beconing

### Analytic Execution

Enter the query into a Splunk search and adjust the time-range as desired. It is recommended to search at 7 day intervals and to focus on 1 port at a time if there is a large ammount of data. Adjust the query to blacklist certain IPs or fields as needed.

### Code Splunk &gt;= 8

```text
index=<ZEEK INDEX> sourcetype=bro_conn id.resp_p=<PORT> proto=tcp
`comment("Creating each flow object group based using the 4-tuple data as a group id")`
| eval tuple='id.orig_h'.":".'id.resp_h'.":".'id.resp_p'.":".'proto' 
| eval timestamp=_time 
| stats  list(timestamp) as timestamp2 list(resp_bytes) as bytes by tuple  
| eval timestamp2=mvsort(timestamp2)
| streamstats count as id 
| eval id = "ID" +":"+tostring(id) 
| eval first_time=mvindex(timestamp2, 0)
| eval last_time=mvindex(timestamp2, -1)
`comment("A basic filter where all flow collections are discarded if n < 10, where n represents the number of flows for a specific host")`
| eval n=mvcount(timestamp2)
| where NOT (n<10) 
| eval timestamp2 = mvmap(timestamp2, timestamp2-first_time)
`comment("Secondly we apply a filter that discards the flow collection if the absolute time ∆t < 30, where t represents the total duration of all flows for that host")`
| eval delta_time=last_time - first_time
| where delta_time>30
`comment("checks if the thresholds are reached for the standard deviation of the byte size (> 100 bytes)")`
| streamstats stdev(bytes) as standard_dev by id 
| where (standard_dev<100 AND standard_dev>0)
`comment("Below is used to take the timestamps from the MultiValue field in each event and use the new single values to calculate R2 values for detection. According to the paper an R2 above 98% is suspect")`
| fields tuple number timestamp2 id standard_dev first_time last_time
| mvexpand timestamp2 
| streamstats count as number by id 
| eventstats count as numevents sum(number) as sumX sum(timestamp2) as sumY sum(eval(number*timestamp2)) as sumXY sum(eval(number*number)) as sumX2 sum(eval(timestamp2*timestamp2)) as sumY2 by id 
    | eval R=((numevents*sumXY) - (sumX*sumY))/sqrt(((numevents*sumX2)-(sumX*sumX))* ((numevents*sumY2)-(sumY*sumY))) 
    | eval R2=R*R
    |where(R2 > .98)
|dedup tuple    
|table tuple id R2 standard_dev first_time last_time
`comment("This query is an attempt to operationalize 3 of the 4 detection techniques within this paper https://delaat.net/rp/2019-2020/p29/report.pdf")`
```

### Code Splunk &lt; 8

```text
index="cpb_ids_bro"  sourcetype=bro:conn:json id.resp_p=* proto=tcp

`comment("Creating each flow object group based using the 4-tuple data as a group id")`
| eval tuple='id.orig_h'.":".'id.resp_h'.":".'id.resp_p'.":".'proto' 
| eval timestamp=_time - 1577836800 
| sort 0 timestamp
| stats  list(timestamp) as timestamp2 list(resp_bytes) as bytes by tuple  
| eval timestamp2=mvsort(timestamp2)
| streamstats count as id 
| eval id = "ID" +":"+tostring(id) 
| eval first_time=mvindex(timestamp2, 0)
| eval last_time=mvindex(timestamp2, -1)

`comment("A basic filter where all flow collections are discarded if n < 10, where n represents the number of flows for a specific host")`
| eval n=mvcount(timestamp2)
| where NOT (n<10) 


`comment("Secondly we apply a filter that discards the flow collection if the absolute time t < 30, where t represents the total duration of all flows for that host")`
| eval delta_time=last_time - first_time| where delta_time>30
`comment("checks if the thresholds are reached for the standard deviation of the byte size (> 100 bytes)")`
| streamstats stdev(bytes) as standard_dev by id 
| where (standard_dev<10)

`comment("Below is used to take the timestamps from the MultiValue field in each event and use the new single values to calculate R2 values for detection. According to the paper an R2 above 98% is suspect")`
| fields tuple number timestamp2 id standard_dev first_time last_time| mvexpand timestamp2 limit=9999 
| streamstats count as number by id 
| eventstats count as numevents sum(number) as sumX sum(timestamp2) as sumY sum(eval(number*timestamp2)) as sumXY sum(eval(number*number)) as sumX2 sum(eval(timestamp2*timestamp2)) as sumY2 by id 
    | eval R=((numevents*sumXY) - (sumX*sumY))/sqrt(((numevents*sumX2)-(sumX*sumX))* ((numevents*sumY2)-(sumY*sumY))) 
    | eval R2=R*R*R    
    |where(R2 > .98)
    |dedup tuple    

|table tuple id R2 standard_dev first_time last_time
`comment("This query is an attempt to operationalize 2 of the 4 detection techniques within this paper https://delaat.net/rp/2019-2020/p29/report.pdf")`
```

### Dashboard

```text
<form>
  <label>Beacon Detection</label>
  <description>This query is an attempt to operationalize 3 of the 4 detection techniques within this paper https://delaat.net/rp/2019-2020/p29/report.pdf</description>
  <fieldset submitButton="false">
    <input type="time" token="field1">
      <label></label>
      <default>
        <earliest>-24h@h</earliest>
        <latest>now</latest>
      </default>
    </input>
    <input type="text" token="field2">
      <label>port</label>
      <default>80</default>
    </input>
  </fieldset>
  <row>
    <panel>
      <title>Beacon Detection</title>
      <table>
        <search>
          <query>index="nipr_traffic" sourcetype=bro_conn id.resp_p=$field2$ proto=tcp
| eval tuple='id.orig_h'.":".'id.resp_h'.":".'id.resp_p'.":".'proto' 
| eval timestamp=_time 
| stats  list(timestamp) as timestamp2 list(resp_bytes) as bytes by tuple  
| eval timestamp2=mvsort(timestamp2)
| streamstats count as id 
| eval id = "ID" +":"+tostring(id) 
| eval first_time=mvindex(timestamp2, 0)
| eval last_time=mvindex(timestamp2, -1)
| eval n=mvcount(timestamp2)
| where NOT (n&lt;10) 
| eval timestamp2 = mvmap(timestamp2, timestamp2-first_time)
| eval delta_time=last_time - first_time
| where delta_time&gt;30
| streamstats stdev(bytes) as standard_dev by id 
| where (standard_dev&lt;100 AND standard_dev&gt;0)
| fields tuple number timestamp2 id standard_dev first_time last_time
| mvexpand timestamp2 limit=9999 
| streamstats count as number by id 
| eventstats count as numevents sum(number) as sumX sum(timestamp2) as sumY sum(eval(number*timestamp2)) as sumXY sum(eval(number*number)) as sumX2 sum(eval(timestamp2*timestamp2)) as sumY2 by id 
    | eval R=((numevents*sumXY) - (sumX*sumY))/sqrt(((numevents*sumX2)-(sumX*sumX))* ((numevents*sumY2)-(sumY*sumY))) 
    | eval R2=R*R
    |where(R2 &gt; .98)

|dedup tuple    
|table tuple id R2 standard_dev first_time last_time</query>
          <earliest>-7d@h</earliest>
          <latest>now</latest>
          <sampleRatio>1</sampleRatio>
        </search>
        <option name="count">20</option>
        <option name="dataOverlayMode">none</option>
        <option name="drilldown">none</option>
        <option name="percentagesRow">false</option>
        <option name="rowNumbers">false</option>
        <option name="totalsRow">false</option>
        <option name="wrap">true</option>
      </table>
    </panel>
  </row>
</form>
```

### Output Data Format

Splunk table of events with hosts who have a perodocity similar to a beacon. Looking at unique tuples returned in a separate search can be used to verify the results.

### Analytic Flags / Alert Message

The output does not guarentee that the tuple is a beacon, only that it behaves like a beacon. Would not recommend generating an alert for this query. Recommended to integrate this into a dashboard on Splunk.

### How to Interpret the Results

Identify tuples within the returned table and further analyze the IP and traffic to determine if the flow is in fact malicious traffic.

### Performance Constraints

Search size of returned events may exceed max\_mem\_usage\_mb = 500 max\_mem\_usage\_mb provides a limitation to the amount of RAM, in megabytes \(MB\), a batch of events or results will use in the memory of a search process.

### Lessons Learned

* This analytic does not return only malcious traffic as the report mentions. When incorperated in an enterprise SIEM, there were a lot of results that appeared to be normal/good traffic.
* This analytic has not been validated in a large-scale enterprise environment yet. FP rate unknown 



