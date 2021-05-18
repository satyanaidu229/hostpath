After deploying manifest file.
kubectl get statefulset web
kubectl get pods -w -l app=nginx
kubectl get pods -l app=nginx
Each Pod has a stable hostname based on its ordinal index. Use kubectl exec to execute the hostname command in each Pod:
This will tell you all hostname of pods
for i in 0 1; do kubectl exec "web-$i" -- sh -c 'hostname'; done

To start new shell:
kubectl run -i --tty --image busybox:1.28 dns-test --restart=Never –rm

The SRV records point to A record entries that contain the Pods' IP addresses.
nslookup web-0.nginx
nslookup web-1.nginx
now exit

In one terminal, watch the StatefulSet's Pods:
kubectl get pod -w -l app=nginx

In second terminal:
kubectl delete pod -l app=nginx

After deleting pods again check their hostname:
for i in 0 1; do kubectl exec web-$i -- sh -c 'hostname'; done
again generate new shell and check their dns name.
The Pods' ordinals, hostnames, SRV records, and A record names have not changed, but the IP addresses associated with the Pods may have changed. In the cluster used for this tutorial, they have. This is why it is important not to configure other applications to connect to Pods in a StatefulSet by IP address.

If you need to find and connect to the active members of a StatefulSet, you should query the CNAME of the headless Service (nginx.default.svc.cluster.local). The SRV records associated with the CNAME will contain only the Pods in the StatefulSet that are Running and Ready.

If your application already implements connection logic that tests for liveness and readiness, you can use the SRV records of the Pods ( web-0.nginx.default.svc.cluster.local, web-1.nginx.default.svc.cluster.local), as they are stable, and your application will be able to discover the Pods' addresses when they transition to Running and Ready

Get the PersistentVolumeClaims for web-0 and web-1:
kubectl get pvc -l app=nginx

Write the Pods' hostnames to their index.html files and verify that the NGINX webservers serve the hostnames:
for i in 0 1; do kubectl exec "web-$i" -- sh -c 'echo "$(hostname)" > /usr/share/nginx/html/index.html'; done
for i in 0 1; do kubectl exec -i -t "web-$i" -- curl http://localhost/; done

Scaling a StatefulSet
In one terminal window, watch the Pods in the StatefulSet:
kubectl get pods -w -l app=nginx

In another terminal window, use kubectl scale to scale the number of replicas to 5:
kubectl scale sts web --replicas=5

Scaling Down
kubectl patch sts web -p '{"spec":{"replicas":3}}'

Get the StatefulSet's PersistentVolumeClaims:
kubectl get pvc -l app=nginx

Updating StatefulSets
RollingUpdate update strategy is the default for StatefulSets.
Rolling Update 
The RollingUpdate update strategy will update all Pods in a StatefulSet, in reverse ordinal order, while respecting the StatefulSet guarantees.
kubectl patch statefulset web -p '{"spec":{"updateStrategy":{"type":"RollingUpdate"}}}'







