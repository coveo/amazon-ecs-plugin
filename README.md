# Amazon Elastic Container Service Plugin for Jenkins

[![Build Status](https://ci.jenkins.io/job/Plugins/job/amazon-ecs-plugin/job/master/badge/icon)](https://ci.jenkins.io/job/Plugins/job/amazon-ecs-plugin/job/master/)
[![Join the chat at https://gitter.im/jenkinsci/amazon-ecs-plugin](https://badges.gitter.im/jenkinsci/amazon-ecs-plugin.svg)](https://gitter.im/jenkinsci/amazon-ecs-plugin?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)


## About

This Jenkins plugin do use [Amazon Elastic Container Service](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/Welcome.html) to host jobs execution inside docker containers.

* see [Jenkins wiki](https://wiki.jenkins-ci.org/display/JENKINS/Amazon+EC2+Container+Service+Plugin) for detailed feature descriptions
* use [JIRA](https://issues.jenkins-ci.org/issues/?jql=project%3DJENKINS%20AND%20status%20in%28Open%2C"In%20Progress"%2CReopened%29AND%20component%3Damazon-ecs-plugin) to report issues / feature requests


## Building the Plugin

```bash
  $ java -version # Need Java 1.8, earlier versions are unsupported for build
  $ mvn -version # Need a modern maven version; maven 3.2.5 and 3.5.0 are known to work
  $ mvn clean install
```

To run locally, execute the following command and open the browser [http://localhost:8080/jenkins/](http://localhost:8080/jenkins/)

```bash
  $ mvn -e hpi:run
```

## Maintainers
Philipp Garbe ([GitHub](https://github.com/pgarbe), [Twitter](https://twitter.com/pgarbe))  
Douglas Manley ([GitHub](https://github.com/tekkamanendless))  
Jan Roehrich ([GitHub](https://github.com/roehrijn))  

## Documentation and Installation

Please find the documentation on the [Jenkins Wiki page Amazon EC2 Container Service Plugin](https://wiki.jenkins-ci.org/display/JENKINS/Amazon+EC2+Container+Service+Plugin).

## FAQ
### My parallel jobs don't start at the same time
Actually, there can be multiple reasons:

* The plugin creates a new agent only when the stage contains an `agent` [definition](https://jenkins.io/doc/book/pipeline/syntax/#agent). If this is missing, the stage inherits the agent definition from the level above and also re-uses the instance. 

* Also, parallel stages sometimes don't really start at the same time. Especially, when the provided label of the `agent` definition is the same. The reason is that Jenkins tries to guess how many instances are really needed and tells the plugin to start n instances of the agent with label x. This number is likely smaller than the number of parallel stages that you've declared in your Jenkinsfile. Jenkins calls the ECS plugin multiple times to get the total number of agents running.

* If launching of the agents takes long, and Jenkins calls the plugin in the meantime again to start n instances, the ECS plugin doesn't know if this instances are really needed or just requested because of the slow start. That's why the ECS plugin subtracts the number of launching agents from the number of requested agents (for a specific label). This can mean for parallel stages that some of the agents are launched after the previous bunch of agents becomes online.
