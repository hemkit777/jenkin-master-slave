Jenkin master is centOS image. It has latest jenkin version and also pre-installed following plugins and packges:

Plugins:
- ssh-slaves
- email-ext
- mailer
- slack
- htmlpublisher
- greenballs
- simple-theme-plugin
- kubernetes

Packages:
- maven
- kubectl
- docker
- JDK
- curl

Build Jenkin master and Slave containers using kubernetes:
---------------------------------------------------------
1) Run below command to build jenkin-master 
- kubelet apply -f jenkins-master/jenkins-deployment.yaml

Note: Before running command, please update hostPath in yaml. This will all master jobs and config stay in local host. And In event of crashing container, will not loss anything. I would also recommand to use nodeselector to run mster on dedicated either kube master or worker node so that when container will recreate it will locate all old config from hostPath.
 
          hostPath:
            path: /var/lib/docker/jenkin

2) Login to master using NodePort and container running hostIP:

http://{hostIP}:32366

3) First time access URL, will have to supply password. This password can be obtain from pod logs using below command:

jenkins logs  jenkins-master-5667fb8fbd-jg9ld

4) Once everythin is setup such as admin account, then update JNLP fixed port by going into Global Settings:

- Configure Global settings -> Agents -> Fixed -> Port No: 50001

5) Configure Kubernetes plugin
- Configure settings -> Add New Cloud 
   -- Name: Kubernetes 
   -- Kubernetes URL: { Cluster API URL } example: https://147.114.144.8:6443
      note: obtaining by running this command kubectl cluster-info 
   -- Click on TEST CONNECTION to make sure jenkin master can talk to kubernetes. Expected message 'Success'
   -- jenkin URL: {step-no:2}
   -- Click on Add POD Template for jenkin-slave setup 
   -- Name: jenkin-slave 
   -- Click on Add Container
   -- Name: jnlp
      Note: Please do not update name of container to other than 'jnlp'
   -- Docker image: hemkit777/jenkin-slave:1.0
   -- (OPtional) ADD Environment -> 
      Key: DOCKER_HOST
      Value: tcp://147.114.144.24:2376
   -- Add Volume -> Host Path Volume 
      Host Path: Your Local path of Scripts etc 
      Mount Point: { Patch of Container to be mounted above host Path }
- Apply and Save

Job Done. Now your job will run in Slave container. As soon as Job is finished, salve container will get distroy.
        
    

