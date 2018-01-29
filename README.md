# kubectl In Action

_Or: How to use your Kubernetes remote control_

This is a talk at [Pre-FOSDEM warmup with Kubernetes](https://www.meetup.com/Brussels-Kubernetes-Meetup/events/245974093/) at Brussels Kubernetes Meetup, 2018-02-02.

Kubernetes comes with kubectl, a CLI that allows you to interact with your cluster, supporting operations from config to managing workloads to administrative tasks. In this talk you'll learn everything you need to know getting the most out of kubectl and beyond.

- [Setup](#setup)
- [Contexts](#contexts)
- [Docs](#docs)
- [Managing workloads](#managing-workloads)
- [Services](#services)
- [Accessing the API](#accessing-the-api)
- [RBAC](#rbac)
- [Debugging](#debugging)
- [Tips and tricks](#tips-and-tricks)
- [Tooling](#tooling)
- [Further reading & watching](#further-reading-watching)

## Setup

I'm new to a cluster, so I have a look around.

What version am I on (client-side and server-side)?

```
$ kubectl version
$ kubectl version --short
```

Can I have a generic dump of `~/.kube/config`?

```
$ kubectl config view
```

## Contexts

What contexts are available?

```
$ kubectl config get-contexts
```

Switch to a certain context:

```
$ kubectl config use minikube
```

Create context `somek` and switch to it:

```
$ kubectl config set-context somek --user=cluster-admin --namespace=meh && \
  kubectl config use-context somek
```

## Docs

What can I do with a command?

```
$ kubectl run -h
```

Got some usage examples?

```
$ kubectl run -h | tail -n+$(kubectl run -h | grep -n Example | grep -Eo '^[^:]+') | head -n $(kubectl run -h | grep -n Options | grep -Eo '^[^:]+')
```

What was that field in the manifest again?

```
$ kubectl explain statefulset.spec.template.spec
```

## Managing workloads

Launching a simple jump pod:

```
$ kubectl run -i -t --rm jumpod --restart=Never --image=quay.io/mhausenblas/jump:v0.1 -- sh
```

Do a dry-run for a long-running process (NGINX):

```
$ kubectl run webserver --image=nginx:1.13 --output=yaml --dry-run
```

Get the name of deployment labelled with `run` as key:

```
$ kubectl get deploy -l=run -o=custom-columns=:metadata.name --no-headers
```

Get the name of pod(s) labelled with `run=webserver`:

```
$ kubectl get po -l=run=webserver -o=custom-columns=:metadata.name --no-headers
```

Scale the `webserver` deployment to 2 replicas and observe the scaling:

```
# in shell terminal 1:
$ kubectl get po -l=run=webserver -o=custom-columns=:metadata.name --no-headers --watch

# in shell terminal 2:
$ kubectl scale deploy/webserver --replicas=2
```

Get rid of `webserver` deployment by scaling down to 0 and deleting it:

```
$ kubectl scale deploy/webserver --replicas=0
$ kubectl delete deploy/webserver
```

## Services

Create a service `webserver` for a deployment and check it:

```
$ kubectl run webserver --image=nginx:1.13
$ kubectl expose deployment webserver --port=80 --target-port=80
$ kubectl get svc -l=run=webserver
```

I'd like to `curl` the new `webserver` service from within the cluster:

```
$ kubectl run -i -t --rm curlpod --restart=Never --image=quay.io/mhausenblas/jump:v0.1 -- curl webserver
```

## Accessing the API

```
$ kubectl proxy
```

### RBAC

Can a certain SA list pods?

```
$ kubectl auth can-i list pods --as=system:serviceaccount:sec:myappsa
```

Create rolebinding for an SA in a specified namespace and just do a dry run:

```
$ kubectl create rolebinding podreaderbinding --role=sec:podreader --serviceaccount=sec:myappsa --namespace=sec --dry-run=true -o=yaml -n=sec
```

## Debugging

```
$ kubectl get events
$ kubectl logs
$ kubectl port-forward $(kubectl get po -l=run=webserver -o=custom-columns=:metadata.name --no-headers) 8080:80
```

## Tips and tricks

- Install and use [auto-complete](https://kubernetes.io/docs/tasks/tools/install-kubectl/#enabling-shell-autocompletion)
- set `KUBE_EDITOR` to your favorite editor

## Tooling

Extend and enhance `kubectl` with:

- [jonmosco/kube-ps1](https://github.com/jonmosco/kube-ps1)
- [ahmetb/kubectx](https://github.com/ahmetb/kubectx)
- [cloudnativelabs/kube-shell](https://github.com/cloudnativelabs/kube-shell)
- [nii236/kk](https://github.com/nii236/kk)
- [CanopyTax/ckube](https://github.com/CanopyTax/ckube)
- [kubed.sh](http://kubed.sh/)

![extending kubectl](img/aab-twitter.jpg)

Source: [Ahmet Alp Balkan](https://twitter.com/ahmetb/status/949064018483802112) on Twitter 01/2018.


## Further reading & watching

- [Troubleshoot Kubernetes Deployments](https://docs.bitnami.com/kubernetes/how-to/troubleshoot-kubernetes-deployments/)
- [What happens when you do kubectl run](https://github.com/jamiehannaford/what-happens-when-k8s/blob/master/README.md)
- [Kubernetes kubectl Tips and Tricks](https://coreos.com/blog/kubectl-tips-and-tricks)
- [Some things you didn’t know about kubectl](http://blog.kubernetes.io/2015/10/some-things-you-didnt-know-about-kubectl_28.html)
