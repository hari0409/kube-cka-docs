# Cluster Maintanance:

## Node OS Upgrades:
* This has to be done on the node but there will be pods running on them. This can be tackled by using the `drain` command which will drain the pods created ReplicaSet, Deployment, DaemonSets etc to anohter pod. 
* This is done by `kubect drain nodeName`. We can also ignore the DaemonSets on the node using the command `kubectl drain nodeName --ignore-daemonsets`.
* If required, we can also force the draining process. `kubectl drain nodeName --force`.
* Mark a node as schedulable, `kubectl uncordon nodeName`.
* Mark a node as unschedulable, `kubectl cordon nodeName`.

## Upgrade Kubernetes Cluster:
* [DOCS](https://v1-28.docs.kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/) can be used for upgrading the cluster.
* This upgrade is for kubeadm & is suitable to be done one version at a time.
* Upgrade in this order:
    * `kubeadm` for controlplane
    * `kubeadm upgrade apply vVERSION` for controlplane
    * `kubelet` for the controlplane
    * `kubeadm` for each node
    * `kubelet` upgrade for each node
    * `kubeadm upgrade node` for each node

## Backup & Restore:
* The backup can be done for three datas:
    * Config Files
    * ETCD Cluster
    * Data Volume
* For the Config Files, we can output the entire config of all using the command `kubectl get all -o yaml`
* For the ETCD Cluster:
    * Set ETCDCTL_API=3 by `export ETCDCTL_API=3`
    * Take a snapshot, we need four properties [requied only for snapshots]
        * `entrypoints`
        * `cert`
        * `cacert`
        * `key`
    ```bash
    ETCDCTL_API=3 etcdctl snapshot save snap.db \
    --endpoint=https://1227.0.0.1:2379
    --cacert=/etc/etcd/ca.crt
    --cert=/etc/etcd/etcd-server/crt
    --key=/etc/etcd/etcd-server.
    ```
    * Status of Snapshot using `ETCDCTL_API=3 etcdctl snapshot status name.db`
    * Restore the snapshot `ETCDCTL_API=3 etcdctl snapshot restore name.db --data-dir=/path` where the path must be without any files or folder.
    * For any help, `ETCDCTL_API=3 etcdctl <command> --help`
* For the Config of the kubernetes, we can use `kubectl config` for changing the config around.
* If the ETCD cluster is installed as a seperate server running, then we can use `ps -aux | grep etcd` to get information about it & SSH into it to get more data about it.
* Move data between servers using SCP `scp node:/path /path`. If required, `scp user@node:/remote_path current_path` where `scp SOURCE TARGET`
* Managing External ETCD Cluster:
    * Backing up & Restoring is same as we do in Stacked ETCD.
    * Modify the permission of ETCD user to access the folder using the command `chmod -R etcd:etcd /path`
    * To edit the options, we have to edit the `/etc/systemd/system/etcd.service` file.
    * Restart the service `systemctl daemon-reload` & then `systemctl restart etcd.service`

