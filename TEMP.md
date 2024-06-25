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

k apply -f -<<EOF
apiVersion: v1
kind: Pod
metadata:
  name: ex2
  labels:
    zone: us-est-coast
    cluster: test-cluster1
spec:
  containers:
    - name: client-container
      image: ubuntu
      command: ["sh", "-c", "sleep 9999"]
      env:
      # - name: NODE_LABELS
      #   valueFrom:
      #     fieldRef:
      #       fieldPath: node.metadata.labels 
      - name: NODE_LABEL_REGION
        valueFrom:
          fieldRef:
            fieldPath: node.metadata.labels['region']  
      # - name: POD_LABELS
      #   valueFrom:
      #     fieldRef:
      #       fieldPath: metadata.labels
      volumeMounts:
        - name: nodelabels
          mountPath: /etc/nodelabels
        - name: podlabels
          mountPath: /etc/podlabels
  volumes:
    - name: nodelabels
      downwardAPI:
        items:
          - path: "nodelabels"
            fieldRef:
              fieldPath: node.metadata.labels
    - name: podlabels
      downwardAPI:
        items:
          - path: "podlabels"
            fieldRef:
              fieldPath: metadata.labels
EOF
sleep 10;
# quick check
kubectl exec -it ex2 -- sh -c "cat /etc/nodelabels/nodelabels"
kubectl exec -it ex2 -- sh -c "cat /etc/podlabels/podlabels"
kubectl exec -it ex2 -- sh -c "env | grep NODE_LABEL_REGION"
```
