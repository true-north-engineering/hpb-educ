# Lab 0 - Prerequisites

To beggin, apply `prerquisites.yaml` to your project.
```
$ oc apply -f prerequisites.yaml
```

This will create objects needed for this lab.

# Lab 1 - Change number of replicas of `cat` deployment

Use either web console or oc command to edit the `cat` deployment and increase number of replicas to 3.

```
$ oc scale deployment cat --replicas 3
```

Try refreshing the `cat` application route in browser and notice that you are still getting same backend each time. That is because routes in openshift use cookies by default to track related connections.

Now try using curl. You will notice that requests are served by different backends.
```
$ curl http://cat-userX.ocp.hpb.tn.hr/
```

I you change number of replicas you can see that openshift automatically balances all pods spreading load as you increase number of replicas and removing pods automatically from pool as you decrease number of replicas, completely transparent to client.

Set number of replicas back to 1.
```
$ oc scale deployment cat --replicas 3
```

# Lab 2 - Add liveliness and readiness probes

`animals` application provides `livez` and `readyz` endpoints to enable health checks.

Add liveleness and readiness probes to `cat` deployment. You can do it in multiple ways, either through Topology view by selecting app and selecting `Add health checks`, or via Administration console by viewing deployment, or via oc by manually editing deployment.

Use `/readyz` on port `5000` for readiness probe, and `/livez` on port `5000` for liveliness probe.

```
$ oc apply -f  cat-with-probes-deployment.yaml
```

You can check container logs and see that `readyz` and `/livez` endpoints are called every few seconds.

# Lab 3 - Check if there are any dragons

We saw that many cats live in our pods, let's check if there are any dragons.

Change the `ANIMAL` variable in configmap to `dragon`.

You can observe following things:

- a new pod was started but it's not ready
- current pod is not terminated
- you can still access the cats on your route

This is because animals application checks whether animals exist in PostgreSQL database. On initialization, it creates entries for cats and dogs, but no dragons. If the animal isn't in database, application never becomes ready. Since new deployment isn't ready, and we use `RollingUpdate` strategy, the pod is never replaced.

You can check the pod logs and see that `readyz` endpoint is returning `500` code.

# Lab 4 - Enable dragons

To enable dragons, we need to add an entry to database. Either use web console to open a terminal to `animalsdb` pod, or use `oc` command to exec remotely.

```
$ oc get pods
NAME                     READY   STATUS      RESTARTS      AGE
animalsdb-1-zlwx8        1/1     Running     1 (13h ago)   26h
$ oc exec -it animalsdb-1-zlwx8 -- sh
```
Once in pod, we can connect to database and insert `dragon` into `animals` table:
```
$ psql animals
INSERT INTO animals (type) VALUES ('dragon');
```

Once the value is inserted, observe that animals pod became ready, and old pod got terminated. Opening the `cat` route now lists all the dragons on our servers.

# Lab 5 - Enable metrics collection

`cat` application exposes prometheus metrics at `/metrics` endpoint.

Open the route in browser and check `/metrics` endpoint to view the metrics.

To add metrics collection to Openshift, add `ServiceMonitor` object to your project.

Use the following template:
```
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: cat-monitor
spec:
  endpoints:
    - interval: <interval>
      path: <path_to_metrics>
      port: http
      scheme: http
  selector:
    matchLabels:
      <cat app label>
```

```
$ oc apply -f cat-metrics-service-metrics.yaml
```

Check the application logs and see that `/metrics` endpoint it requested every `<interval>`.

```
$ oc get pods
NAME                     READY   STATUS      RESTARTS      AGE
cat-7fcd74845f-8j82j     1/1     Running     0             13m
$ oc logs cat-7fcd74845f-8j82j
```

# Lab 6 - Find application metrics in observability tab

In Administrator web console open Observe tab and select Metrics. Try to find metrics from `/metrics` endpoint of `cat` application.

Open `cat` application url in web browser and go to `/metrics` endpoint (`http://cat-userX.ocp.hpb.tn.hr/metrics`). Look at the available metrics, for example `flask_http_request_total`.

Enter `flask_http_request_total` into search query and press run queries.

# Lab 7 - Find application logs in observability tab

In Administrator web console open Observe tab and select Logs. Try to find logs from your `cat` application.

Under filters select your namespace and your pod.

# Lab 8 - Scale `cat` app to 10 replicas

Scale the `cat` app to 10 replicas.
```
$ oc scale deployment cat --replicas 10
```
Observe the number of replicas and diagnose the problem.

Find the `ReplicaSet` for `cat` deployment and look at the events. You will see error similar to:
```
Error creating: pods "cat-7fcd74845f-v5hlg" is forbidden: exceeded quota: pod-limit, requested: pods=1, used: pods=5, limited: pods=5
```

Your namespace has a quota limit of 5 pods so no new pods can be scheduled.

Once you find out the problem scale the `cat` app back to 1 replica.

```
$ oc scale deployment cat --replicas 10
```

# Lab 9 - Run all `cat` instances on a specific node

One of the nodes in cluster has `catnode` label set to `"true"`. Configure `cat` deployment to only run on that node.

Add:
```
  nodeSelector:
    catnode: 'true'
```
to `cat` deployment.

Look at the node where `cat` application is deployed and find the label `catnode: "true"` on the node.

# Lab 10 - Deploy `unicorn` app

There is a very special application available in `unicorns` namespace called `unicorns`. Unicorns are very particular and require to run under user id `1000`.

You can find the `unicorns` image at `image-registry.openshift-image-registry.svc:5000/user50/unicorns:latest`.

To pull the image from repo you have create a new `serviceaccount` and add `system:image-puller` role to it in `unicorns` project. The role also has to have `anyuid` `scc`.

Create the service acount:
```
$ oc create sa unicorn
```
Add `image-puller` role to service account:
```
$ oc adm policy add-role-to-user system:image-puller system:serviceaccount:userX:unicorn -n unicorns
```
Add `anyuid` `scc` to service account:
```
$ oc adm policy add-scc-to-user anyuid system:serviceaccount:user49:unicorn
```

Make sure you don't add image pull secret to `unicorns` deployment.

Use the same config for database as for `cat` application.

```
$ oc apply -f unicorns-deployment.yaml
```