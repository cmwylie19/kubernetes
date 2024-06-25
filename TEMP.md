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
k label no kind-control-plane region=us-east-1
sleep 10;
k delete po --all --force;
k apply -f -<<EOF
apiVersion: v1
kind: Pod
metadata:
  name: ex3
spec:
  containers:
    - name: client-container
      image: ubuntu
      command: ["sh", "-c", "env | grep NODE_LABEL_REGION && cat /etc/nodelabels/nodelabels && sleep 9999"]
      env:
      - name: NODE_LABEL_REGION
        valueFrom:
          fieldRef:
            fieldPath: node.metadata.labels['region']  
      volumeMounts:
        - name: nodelabels
          mountPath: /etc/nodelabels
  volumes:
    - name: nodelabels
      downwardAPI:
        items:
          - path: "nodelabels"
            fieldRef:
              fieldPath: node.metadata.labels
---
EOF
sleep 3;
kubectl logs ex3
```



Running Test:

```bash
make WHAT=test/e2e/e2e.test
./_output/bin/ginkgo --focus="should provide node label region as an env var and mount a volume from node labels" ./_output/bin/e2e.test -- --kubeconfig=$HOME/.kube/config --provider=skeleton

```
