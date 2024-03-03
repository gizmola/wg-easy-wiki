Example deployment on a self hosted K3s cluster with local storage.

Some caveats and notes:
- K3s provides an `eth0` device for every pod, you'll have to check your distribution to find out what device is available within the pod. I'm 99% sure it's eth0 everywhere, but for example `aws-vpc-cni`, containerd, or dockerd may call it something else. If you see inbound traffic but no outbound traffic, and can only reach the wg-easy http interface on `http://10.8.0.1:51821` then this will most likely be your problem and you'll need to specify the `WG_DEVICE` environment variable.
- This example is using local storage on a single node in a self-hsoted k3s cluster. It is assumed if you're reading this you know how to create persistent volumes. If not then it's best to start with the [Kuberenetes documentation](https://kubernetes.io/docs/concepts/storage/persistent-volumes/). There are many other options for storage here which are outside the scope of this doc.
- It's assumed you can already `kubectl apply` or pass this in to argocd/flux or whatever CI/CD system you want.
- There is no UI password protection here. It's assumed you can add your own Secret and adjust the UI accordingly if needed.

## Create a namespace
Example below
```
apiVersion: v1
kind: Namespace
metadata:
  name: wg-easy
```

## Create a deployment
An example is below, adjust as necessary.
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wg-easy
  namespace: wg-easy
spec:
  replicas: 1
  revisionHistoryLimit: 1
  selector:
    matchLabels:
      app.kubernetes.io/name: wg-easy
  strategy:
    # Restrict to a Single wg-easy instance, on redeploys it will tear down the old one before bring a new one up.
    type: Recreate
  template:
    metadata:
      labels:
        app.kubernetes.io/name: wg-easy
    spec:
      containers:
        - name: wg-easy
          # Specify external hostname and port as environment variables
          env:
            - name: WG_HOST
              value: wg.hostname.com
            - name: WG_PORT
              value: "30000"
          image: ghcr.io/wg-easy/wg-easy
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 51820
              name: wg
              protocol: UDP
            - containerPort: 51821
              name: http
              protocol: TCP
          # Use the http server for pod health checks
          livenessProbe:
            failureThreshold: 3
            periodSeconds: 10
            successThreshold: 1
            tcpSocket:
              port: http
            timeoutSeconds: 1
          readinessProbe:
            failureThreshold: 3
            periodSeconds: 10
            successThreshold: 1
            tcpSocket:
              port: http
            timeoutSeconds: 1
          startupProbe:
            failureThreshold: 30
            periodSeconds: 5
            successThreshold: 1
            tcpSocket:
              port: http
            timeoutSeconds: 1
          # Give pod permissions to modify iptables and load the wireguard kernel module
          securityContext:
            capabilities:
              add:
                - NET_ADMIN
                - SYS_MODULE
          # Persistent storage location
          volumeMounts:
            - mountPath: /etc/wireguard
              name: config
      restartPolicy: Always
      volumes:
        - name: config
          persistentVolumeClaim:
            claimName: wg-easy-storage-claim
```

## Create services

Two services will be created:
1. A udp service for connecting to the wireguard daemon
1. A http service for connecting to the UI via an Ingress

### Wireguard UDP service

This will map a nodePort to the wg-easy pod. It's possible to use a LoadBalancer service here also but for this example it's assumed you are able to bring external traffic in to the node port address via port forwarding.

```
apiVersion: v1
kind: Service
metadata:
  name: wg-easy-wg
  namespace: wg-easy
spec:
  ports:
    - name: wg
      port: 30000
      nodePort: 30000
      protocol: UDP
      targetPort: wg
  selector:
    app.kubernetes.io/name: wg-easy
  type: NodePort
```

### UI Service

The UI will be exposed externally via a K8s Ingress so does not require a NodePort.
```
apiVersion: v1
kind: Service
metadata:
  name: wg-easy-http
  namespace: wg-easy
spec:
  ports:
    - name: http
      port: 51821
      protocol: TCP
      targetPort: http
  selector:
    app.kubernetes.io/name: wg-easy
  type: ClusterIP
```

## Create UI Ingress

Example using ingress-nginx to connect to the UI service running in the wg-easy pod.
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    cert-manager.io/cluster-issuer: letsencryptcertmanager # Or your certificate issuer
  name: wg-easy
  namespace: wg-easy
spec:
  ingressClassName: nginx
  rules:
    - host: wg.ingress-hostname.com
      http:
        paths:
          - backend:
              service:
                name: wg-easy-http
                port: 
                  name: http
            path: /
            pathType: Prefix
  tls:
    - hosts:
        - wg.ingress-hostname.com
      secretName: wg-ingress-hostname-com-tls
```

## Add Persistent Volume Claim

Example persistent volume claim. The persistent volume and storage class are left out since these are unique to the cluster.
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: wg-easy-storage-claim
  namespace: wg-easy
spec:
  accessModes:
  - ReadWriteMany
  storageClassName: local-storage-zfs
  volumeName: wg-easy-pv
  resources:
    requests:
      storage: 256Mi
```

## Direct internet traffic to your wiregard NodePort service

Once objects are deployed you will need to direct internet traffic to your cluster. For example:
- Create port forward from your router to any K8s node for UDP traffic on the NodePort
- Create DNS entry to point 

These may be automated with a UPnP client and a dynamic DNS service.