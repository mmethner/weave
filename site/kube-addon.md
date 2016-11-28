---
title: Integrating Kubernetes via the Addon
menu_order: 63
---

The following topics are discussed:

* [Installation](#install)
* [Network Policy Controller](#npc)
 * [Upgrading the Daemon Sets](#daemon-sets)
* [Network Policy Controller](#npc)
 * [Troubleshooting Blocked Connections](#blocked-connections)
 * [Changing Configuration Options](#configuration-options)


##<a name=”install”></a> Installation

In versions of Kubernetes that are 1.4 or greater and that is also configured to use
[CNI](/site/cni-plugin.md), Weave Net can be installed onto your cluster with a single
command:

```
kubectl apply -f https://git.io/weave-kube
```

After a few seconds, a Weave Net pod should be running on each
Node and any further pods you create will be automatically attached to the Weave
network.

> If you do not already have a CNI-enabled cluster, you can bootstrap
> one easily with
> [kubeadm](http://kubernetes.io/docs/getting-started-guides/kubeadm/).
> Alternatively if you are using the older cluster set-up scripts from
> the Kubernetes repo, then use:
>
>
>     NETWORK_PROVIDER=cni cluster/kube-up.sh

**Note:** If using the [Weave CNI
Plugin](/site/cni-plugin.md) from a prior full install of Weave Net with your
cluster, you must first uninstall it before applying the Weave-kube addon.
Shut down Kubernetes, and _on all nodes_ perform the following:

 * `weave reset`
 * Remove any separate provisions you may have made to run Weave at
   boot-time, e.g. `systemd` units
 * `rm /opt/cni/bin/weave-*`

Then relaunch Kubernetes and install the addon as described
above.

The URL [https://git.io/weave-kube](https://git.io/weave-kube) points
to the YAML file for the latest release of the Weave Net addon.
Historic versions are archived on our [GitHub release
page](https://github.com/weaveworks/weave/releases).

###<a name=”daemon-sets”></a> Upgrading the Daemon Sets

Kubernetes does not currently support rolling upgrades of daemon sets,
and so you will need to perform the procedure manually:

* Apply the updated addon manifest `kubectl apply -f https://git.io/weave-kube`
* Kill each Weave Net pod with `kubectl delete` and then wait for it to reboot before moving on to the next pod.

##<a name="npc"></a>Network Policy Controller

The addon also supports the [Kubernetes policy
API](http://kubernetes.io/docs/user-guide/networkpolicies/) so that
you can securely isolate pods from each other based on namespaces and
labels. For more information on configuring network policies in
Kubernetes see the
[walkthrough](http://kubernetes.io/docs/getting-started-guides/network-policy/walkthrough/)
and the [NetworkPolicy API object
definition](http://kubernetes.io/docs/api-reference/extensions/v1beta1/definitions/#_v1beta1_networkpolicy).

###<a name=”blocked-connections”></a> Troubleshooting Blocked Connections

If you suspect that legitimate traffic is being blocked by the Weave Network Policy Controller, check the `weave-npc` container's logs:

```
$ kubectl logs $(kubectl get pods --all-namespaces | grep weave-net | awk '{print $2}') -n kube-system weave-npc
```

When the Weave Network Policy Controller blocks a connection, it logs the following details about it:

* protocol used, 
* source IP and port, 
* destination IP and port, 

as per the below example:

```
TCP connection from 10.32.0.7:56648 to 10.32.0.11:80 blocked by Weave NPC.
UDP connection from 10.32.0.7:56648 to 10.32.0.11:80 blocked by Weave NPC.
```

###<a name=”configuration-options”></a> Changing Configuration Options

The default configuration settings can be changed by saving and editing the
addon YAML before running `kubectl apply`. Additional arguments may be
supplied to the Weave router process by adding them to the `command:`
array in the YAML file.

Some parameters are changed by environment variables and these can be
inserted into the YAML file as follows:

```
      containers:
        - name: weave
          env:
            - name: IPALLOC_RANGE
              value: 10.0.0.0/16
```

The list of variables you can set is:

* **CHECKPOINT_DISABLE** - if set to 1, disable checking for new Weave Net
  versions (default is blank, i.e. check is enabled)
* **IPALLOC_RANGE** - the range of IP addresses used by Weave Net
  and the subnet they are placed in (CIDR format; default 10.32.0.0/12)
* **EXPECT_NPC** - set to 0 to disable Network Policy Controller (default is on)
* **KUBE_PEERS** - list of addresses of peers in the Kubernetes cluster
  (default is to fetch the list from the api-server)
* **IPALLOC_INIT** - set the initialization mode of the [IP Address
  Manager](/site/operational-guide/concepts.md#ip-address-manager)
  (defaults to consensus amongst the KUBE_PEERS)

