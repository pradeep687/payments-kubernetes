# Payments Kubernetes Deployment

## Application End URL

The application is deployed on Kubernetes and exposed through a NodePort service.

**Live Application URL:**
https://1f43f5d7da9c-10-244-1-186-31263.papa.r.killercoda.com/
<img width="1329" height="533" alt="image" src="https://github.com/user-attachments/assets/df7580f7-71bc-43f1-a4d3-08b4be0940dd" />

The application is accessible through the above Killercoda public endpoint.
Note: The above endpoint is running on a temporary Killercoda environment and may become unavailable after the session expires. So attached the screenshots below at the end.



## Kubernetes Deployment

The Payments application consists of the following components:

* **web-frontend** – Exposed externally using a Kubernetes NodePort service.
* **order-processor** – Internal ClusterIP service responsible for processing orders.
* **payment-gateway** – Internal ClusterIP service handling payment processing.

### Secure Communication Flow

NetworkPolicies restrict service-to-service communication to the following flow:

```text
web-frontend
      |
      v
order-processor
      |
      v
payment-gateway
```

* `web-frontend` can communicate with `order-processor`.
* `order-processor` can communicate with `payment-gateway`.
* `payment-gateway` only accepts traffic from `order-processor`.
* The payment gateway is not directly exposed externally.


### Pods

All application pods are running successfully:

root@controlplane:~/payments-kubernetes/helm/payments$ kubectl get pods
NAME                               READY   STATUS    RESTARTS   AGE
order-processor-f95454695-zqf5f    1/1     Running   0          14m
payment-gateway-84ccbbdcff-22s77   1/1     Running   0          17m
web-frontend-5df46cd84b-vvwd9      1/1     Running   0          17m

### Services

root@controlplane:~/payments-kubernetes/helm/payments$ kubectl get svc
NAME              TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
kubernetes        ClusterIP   10.96.0.1        <none>        443/TCP        23d
order-processor   ClusterIP   10.96.245.134    <none>        8080/TCP       17m
payment-gateway   ClusterIP   10.109.153.123   <none>        8080/TCP       17m
web-frontend      NodePort    10.98.183.38     <none>        80:31263/TCP   17m

The `web-frontend` service is exposed externally using **NodePort 31263**.

### Deployments

root@controlplane:~/payments-kubernetes/helm/payments$ kubectl get deployments
NAME              READY   UP-TO-DATE   AVAILABLE   AGE
order-processor   1/1     1            1           14m
payment-gateway   1/1     1            1           17m
web-frontend      1/1     1            1           17m


### Helm Release

The application was deployed using Helm.

root@controlplane:~/payments-kubernetes/helm/payments$ helm list
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
payments        default         2               2026-08-13 18:45:17.384743285 +0000 UTC deployed        payments-0.1.0  1.0.0   

**Helm Release:** `payments`
**Chart Version:** `payments-0.1.0`
**Release Status:** `deployed`

## Application Verification

The application endpoint was tested successfully using:

```bash
curl http://172.30.1.2:31263
```

The request returned the Nginx welcome page, confirming that the NodePort, Kubernetes Service, Pod, and Nginx web server are reachable and functioning.

## Screenshots

### Application End URL

<img width="1329" height="533" alt="image" src="https://github.com/user-attachments/assets/422059fb-2302-42e6-956c-ca4d1dfcfd10" />


### Kubernetes Pods, Services, Deployments, Helm release

<img width="1309" height="522" alt="image" src="https://github.com/user-attachments/assets/441c23a8-6ec5-4061-ad74-a2cd52e0eb33" />

<img width="776" height="407" alt="image" src="https://github.com/user-attachments/assets/dad9d40f-5842-460a-bf87-1ff635b8026c" />

<img width="926" height="450" alt="image" src="https://github.com/user-attachments/assets/1240db7d-64ad-4fa4-b578-36fbe7c8a250" />
