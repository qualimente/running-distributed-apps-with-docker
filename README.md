# Overview #

This repo contains resoruces for the ['Running Distributed Apps from localhost to AWS'](https://www.meetup.com/Docker-Phoenix/events/232976147/) talk.

## Script ##

```
wget -O docker-compose https://github.com/docker/compose/releases/download/1.8.0/docker-compose-`uname -s`-`uname -m`
chmod 755 docker-compose

git clone git@github.com:qualimente/running-distributed-apps-with-docker.git

or

wget https://raw.githubusercontent.com/qualimente/running-distributed-apps-with-docker/master/example-voting-app/docker-compose.yml

```

Create SwarmV2 cluster using Docker for AWS

```
Create Docker for AWS via CloudFormation
ssh -i ~/.ssh/docker-phoenix-2016-08-03.pem docker@<docker-elb>
```

Look-around Docker for AWS
```
docker node ls

# launch container in host's process, network, and fs namespaces
docker run --rm -it --privileged --net=host --pid=host -v /:/host alpine sh
hostname
ls /host/etc/*release
cat /host/etc/os-release
ps -ef | grep docker
export DOCKER_HOST='0.0.0.0:2375'
/host/usr/bin/docker ps
```

Application Demos

```
# ---------------- example-voting-app ----------------

# deploy on dev machine
cd example-voting-app
unset DOCKER_HOST
docker-compose up --force-recreate
docker-compose bundle

# deploy to SwarmV2
export DOCKER_HOST=ec2-52-53-218-126.us-west-1.compute.amazonaws.com:2375
docker stack deploy examplevotingapp
docker stack ps examplevotingapp
less examplevotingapp.dab

# check-out on SwarmV2
docker service ls
docker service inspect examplevotingapp_vote | jq '.[0] | { EndpointPorts: .Endpoint.Ports }'
docker service inspect examplevotingapp_result | jq '.[0] | { EndpointPorts: .Endpoint.Ports }'

# optional, but must match ELB
docker service update examplevotingapp_vote --publish-add "5000:80"
docker service update examplevotingapp_result --publish-add "5001:80"

# Add to Docker-ELB
https://us-west-1.console.aws.amazon.com/ec2/v2/home?region=us-west-1#LoadBalancers:

# check
http://docker-elb-430443134.us-west-1.elb.amazonaws.com:5000/
http://docker-elb-430443134.us-west-1.elb.amazonaws.com:5001/

```

```

# ---------------- Log Collection System ----------------

# on dev machine
docker network create log
cd log-system
docker-compose bundle
docker stack deploy logsystem
cd ../log-shipper
```

## Tech Details ##

CloudFormation templates came-from:

* https://docker-for-aws.s3.amazonaws.com/v1.12.0-rc3-beta1/docker_for_aws.json
* https://docker-for-aws.s3.amazonaws.com/aws/beta/aws-v1.12.0-beta4.json

# References #

* https://blog.docker.com/2016/07/docker-built-in-orchestration-ready-for-production-docker-1-12-goes-ga/


# Questions #

* how is DynamoDB being used? function, implementation
