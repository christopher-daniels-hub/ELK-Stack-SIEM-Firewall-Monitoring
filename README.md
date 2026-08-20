# ELK Stack SIEM Firewall Monitoring 🔍

## Project Overview

This project documents the development of an end-to-end ELK Stack security-monitoring workflow for UFW firewall logs. I configured Logstash pipelines, parsed raw firewall events with Grok, stored the resulting fields in Elasticsearch, investigated activity in Kibana, and created a security-monitoring dashboard.

The project gave me hands-on experience with the complete SIEM data lifecycle:

```text
Collect → Parse → Index → Search → Investigate → Visualize → Report
```

> **Featured Result:** I successfully processed thousands of firewall events and created a Kibana dashboard for monitoring traffic volume, blocked source addresses, destination ports, and network activity over time.

![Final Kibana firewall monitoring dashboard](Screenshots/11-firewall-monitoring-dashboard-fixed.png)

## Project Objectives 🎯

The objectives of this project were to:

- Build an ELK Stack environment on Ubuntu Server
- Understand Logstash input, filter, and output stages
- Collect Linux authentication and UFW firewall logs
- Create a custom Grok pattern for parsing firewall events
- Correct and standardize event timestamps
- Create an Elasticsearch index template
- Verify that parsed fields were searchable in Kibana
- Investigate blocked network traffic using KQL
- Create security visualizations and a monitoring dashboard
- Develop practical troubleshooting and SIEM-analysis skills

## Lab Environment

- Oracle VirtualBox
- Ubuntu Server 24.04
- Elasticsearch
- Logstash
- Kibana
- UFW firewall logs
- Linux authentication logs
- Kibana Query Language
- Logstash Grok patterns

The original virtual lab environment is no longer active. This repository preserves the project through sanitized configuration examples and screenshots captured during the completed lab.

## General Workflow

The following diagram illustrates how raw Linux logs moved through the ELK Stack and became searchable security findings, visualizations, and recommendations.

![ELK Stack SIEM firewall monitoring workflow](./Screenshots/elk-siem-workflow.png)

### Workflow Breakdown

1. **Collect:** Logstash collected Linux authentication and UFW firewall logs.
2. **Parse:** Grok patterns separated raw firewall messages into searchable fields.
3. **Transform:** Logstash standardized timestamps and removed unnecessary fields.
4. **Index:** Elasticsearch stored the structured events in searchable indices.
5. **Search:** Kibana Discover provided access to the indexed events.
6. **Investigate:** KQL filters isolated blocked traffic and other activity of interest.
7. **Visualize:** Kibana displayed source addresses, destination ports, bytes, and event trends.
8. **Report:** The dashboard and written findings summarized the security analysis.

## 1. Basic Logstash Pipeline

I began by creating a basic Logstash pipeline named `authlog.conf`. The pipeline read events from `/var/log/auth.log` and wrote the processed output to a separate authentication log file.

This step helped me understand how Logstash moves data through its input, filter, and output stages.

![Basic Logstash authentication pipeline](./Screenshots/01-authlog-pipeline.png)

The sanitized configuration is available here:

[View authlog.conf](configs/authlog.conf)

## 2. Logstash Output Verification

After configuring the pipeline, I verified that Logstash successfully created the expected output file and copied the authentication events into it.

This confirmed that the pipeline could read the input source and produce the intended output.

![Logstash authentication output](./Screenshots/02-authlog-output.png)

## 3. Elasticsearch Index Verification

I configured Logstash to forward processed events to Elasticsearch. I then verified in Kibana Index Management that the expected index had been created.

This demonstrated that Logstash and Elasticsearch were communicating successfully.

![Elasticsearch index verification](./Screenshots/03-elasticsearch-index.png)

## 4. Kibana Data View

I created a corresponding Kibana data view so the indexed events could be searched, filtered, and analyzed through Kibana Discover.

![Kibana data view](./Screenshots/04-kibana-data-view.png)

