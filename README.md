# Practical implementations of Kubernetes network policies using Calico to secure pod-to-pod communication


![arch](https://github.com/user-attachments/assets/179f8573-57a5-4af6-8eb1-f39f3c3e6fcc)



# Kubernetes Network Policies with Calico:
Kubernetes Network Policies provide a robust mechanism to control the communication between pods and other network endpoints. They are essential for securing your applications by defining rules that specify how pods can communicate with each other and other network resources. In this hands-on repo, we will cover the fundamentals and demonstrate how to implement network policies with a practical example involving pods in **three namespaces** i.e **dev, qa and prod**.

# Understanding Network Policies:
Network Policies are implemented using Kubernetes resources that define how pods are allowed to communicate with each other and other network endpoints. They use labels to select pods and define rules for ingress (incoming) and egress (outgoing) traffic.
Key Concepts:

•	**Pod Selector**: Selects the group of pods to which the policy applies. \
•	**Ingress Rules**: Define allowed incoming traffic. \
•	**Egress Rules**: Define allowed outgoing traffic. \
•	**Policy Types**: Can be either ingress, egress, or both.


# Features:

•	**Namespace Isolation**: Policies to isolate network traffic within specific namespaces, preventing unintended access between different parts of the cluster. \
•	**Granular Traffic Control**: Define rules that allow or block traffic between namespaces, ensuring that only authorized communication paths are permitted. \
•	**Environment Segmentation**: Securely manage traffic flow between development, QA, and production environments, helping to maintain a strict separation of concerns. \
•	**Ingress and Egress Management**: Control incoming and outgoing traffic at a fine-grained level, which is crucial for both security and performance optimization. \
•	**Port-Specific Rules**: Implement policies that allow or restrict traffic on specific TCP/UDP ports, such as allowing QA to communicate with production over a specific port.


# Files Included:
1.	**deny-traffic-inside-namespace.yaml**
o	Denies all traffic within a specific namespace, ensuring strict isolation.
2.	**Creating-ns-pods-labels.yaml**
o	Creates namespaces and applies necessary labels to pods for easier policy application.
3.	**block-all-traffic-namespace.yaml**
o	Blocks all incoming and outgoing traffic to and from a specific namespace, providing a secure baseline.
4.	**Allow-Traffic-single-namespace.yaml**
o	Allows internal traffic within a single namespace, enabling services within the namespace to communicate freely.
5.	**allow-dev-to-prod-traffic.yaml**
o	Permits traffic from the development namespace to the production namespace under controlled conditions.
6.	**allow-prod-to-qa-tcp-8888.yaml**
o	Allows communication from production to QA namespace over TCP port 8888, ensuring specific services can interact.
7.	**allowing-coredns-ingress-prod.yaml**
o	Allows ingress traffic to CoreDNS in the production namespace, ensuring that DNS services remain accessible.


# HandsOn Practicals:
o Let's implement all one by one:

# 1.	**deny-traffic-inside-namespace.yaml**

Lets Create two pods in a namespace and check the network connectivity b/w them:

```bash
kubectl run trouble1 --image=rajpractise/troubleshootingtools:v1 -n default
kubectl run trouble2 --image=rajpractise/troubleshootingtools:v1 -n default
```

![1podtopod-bydefaultcommyes](https://github.com/user-attachments/assets/27ac1a20-a86c-4ca1-8315-40222858a42b)



We could see that by default pod 1 i.e trouble1 is able to communicate with pod2 i.e trouble2 in same namespace.

# Now lets apply the policies to restrict traffic inside namespace :

first create policy for **ingress**:

```bash
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: deny-ingress
  namespace: default
spec:
  podSelector: 
    matchLabels:
      run: trouble1
  policyTypes:
  - Ingress
```

# Now lets check the connectivity:

![2-pod1-ingress-block](https://github.com/user-attachments/assets/edb645c4-0e6a-4a9b-b4ce-d6c8f445acde)


We can see that trouuble1 pod is able to communicate with trouble2 pod but from trouble2 pod we can't communicate with trouble1 pod because we have blocked ingress traffic for trouble1 pod.


# Now  create policy for **egress**:

```bash
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: deny-egress
  namespace: default
spec:
  podSelector: 
    matchLabels:
      run: trouble1
  policyTypes:
  - Egress
```

# Now lets check the connectivity:


![3-pod1-egress-block](https://github.com/user-attachments/assets/a5a22bd2-d0ac-466b-9b3c-35b14b0cee22)



We can see that since we have applied both ingress and egress rule so now neither trouble1 nor trouble2 pod is able to communicate within same namespace.


########################################################################################################################



2.	**Creating-ns-pods-labels.yaml**


   Now lets create three namespaces **dev,prod and qa** and apply labels for all:

```bash
#Creating Namespaces
kubectlbectl create ns prod
kubectlbectl create ns dev
kubectlbectl create ns qa

#Labling Namespaces
kubectlbectl label ns prod nsp=prod
kubectlbectl label ns dev nsp=dev
kubectlbectl label ns qa nsp=qa
```



![4-threenscreatedandlabelapplied](https://github.com/user-attachments/assets/ef543ba3-8c5e-45fd-8bd1-eae8c621b779)



# Now lets deploy two pods in each namespace i.e dev, qa and prod:

```bash
kubectl run -n prod prod1 --image=rajpractise/troubleshootingtools:v1 -l ns=prod
kubectl run -n prod prod2 --image=rajpractise/troubleshootingtools:v1 -l ns=prod

kubectl run -n dev dev1 --image=rajpractise/troubleshootingtools:v1 -l ns=dev
kubectl run -n dev dev2 --image=rajpractise/troubleshootingtools:v1 -l ns=dev

kubectl run -n qa qa1 --image=rajpractise/troubleshootingtools:v1 --labels ns=qa
kubectl run -n qa qa2 --image=rajpractise/troubleshootingtools:v1 --labels ns=qa
```




![5-twopodscreatedindevqaprodeach](https://github.com/user-attachments/assets/7c6e8405-aa61-44f3-9039-4d19b7bbf24e)





# Now lets try to get into each pod and try to ping to other two pods and see the default behaviour:
from dev pod: ping qa and prod pod
from qa pod: ping dev and prod pod
from prod pod: ping dev and qa pod


```bash
kubectl exec -it  prod1 -n prod -- ping -c 3  100.102.231.5 \
&& kubectl exec -it  prod2 -n prod -- ping -c 3   100.118.47.197 \
&& kubectl exec -it  prod1 -n prod -- ping -c 3   100.118.47.198 \
&& kubectl exec -it  prod2 -n prod -- ping -c 3   100.102.231.6


kubectl exec -it  dev1 -n dev -- ping -c 3    100.118.47.198 \
&& kubectl exec -it  dev2 -n dev -- ping -c 3  100.102.231.6 \
&& kubectl exec -it  dev1 -n dev -- ping -c 3 100.102.231.4 \
&& kubectl exec -it  dev2 -n dev -- ping -c 3 100.102.231.7


kubectl exec -it  qa1 -n qa -- ping -c 3     100.102.231.4  \
&& kubectl exec -it  qa2 -n qa -- ping -c 3  100.102.231.7  \
&& kubectl exec -it  qa1 -n qa -- ping -c 3  100.102.231.5 \
&& kubectl exec -it  qa2 -n qa -- ping -c 3  100.118.47.197
```


![6-pingingothertwopodsfromeachpod](https://github.com/user-attachments/assets/c02d7897-47f1-47c5-afee-7c99ed0fc7ed)




We could see that by default each pod from one namespace is able to communicate with two other pods in different namespaces.


now lets try to block all traffic :

# 3.	**block-all-traffic-namespace.yaml**
o	Blocks all incoming and outgoing traffic to and from a specific namespace, providing a secure baseline.


```bash
---
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: deny-all-traffic-prod
  namespace: prod
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress

---
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: deny-all-traffic-dev
  namespace: dev
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress

---
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: deny-all-traffic-qa
  namespace: qa
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
```


Now lets try again to communicate pod from one namespace to other pods in different namespaces.

from dev pod: ping qa and prod pod
from qa pod: ping dev and prod pod
from prod pod: ping dev and qa pod


```bash
kubectl exec -it  prod1 -n prod -- ping -c 3  100.102.231.5 \
&& kubectl exec -it  prod2 -n prod -- ping -c 3   100.118.47.197 \
&& kubectl exec -it  prod1 -n prod -- ping -c 3   100.118.47.198 \
&& kubectl exec -it  prod2 -n prod -- ping -c 3   100.102.231.6


kubectl exec -it  dev1 -n dev -- ping -c 3    100.118.47.198 \
&& kubectl exec -it  dev2 -n dev -- ping -c 3  100.102.231.6 \
&& kubectl exec -it  dev1 -n dev -- ping -c 3 100.102.231.4 \
&& kubectl exec -it  dev2 -n dev -- ping -c 3 100.102.231.7


kubectl exec -it  qa1 -n qa -- ping -c 3     100.102.231.4  \
&& kubectl exec -it  qa2 -n qa -- ping -c 3  100.102.231.7  \
&& kubectl exec -it  qa1 -n qa -- ping -c 3  100.102.231.5 \
&& kubectl exec -it  qa2 -n qa -- ping -c 3  100.118.47.197
```



![7-denyingressegressforallpodsandcheckcomm](https://github.com/user-attachments/assets/e6454dcb-2442-4814-9f10-a82fe465f8ad)



We could see that all communication from each namespace is blocked.


#########################################################################################################################



4.	**Allow-Traffic-single-namespace.yaml**

Lets see the current behaviour of resources in prod namespace:



![8-podsinprodnsaccessblockedduetoalreadyappliedblockedpoliciesingressegress](https://github.com/user-attachments/assets/f50f2940-1200-44d6-8f87-377b9c17aa65)



Since we blocked all ingress and egress traffic previously in prod namespace hence pods in the **prod namespace** are not able to communicate each other.



Now lets Allows internal traffic within a single namespace i.e prod , enabling services within the namespace to communicate freely.


```bash
#This Policy Will Allow Traffic Between PODs in prod NS with Label ns=prod

kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: allow-traffic-within-prod-namespace
  namespace: prod
spec:
  podSelector: 
   matchLabels:
    ns: prod
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          ns: prod
  egress:
  - to:
    - podSelector:
        matchLabels:
          ns: prod
```



# Now lets check the communication within the prod namespace:


![pol](https://github.com/user-attachments/assets/74658bbf-ce24-456e-9dca-0f108defd497)



each pod prod1 and prod2 are able to communicate each other now...


######################################################################################################################################



# 5.	**allow-dev-to-prod-traffic.yaml**
o	Permits traffic from the development namespace to the production namespace under controlled conditions.


lets see the current behaviour of connectivity from dev namespace to prod namespace:


![10-checkconnfromdevtoprodns](https://github.com/user-attachments/assets/24fb4372-83a6-4c0e-bf85-92a12b969650)



Currently pods in dev namespace is unable to communicate with pods in the prod namespace.


Lets apply the policies and enable communication from dev ns to prod ns:


```bash
#Allow Ingress Traffic From dev To prod applied at prod Namespace
---
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: allow-traffic-from-dev-to-prod
  namespace: prod
spec:
  podSelector: 
   matchLabels:
    ns: prod
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          nsp: dev
    - podSelector:
        matchLabels:
          ns: dev

#Allow Egress Traffic From dev To prod applied at dev Namespace
---
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: allow-traffic-from-dev-to-prod
  namespace: dev
spec:
  podSelector: 
   matchLabels:
    ns: dev
  policyTypes:
  - Egress
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          nsp: prod
    - podSelector:
        matchLabels:
          ns: prod
```


# Now lets check the communication:


![11-policy-4-devtoprodcommstarted](https://github.com/user-attachments/assets/4e110a1d-6760-47a4-808f-81f6f51a843d)




We could see that now pods in dev namespace is able to communicate with pods in the prod namespace...


########################################################################################################################



# 6.	**allow-prod-to-qa-tcp-8888.yaml**

o	Allows communication from production to QA namespace over TCP port 8888, ensuring specific services can interact.

```bash
---
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: allow-traffic-from-prod-to-qa-on-tcp8888
  namespace: qa
spec:
  podSelector: 
   matchLabels:
    ns: qa
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          nsp: prod
    - podSelector:
        matchLabels:
          ns: prod
    ports:
    - protocol: TCP
      port: 8888

---
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: allow-traffic-from-prod-to-qa-on-tcp8888
  namespace: prod
spec:
  podSelector: 
   matchLabels:
    ns: prod
  policyTypes:
  - Egress
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          nsp: qa
    - podSelector:
        matchLabels:
          ns: qa
    ports:
    - protocol: TCP
      port: 8888
```


# Lets check whether pods in prod namespace is able to communicate with pods in the QA namespace over TCP port 8888 , ensuring only specific services can interact:




![12-policy-5-prodtoqacommstarted](https://github.com/user-attachments/assets/ea99bcec-652e-475d-9235-44bb17a552c9)




Communication is successfull between resources from prod to the resources in qa namespace over a specific port...


###################################################################################################################################




# 7.	**allowing-coredns-ingress-prod.yaml**

o	Allows ingress traffic to CoreDNS in the production namespace, ensuring that DNS services remain accessible.


Lets check the current behaviour whether pods in the prod namespace is able to communicate over the internet or not.



![13-fromprodpodtointernetaccessbydefntallow](https://github.com/user-attachments/assets/0a65ce03-8879-439e-ab8d-db9b41746277)



Communication is blocked over the internet.


Lets apply the policy to 	Allows ingress traffic to CoreDNS in the production namespace, ensuring that DNS services remain accessible.



```bash
---
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: allow-traffic-from-prod-to-coredns-on-tcp53
  namespace: prod
spec:
  podSelector: 
   matchLabels:
    ns: prod
  policyTypes:
  - Egress
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: kube-system
    - podSelector:
        matchLabels:
          k8s-app: kube-dns
    ports:
    - protocol: TCP
      port: 53
    - protocol: UDP
      port: 53
    - protocol: TCP
      port: 9153
```


# Lets check the connectivity now:


![14-policy-6-DNSresworksNetPolcorrectlyallowingtraffictoCoreDNSonport 53](https://github.com/user-attachments/assets/54e14f3e-c9b8-485c-b597-33bd9f61ffab)



# We could see that now ingress traffic to CoreDNS in the production namespace is enabled making sure that DNS services remain accessible.




# Hence now all the different network policies has been successfully implemented and tested...



# Conclusion
Kubernetes Network Policies are a powerful tool for securing your cluster by controlling pod communication. By understanding and implementing these policies, you can significantly enhance the security of your applications. Experiment with different policies and use this guide as a reference to master Network Policies in Kubernetes.


































