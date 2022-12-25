# OpenFaaS

**Serverless Architecture with OpenFaaS**


<p align="center">
  <img src="https://user-images.githubusercontent.com/38450758/209446422-ab7db3a2-66b4-4af6-8031-394848dc8169.png">
</p>

<br>
<br>

**If you want to deploy OpenFaaS in a kubernetes cluster then follow below link** 
<br>
**Create kubernetes cluster with K3D** [ Click here](https://github.com/santoshcisco/k3d)

<br>

**<p>Deploy the OpenFaaS</p>**

•	OpenFaaS is a Serverless Framework.

•	The term serverless gained much popularity. However, many people are still unsure what it is for, and how it can help them build applications faster than traditional approaches

•	The Community Edition (CE) of OpenFaaS uses our legacy scaling technology, which is meant for development only

•	OpenFaaS Pro Edition gives auto scaling feature. If you are looking for Production, then go for OpenFaaS Pro Edition



**What is PLONK?**

The PLONK stack also requires components like a Container Registry and Container Runtime such as Docker or containerd. You can then build on top of it by introducing new projects, such as OpenFaaS Cloud, GitHub and GitLab
 

**Infrastructure Layer:**

1.	Docker provides a packaging image format.

2.	A container registry holds each version of our function, we can version it and benefit from distribution.

3.	Kubernetes provides a control plane to run our functions, including fail-over, high availability (HA), scale-out and secret management.


**Application Layer:**

1.	The OpenFaaS Gateway is conceptually similar to a reverse proxy like Nginx, Kong or Caddy; however, its job is to expose and manage containers running our functions, rather than REST APIs. It does have its own REST API and can be automated. The most popular client for the OpenFaaS Gateway is the CLI (faas-cli), followed by the UI.

2.	Prometheus is a CNCF project which provides metrics and instrumentation. It can be used to help inform autoscaling decisions, along with understanding the health and performance of OpenFaaS and our set of functions.

3.	NATS is another CNCF project which, when combined with OpenFaaS, provides a way to queue up requests and defer them for later execution.

**GitOps/laac Layer:**

1.	OpenFaaS Cloud orchestrates all the lower layers to provide a multi-user dashboard with authentication, built-in CI/CD and integration to GitHub or GitLab.

2.	GitHub can be used to build and deploy functions using its Travis integration, or its own GitHub actions and container registry.

3.	GitLab comes with a full suite of GitOps-like tooling that can be used to create build and deployment pipelines directly into OpenFaaS.
 

**Once we configured Kubernetes and explored both Helm and arkade, let’s go ahead and deploy OpenFaaS to our cluster. Use arkade to install OpenFaaS. If you do not have Helm3 installed**


**Get the faas-cli**

    $ curl -SLsf https://cli.openfaas.com | sudo sh
    $ arkade install openfaas


**After the installation has completed, we’ll receive the commands we need to run, to log in and access the OpenFaaS Gateway service in Kubernetes**

**Forward the gateway to our machine**

    $ kubectl rollout status -n openfaas deploy/gateway
    $ kubectl port-forward -n openfaas svc/gateway 8080:8080 &


**If basic auth is enabled, we can now log into your gateway**

    PASSWORD=$(kubectl get secret -n openfaas basic-auth -o

    jsonpath="{.data.basic-auth-password}" | base64 --decode; echo)

    echo -n $PASSWORD | faas-cli login --username admin --password-stdin
    
    faas-cli store deploy figlet
    
    faas-cli list


**We can get this message again at any time with arkade info openfaas. Let’s break it down, line by line**

The kubectl rollout status command checks that all the containers in the core OpenFaaS stack have started and are healthy.
The kubectl port-forward command securely forwards a connection to the OpenFaaS Gateway service within your cluster to your laptop on port 8080. It will remain open for as long as the process is running, so if it appears to be inaccessible later, just run this command again.
The faas-cli login command and preceding line populate the PASSWORD environment variable. You can use this to get the password to open the UI at any time.
We then have faas-cli store deploy figlet and faas-cli list. The first command deploys an ASCII generator function from the Function Store and the second command lists the deployed functions, you should see figlet listed.


**We will also find the PLONK stack components deployed, such as Prometheus and NATS. We can see them in the openfaas Kubernetes namespace:**


**List several NameSpaces in the cluster**
    
    $ kubectl get all --all-namespaces
 

**Check the OpenFaaS deployment**
    
    $ kubectl get deploy -n openfaas
 

**To see what are the pods deployed in openfaas namespace**
    
    $ kubectl get pods -o wide -n openfaas
 

**To see what are the services deployed in openfaas namespace**
    
    $ kubectl get svc -n openfaas
 

**Once we deploy a Kubernetes cluster and OpenFaaS then we can create a Function and deploy it on cluster**


**To see the faas version. We will get the output like below**

    $ faas-cli version

 

**Note:** Before we create a function, we need to know where it will be stored as a container image. This could be a local container registry, a registry hosted by your cloud provider, or a managed registry. We will be using Docker Hub in this lab.


**Create a sample function named hello-openfaas in python language**


**Before starting this lab, create a new folder for your sample function**

    $ mkdir test && cd test


**There are two ways to create a new function**

•	scaffold a function using a built-in or community code template (default)

•	take an existing binary and use it as your function (advanced)


**Pull the template from Templates**
    
    $ faas-cli template pull


**List out the languages by using below command**

    $ faas-cli template


**Note:** If we would prefer to use Python 2.7 instead of Python 3 then swap faas-cli new --lang python3 for faas-cli new --lang python**


**We will create a hello world sample function in Python language**

    $ faas-cli new  - It is used for create a new function

    $ faas-cli new --lang python3 hello-openfaas --prefix="<docker-username>"


**This will create three files and a directory as we see below**

![image](https://user-images.githubusercontent.com/38450758/209444837-65743740-389b-474b-a864-3b4e515251b5.png)


**This hello-openfaas.yml file is used to configure the CLI for building, pushing, and deploying our function**

**Note:** If we deploy a function on local Kubernetes cluster or on cloud, we need to override the default gateway URL of 127.0.0.1:8080 with an environmental 

Variable: OPENFAAS_PREFIX

OPENFAAS_URL=127.0.0.1:8080


**Below is the hello-openfaas.yml file content**

![image](https://user-images.githubusercontent.com/38450758/209444893-fe89b7cd-5a0d-4b3b-9ed1-e8f52f5270ae.png)


**Remember that the gateway URL can be overridden in the YAML file (by editing the gateway: value under provider:) or on the CLI (by using --gateway or setting the OPENFAAS_URL environment variable)**


**Below is the handler.py file content**

![image](https://user-images.githubusercontent.com/38450758/209445016-b2c870e3-7cce-47fc-9d74-71c08ba042dc.png)


**Build, Push and Deploy using below commands**

**Build the image**

    $ faas-cli build -f Yaml filename - build an image into the local Docker library

![image](https://user-images.githubusercontent.com/38450758/209445193-19675433-5bf0-4d30-95ce-21a803d125c8.png)


**Push the image to local/remote repo as per our configuration**

    $ faas-cli push - f Yaml filename - push that image to a remote container registry

![image](https://user-images.githubusercontent.com/38450758/209445207-9c589400-b2e6-4863-96b6-27ef8029d4d2.png)


**Deploy the image into kubernetes cluster**

    $ faas-cli deploy - f Yaml filename - deploy our function into a Kubernetes cluster

![image](https://user-images.githubusercontent.com/38450758/209445231-421b0be6-e678-49ba-b27c-83ea85ecd3b5.png)


**We can Automate the process of Build, Push and Deploy using single below command**

    $ faas-cli up -f  Yaml filename


**Note:** Please make sure that you have logged in to docker registry with docker login command before running above command.


**List, inspect, invoke, and troubleshoot your functions**

•	faas-cli list

![image](https://user-images.githubusercontent.com/38450758/209445240-d71f8068-c01a-435d-9151-9e0ebb359d1f.png)

 
•	faas-cli describe

•	faas-cli invoke

•	faas-cli logs


**Authenticate to the CLI, and create secrets for your functions**

•	faas-cli login

•	faas-cli secret

<br>

**Now Scaling it**

    $ kubectl autoscale deploy/hello-openfaas -n openfaas-fn --min=2 --max=5


**We can see the number of deployments details by using below command**

    $ Kubectl get deploy -n openfaas


**We can see the openfaas service details by using below command**

    $ kubectl get svc -n openfaas


**We can see the deployment pods details by using below command**

    $ kubectl get pods -n openfaas 


**We can see check that function is responding or not**

    $ curl -v http://127.0.0.1:8080/function/hello-openfaas/ping
 
 ![image](https://user-images.githubusercontent.com/38450758/209445272-cd5bb299-d24d-4340-9d7c-3cff30169827.png)

 

**We can see the deployment external gateway details by using below command**

    $ kubectl get svc -n openfass -o wide


**Now clone a tester application written in Go and deploy it to our cluster**

    $ git clone https://github.com/alexellis/echo-fn && cd echo-fn && faas-cli template store pull golang-http && faas-cli deploy --label com.openfaas.scale.max=10 --label com.openfaas.scale.min=1


**Deploy a function from marketplace using OpenFaaS UI**

Run below command for password and use this password for logging into OpenFaaS UI

    $ echo "OpenFaaS admin password: $PASSWORD"
    

**To open the UI navigate to http://127.0.0.1:8080 in a browser. There is no need to worry that this is using HTTP (plaintext) instead of HTTPS because the previous port-forwarding command runs over an encrypted connection**
 
![image](https://user-images.githubusercontent.com/38450758/209445308-38ede202-dcd1-4956-ad00-0cb7cee6f6ba.png)


**When prompted, the user is admin and the password is the value from above. The password can be changed at any time. A commercial solution using OpenID Connect is also available separately**


**Click on Deploy New Function then enter the function name in the search. Once we found, select that function then click on Deploy as shown in the below image. It will pull from the marketplace**
 
![image](https://user-images.githubusercontent.com/38450758/209445343-68cec21f-acce-46c2-b156-de7feb450638.png)



**Select the same function from available list then click on INVOKE**
 
![image](https://user-images.githubusercontent.com/38450758/209445358-7b6b38e9-5166-4605-a6c8-803fd26786d8.png)



**Once we click on INVOKE, we can see the output of the function. And we can see the Response Status, Replicas, Image repo URL**
 
![image](https://user-images.githubusercontent.com/38450758/209445404-cc28e4f7-db71-467d-aa32-d62895382edb.png)



**Enter deployed function name in the search then select the function then click INVOKE. Will see the output**
 
![image](https://user-images.githubusercontent.com/38450758/209445418-638a5bc4-4315-44b8-ae97-d1841053ced1.png)


**We can see the output of existing hello-openfaas function deployed from CLI**

![image](https://user-images.githubusercontent.com/38450758/209445451-352027b8-6c61-41d8-adcd-cc9b6491e8f0.png)

 
<br>


**Monitoring with Prometheus & Grafana**

<br>

![image](https://user-images.githubusercontent.com/38450758/209445481-984665fc-be00-4620-ac41-2371fce68364.png)

<br>
   
**Exploring the Metrics**

Prometheus is a time-series database used by OpenFaaS to track the requests per second being sent to an individual function along with the success and failure of those requests and their latency. This can be referred to as RED metrics or - rate, error, and duration.

Prometheus does not come with any kind of authentication, so we keep it hidden by default. Port-forward Prometheus to your local computer:

    $ kubectl port-forward deployment/prometheus 9090:9090 -n openfaas &

Now open its UI using http://127.0.0.1:9090
 
![image](https://user-images.githubusercontent.com/38450758/209445505-c29752ea-d4fb-476e-8937-fde98c89ce34.png)

<br>

**Monitoring the Functions with a Grafana Dashboard**

OpenFaaS tracks metrics on our functions automatically using Prometheus. The metrics can be turned into a useful dashboard with free and Open-Source tools like Grafana.

<br>

**Deploy Grafana in OpenFaaS namespace**

    $ kubectl -n openfaas run --image=stefanprodan/faas-grafana:4.6.3 --port=3000 grafana


**Expose Grafana with a NodePort**

    $ kubectl -n openfaas expose pod grafana --type=NodePort --name=grafana


**Find Grafana node port address**

    $ GRAFANA_PORT=$(kubectl -n openfaas get svc grafana -o jsonpath="{.spec.ports[0].nodePort}")

    $ GRAFANA_URL=http://IP_ADDRESS:$GRAFANA_PORT/dashboard/db/openfaas

Where IP_ADDRESS is our corresponding IP for Kubernetes.


Or 


We may run this port-forwarding command to be able to access Grafana on http://127.0.0.1:3000:

    $ kubectl port-forward pod/grafana 3000:3000 -n openfaas

If you're using Kubernetes 1.17 or older, use deploy/grafana instead of pod/ in the command above.

After the service has been created open Grafana in your browser, login with username admin password admin and navigate to the pre-made OpenFaaS dashboard at $GRAFANA_URL.

<br>

**We can observe in below screenshots, how the load/traffic is increasing**  
 
<br>

![image](https://user-images.githubusercontent.com/38450758/209445657-d190b187-229c-4772-baa3-495310ce6278.png)

![image](https://user-images.githubusercontent.com/38450758/209445715-28ed1532-b680-4ddc-b878-9bb7748f3f79.png)

![image](https://user-images.githubusercontent.com/38450758/209445768-0135b6d2-43d5-42a0-8cc5-ac4983c62e07.png)

![image](https://user-images.githubusercontent.com/38450758/209445809-7d33efbd-2476-41cd-b91c-4dc43f97539f.png)

![image](https://user-images.githubusercontent.com/38450758/209445826-e4e217da-f652-4c92-ab17-b5ec29e42650.png)

<br>

**Auto-Scaling Work**

We can read more on how auto-scaling works in the OpenFaaS documentation.

OpenFaaS can scale functions down to zero automatically when they are idle using its faas-idler component. You can read about how to enable it in the OpenFaaS documentation. However, it is turned off by default to avoid premature optimizations.

For this example, we will manually scale down the function, and then invoke it again.

Open four terminal windows and type in the following commands, one into each terminal, so that we can monitor what happens when you invoke the function that is scaled to zero.

<br>

**Show the pods that are removed, and then created again**

    $ kubectl get pods -n openfaas-fn -w


**Show the container image being pulled and a pod being scheduled**

    $ kubectl get events --sort-by=.metadata.creationTimestamp -n openfaas-fn -w


**Show the gateway finding out how many replicas are present, and then blocking the request until the desired state is met**

    $ kubectl logs -n openfaas deploy/gateway -c gateway -f


**Scale the function down manually, then wait a few seconds**

    $ kubectl scale deployment/go-echo -n openfaas-fn -replicas=0; sleep 10


**Now we are ready to invoke the function again**

    $ curl -d "hi" http://127.0.0.1:8080/function/go-echo
    
![image](https://user-images.githubusercontent.com/38450758/209445847-07773445-9bb4-44f5-8450-40da53c39d67.png)

<br>
 
**Scale from Zero**


**Kubernetes is an event-driven system which relies on events being propagated throughout its cluster when actions take place. In this example, the following happened**

•	The gateway saw no pods were present in openfaas-fn for the function.

•	The gateway asked Kubernetes to scale up to 1.

•	The gateway called GetReplicas in a loop.

•	Kubernetes scheduled the pod for the function.

•	The Kubernetes node started to pull the image from Docker Hub.

•	Kubernetes went into a loop trying to call the function’s HTTP health endpoint, to see if it was ready to serve traffic.

•	Kubernetes marked the endpoint as ready for traffic.

•	The gateway stopped blocking the request and let it through to the function and we got the result.


Kubernetes is also called “eventually consistent” and requires some tuning to get the cold-start we saw above under 1-2 seconds. You will find more about this in the 

OpenFaaS documentation. Ultimately, you can avoid all cold-starts by having some minimum amount of available replicas i.e. 1-5. You can also run functions asynchronously, which will hide any scaling-up from the user.

Another option we mentioned earlier in the course was faasd, which runs on a single host, and eliminates the eventually-consistent nature of a cluster and can cold-start in as little as 0.19s.
<br>
<br>
<br>
<br>
![image](https://user-images.githubusercontent.com/38450758/209445873-33f9482a-8a22-428b-b6c5-9a284447d188.png)


<p align="center">
  <img src="https://user-images.githubusercontent.com/38450758/209445882-a0b76e6d-2f08-4c4f-bcfc-00523939125d.png">
</p>

