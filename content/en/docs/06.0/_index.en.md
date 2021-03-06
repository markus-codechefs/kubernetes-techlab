---
title: "6. Scaling"
weight: 6
sectionnumber: 6
---

In this lab, we are going to show you how to scale applications on Kubernetes. Further, we show you how Kubernetes makes sure that the number of requested Pods is up and running and how an application can tell the platform that it is ready to receive requests.


## Task {{% param sectionnumber %}}.1: Scale the example application

Create a new Deployment in your namespace:
{{< onlyWhenNot mobi >}}

```bash
kubectl create deployment example-web-python --image=acend/example-web-python --namespace <namespace>
```

{{< /onlyWhenNot >}}
{{< onlyWhen mobi >}}

```bash
kubectl create deployment example-web-python --image=docker-registry.mobicorp.ch/puzzle/k8s/kurs/example-web-python --namespace <namespace>
```

{{< /onlyWhen >}}
If we want to scale our example application, we have to tell the Deployment that we want to have three running replicas instead of one.
Let's have a closer look at the existing ReplicaSet:

```bash
kubectl get replicasets --namespace <namespace>
```

Which will give you an output similar to this:

```
NAME                            DESIRED   CURRENT   READY   AGE
example-web-python-86d9d584f8   1         1         1       110s
```


Or for even more details:

```bash
kubectl get replicaset <replicaset> -o json --namespace <namespace>
```

The ReplicaSet shows how many instances of a Pod that are desired, current and ready.


Now we scale our application to three replicas:

```bash
kubectl scale deployment example-web-python --replicas=3 --namespace <namespace>
```

Check the number of desired, current and ready replicas:

```bash
kubectl get replicasets --namespace <namespace>
```

```
NAME                            DESIRED   CURRENT   READY   AGE
example-web-python-86d9d584f8   3         3         1       4m33s

```

Look at how many Pods there are:

```bash
kubectl get pods --namespace <namespace>
```

Which gives you an output similar to this:

```
NAME                                  READY   STATUS    RESTARTS   AGE
example-web-python-86d9d584f8-7vjcj   1/1     Running   0          5m2s
example-web-python-86d9d584f8-hbvlv   1/1     Running   0          31s
example-web-python-86d9d584f8-qg499   1/1     Running   0          31s

```

{{% alert title="Note" color="primary" %}}
Kubernetes even supports [autoscaling](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/).
{{% /alert %}}


## Check for uninterruptible Deployments

Now we create a new Service of type `NodePort`:


```bash
kubectl expose deployment example-web-python --type="NodePort" --name="example-web-python" --port=5000 --target-port=5000 --namespace <namespace>
```

Let's look at our Service. We should see all three corresponding Endpoints:

```bash
kubectl describe service example-web-python --namespace <namespace>
```

```
Name:                     example-web-python
Namespace:                philipona-scale
Labels:                   app=example-web-python
Annotations:              <none>
Selector:                 app=example-web-python
Type:                     NodePort
IP:                       10.39.245.205
Port:                     <unset>  5000/TCP
TargetPort:               5000/TCP
NodePort:                 <unset>  32193/TCP
Endpoints:                10.36.0.10:5000,10.36.0.11:5000,10.36.0.9:5000
Session Affinity:         None
External Traffic Policy:  Cluster
Events:
  Type    Reason                Age   From                Message
  ----    ------                ----  ----                -------
```


Scaling of Pods within a Service is fast, as Kubernetes simply creates a new container.

You can check the availability of your Service while you scale the number of replicas up and down in your browser: `http://<node-ip>:<node-port>`.

{{% alert title="Note" color="primary" %}}
Check the [previous lab](../05.0) on how to get the `<node-ip>` and `<node-port>` or how to use an Ingress instead.
{{% /alert %}}

Now, execute the corresponding loop command for your operating system in another console.

Linux:

```bash
URL=http://<node-ip>:<node-port>
while true; do sleep 1; curl -s $URL/pod/; date "+ TIME: %H:%M:%S,%3N"; done
```

Windows PowerShell:

```bash
while(1) {
  Start-Sleep -s 1
  Invoke-RestMethod http://[URL]/pod/
  Get-Date -Uformat "+ TIME: %H:%M:%S,%3N"
}
```

Scale from 3 replicas to 1 replica.
The output shows which Pod is still alive and is responding to requests:

