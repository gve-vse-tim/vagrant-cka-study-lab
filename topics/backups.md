# ETCD backups

There's a v2 and v3 API issue going on. What's strange is that the CLI arguments
are also API version dependent (--ca-cert vs --cacert).  So, the solution to the
backup issue is to ensure the etcdctl binary is install, the CLI args are checked
so that the correct CA certs can be used (in order to connect).

Ideally, we need to install the etcd-client into an Ubuntu OS (ideally the control
node?) and then access to the various ETCD certs from the control node.  In this
way, we can store the etcd snapshot in the host OS to be able to leverage it for
restoration.

/etc/kubernetes/pki/etcd is mapped into the etcd container.
/var/lib/etcd is also mapped from the host into the etcd container.

## Requirements

- API servers must be stopped

## Commands

- Backup
    - sudo su -c "ETCDCTL_API=3 etcdctl snapshot save etcd.db \
                  --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt \
                  --key=/etc/kubernetes/pki/etcd/server.key --endpoints=https://127.0.0.1:2379"
- Restore
    - sudo su -c "ETCDCTL_API=3 etcdctl snapshot restore etcd.db \
                  --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt \
                  --key=/etc/kubernetes/pki/etcd/server.key --endpoints=https://127.0.0.1:2379 \
                  --name=controlplane --data-dir /var/lib/etcd-from-backup \
                  --initial-cluster=controlplane=https://127.0.0.1:2380 \
                  --initial-cluster-token=etcd-cluster-1 \
                  --initial-advertise-peer-urls=https://127.0.0.1:2380"


## References

- https://brandonwillmott.com/2020/09/03/backup-and-restore-etcd-in-kubernetes-cluster-for-cka-v1-19/
- https://etcd.io/docs/v3.4/op-guide/recovery/
- https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#backing-up-an-etcd-cluster

## etcd.yaml manifest

