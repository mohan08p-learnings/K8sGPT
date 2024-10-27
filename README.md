# K8sGPT
K8sGPT

#### ChatGPT for your K8s cluster

K8sGPT gives Kubernetes SRE superpowers to everyone

#### Installation

Install K8sGPT on your machine with the following commands on MacOS:

``` 
brew tap k8sgpt-ai/k8sgpt
brew install k8sgpt
```

RPM-based installation (RedHat/CentOS/Fedora)

```
curl -LO https://github.com/k8sgpt-ai/k8sgpt/releases/download/v0.3.24/k8sgpt_amd64.rpm
sudo rpm -ivh -i k8sgpt_amd64.rpm
```

DEB-based installation (Ubuntu/Debian)

```
curl -LO https://github.com/k8sgpt-ai/k8sgpt/releases/download/v0.3.24/k8sgpt_amd64.deb
sudo dpkg -i k8sgpt_amd64.deb
```

#### Verify Installation

```
mohanpawar@MohanPawars-MacBook-Pro K8sGPT % k8sgpt version
k8sgpt: 0.3.41 (Homebrew), built at: 2024-09-23T05:33:03Z
```

To view all the commands provided by K8sGPT, used the `--help` flag:

```
k8sgpt --help
```

You can see an overview of the different commands also on the [documentation](https://docs.k8sgpt.ai/).

#### Prerequisites

The prerequisites to follow the next sections is to have an [OpneAI](https://openai.com/?ref=anaisurl.com) account and a running Kubernetes cluster; any cluster, such as microk8s or minikube will be sufficient.

Once you have the OpneAI account, you want to go to the following site to generate a new API key https://platform.openai.com/account/api-keys

Alternatively, you can run the following command and K8sGPT will open the same site in your default Browser:

```
k8sgpt generate
```

This key is needed for K8sGPT to interact with OpenAI. Authorise K8sGPT with the newly created API key/token:

```
k8sgpt auth add openai
Enter openai Key: openai added to the AI backend provider list
```

You can list your backends with the following command:

```
k8sgpt auth list
```

Again, [K8sGPT documentation](https://docs.k8sgpt.ai/reference/providers/backend/?ref=anaisurl.com) provides further information on the different AI backends available.

From the K8s cluster I have minikube cluster installed on my workstation as below,

```
mohanpawar@MohanPawars-MacBook-Pro K8sGPT % kubectl get nodes
NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   83d   v1.30.0
```

Next, we will install a deployment into our K8s cluster. The pod will go into `CrashLoopBackOff`. 

```
mohanpawar@MohanPawars-MacBook-Pro K8sGPT % kubectl apply -f nginx-deploy.yaml    
deployment.apps/nginx-deployment created

mohanpawar@MohanPawars-MacBook-Pro K8sGPT % kubectl get po                        
NAME                               READY   STATUS             RESTARTS      AGE
myapp1-mychart1-86bd77bbfd-b4vm4   1/1     Running            5 (63m ago)   82d
nginx-deployment-6dd5d548f-2kxk7   0/1     CrashLoopBackOff   1 (11s ago)   15s
nginx-deployment-6dd5d548f-jnttz   0/1     CrashLoopBackOff   1 (11s ago)   15s
nginx-deployment-6dd5d548f-znw88   0/1     CrashLoopBackOff   1 (11s ago)   15s
```

If we look at the events for one of the pods, we will not become much smarter:

```
mohanpawar@MohanPawars-MacBook-Pro K8sGPT % kubectl describe po nginx-deployment-6dd5d548f-2kxk7
Name:             nginx-deployment-6dd5d548f-2kxk7
Namespace:        default
...
...
Events:
  Type     Reason     Age                 From               Message
  ----     ------     ----                ----               -------
  Normal   Scheduled  101s                default-scheduler  Successfully assigned default/nginx-deployment-6dd5d548f-2kxk7 to minikube
  Normal   Pulled     14s (x5 over 101s)  kubelet            Container image "nginx:1.14.2" already present on machine
  Normal   Created    14s (x5 over 101s)  kubelet            Created container nginx
  Normal   Started    14s (x5 over 100s)  kubelet            Started container nginx
  Warning  BackOff    1s (x9 over 98s)    kubelet            Back-off restarting failed container nginx in pod nginx-deployment-6dd5d548f-2kxk7_default(1ef0c450-4628-4602-bbec-8d124ee4f4fe)
```

So what we can do instead to access more details on why these pods are erroring, we can run a K8sGPT command:

```
mohanpawar@MohanPawars-MacBook-Pro K8sGPT % k8sgpt analyse
AI Provider: AI not used; --explain not set

0: Pod default/nginx-deployment-6dd5d548f-2kxk7(Deployment/nginx-deployment)
- Error: the last termination reason is Error container=nginx pod=nginx-deployment-6dd5d548f-2kxk7

1: Pod default/nginx-deployment-6dd5d548f-jnttz(Deployment/nginx-deployment)
- Error: the last termination reason is Error container=nginx pod=nginx-deployment-6dd5d548f-jnttz

2: Pod default/nginx-deployment-6dd5d548f-znw88(Deployment/nginx-deployment)
- Error: the last termination reason is Error container=nginx pod=nginx-deployment-6dd5d548f-znw88
```

To receive further information as well as recommendations on how to fix the issues, we can use the --explain flag:

```
mohanpawar@MohanPawars-MacBook-Pro K8sGPT % k8sgpt analyse --explain
 100% |██████████████████████████████████████████████████████████████████████████| (2/2, 18 it/hr)           
AI Provider: localai

0: Pod default/nginx-deployment-6dd5d548f-2kxk7(Deployment/nginx-deployment)
- Error: the last termination reason is Error container=nginx pod=nginx-deployment-6dd5d548f-2kxk7
Error: The last termination reason is Error container=nginx pod=nginx-deployment-6dd5d548f-2kxk7.

Solution: 
1. Check nginx container logs for errors.
2. Verify if the nginx configuration file is correct and not causing any issues.
3. Ensure that the deployment is correctly configured and has sufficient resources (CPU, memory).
4. Try restarting the pod or rolling back to a previous version if necessary.
1: Pod default/nginx-deployment-6dd5d548f-znw88(Deployment/nginx-deployment)
- Error: the last termination reason is Error container=nginx pod=nginx-deployment-6dd5d548f-znw88
Error: The pod "nginx-deployment-6dd5d548f-znw88" terminated with an Error reason.

Solution: 
1. Check the container logs for errors using `kubectl logs -n <namespace> nginx-deployment-6dd5d548f-znw88`
2. Verify the container's configuration and environment variables.
3. If the issue persists, try restarting the pod or rolling back to a previous version if it's a deployment.
```

#### Integration

The value of most tools in the cloud native ecosystem originates from how well they integrate with other tools.

At the time of writing, K8sGPT offers easy integration with observability tools, such as Gafana and Prometheus. Additionally, it is possible to write plugins for K8sGPT. The first plugin that was provided by the core maintainers is for Trivy, an all in one, cloud native security scanner.

You can list all available integration with the following command:

```
mohanpawar@MohanPawars-MacBook-Pro K8sGPT % k8sgpt integration list
Active:
Unused: 
> trivy
> prometheus
> aws
> keda
> kyverno
```

Next, let's activate `trivy` integration:

```
mohanpawar@MohanPawars-MacBook-Pro K8sGPT % k8sgpt integration activate trivy

2024/10/27 19:23:44 creating 1 resource(s)
2024/10/27 19:23:45 beginning wait for 12 resources with timeout of 1m0s
2024/10/27 19:23:47 creating 21 resource(s)
2024/10/27 19:23:48 release installed successfully: trivy-operator-k8sgpt/trivy-operator-0.24.1
Activated integration trivy

```

This installed the Trivy operator inside your cluster.

Once the integration is activated, we can use the Vulnerability Reports created by Trivy as part of our K8sGPT analysis thought K8sGPT filters

```
mohanpawar@MohanPawars-MacBook-Pro K8sGPT % k8sgpt filters list
Active: 
> Pod
> Deployment
> PersistentVolumeClaim
> MutatingWebhookConfiguration
> ConfigAuditReport (integration)
> ReplicaSet
> ValidatingWebhookConfiguration
> Ingress
> StatefulSet
> Node
> VulnerabilityReport (integration)
> CronJob
> Service
Unused: 
> HorizontalPodAutoScaler
> PodDisruptionBudget
> NetworkPolicy
> Log
> GatewayClass
> Gateway
> HTTPRoute
```

The filters correspond to specific analyzers in the k8sgpt code. Analysers only look at the relevant information e.g. the most critical vulnerabilities.

To use the VulnerabilityReport filter, use the following command:

```
mohanpawar@MohanPawars-MacBook-Pro K8sGPT % k8sgpt analyse --filter=VulnerabilityReport
AI Provider: AI not used; --explain not set

No problems detected
```

Similar to before, we can also ask K8sGPT to provide further explanations on the scan:

```
mohanpawar@MohanPawars-MacBook-Pro K8sGPT % k8sgpt analyse --explain --filter=VulnerabilityReport
AI Provider: localai

No problems detected
```