```
POD: example-web-python-86d9d584f8-7vjcj TIME: 17:33:07,289
POD: example-web-python-86d9d584f8-7vjcj TIME: 17:33:08,357
POD: example-web-python-86d9d584f8-hbvlv TIME: 17:33:09,423
POD: example-web-python-86d9d584f8-7vjcj TIME: 17:33:10,494
POD: example-web-python-86d9d584f8-qg499 TIME: 17:33:11,559
POD: example-web-python-86d9d584f8-hbvlv TIME: 17:33:12,629
POD: example-web-python-86d9d584f8-qg499 TIME: 17:33:13,695
POD: example-web-python-86d9d584f8-hbvlv TIME: 17:33:14,771
POD: example-web-python-86d9d584f8-hbvlv TIME: 17:33:15,840
POD: example-web-python-86d9d584f8-7vjcj TIME: 17:33:16,912
POD: example-web-python-86d9d584f8-7vjcj TIME: 17:33:17,980
POD: example-web-python-86d9d584f8-7vjcj TIME: 17:33:19,051
POD: example-web-python-86d9d584f8-7vjcj TIME: 17:33:20,119
POD: example-web-python-86d9d584f8-7vjcj TIME: 17:33:21,182
POD: example-web-python-86d9d584f8-7vjcj TIME: 17:33:22,248
POD: example-web-python-86d9d584f8-7vjcj TIME: 17:33:23,313
POD: example-web-python-86d9d584f8-7vjcj TIME: 17:33:24,377
POD: example-web-python-86d9d584f8-7vjcj TIME: 17:33:25,445
POD: example-web-python-86d9d584f8-7vjcj TIME: 17:33:26,513
```

The requests get distributed amongst the three Pods. As soon as you scale down to one Pod, there should be only one remaining Pod that responds.
What happens if you start a new Deployment while our request generator is still running?


{{% alert title="Warning" color="secondary" %}}
On Windows, execute the following command in Git Bash; PowerShell seems not to work.
{{% /alert %}}

```bash
kubectl patch deployment example-web-python -p "{\"spec\":{\"template\":{\"metadata\":{\"labels\":{\"date\":\"`date +'%s'`\"}}}}}" --namespace <namespace>
```


During a short period we won't get a response:

```
POD: example-web-python-86d9d584f8-7vjcj TIME: 17:37:24,121
POD: example-web-python-86d9d584f8-7vjcj TIME: 17:37:25,189
POD: example-web-python-86d9d584f8-7vjcj TIME: 17:37:26,262
POD: example-web-python-86d9d584f8-7vjcj TIME: 17:37:27,328
POD: example-web-python-86d9d584f8-7vjcj TIME: 17:37:28,395
POD: example-web-python-86d9d584f8-7vjcj TIME: 17:37:29,459
POD: example-web-python-86d9d584f8-7vjcj TIME: 17:37:30,531
POD: example-web-python-86d9d584f8-7vjcj TIME: 17:37:31,596
POD: example-web-python-86d9d584f8-7vjcj TIME: 17:37:32,662
# no answer
POD: example-web-python-f4c5dd8fc-4nx2t TIME: 17:37:33,729
POD: example-web-python-f4c5dd8fc-4nx2t TIME: 17:37:34,794
POD: example-web-python-f4c5dd8fc-4nx2t TIME: 17:37:35,862
POD: example-web-python-f4c5dd8fc-4nx2t TIME: 17:37:36,929
POD: example-web-python-f4c5dd8fc-4nx2t TIME: 17:37:37,995
POD: example-web-python-f4c5dd8fc-4nx2t TIME: 17:37:39,060
POD: example-web-python-f4c5dd8fc-4nx2t TIME: 17:37:40,118
POD: example-web-python-f4c5dd8fc-4nx2t TIME: 17:37:41,187
```

In our example, we use a very lightweight Pod. If we had used a more heavyweight Pod that needed a longer time to respond to requests we would of course see a larger gap.
An example for this would be a Java application with a startup time of 30 seconds:

```
Pod: example-spring-boot-2-73aln TIME: 16:48:25,251
Pod: example-spring-boot-2-73aln TIME: 16:48:26,305
Pod: example-spring-boot-2-73aln TIME: 16:48:27,400
Pod: example-spring-boot-2-73aln TIME: 16:48:28,463
Pod: example-spring-boot-2-73aln TIME: 16:48:29,507
<html><body><h1>503 Service Unavailable</h1>
No server is available to handle this request.
</body></html>
 TIME: 16:48:33,562
<html><body><h1>503 Service Unavailable</h1>
No server is available to handle this request.
</body></html>
 TIME: 16:48:34,601
 ...
Pod: example-spring-boot-3-tjdkj TIME: 16:49:20,114
Pod: example-spring-boot-3-tjdkj TIME: 16:49:21,181
Pod: example-spring-boot-3-tjdkj TIME: 16:49:22,231

```

It is even possible that the Service gets down, and the routing layer responds with the status code 503 as can be seen in the example output above.

In the following chapter we are going to look at how a Service can be configured to be highly available.


## Uninterruptible Deployments

