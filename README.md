# Opensearch Docker Test
A simple test of using Logstash and Opensearch with docker compose. This includes shipping syslog logs
from docker, and netflow data, to logstash, and onwards to opensearch.
It also includes an example of how to set up LDAP login in opensearch, but no users are created.


## Usage
Use `docker-compose up` to start the containers, then go to `localhost:5601` to see the dashboard.
Login with username "admin" and password "admin".


### First time setup
The first time you start the Opensearch cluster you have to create an index to view the data.
Go to the burger menu -> Management -> Stack management -> Index Patterns -> Create index pattern -> Enter "docker-syslog-*" -> Next -> Select @timestamp as the time field -> Create.

Do the same, but enter "netflow-*" instead of "docker-syslog-*" to add an index for the netflow data.

Now, to view the data, you can go back to the front page -> Visualize & analyse -> Discover.

### Adding logs from other containers
Right now only logs from the log-generator container is shipped to logstash. Logs from other containers can
be included by adding the line "logging: *logging" to the docker-compose definition. Look at the log-generator
container for an example.

## FAQ

### Why doesn't the logs show the IP/hostname of the container that sent them?
This is because the container doesn't send the logs, the docker daemon does. And because the docker daemon
runs on the host, not inside the network created by docker compose, we have to open a port from the outside
to logstash. This means that all logs gets an IP on the form 172.*.0.1, which is the IP of the virtual router.

### Why use UDP instead of TCP?
When using TCP docker crash if it can't send logs to the log sink. Since logstash takes a while to get started
docker will always crash. UDP doesn't have this problem, since docker doesn't know if the packets are received or not.

This could be mitigated by adding a 20 second delay to the containers that send logs.
