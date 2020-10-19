
## Work flow
- mylog -> filebeat -> logstash -> elasticsearch <- kibana
- logs will be send to filebeat and then to logstash for filtering. 
- After filtering logstash will load the data to elasticsearch
- elastalert will also connect to elastic search and will look for the query
- elasastic search will ship the logs to kibana with the custom index name specified in the longstash.template.config in logstash-config folder
- make the index in kibana to view the logs 
## Usage
```
Go to logstash-config folder and build the image
After that run the docker-compose file `make sure you add logstash build image`
```
### configuring filebeat

###### Elk stack (Amazon ami 2)
```
Docker:
sudo yum update -y
sudo amazon-linux-extras install docker
sudo service docker start
sudo usermod -a -G docker ec2-user
Docker-compose:
sudo curl -L https://github.com/docker/compose/releases/download/1.21.0/docker-compose-`uname -s`-`uname -m` | sudo tee /usr/local/bin/docker-compose > /dev/null
sudo chmod +x /usr/local/bin/docker-compose
ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
docker-compose --version
Go to elk stack folder and run docker-compose up to start elastic search, log stash and kibana
Filebeat (Amazon Ami)
curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.9.2-x86_64.rpm
sudo rpm -vi filebeat-7.9.2-x86_64.rpm
sudo su	
cd /etc/filebeat	
vi filebeat.yml	
Then add the log file path to filebeat.input	
Comment the output.elasticsearch	
Uncomment output.logstash and add the ip followed by port in the host section	i.e hosts: ["18.191.162.70:5044"]	
Save the file and then start the filebeat	
sudo systemctl start filebeat	
sudo systemctl enable filebeat	
If you want to restart the filebeat (after changes) then use this command:	
sudo systemctl restart filebeat	
Check the status of filebeat using:	
sudo systemctl status filebeat	
```
###### Filebeat (ubuntu)
```
curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.9.2-amd64.deb	
sudo dpkg -i filebeat-7.9.2-amd64.deb	
sudo su	
cd /etc/filebeat	
vi filebeat.yml	
Then add the log file path to filebeat.input	
Comment the output.elasticsearch	
Uncomment output.logstash and add the ip followed by port in the host section	i.e hosts: ["18.191.162.70:5044"]	
Save the file and then start the filebeat	
sudo systemctl start filebeat	
sudo systemctl enable filebeat	
If you want to restart the filebeat (after changes) then use this command:	
sudo systemctl restart filebeat	
Check the status of filebeat using:	
sudo systemctl status filebeat	
```
### configuring elastalert locally and connecting it to elastic search
```
- git clone https://github.com/Yelp/elastalert.git
- pip install "setuptools>=11.3"
- python setup.py install
`Elasticsearch 5.0+:`
- pip install "elasticsearch>=5.0.0"
`Elasticsearch 2.X:`
- pip install "elasticsearch<3.0.0"
- save the example.config.yaml file as config.yaml
- elastalert-create-index
- cd to example_rules/example_frequency.yaml and add the following content
```
###### config.yaml
```
rules_folder: example_rules
run_every:
  minutes: 1
buffer_time:
  minutes: 15
es_host: localhost
es_port: 9200
writeback_index: elastalert_status
writeback_alias: elastalert_alerts

alert_time_limit:
  days: 2
```
###### example_rules/example_frequency.yaml
```
name: Slack-demo

type: frequency

index: adserver-*

num_events: 1

timeframe:
  hours: 1

filter:
- query:
    query_string:
      query: "RabbitMQ connection is closed"
alert_text: "AdServer down on hostname: {1} , message: []"
alert_text_type: alert_text_only
alert_text_args: ["logLevel","agent.hostname","message","tags","type"]
# include: ["logLevel","agent.hostname","message","tags","type"]

alert:
- "slack"

slack:
slack_webhook_url:
```
### Applying elastalert to elastic search command
```
- elastalert-test-rule example_rules/example_frequency.yaml
- elastalert-test-rule --config config file example_rules/example_frequency.yaml
```