The [rolling](https://kubernetes.io/docs/tutorials/kubernetes-basics/update/update-intro/) update strategy makes it possible to deploy Pods without interruption. The rolling update strategy means that the new version of an application gets deployed and started. As soon as the application says it is ready, Kubernetes forwards requests to the new instead of the old version of the Pod, and the old Pod gets terminated.

Additionally, [container health checks](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/) help Kubernetes to precisely determine what state the application is in.

Basically, there are two different kinds of checks that can be implemented:

* Liveness probes are used to find out if an application is still running
* Readiness probes tell us if the application es ready to receive requests (which is especially relevant for above-mentioned rolling updates)

These probes can be implemented as HTTP checks, container execution checks (the execution of a command or script inside a container), or TCP socket checks.

In our example, we want the application to tell Kubernetes that it is ready for requests with an appropriate readiness probe. Our example application has a health check context named health: `http://<node-ip>:<node-port>/health`


## Task {{% param sectionnumber %}}.2: Availability during Deployment

In our deployment configuration inside the rolling update strategy section we define that our application has to be always be available during an update: `maxUnavailable: 0`

You can directly edit the deployment (or any resource) with:

```bash
kubectl edit deployment example-web-python --namespace <namespace>
```


**YAML:**

```yaml
...
spec:
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 0 # <- change this line
    type: RollingUpdate
...
```


If you prefer JSON formatting to YAML, use the corresponding `--output`/`-o` parameter to edit the resource in JSON:

```bash
kubectl edit deployment example-web-python -o json --namespace <namespace>
```


**JSON:**

```json
...
"strategy": {
    "rollingUpdate": {
        "maxSurge": "25%",
        "maxUnavailable": "0" // <- change this line
    },
    "type": "RollingUpdate"
},
...
```

Now insert the readiness probe at `.spec.template.spec.containers` above the `resources: {}` line:

**YAML:**

```yaml
...
     containers:
      - image: acend/example-web-python
        imagePullPolicy: Always
        name: example-web-python
        # start to copy here
        readinessProbe:
          httpGet:
            path: /health
            port: 5000
            scheme: HTTP
          initialDelaySeconds: 10
          timeoutSeconds: 1
        # stop to copy here
        resources: {}
...
```

**JSON:**

```json
...
                        "image": "acend/example-web-python",
                        "imagePullPolicy": "Always",
                        "name": "example-web-python",
                        // start to copy here
                        "readinessProbe": {
                            "httpGet": {
                                "path": "/health",
                                "port": 5000,
                                "scheme": "HTTP"
                            },
                            "initialDelaySeconds": 10,
                            "periodSeconds": 10,
                            "timeoutSeconds": 1
                        },
                        // stop to copy here
                        "resources": {},
...
```


The `containers` configuration then looks like:

**YAML:**

```yaml
...
      containers:
      - image: acend/example-web-python
        imagePullPolicy: Always
        name: example-web-python
        readinessProbe:
          failureThreshold: 3
          httpGet:
            path: /health
            port: 5000
            scheme: HTTP
          initialDelaySeconds: 10
          periodSeconds: 10
          successThreshold: 1
          timeoutSeconds: 1
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
...
```

**JSON:**

```json
...
                "containers": [
                    {
                        "image": "acend/example-web-python",
                        "imagePullPolicy": "Always",
                        "name": "example-web-python",
                        "readinessProbe": {
                            "failureThreshold": 3,
                            "httpGet": {
                                "path": "/health",
                                "port": 5000,
                                "scheme": "HTTP"
                            },
                            "initialDelaySeconds": 10,
                            "periodSeconds": 10,
                            "successThreshold": 1,
                            "timeoutSeconds": 1
                        },
                        "resources": {},
                        "terminationMessagePath": "/dev/termination-log",
                        "terminationMessagePolicy": "File"
                    }
                ],
...
```

We are now going to verify that a redeployment of the application does not lead to an interruption.

Set up the loop to periodically check the application's response:

```bash
URL=http://<node-ip>:<node-port>
while true; do sleep 1; curl -s $URL/pod/; date "+ TIME: %H:%M:%S,%3N"; done
```

Start a new deployment by editing it (the so-called _ConfigChange_ trigger creates the new Deployment automatically):

```bash
kubectl patch deployment example-web-python -p "{\"spec\":{\"template\":{\"metadata\":{\"labels\":{\"date\":\"`date +'%s'`\"}}}}}" --namespace <namespace>
```


## Self healing

Via ReplicaSet we told Kubernetes how many replicas we want. So what happens if we simply delete a Pod?

Look for a running Pod (status `RUNNING`) that you can bear to kill via `kubectl get pods`.

Show all Pods and watch for changes:

```bash
kubectl get pods -w --namespace <namespace>
```

Now delete a Pod (in another terminal) with the following command:

```bash
kubectl delete pod <pod> --namespace <namespace>
```

Observe how Kubernetes instantly creates a new Pod in order to fulfill the desired number of running instances.
