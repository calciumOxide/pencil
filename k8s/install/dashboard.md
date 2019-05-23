# 安装dashboard

*docker pull docker.io/siriuszg/kubernetes-dashboard-amd64*

## 1、配置kube-dashboard.yaml
```text
kind: Deployment
apiVersion: apps/v1beta2
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kube-system
spec:
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      k8s-app: kubernetes-dashboard
  template:
    metadata:
      labels:
        k8s-app: kubernetes-dashboard
    spec:
      containers:
      - name: kubernetes-dashboard
        image: kubernetes-dashboard-amd64:v1.8.3 
        ports:
        - containerPort: 9090
          protocol: TCP
        args:
          - --apiserver-host=http://10.112.118.166:8080          ###修改为Master的IP
        volumeMounts:
        - mountPath: /tmp
          name: tmp-volume
        livenessProbe:
          httpGet:
            path: /
            port: 9090
          initialDelaySeconds: 30
          timeoutSeconds: 30
      volumes:
      - name: tmp-volume
        emptyDir: {}
      serviceAccountName: kubernetes-dashboard
      # Comment the following tolerations if Dashboard must not be deployed on master
      tolerations:
      - key: node-role.kubernetes.io/master
        effect: NoSchedule

---

kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kube-system
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: 9090
    nodePort: 30090
  selector:
    k8s-app: kubernetes-dashboard
```

## 2、创建dashboard
```
kubectl create -f kube-dashboard.yaml
```

