# urllookup
url information service


####### Assumptions ############

1. The Machines where use exceute maven build  have docker engine installed.

2. We also have a highly available kubernetes cluster present, hence "kubectl" or "oc" (for opnshift)  command line tools will be present in machine . mostly master where user will exceute kubernetes resoource definition yaml file.
3. A local docker repo is already created and configured in pom.xml , hence images post build are getting directly pushed to local repo and same is accessible from machine where we execue maven build



##### Steps to Execute ############

1. Assuming we are building docker image and project locally in any external machine (other than kubernetes cluster machine )
go to   "..\urllookup\URLLookUpService\src\main\docker" and execute   :  mvn clean install docker:build -DpushImage
2. place "..\urllookup\Kubernetes yamls\url_service.yaml"   in any of Kubernetes cluster masters and go to file location and execute:

   kubectl create -f url_service.yaml

 The same can also be exceuted via kubernetes dashboard
 
 3. User can either monitor application via Kube Dashboard or execute :
 
    kubectl get svc -n test-env
    
    service with name "url_service" will be is running state and user have to node external port in range of 3000-5000.
    
 4. User can now hit :    <master_ip>:<external_port>/users_host$port/<url_to_inquire>
 
 
 
###### Considerations##################33

1. Here H2 in meomry DB is considered for demo purpose ,where a table with two columns "URL" & "STATUS" will be inserted. In reality sharded mongodb clusters will be used and not H2 , and this mongo DB cluster will not be a part of Kuberntes clluster.

Sharded mongo DB Clusters will solve the issues pointed in exercise :

  a. The size of the URL list could grow infinitely, how might you scale this beyond the memory capacity of the system? Bonus if you implement this.
  
  
2. Here we are using horizontal pod autoscaler and assume that on basis of transactions per minute analysis , we create 7 worker node kubernetes cluster with specific RAM, CPU and IO's for every node.

3. Every pod has been assigned a limit and request of CPU's here (assume we have cacluted that for initial phase of traffic) , as load increases the autoscaling works for pod ( Can explain how) and these pods will be scheduled anywhere in the cluster, monitoirng is defintely required once compute resources of all cluster is crossing a limit and appdynamics or new relic can be used here to generate an alert and on basis of where this kubernetes cluster is , user can also keep buffer nodes. The same works for update service , at appication layer we are doing autoscaling and the pod scales on basis of cpu , ram or no. of hits and the DB layer we are using sharded clusters.

 it solves the below issue of exercise:
   
   b. The number of requests may exceed the capacity of this system, how might you solve that? Bonus if you implement this.

   c. What are some strategies you might use to update the service with new URLs? Updates may be as much as 5 thousand URLs a day with updates arriving every minute

4. There will be lot of performaance optimisation issues , so there are various things available like liveness , rediness probes and network related configurations , what kind of load balancers we are using for our pod endpoints etc.

5. Creating basic service and not going deeper into springboot service's optimisation as , i wanted to present high level overview of my approach.
