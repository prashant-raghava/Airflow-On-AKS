# Install Airflow on AKS

Build and execute a multi-node Airflow on Azure kubernets cluster (AKS).

### Airflow
Apache Airflow, also known simply as Airflow, is a system that enables the creation, scheduling, and monitoring of workflows in a programmatic way. By defining workflows as code, they become easier to maintain, can be versioned, tested, and foster collaboration.

Airflow is used to create workflows as directed acyclic graphs (DAGs) of tasks. The Airflow scheduler carries out your tasks on a group of workers while adhering to the defined dependencies. Comprehensive command line tools make it straightforward to carry out complex operations on DAGs. The user-friendly interface makes it simple to visualize pipelines that are running in production, monitor their progress, and troubleshoot if necessary.


### Prerequisites

[Documentation](https://airflow.apache.org/docs/apache-airflow/stable/index.html)
prior knowledge on Airflow as well as Kubernetes is a must. Other items are given below:

- In this article I will not cover how to create Azure AKS cluster. So make sure you have Azure AKS cluster to deploy Airflow.
- Create `Azure storage account` and fileshare.
- Create a `Namespace`.
- Make sure you have `az-cli` installed on your window machine

### Setup Helm and kubectl
I am using windows so here are the steps:

* **Install choco**: Run the follwoing command on command prompt:

```
@"%SystemRoot%\System32\WindowsPowerShell\v1.0\powershell.exe" -NoProfile -InputFormat None -ExecutionPolicy Bypass -Command "[System.Net.ServicePointManager]::SecurityProtocol = 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))" && SET "PATH=%PATH%;%ALLUSERSPROFILE%\chocolatey\bin"
```
```
choco install kubernetes-helm
``` 
Verify your version of `helm`
```
helm version
``` 
Now you are all set with `helm`.

* **Install kubectl on Windows**: Execute the following command.
```
choco install kubernetes-helm
``` 
Test to ensure the version you installed is up-to-date:

```
kubectl version --client
```
* **Add Helm repositories**: We are going to add official Airflow repo.
```
helm repo add apache-airflow https://airflow.apache.org
```
If we want to install Airflow for lower environment where we are just using inbuilt features provided by official Airflow realease. please execute below command and it will create all necessory infrastructure for you:
```
helm upgrade --install airflow apache-airflow/airflow --namespace airflow --create-namespace
```
Above command deploys Airflow on the Kubernetes cluster in the default configuration.

To uninstall/delete the airflow deployment:
```
helm delete airflow --namespace airflow
```
Above command removes all the Kubernetes components associated with the chart and deletes the release.

## Output
```
Release "airflow" does not exist. Installing it now.
NAME: airflow
LAST DEPLOYED: Wed Jul 26 17:39:15 2023
NAMESPACE: airflow
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
Thank you for installing Apache Airflow 2.6.2!

Your release is named airflow.
You can now access your dashboard(s) by executing the following command(s) and visiting the corresponding port at localhost in your browser:

Airflow Webserver:     kubectl port-forward svc/airflow-webserver 8080:8080 --namespace airflow
Default Webserver (Airflow UI) Login credentials:
    username: admin
    password: admin
Default Postgres connection credentials:
    username: postgres
    password: postgres
    port: 5432

You can get Fernet Key value by running the following:

    echo Fernet Key: $(kubectl get secret --namespace airflow airflow-fernet-key -o jsonpath="{.data.fernet-key}" | base64 --decode)

###########################################################
#  WARNING: You should set a static webserver secret key  #
###########################################################

You are using a dynamically generated webserver secret key, which can lead to
unnecessary restarts of your Airflow components.

Information on how to set a static webserver secret key can be found here:
https://airflow.apache.org/docs/helm-chart/stable/production-guide.html#webserver-secret-key
```
* Check the helm list
```
C:\Users\Prashant>helm ls -n airflow
NAME    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
airflow airflow         1               2023-07-26 17:39:15.567555 -0400 EDT    deployed        airflow-1.10.0  2.6.2
```
* Default pods installed will be shown below:
```
C:\Users\Prashant>kubectl get pods -n airflow
NAME                                 READY   STATUS    RESTARTS   AGE
airflow-postgresql-0                 1/1     Running   0          10m
airflow-redis-0                      1/1     Running   0          10m
airflow-scheduler-7689f9f48d-lgl29   2/2     Running   0          10m
airflow-statsd-67dbbcd9b-5rj79       1/1     Running   0          10m
airflow-triggerer-0                  2/2     Running   0          10m
airflow-webserver-59976594dc-sq5sp   1/1     Running   0          10m
airflow-worker-0                     2/2     Running   0          10m
```
* Check the logs of the service:
```
C:\Users\Prashant>kubectl logs airflow-scheduler-7689f9f48d-lgl29 -n airflow -c scheduler


  ____________       _____________
 ____    |__( )_________  __/__  /________      __
____  /| |_  /__  ___/_  /_ __  /_  __ \_ | /| / /
___  ___ |  / _  /   _  __/ _  / / /_/ /_ |/ |/ /
 _/_/  |_/_/  /_/    /_/    /_/  \____/____/|__/
[2023-07-26T21:40:22.945+0000] {executor_loader.py:114} INFO - Loaded executor: CeleryExecutor
[2023-07-26T21:40:22.987+0000] {scheduler_job_runner.py:788} INFO - Starting the scheduler
[2023-07-26T21:40:22.988+0000] {scheduler_job_runner.py:795} INFO - Processing each file at most -1 times
[2023-07-26T21:40:22.992+0000] {manager.py:165} INFO - Launched DagFileProcessorManager with pid: 16
[2023-07-26T21:40:22.994+0000] {scheduler_job_runner.py:1553} INFO - Resetting orphaned tasks for active dag runs
[2023-07-26T21:40:23.007+0000] {settings.py:60} INFO - Configured default timezone Timezone('UTC')
[2023-07-26T21:40:23.045+0000] {settings.py:499} INFO - Loaded airflow_local_settings from /opt/airflow/config/airflow_local_settings.py .
[2023-07-26T21:45:23.271+0000] {scheduler_job_runner.py:1553} INFO - Resetting orphaned tasks for active dag runs
[2023-07-26T21:50:23.310+0000] {scheduler_job_runner.py:1553} INFO - Resetting orphaned tasks for active dag runs
```
* **Access the Airflow Web UI**
```
kubectl port-forward svc/airflow-webserver 8080:8080 --namespace airflow
```
# Production ready setup:
To install a Production ready setup we need:
- External PostgreSQL DB for Airflow metadata.
- Load Balancer (`nginx-ingress`). 
- Attach Volume(Storage) for DAG's and Logs.    
- KubernetesExecutor - Uses Kubernetes pods to run the worker tasks

### Setup Database
Create PostgreSQL Database
```sql
postgres=# CREATE ROLE airflow LOGIN PASSWORD 'airflow';
postgres=# CREATE DATABASE airflow OWNER airflow ENCODING 'UTF8';
postgres=# GRANT ALL ON DATABASE airflow TO airflow;
```
**Things we need moving forward**
- **Namespace**: airflow (Create it in Azure)
- **Storage Account Name**: airflowstorage 
- **Share-Name**: airflow-logs and airflow-Dags 
- **account-key**: <*your account key*>
- **AKS Cluster** Airflow-AKS  

### Let's start:

- Login to your cloud account with AZ-CLI: Execute below commands on window machine `command prompt`. I am using [VS code editor](https://code.visualstudio.com/) to execute these commands:
```
az login --service-principal -u "a4a868b6-e943-3771-4121-1288d28ce522" -p "~187Q~G~24sulh6.9i4r8GYIs-qqrHH723#OoqbUI" --tenant "7654ece1-g80q-40we-9w23-2a7bewwdb345"
az aks get-credentials --resource-group my-resource-group-aks --name Airflow-AKS
```
If you have multiple context, execute below command to set the right context:
```
kubectl config use-context Airflow-AKS
```
Most of the interaction with kubelogin is around convert-kubeconfig subcommand which uses the input kubeconfig specified in --kubeconfig or KUBECONFIG environment variable to convert to the final kubeconfig in exec format based on specified login mode.
```
kubelogin convert-kubeconfig -l azurecli
```
Congratulation now we are ready to take off 🚀

#### Mount a Shared Persistent Volume: 
This method stores your DAGs and logs in a Kubernetes Persistent Volume Claim (PVC), you must use some external system to ensure this volume has your latest DAGs and logs. 
let's attach persistant volume and claim that volume: execute below comannd to configure this:
```
kubectl apply -f airflow-dags-storage-pv.yaml -n airflow
kubectl apply -f airflow-dags-storage-pvc.yaml -n airflow
kubectl apply -f airflow-logs-storage-pv.yaml -n airflow
kubectl apply -f airflow-logs-storage-pvc.yaml -n airflow
```
Check PVC:
```
PS C:\Users\Prashant> kubectl get pvc -n airflow
NAME           STATUS   VOLUME         CAPACITY   ACCESS MODES   STORAGECLASS   AGE
airflow-dags   Bound    airflow-dags   10Gi       RWX            default        1h
airflow-logs   Bound    airflow-logs   10Gi       RWX            default        1h
```
Check PV:
```
PS C:\Users\Prashant> kubectl get pv -n airflow 
NAME            CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                   STORAGECLASS   REASON   AGE
airflow-dags    10Gi       RWX            Retain           Bound    airflow/airflow-dags    default                 4d1h
airflow-logs    10Gi       RWX            Retain           Bound    airflow/airflow-logs    default                 4d1h
```
#### Deploying an ingress controller
The NGINX Ingress Controller plays an important role in Kubernetes by managing incoming traffic to the cluster. It acts as a reverse proxy and load balancer, routing incoming requests to the correct service based on the hostname and path of the request. This allows for easy management of the traffic routing rules, and it does not require changes to the application code.
To deploy an NGINX Ingress Controller using Helm, you can do the following: 
- Add the nginx-stable repository to helm
- Run helm repo update
- Deploy using the chart nginx-stable/nginx-ingress
Use the following commands to setup nginx ingress: 

```
> helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx helm repo update
> helm install ingress-nginx ingress-nginx/ingress-nginx -n ingress-nginx --create-namespace
> helm repo update
```
- Validate that NGINX is Running
Make sure the NGINX Ingress controller is running.
```
kubectl get all -n ingress-nginx
```
- Exposing Services using NGINX Ingress Controller
Now that an ingress controller is running in the cluster, you will need to create services that leverage it using either host, URI mapping, or even both.

Sample of a host-based service mapping through an ingress controller using the type “Ingress”:
Use `airflow-ingress.yaml` file to configure ingress controller. Execute following command to expose service:
```
> kubectl apply -f airflow-ingress.yaml -n airflow
```
**Note:** *Make sure to use same namespace where we are creating airflow. Ingress controller is running in ingress-nginx namespace.*
## Final Step
Now We will use `values.yaml` file to install airflow. In this file we will define everything we need to configure Airflow. You can get this file in respository.

Execute the following command to install Airflow on AKS:
```
helm upgrade --install airflow apache-airflow/airflow --namespace airflow -f values.yaml --debug --timeout 10m0s
```


