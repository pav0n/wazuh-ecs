# wazuh-ecs

This project described the parsing of wazuh[HIDS] alert logs for elasticsearch with Elastic common schema using filebeat.

Goal of the project is to parse wazuh alerts logs directly from wazuh manager as simple as possible with Elastic common schema.

<img width="369" alt="Screenshot 2020-04-27 at 11 41 23 AM" src="https://user-images.githubusercontent.com/40884455/80339538-7cd6de00-887c-11ea-9f9e-f4c6db8ae9a4.png">

**Warning** : The parsing and conversion of alerts data with Elastic common schema is experimental. As much as possible ECS fields are parsed and added as initial release. 

## Assumption
The project assumes that wazuh manager and elasticsearch are already installed. [Wazuh](https://documentation.wazuh.com/3.7/installation-guide/index.html) official Installation guide and [Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/install-elasticsearch.html) official guide to be used for more details. 

The quickest way to get elasticsearch up and running is by using [Elastic cloud](https://www.elastic.co/elasticsearch/service) - however feel free to deploy elasticsearch whereever you like.

## Data parsing
Wazuh alerts which in JSON format are to be read and decoded using structured logging with filebeat.

Filebeat processors are used in decoding the JSON message into structured JSON objects and uniform naming convention with prefix as “wazuh” and it will be used across all the data parsed with wazuh alerts. 

```
Processor:
    - decode_json_fields:
        fields: ['message']
        target: "wazuh"
    - drop_fields:
        fields: ['message']
```
Filebeat is configured with a configuration file that decodes json alerts from wazuh and uses processors to decode into common naming convention. Ingest pipeline is used for ECS parsing.   

## ECS parsing
straightforward mapping of the original fields in the wazuh alert data to ECS related fields are  created with [ecs-mapper](https://github.com/elastic/ecs-mapper). ecs-mapper  is a tool to generate starter pipelines to help you get started quickly in mapping your event sources to ECS.

wazuh-ECS-mapping template sheet is maintained for mapping of alert data into ECS and used with ecs-mapper tool in generating pipelines. 

ECS mapper turns a field mapping CSV to roughly equivalent pipelines for:
- Beats
- Elasticsearch
- Logstash

We would be using Elasticsearch ingest processor pipelines in the filebeat configuration for simple and less resource intensive field parsing. 

Additional fields added with ECS parsing on wazuh alerts are as below

| ECS field      | Field value         |
|----------------|---------------------|
| event.module   | wazuh               |
| event.kind     | alert               |
| event.category | intrusion_detection |


## Installation and Configuration
1. Install filebeat

Filebeat to be installed in the wazuh management server.  Repositories for YUM and APT is available with [official documentation](https://www.elastic.co/guide/en/beats/filebeat/current/setup-repositories.html)

2. Define the Pipeline

Define the pipeline in Elasticsearch that converts and adds ECS data to wazuh alerts.

```
# curl -so /etc/filebeat/wazuh-ecs-pipeline.json https://raw.githubusercontent.com/HKcyberstark/wazuh-ecs/master/wazuh-ecs-pipeline.json
# chmod go+r /etc/filebeat/wazuh-ecs-pipeline.json
```

```
# cd /etc/filebeat
```
<<YOUR_ELASTIC_SERVER_IP>> is the IP address of your elasticsearch to be used in the below command.
```
# curl -H 'Content-Type: application/json' -XPUT 'https://<<YOUR_ELASTIC_SERVER_IP>>/_ingest/pipeline/wazuh-ecs-pipeline-pipeline' -d@wazuh-ecs-pipeline.json
```

3. Filebeat configuration

```
# curl -so /etc/filebeat/filebeat.yml https://raw.githubusercontent.com/HKcyberstark/wazuh-ecs/master/filebeat.yml
# chmod go+r /etc/filebeat/filebeat.yml
```
Edit the file /etc/filebeat/filebeat.yml and replace YOUR_ELASTIC_SERVER_IP with the IP address or the hostname of the Elasticsearch server. For example:

```
Output.elasticsearch
    host: [“https://YOUR_ELASTIC_SERVER_IP:9200”]
```

4. Run filebeat

For Systemd:
```
systemctl daemon-reload
systemctl enable filebeat.service
systemctl start filebeat.service
```
For SysV Init:
```
chkconfig --add filebeat
service filebeat start
```

## Field descriptions

| Field format | Description                                                    |
|--------------|----------------------------------------------------------------|
| host.*       | Details of server running the filebeat [usually wazuh manager] |
| agent.*      | Wazuh agent details                                            |
| event.*      | events related to wazuh alerts mapped with ECS                 |
| rule.*       | wazuh rule details mapped to ECS                               |
| wazuh.*      | original data fields from wazuh alert                          |

## Screenshots

<img width="1315" alt="Screenshot 2020-04-27 at 12 05 28 PM" src="https://user-images.githubusercontent.com/40884455/80341409-020fc200-8880-11ea-8b08-774ad7436055.png">

<img width="1611" alt="Screenshot 2020-04-27 at 12 08 39 PM" src="https://user-images.githubusercontent.com/40884455/80341378-f3c1a600-887f-11ea-8e70-345444c870ed.png">

## Next steps
- [x] Initial ECS parsing for wazuh alerts
- [x] wazuh alerts to be populated in Elastic SIEM
- [ ] Pre built ECS dashboards for wazuh
- [ ] Elastic SIEM rules for wazuh alerts
- [ ] MITRE ATT&CK mapping for wazuh alerts

## Contribution
Continuous enhancement and improvement with ECS parsing with the latest version of elasticstack is the development goal of the project. Help with your expertise to enhance the project.