## 5. Custom Firewall Pipeline

I created a more advanced Logstash pipeline for processing raw UFW firewall logs. The pipeline used a custom Grok pattern to extract important information from each event.

The extracted fields included:

- Event timestamp
- Hostname
- Firewall action
- Source IP address
- Destination IP address
- Source port
- Destination port
- Protocol
- Packet size in bytes
- Network-interface information

The pipeline also corrected the event timestamp, removed unnecessary fields, and forwarded the structured events to an Elasticsearch index named `firewall`.

![Sanitized Logstash firewall pipeline](./Screenshots/05-firewall-logstash-pipeline.png)

A sanitized portfolio version of the configuration is available here:

[View firewall.conf](configs/firewall.conf)

The public configuration uses environment-variable and hostname placeholders. No working API keys, passwords, or authentication tokens are included.

## 6. Elasticsearch Index Template

I created an Elasticsearch index template to assign appropriate data types to the parsed firewall fields.

| Field | Data type | Purpose |
|---|---|---|
| `@timestamp` | Date | Supports timelines and time-based searches |
| `action` | Keyword | Identifies ALLOW, BLOCK, and AUDIT events |
| `bytes` | Numeric | Measures network-traffic volume |
| `srcip` | IP address | Identifies the connection source |
| `dstip` | IP address | Identifies the connection destination |
| `srcport` | Numeric | Records the source port |
| `dstport` | Numeric | Records the destination port |
| `protocol` | Keyword | Identifies TCP, UDP, and other protocols |
| `hostname` | Keyword | Identifies the system generating the event |

Correct field mappings allowed Kibana to search, sort, filter, and aggregate the data accurately.

![Firewall index-template mappings](./Screenshots/06-firewall-index-template.png)

## 7. Parsed Events in Kibana Discover

After completing the pipeline, I reviewed the processed events in Kibana Discover. The data view contained 4,821 documents with searchable fields for firewall actions, addresses, ports, protocols, timestamps, bytes, and geographic information.

This confirmed that the complete data pipeline was working:

```text
UFW Logs → Logstash → Elasticsearch → Kibana
```

![Parsed firewall events in Kibana Discover](./Screenshots/07-parsed-firewall-event.png)

## 8. Blocked-Traffic Investigation 🛡️

I used Kibana Query Language to isolate firewall events where the action was `BLOCK`.

I reviewed the resulting events to identify:

- Frequently blocked source addresses
- Targeted destination addresses
- Common destination ports
- Event timestamps
- Repeated connection attempts
- Geographic information associated with the traffic

This investigation demonstrated how a SOC analyst can reduce a large dataset to a smaller group of events requiring additional review.

![Blocked traffic investigation](./Screenshots/08-blocked-traffic-investigation.png)

## 9. Top Blocked Sources

I created a bar chart showing the source addresses responsible for the highest number of blocked events.

This visualization helped identify persistent sources that may have been conducting repeated connection attempts, network probing, or scanning activity.

A high number of blocked events alone does not prove malicious intent, so these findings would require correlation with additional logs and threat intelligence.

![Top blocked source addresses](./Screenshots/09-top-blocked-sources.png)

## 10. Targeted Destination Ports

I created a treemap showing the destination ports most frequently associated with blocked traffic.

The observed ports included:

- Port 23 — Telnet
- Port 22 — SSH
- Port 53 — DNS
- Port 80 — HTTP
- Port 16345 — Uncommon high-numbered port

Port 23 represented the largest portion of the reviewed blocked-port activity. Because Telnet transmits information without modern encryption protections, repeated traffic targeting this service would deserve additional investigation.

![Targeted destination ports](./Screenshots/10-targeted-destination-ports.png)

## 11. Final Kibana Dashboard 📊

I combined the saved visualizations into a unified Kibana dashboard named **Firewall Data Analysis**.

The dashboard included:

