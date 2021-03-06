---
title: "3. First steps in the lab environment"
weight: 3
sectionnumber: 3
---


In this lab, we will interact with the Kubernetes cluster for the first time.

{{% alert title="Warning" color="secondary" %}}
Please make sure you completed [lab 2](../02.0/) before you continue with this lab.
{{% /alert %}}


## Login and choose a Kubernetes cluster

{{% alert title="Note" color="primary" %}}
Authentication depends on the specific Kubernetes cluster environment. You may need special instructions if you're not using our lab environment.
{{% /alert %}}

{{< onlyWhen rancher >}}
Our Kubernetes cluster of the lab environment runs on [cloudscale.ch](https://cloudscale.ch) (a Swiss IaaS provider) and has been provisioned with [Rancher](https://rancher.com/). You can log in into the cluster with a Rancher user.

{{% alert title="Note" color="primary" %}}
Your teacher will provide you with the credentials to log in.
{{% /alert %}}

Log in to the Rancher web console and choose the desired cluster.

You now see a button at the top right that says **Kubeconfig File**. Click it, scroll down to the bottom and click **Copy to Clipboard**.

![Download kubeconfig File](kubectlconfigfilebutton.png)

The copied kubeconfig now needs to be put into a file. The default location for the kubeconfig file is `~/.kube/config`.

{{% alert title="Note" color="primary" %}}
If you already have a kubeconfig file, you might need to merge the Rancher entries with yours. Or use a dedicated file as described below.
{{% /alert %}}

Put the copied content into a kubeconfig file on your system.
If you decide to not use the default kubeconfig location at `~/.kube/config` then let `kubectl` know where you put it with the KUBECONFIG environment variable:

```
export KUBECONFIG=$KUBECONFIG:~/.kube-techlab/config
```

{{< /onlyWhen >}}

{{< onlyWhen mobi >}}
We are using the Mobi `kubedev` Kubernetes cluster. Use the following command to set the appropriate context:

```bash
kubectl config use-context kubedev
```

{{% alert title="Warning" color="secondary" %}}
Make sure you have setup your kubeconfig file correctly. Check your [CWIKI](https://cwiki.mobicorp.ch/confluence/display/ITContSol/Set+up+Kubectl) for instructions on how to configure it.
{{% /alert %}}
{{< /onlyWhen >}}


## Namespaces

As a first step on the cluster we are going to create a new Namespace.

A Namespace is the logical design used in Kubernetes to organize and separate your applications, Deployments, Pods, Ingresses, Services, etc. on a top-level basis. Take a look at the [Kubernetes docs](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/). Authorized users inside a namespace are able to manage those resources. Namespace names have to be unique in your cluster.

{{< onlyWhen rancher >}}
{{% alert title="Note" color="primary" %}}
Additionally, Rancher knows the concept of a [*Project*](https://rancher.com/docs/rancher/v2.x/en/cluster-admin/projects-and-namespaces/) which encapsulates multiple Namespaces.
{{% /alert %}}

In the Rancher web console choose the Project called `techlab`.

{{< onlyWhen mobi >}}
We use the project `kubernetes-techlab` on the `kubedev` cluster.
{{< /onlyWhen >}}

{{< onlyWhenNot mobi >}}
![Rancher Project](chooseproject.png)
{{< /onlyWhenNot >}}

{{< /onlyWhen >}}


### Task {{% param sectionnumber %}}.1: Create a Namespace

Create a new namespace in the lab environment. The `kubectl help` output can help you figure out the right command.

{{% alert title="Note" color="primary" %}}
Please choose an identifying name for your Namespace, e.g. your initials or name as a prefix. We are going to use `<namespace>` as a placeholder for your created Namespace.
{{% /alert %}}


### Solution

To create a new Namespace on your cluster use the following command:

```bash
kubectl create namespace <namespace>
```

{{< onlyWhen rancher >}}
{{% alert title="Note" color="primary" %}}
Namespaces created via `kubectl` have to be assigned to the correct Rancher Project in order to be visible in the Rancher web console. Please ask your teacher for this assignment. Or you can create the Namespace directly within the Rancher web console.
{{% /alert %}}
{{< /onlyWhen >}}

{{% alert title="Note" color="primary" %}}
By using the following command, you can switch into another Namespace instead of specifying it for each `kubectl` command.

Linux:

```bash
kubectl config set-context $(kubectl config current-context) --namespace <namespace>
```

Windows:

```bash
kubectl config current-context
SET KUBE_CONTEXT=[Insert output of the upper command]
kubectl config set-context %KUBE_CONTEXT% --namespace <namespace>
```

Some prefer to explicitly select the Namespace for each `kubectl` command by adding `--namespace <namespace>` or `-n <namespace>`. Others prefer helper tools like `kubens` (see [lab 2](../02.0)).
{{% /alert %}}


{{< onlyWhen rancher >}}


## Task {{% param sectionnumber %}}.2: Discover the Rancher web console

Check the menu entries, there should neither appear any Deployments nor any Pods or Services in your Namespace.

Display all existing Pods in the previously created Namespace with `kubectl` (there shouldn't yet be any):

```bash
kubectl get pod -n <namespace>
```

With the command `kubectl get` you can display all kinds of resources.
{{< /onlyWhen >}}
