# DataDog Agent App

Deploy the DataDog Agent and API key as a standard Convox app.

## Quick Start

```bash
# check out the repo
$ git clone https://github.com/convox-examples/dd-agent.git
$ cd dd-agent

# create the app and secrets
$ convox apps create
$ convox env set API_KEY=<your api key>
$ convox deploy

# monitor the app
$ convox logs
2016-12-07T18:24:02Z agent:0.70/i-603780f8 Starting agent process 8d510c19e42c
2016-12-07T18:24:02Z agent:0.70/i-f6dad10e Starting agent process a3ade3d781d1
2016-12-07T18:24:02Z agent:0.70/i-a89a51bf Starting agent process 96d1195ed218
2016-12-07T18:24:03Z agent:RADWUQIPBCQ/8d510c19e42c 2016-12-07 18:24:03,173 INFO supervisord started with pid 1
```

## Motivation

The [Datadog-AWS ECS Integration](http://docs.datadoghq.com/integrations/ecs/) guide requires:

* Creating a ECS Task Definition
* Customizing the Task Definition with your API Key
* Modifying IAM Policies
* Writing an EC2 User Data script

Convox removes all this complexity by deploying the agent as an app with a very simple `docker-compose.yml` manifest:

```yaml

version: '2'
services:
  agent:
    image: datadog/docker-dd-agent:latest
    environment:
     - API_KEY
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /proc/:/host/proc
      - /cgroup/:/host/sys/fs/cgroup
    ports:
      - 8125/udp
    privileged: true
    labels:
      - convox.agent=true
```

## Custom Metrics

In addition to collecting instance and container metrics, you can write custom metrics to the local Datadog agent with a statsd client over UDP. An example Ruby producer is:

```ruby
require "sinatra"
require "datadog/statsd"

agent  = `ip route list match 0/0 | awk '{print $3}'`.chomp
statsd = Datadog::Statsd.new(agent, 8125)

get "/health" do
  UDPSocket.new.send 'page.views:1|c', 0, agent, 8125
  statsd.increment('health')
  status 200
end
```
