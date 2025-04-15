# Kubernetes_by_Ayushi
What is kubernetes?(k8s)
 kubernetes is an open source orchestration platform ( ye containers ko automate , scale , deploy aur manage krta hai)

 Why do we need kubernetes?
 image you have 50 Docker containers now it is difficult to manage them like start, stop , monitor and if any container crashes then how can we restart manually
 Kubernetes automatically container ko restart krega, Load jyada h naye container ko launch karega 
 container down hai ? health check karega aur replace karega
 Blue green deployments, Rolling updates bhi easy bna deta hai.

Kubernetes Architecture is divided into 2 parts
1. Master Node(control plane)- Brain of the cluster
   a)API Server:kubectl controller, schedular sab isse baat krte h client req yahi aati h authentication, authorization , validation sab yahi hota hai.
    Its like a gatekeeper- har command chahe kubectl se aaye ya dashboard se , phle yahi aata h, ye REST API provide krta h cluster se baat krne ke liye, hr resource (Pod, Deployment, etc ) yahin se create/update hote hain.
   jab kubectl apply -f pod.yaml dia >- API Server accept karega -> etcd me store hoga -.baki components ko bolega kam kro
   b) etcd : is a key-value store, pure ccluster ka state yahi pr save hota hai, agar tumhare clutser me kya chal rha ha, kya desired state hai - sab kuch etcd me hota hai.
   example) 3 pods chahiye etcd me likha hoga
    pod crash ho gya: controller manager check karega aur fir se banayega.
   c) Controller Manager : is a component which constantly monitor the cluster actual state. aur agar kio difference ho desired state se to use fix krta hai.
    Desired state =tumne bola tha 3 pods chhaiye
    Actual state = 1 poddown ho gya
    controller manager bolega chinta mat kr main naya pod bana deta hu.
   Controller manager ke pass multiple controllers hote hain jo cluster ke alag -alag objects ko handle krte hain.
   Node - Controller: Check krta hai ki node zinda hai ki nhi. Agar node down ho gyi to uske pods ko dusri node pe shift krne me help karega.
   Replication Controller- Ensure krta hai ki specified no. of pods replicas hamesha chla rhe hain, Agar kam ho gye toh naya create karega.
   Deployment - controller - Rolling updates, roolback, scale -up/down sab handle krta hai.
   Job Controller - Ek ya jyada pods ko run krta hai jab tak task complete na ho jaye
   Endpoint Controller: Service ke corresponding pods ke endpoints maintain krta hai.
   Service Account & Token Controllers: New namespaces me default service account aur token generate krta hai, authentiation ke liye.
   d) Scheduler: Pod assign krna node pe. jab ek new pod create hota hai initially wo pending state me hota hai kube schedular resource availiability taints, affinity etc check krke best node select krta hai.
   Factors dekhta hai: Node ka CPU/RAM
                       Node healthy hai ya nhi
                       Taints/Tolerations, node Affinity.

   WORKER NODE: (jahan actual containers chalte hain)
   a) kubelet: yeh control plane ka agent hai , yeh hr node pe chalta hai , node ke pods ko monitor krta hai. Pod Spec leke container run krta hai via container runtime, health check run krta hai.
   Master se instruction leta hai (e.g, "nginx pod chalao") aur node pe wo pod chalata hai.
   ye continously check rkta hai ki containers sahi se chal rhe ha ya nhi.
   b) kube-proxy : yeh network traffic handle krta hai, hr node pe chalta hai service se pod tak traffic route krta hai, Load balancing krta hai.
   Pods ke beech communication, ya pods se service tak ka traffic ye hi handle krta hai.
   ye iptables ya IPVS ka use krke traffic forward krta hai.
   c) Container Runtime : Ye actual container ko chalata h, Mostly Docker, containerd , CRI-O.
     tumhare pod me jo container image h wo yahi se launch hoti h
   d) Core DNS: har pod ya service ko ek dns name milta hai
   nginx.default.svc.cluster.local
   Pod to Pod communication me help krta h.
   e) Container network interface : ek standard/ specification hai jo batata hai ki containers ko network kase assign kiye jaye.
   Jab bbhi kio pod banta hai , to usko IP address dena, networking rules apply krna, routing set krna yeh sab CNI pugin ka kam hota hai.
   Kubernetes me CNI ka role kya hai?
   kubernetes khud networking nhi krta wo bs instruction deta hai :bhai pod bna dia ab ise ip de do connect kr do network se" Aur fr CNI ka plugin aata h picture me ye setup krta h, popular plugins Flannel, Calico , Weave, Cilium, canal
   - IP address  ,  Routing  ,  DNS  ,  ]Firwall rules

 Workloads
 Pod , ReplicaSets, Deployments, statefulset, DaemonSet 
 What is a Pod? -Basic unit in kubernetes , ye ek se jyada containers ko group krta hai jo ek share netwrok namespace (IP , ports) aur storage (volumes use krta h) har pod ka ek unique ip hota hai jo commmunication ke liye use hota hai.
 What is a Replica set? - ka kaam hai ensure krna ki cluster me hamesha ek specified no. of pod replicas chal rhe ho, ye pod failures ko handle krta hai agar kio pod down ho jaye, toh naya pod launch kr deta hai taki desired count maintained rhe.
                         ek replicaset ke jariye hum yeh declare krte h kitne no of identical pod chahiye. agar pod kisi wajah se crash ho jata hai, Replicaset turant ek replacement create kr deta hai.
                         Replicaset ke yaml me spec.replicas key se desired replica count set krte hain aur selector se define krte hain ki kaun se pods target krne hain.
