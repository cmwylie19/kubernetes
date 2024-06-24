Export Kind Logs:

```bash
kind export logs   
```

Quick Build:

```bash
kind delete clusters --all; 
docker system prune -a -f; 
kind build node-image .;
kind create cluster --image kindest/node:latest;
sleep 10;
k apply -f -<<EOF
apiVersion: v1
kind: Pod
metadata:
  name: kubernetes-downwardapi-volume-example
  labels:
    zone: us-est-coast
    cluster: test-cluster1
    rack: rack-22
  annotations:
    build: two
    builder: john-doe
spec:
  containers:
    - name: client-container
      image: registry.k8s.io/busybox
      command: ["sh", "-c"]
      args:
      - while true; do
          if [[ -e /etc/podinfo/labels ]]; then
            echo -en '\n\n'; cat /etc/podinfo/labels; fi;
          if [[ -e /etc/podinfo/annotations ]]; then
            echo -en '\n\n'; cat /etc/podinfo/annotations; fi;
          sleep 5;
        done;
      volumeMounts:
        - name: podinfo
          mountPath: /etc/podinfo
  volumes:
    - name: podinfo
      downwardAPI:
        items:
          - path: "nodelabels"
            fieldRef:
              fieldPath: node.metadata.labels
          - path: "labels"
            fieldRef:
              fieldPath: metadata.labels
          - path: "annotations"
            fieldRef:
              fieldPath: metadata.annotations
EOF
```


apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: hithere
  name: hithere
spec:
  volumes:
    - name: proof
      downwardAPI:
        items:
          - path: "nodelabels"
            fieldRef:
              fieldPath: node.metadata.labels
  containers:
  - image: nginx
    name: hithere
    resources: {}
    volumeMounts:
      - name: proof
        mountPath: /etc/proof
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
