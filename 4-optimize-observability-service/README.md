We will optimize the availability of our service and avoid the problems found in the third step.

1. Add limits to resource
    ```
    resources:
      limits:
        cpu: 120m
        memory: 520Mi
      requests:
        cpu: 80m
        memory: 120Mi
    ```

2. We are making the following changes to vmcluster to improve the availability of our services.
   - Deploy multiple replicas of the same service. Checkout the yaml:
     ```
       replicaCount: 2
       # This configuration will start two replicas (pods) of the vminsert service.
       # When one of the pods crashes, the other pod can still provide services
     ```
   - Deploy multiple copies of the same service to different EC2. Checkout the yaml:
       ```
       podAntiAffinity:
       #the replicas get the label app.kubernetes.io/name=vminsert.
       #The podAntiAffinity rule tells the scheduler to avoid placing multiple replicas with the app.kubernetes.io/name=vminsert label on a single node.
       preferredDuringSchedulingIgnoredDuringExecution:
       - podAffinityTerm:
         labelSelector:
         matchExpressions:
         - key: app.kubernetes.io/name
         operator: In
         values:
         - vminsert
         topologyKey: kubernetes.io/hostname
         weight: 100     
       ```
   - Enable the data multi-copy mode in the service. Checkout the yaml:
       ```
       replicationFactor: 2
       #replicationFactor instructs vminsert to store N copies for every ingested sample on N distinct vmstorage nodes.
       #This guarantees that all the stored data remains available for querying if up to N-1 vmstorage nodes are unavailable.
       ```
   - Add more resources to the service. Checkout the yaml:
       ```
       requests:
         cpu: "0.2"
       ```

   Update the yaml of vmcluster:
    ```
    $ kubectl apply -f 4-optimize-observability-service/4-2-high-availability-deployment/vmcluster.yaml
    $ kubectl get pod | grep vmcluster
    ```
   
   Now let's check our results, 

3. Isolate data or services of different tenants.
   - Deploy a separate vmagent for each tenant, these vmagents may be on different VPCs or even different clouds in production environment.  Checkout the yaml:
     ```
     metadata:
     name: vmagent-1
     namespace: observability
     ```
   
   - Each tenant's vmagent writes data to its own space.  Checkout the yaml:
     ```
     - url: "http://vminsert-vmcluster.observability.svc.cluster.local:8480/insert/1/prometheus/api/v1/write"
     ```
   - Apply
    ```
    $ kubectl apply -f 4-optimize-observability-service/4-3-multi-tenant-isolation/vmagent-tenant-1.yaml
    $ kubectl apply -f 4-optimize-observability-service/4-3-multi-tenant-isolation/vmagent-tenant-2.yaml
   
    # Let's confirm that vmcluster collects data from different tenants
   
    $ kubectl port-forward svc/vmselect-vmcluster -n observability 8481:8481 
   
    #  http://127.0.0.1:8481/select/1/vmui/?#/cardinality
    #  http://127.0.0.1:8481/select/2/vmui/?#/cardinality
    ```

   - Deploy a separate vmalert for each tenant, they have different configurations same as vmagent.
    ```
    $ kubectl apply -f 4-optimize-observability-service/4-3-multi-tenant-isolation/vmalert-tenant-1.yaml
    $ kubectl apply -f 4-optimize-observability-service/4-3-multi-tenant-isolation/vmalert-tenant-2.yaml
    ```

4. Provides disaster recovery for possible failures.
   When vmagent fails to report data due to vmcluster failure, we first store the data in the buffer of vmcluster. After the vmcluster service is restored, vmagent will make up the cached data to vmcluster to ensure that no data is lost.
   ```
   $ kubectl apply -f 4-optimize-observability-service/4-4-disaster-recovery/vmagent-tenant-1.yaml
   ```
   ```
   # now we set the replicaCount of vminsert to 0, thus injecting a write fault
   $ kubectl apply -f 4-optimize-observability-service/4-4-disaster-recovery/vmcluster.yaml
   
   # (-_-) zzz
   # you can sleep for three minutes.
   # (-_-) zzz
   ```
   ```
   # After waiting 3 minutes, restore vminsert with 4-2-high-availability-deployment/vmcluster.yaml
   
   $ kubectl apply -f 4-optimize-observability-service/4-2-high-availability-deployment/vmcluster.yaml
   ```
   ```
   # OK. Let's check the data difference between tenant-1 and tenant-2
   $ kubectl port-forward svc/vmselect-vmcluster -n observability 8481:8481 
   
   # http://127.0.0.1:8481/select/1/vmui/#/?g0.expr=vm_http_requests_total&g0.range_input=30m&g0.end_input=2023-06-12T10%3A27%3A11&g0.relative_time=last_30_minutes&g0.tenantID=2
   # We will see that part of the data is lost, while in tenant-2, no data is lost
   # http://127.0.0.1:8481/select/2/vmui/#/?g0.expr=vm_http_requests_total&g0.range_input=30m&g0.end_input=2023-06-12T10%3A27%3A11&g0.relative_time=last_30_minutes&g0.tenantID=2
   ```
   