```
$ kubectl get pod -n kube-system etcd-control1 -o yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubeadm.kubernetes.io/etcd.advertise-client-urls: https://192.168.56.11:2379
    kubernetes.io/config.hash: c1b407ff6b2eba6b19ae38cadb8e2232
    kubernetes.io/config.mirror: c1b407ff6b2eba6b19ae38cadb8e2232
    kubernetes.io/config.seen: "2021-05-17T03:31:02.135459400Z"
    kubernetes.io/config.source: file
  creationTimestamp: "2021-05-17T03:31:08Z"
  labels:
    component: etcd
    tier: control-plane
  managedFields:
  - apiVersion: v1
    fieldsType: FieldsV1
    fieldsV1:
      f:metadata:
        f:annotations:
          .: {}
          f:kubeadm.kubernetes.io/etcd.advertise-client-urls: {}
          f:kubernetes.io/config.hash: {}
          f:kubernetes.io/config.mirror: {}
          f:kubernetes.io/config.seen: {}
          f:kubernetes.io/config.source: {}
        f:labels:
          .: {}
          f:component: {}
          f:tier: {}
        f:ownerReferences:
          .: {}
          k:{"uid":"6c4789e3-2280-4b3e-846f-5c42c2373292"}:
            .: {}
            f:apiVersion: {}
            f:controller: {}
            f:kind: {}
            f:name: {}
            f:uid: {}
      f:spec:
        f:containers:
          k:{"name":"etcd"}:
            .: {}
            f:command: {}
            f:image: {}
            f:imagePullPolicy: {}
            f:livenessProbe:
              .: {}
              f:failureThreshold: {}
              f:httpGet:
                .: {}
                f:host: {}
                f:path: {}
                f:port: {}
                f:scheme: {}
              f:initialDelaySeconds: {}
              f:periodSeconds: {}
              f:successThreshold: {}
              f:timeoutSeconds: {}
            f:name: {}
            f:resources: {}
            f:startupProbe:
              .: {}
              f:failureThreshold: {}
              f:httpGet:
                .: {}
                f:host: {}
                f:path: {}
                f:port: {}
                f:scheme: {}
              f:initialDelaySeconds: {}
              f:periodSeconds: {}
              f:successThreshold: {}
              f:timeoutSeconds: {}
            f:terminationMessagePath: {}
            f:terminationMessagePolicy: {}
            f:volumeMounts:
              .: {}
              k:{"mountPath":"/etc/kubernetes/pki/etcd"}:
                .: {}
                f:mountPath: {}
                f:name: {}
              k:{"mountPath":"/var/lib/etcd"}:
                .: {}
                f:mountPath: {}
                f:name: {}
        f:dnsPolicy: {}
        f:enableServiceLinks: {}
        f:hostNetwork: {}
        f:nodeName: {}
        f:priorityClassName: {}
        f:restartPolicy: {}
        f:schedulerName: {}
        f:securityContext: {}
        f:terminationGracePeriodSeconds: {}
        f:tolerations: {}
        f:volumes:
          .: {}
          k:{"name":"etcd-certs"}:
            .: {}
            f:hostPath:
              .: {}
              f:path: {}
              f:type: {}
            f:name: {}
          k:{"name":"etcd-data"}:
            .: {}
            f:hostPath:
              .: {}
              f:path: {}
              f:type: {}
            f:name: {}
      f:status:
        f:conditions:
          .: {}
          k:{"type":"ContainersReady"}:
            .: {}
            f:lastProbeTime: {}
            f:lastTransitionTime: {}
            f:status: {}
            f:type: {}
          k:{"type":"Initialized"}:
            .: {}
            f:lastProbeTime: {}
            f:lastTransitionTime: {}
            f:status: {}
            f:type: {}
          k:{"type":"PodScheduled"}:
            .: {}
            f:lastProbeTime: {}
            f:lastTransitionTime: {}
            f:status: {}
            f:type: {}
          k:{"type":"Ready"}:
            .: {}
            f:lastProbeTime: {}
            f:lastTransitionTime: {}
            f:status: {}
            f:type: {}
        f:containerStatuses: {}
        f:hostIP: {}
        f:phase: {}
        f:podIP: {}
        f:podIPs:
          .: {}
          k:{"ip":"192.168.56.11"}:
            .: {}
            f:ip: {}
        f:startTime: {}
    manager: kubelet
    operation: Update
    time: "2021-05-19T19:19:32Z"
  name: etcd-control1
  namespace: kube-system
  ownerReferences:
  - apiVersion: v1
    controller: true
    kind: Node
    name: control1
    uid: 6c4789e3-2280-4b3e-846f-5c42c2373292
  resourceVersion: "607700"
  selfLink: /api/v1/namespaces/kube-system/pods/etcd-control1
  uid: 1d5b41b3-fa27-4bab-84d6-56defdaf2ee7
spec:
  containers:
  - command:
    - etcd
    - --advertise-client-urls=https://192.168.56.11:2379
    - --cert-file=/etc/kubernetes/pki/etcd/server.crt
    - --client-cert-auth=true
    - --data-dir=/var/lib/etcd
    - --initial-advertise-peer-urls=https://192.168.56.11:2380
    - --initial-cluster=control1=https://192.168.56.11:2380
    - --key-file=/etc/kubernetes/pki/etcd/server.key
    - --listen-client-urls=https://127.0.0.1:2379,https://192.168.56.11:2379
    - --listen-metrics-urls=http://127.0.0.1:2381
    - --listen-peer-urls=https://192.168.56.11:2380
    - --name=control1
    - --peer-cert-file=/etc/kubernetes/pki/etcd/peer.crt
    - --peer-client-cert-auth=true
    - --peer-key-file=/etc/kubernetes/pki/etcd/peer.key
    - --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
    - --snapshot-count=10000
    - --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
    image: k8s.gcr.io/etcd:3.4.13-0
    imagePullPolicy: IfNotPresent
    livenessProbe:
      failureThreshold: 8
      httpGet:
        host: 127.0.0.1
        path: /health
        port: 2381
        scheme: HTTP
      initialDelaySeconds: 10
      periodSeconds: 10
      successThreshold: 1
      timeoutSeconds: 15
    name: etcd
    resources: {}
    startupProbe:
      failureThreshold: 24
      httpGet:
        host: 127.0.0.1
        path: /health
        port: 2381
        scheme: HTTP
      initialDelaySeconds: 10
      periodSeconds: 10
      successThreshold: 1
      timeoutSeconds: 15
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/lib/etcd
      name: etcd-data
    - mountPath: /etc/kubernetes/pki/etcd
      name: etcd-certs
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  hostNetwork: true
  nodeName: control1
  preemptionPolicy: PreemptLowerPriority
  priority: 2000001000
  priorityClassName: system-node-critical
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext: {}
  terminationGracePeriodSeconds: 30
  tolerations:
  - effect: NoExecute
    operator: Exists
  volumes:
  - hostPath:
      path: /etc/kubernetes/pki/etcd
      type: DirectoryOrCreate
    name: etcd-certs
  - hostPath:
      path: /var/lib/etcd
      type: DirectoryOrCreate
    name: etcd-data
status:
  conditions:
  - lastProbeTime: null
    lastTransitionTime: "2021-05-17T03:31:08Z"
    status: "True"
    type: Initialized
  - lastProbeTime: null
    lastTransitionTime: "2021-05-17T20:46:10Z"
    status: "True"
    type: Ready
  - lastProbeTime: null
    lastTransitionTime: "2021-05-17T20:46:10Z"
    status: "True"
    type: ContainersReady
  - lastProbeTime: null
    lastTransitionTime: "2021-05-17T03:31:08Z"
    status: "True"
    type: PodScheduled
  containerStatuses:
  - containerID: docker://7065c6a5276f582a21f1a8b83b6d77d0c4664c07ad2031e56dd2b57d4247f70b
    image: k8s.gcr.io/etcd:3.4.13-0
    imageID: docker-pullable://k8s.gcr.io/etcd@sha256:4ad90a11b55313b182afc186b9876c8e891531b8db4c9bf1541953021618d0e2
    lastState:
      terminated:
        containerID: docker://31e3169669474d6409a42e73ea99b3b210548e284b1fea9129ec30fafc05261e
        exitCode: 0
        finishedAt: "2021-05-19T19:19:30Z"
        reason: Completed
        startedAt: "2021-05-17T20:44:49Z"
    name: etcd
    ready: true
    restartCount: 3
    started: true
    state:
      running:
        startedAt: "2021-05-19T19:19:31Z"
  hostIP: 192.168.56.11
  phase: Running
  podIP: 192.168.56.11
  podIPs:
  - ip: 192.168.56.11
  qosClass: BestEffort
  startTime: "2021-05-17T03:31:08Z"
```