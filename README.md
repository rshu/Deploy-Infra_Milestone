# Deploy_Infra_Milestone
Final Milestone of Devops Class (CSC 519)

### Team members:
* Nawshin Tabassum - ntabass
* Joymallya Chakraborty - jchakra
* Rui Shu - rshu
* Pingping Chen - pchen23


### Contribution: 
ntabass,jchakra worked on the deployment of iTrust and checbox.io, hook to deploy iTrust and the featureflags. rshu and pchen23 worked on the special milestone(monitoring) and infrastructure upgrade, also work on the [final demo slides](https://docs.google.com/presentation/d/1H3ndPuoWWungxkaeSIxTwQZ05RzQTUDxUoOP-JOssxA/edit#slide=id.g35f391192_00).


#### Instructions to run the project
To run the project, first do the following:

`git clone https://github.ncsu.edu/jchakra/Deploy_Infra_Milestone.git `

`cd Deploy_Infra_Milestone`

After setting up the key, you can run the following command:

`ansible-playbook playbook.yml -i inventory.ini`

This will deploy iTrust and checkbox.io on two separate aws ec2 instances. 



### Deployment Components

There are 2 role modules for iTrust deployment - 1> iTrust 2> tomcat 3> deploy

There is a single role to deploy checbox.io - > checkbox


### Hook 

We have created a [post-receive](https://github.ncsu.edu/jchakra/Deploy_Infra_Milestone/blob/master/post-receive) hook in the jenkins server to automatically deploy iTrust after making a push to production. 

### Feature Flag

We have changed iTrust source code in the iTrust2/src/main/java/edu/ncsu/csc/itrust2/controllers/api/APIHospitalController.java and used redis.clients.jedis.Jedis for feature flag implemenation.

From the redis server if we set the flag value `true` (set test "true") then admin user is allowed to create a new hospital otherwise (set test "false") not allowed to create.


For this change, we had to add `jedis` dependency in pom.xml file.




### Infrastructure Upgrade

**More details can be found in github repository [checkbox.io]**(https://github.com/rshu/checkbox.io)

In this part, we use the Nomad cluster upgrade our infrastructure, i.e., we run three markdown microservices in Docker containers in each Nomad client, and run checkbox.io in Nomad server. The checkbox.io will use our mircoservice instead of local markdown render API.

We have created and pushed Docker images into Docker Hub, including checkbox.io and markdown microservice. 

Docker Hub:

[checkbox.io](https://cloud.docker.com/u/tamitito/repository/docker/tamitito/checkbox) - The checkbox image

[markdown](https://cloud.docker.com/u/tamitito/repository/docker/tamitito/marqdown) - The markdown microservice image

(Note: tamitito is Docker account of Rui Shu)

* Check nomad server and client status:

```
nomad server members
nomad node status
```

Nomad server creates job to Nomad client:

* Run markdown job

```
nomad job run checkbox.nomad
```

* Check job status

```
nomad status Checkbox.Job
```

* If we have updated the job

```
nomad job plan checkbox.nomad
```

* Run the modified job again with index

```
nomad job run -check-index 512 checkbox.nomad
```

* Stop a job

```
nomad job stop Checkbox.Job
```

Since we have commited and run the job in Nomad clients, we check whether our markdown service is available or not. We use postman:

![alt text](https://github.ncsu.edu/jchakra/Deploy_Infra_Milestone/blob/master/postman.png)

This shows that our microservices are available.

We have updated the server_callMicroservice.js code. In this case, the checkbox.io will use microservice of the client if the client host is available (i.e., not in shutdown status), instead of using local API. Thus, the service is still available after truning off the nodes.

### Special Component - Monitoring/Analysis

**More details can be found in github repository [checkbox.io]**(https://github.com/rshu/checkbox.io)

In this part, we use docker-compose to manage tools to monitoring system-level metrics of docker containers and the host. 


Major Tools:

* cAdvisor (container metrics collector)
* NodeExporter (host metric collector)
* Prometheus (metrics database)
* Grafana (Visualization tool)


If we start checkbox.io in Docker container, run

```
sudo docker run -it -p 80:80 --env MONGO_PORT=27017 --env MONGO_IP=127.0.0.1 --env MONGO_USER=myUserAdmin --env MONGO_PASSWORD=abc123 --env MAIL_USER=csc519s19.rshu --env MAIL_SSWORD=devops2019! --env MAIL_SMTP=smtp.gmail.com --env APP_PORT=3002 --network host tamitito/checkbox
```

Steps:

* Start monitoring services

```
ADMIN_USER=admin ADMIN_PASSWORD=admin sudo docker-compose up -d
```

* Configure Grafana visualiation

```
http://34.239.228.117:3000
Login: admin
Password: admin
```
* Monitoring Screenshot

![alt text](https://github.ncsu.edu/jchakra/Deploy_Infra_Milestone/blob/master/Monitoring.png)

Screencast for this milestone is [here](https://drive.google.com/open?id=1yuvSECHQlkArbd-cXF3Tr6ImkAZY-Jrj)
Final demo link is [here](https://drive.google.com/file/d/142DNp7x9AYIEQZlKn1wPWHgzn963WvW4/view?usp=sharing)