- Network bytes by destination address
- Top source addresses
- Destination ports by firewall action
- Network-traffic volume over time

The dashboard provided a centralized view of firewall activity and supported faster identification of persistent sources, frequently targeted services, and traffic-volume patterns.

![Final Kibana firewall monitoring dashboard](./Screenshots/11-firewall-monitoring-dashboard.png)

## Key Findings

The analysis produced several useful observations:

- The firewall dataset contained thousands of searchable events.
- BLOCK events revealed sources making repeated connection attempts.
- Telnet was the most frequently represented destination port among the reviewed blocked-port activity.
- Several external systems accounted for a large portion of the observed traffic volume.
- The traffic-over-time visualization did not reveal a single extreme or unexplained spike.
- Structured fields made it possible to move from raw firewall messages to targeted security investigations.

These observations represent findings from a controlled training dataset. Additional endpoint, identity, IDS, and threat-intelligence data would be required before declaring any source definitively malicious.

## Troubleshooting Experience

One of the main challenges was a `401 Unauthorized` response when Logstash attempted to connect to Elasticsearch.

I resolved the issue by:

1. Reviewing the Logstash error output
2. Identifying an invalid or missing API key
3. Generating a new authorized API key
4. Updating the Logstash configuration
5. Restarting the Logstash service
6. Verifying that events appeared in Elasticsearch
7. Confirming successful access through Kibana Discover

I also repeatedly validated configuration syntax and corrected errors in the Logstash and YAML configuration files. This strengthened my ability to read error messages, isolate configuration problems, and verify solutions systematically.

## Security Recommendations

Based on the firewall analysis, I would recommend:

- Disable unnecessary services, especially unencrypted legacy services
- Restrict administrative ports to approved source networks
- Review repeated blocks from the same source addresses
- Alert on sudden increases in blocked traffic
- Monitor frequently targeted ports
- Correlate firewall data with IDS, endpoint, and authentication logs
- Apply network segmentation
- Use threat intelligence to enrich suspicious addresses
- Maintain accurate timestamps across log sources
- Protect SIEM credentials with environment variables or a secrets manager
- Review and tune detection logic to reduce false positives

## Skills Demonstrated

- ELK Stack deployment and configuration
- Linux command-line administration
- Logstash pipeline development
- Grok pattern creation
- Log parsing and transformation
- Elasticsearch indexing
- Index-template creation
- Field mapping
- Kibana Discover
- Kibana Query Language
- Firewall-log analysis
- SIEM investigation
- Dashboard development
- Data visualization
- Authentication troubleshooting
- Security finding validation
- Technical documentation

## Lessons Learned

This project taught me that deploying a SIEM involves much more than installing software. The quality of the analysis depends on correctly collecting, parsing, mapping, timestamping, and validating the underlying data.

I also learned how important troubleshooting is to security engineering. Configuration mistakes, authentication failures, and incorrect field mappings can disrupt the entire pipeline. Working through these problems improved my technical confidence and reinforced the importance of methodical verification.

Most importantly, the project helped me connect technical configuration work with the responsibilities of a SOC analyst: reviewing data, identifying patterns, validating findings, and communicating practical security recommendations.

## Future Improvements 🚀

If I recreate or expand this environment, I would:

- Add automated alerting for repeated blocked connections
- Create detection rules for port scanning and brute-force patterns
- Ingest Windows and endpoint-security logs
- Add threat-intelligence enrichment
- Build dashboards for authentication and endpoint activity
- Apply MITRE ATT&CK mappings to detection use cases
- Use a secrets manager for production credentials
- Create a documented incident-triage workflow
- Integrate cloud-security logs from AWS or Azure

## Ethical and Privacy Statement

This project was completed in an authorized academic lab environment using training data. The repository contains only sanitized screenshots and configuration examples.

API keys, passwords, authentication tokens, private addresses, and other sensitive information have been removed or replaced with safe placeholders. This project should not be used to monitor or access systems without explicit authorization.