What is Deployment ? - Higher level abstraction hai jo replicasets ko manage krta hai jo replicaset of manage krta hai, iske use krne se hum application ke lifecycle ko easily manage kr sakte hain - deployment , updates, rollbacks, aur scaling.
                     Features>>> Rolling Updates , Declarative Update , Rollback Feature
Deployment VS Replicaset
Replicaset : sirf pod count ensure krta hai
Deployment : Replicaset ko control krne ke sath sath application update strategy (roollout, rollback) bhi manage krta hai.

StatefulSet & DaemonSet:
StatefulSet: For Sateful Applications: Statefulset un applications ke liye hota hai jinko persistent identity chhahiye ex databases. hr pod ko stable, unique network identity (like hostname) milta hai (Pod-0,pod-1 etc)
Daemonsets : for Node-level pods : Daemonset ensure krta h cluster ke hr node pe ek specific pod run ho. ye specifically useful hai monitoring tools ya log collectors jise hr machine pe run krna ho.
Analogy: socho hr building (node) me ek security gurad (pod) chhaiye. Daemonset ek aisa mechanism hai jo hr building me automatically guard bhej deta hai. 

Kubernetes Services- Connecting your applications
what is a Service?--service ek stable network endpoint hai jo ek logical group of pods ko represent krta hai ye ensure krta hai ki pods ke dynamic IP chnages ke nawajood, ek constant access point mil jaye.
use case: Pods ke set ko ek DND name aur stable IP provide krna, Load balancing krna taki traffic evenly pods me dirtribute ho, Pods ke lifecycle chnages (restart, update) ke bawajood connectivity maintain rakhna.
Types of Services :
a) ClusterIP (Default): pods ke liye ek internal, virtual ip banata hai jo sirf cluster ke ander hi accessible hota hai.
use case: jab tumhare services ko sirf internal communication (microservices architecture) ke liye use krna hota hai.
b) Nodeport: har node ke ip ke sath ek specific port open krta hai.
External traffic node ke IP:Port ko hit krke pods se communicate kr skta hai.
Use case: Testing ya development env me jab directly node ke through access chahiye.
c) LoadBalancer: Cloud providers (like AWS , GCP ) me external load balancer create krta hai jo service ke sath bind ho jata hai.
Automatically external IP provide krta  hai.
Use case: Production enviroment me jab high availability aur scalability required ho.

How Services work in kubernetes
1. Label Selectors:
   Services use label selectors to automatically pick the right group of pods. ex agar pod me label app: myapp diya ho, to service configuration me bhi same selector specify krte ho.
2.Endpoints:
   Services ke IP ke piche actual pod IP addresses list hoti hain, yeh endpoints dynamically update hote hain jab pods add/delete ho rhe hote hain.
3.DNS- Based Service Discovery:
   Cluster ke ander, har service ko ek DNS name milta hai (e.g myapp-service.default.svc.cluster.local), jisse pods easily connect ho skate hain.
4.Load Balancing:
   Service automatic load balancing krta hai incoming traffic ko available pods me distribute krke.
   Analogy: Pod= taxi, Replicaset= taxi company ki fleet jo ensure krte h hamesha certain number me taxi available ho, Service =booking system jo automatically available taxi assign krta hai.

Kubernetes Networking : 
1. Pod to Pod:
    har pod ko ek unique ip Address milta hai ( CNI se),  Sabhi pods same flat network me hote hain, isliye ek pod dusre pod ko direct IP se access kr sakta hai - bina NAT ke, lekin - IPs dynamic hote hain, to isse stable solution nhi mana jata.
2. Pod to Service:
   isliye service banay jata hai, ye ek permanent IP+DNS name deta hai jo multiple pods ko represent krta hai, agar pod delete ho gya, naye wale ko connect krne me kio dikkat nhi
3. Cluster ke bahar se agar traffic lana hai to teen tareeke hate hai
   Node Port -Cluster ke node ke IP+ port ke through access
   LoadBalancer- Cloud provider ke external LB se connect hota hai
   Ingress- Domain based routing krta hai , reverse proxy jase

