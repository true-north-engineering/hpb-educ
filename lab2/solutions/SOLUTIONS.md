#Lab1 - Openshift access:

Open OpenShift console:
`https://console-openshift-console.ocp.hpb.tn.hr`

Login with your `IBM account`.

In top right corner, click on your `Username` and select `Copy login command`.

Open terminal and use `oc` to login into cluster.

Check your ID by running `oc whami`.

List all visible projects.
```
$ oc projects
```

#Lab2 - Create Project

Create a project for rest of the labs.

Use either Openshift web console, or `oc` command to create a new project named `userX`, same as your user on day 1.

```
$ oc new-project userX
```

#Lab3 - Deploy `PostgreSQL (Ephemeral)` database from template.

Login to OpenShift web console.

Choose the `Developer` console from top left dropdown.

Add `PostgreSQL (Ephemeral)` database to your project.

Set `Service Name` to `animalsdb`
Set `Connection Username` to `animal`
Set `Connection Password` to `4n1m4l!`
Set `Database Name` to `animals`

Leave rest of the fields blank.

Wait for `PostgreSQL (Ephemeral)` to be ready.

Run:
```
$ oc get pods
```
And wait for output similar to:
```
NAME                                 READY   STATUS      RESTARTS   AGE
animalsdb-1-q9lm7                    1/1     Running     0          3h20m
```
`animalsdb` pod should be in `Runnning` state.

#Lab4 - Create image pull secret

Login to OpenShift console.

Select secrets in ether administrator console under workloads, or developer console at bottom of menu.

Choose `Create` and select `Image pull secret` from dropdown menu.

Use following parameters
```
name: nexus-pull-secret
url: nexus-docker.ocp.hpb.tn.hr`
user: admin
password: <user password from day1>
```

#Lab5 - Deploy `animals` application using `oc` command

`animals` is an aplication that tells you what animals live on your servers. When deploying app you can choose what animal you want to check by setting `ANIMAL` environment variable, and if that animal exists, it will tell you their names.

To know whether animal exists, it checks in PostgreSQL database you just created. Let's check if there are any cats.

Create the deployment using oc command:
```
$ oc apply -f animals-cat-deployment.yaml
```

#Lab 6 - Expose `animals` application

Use `oc` command to expose `animals` application.
```
$ oc expose deployment cat
```
Once deployment is exposed, find the service that was created using `oc` comand and in web interface.
```
$ oc get services
```

#Lab 7 - Allow external acces for `animals` application with a route

Use either web console or oc command to create a route for `animals` application.
```
$ oc expose service cat
```
Once you have the route created, note the url that was created.
```
$ oc get routes
```
#Lab 8 - Call `animals` application

Open the url from `animals` route in your browser and see the result.
In previous lab you should get output similar to:
```
NAME   HOST/PORT                  PATH   SERVICES   PORT   TERMINATION   WILDCARD
cat    cat-userX.ocp.hpb.tn.hr          cat        5000                 None
```
In this case `cat-userX.ocp.hpb.tn.hr` is the url to service. Open it in web browser to get the result:
```
It's a cat named Sundance living on cat-d77c74f76-rnqpd.
```
Each time you call the service you get a different name. The `cat-d77c74f76-rnqpd` in exmaple is the name of the pod that served the request.

You can do the same thing in console using curl:
```
$ curl http://animals-userX.ocp.hpb.tn.hr/
It's a cat named Lucy living on cat-d77c74f76-rnqpd.
```

#Lab 9 - Add environmet variables from secret

When you deployed your `PostgreSQL` database, a `secret` with the same name was created. Inside that secret are parameters to connect to DB.

Open the deployment config of `animals` application.

Under environment tab replace static environment variables with those from `PostgreSQL` secret.

Compare the new `yaml` configuration with your initial deployment file.

You should see new config in yaml file:
```
            - name: DB_USER
              valueFrom:
                secretKeyRef:
                  name: animalsdb
                  key: database-user
            - name: DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: animalsdb
                  key: database-password
            - name: DB_NAME
              valueFrom:
                secretKeyRef:
                  name: animalsdb
                  key: database-name
```
#Lab 9 - Add configurations from configmap

Now that we have database config from secret, externalize rest of the config by creating a config map.

Add following values to configmap:
```
ANIMAL
DB_HOST
DB_PORT
```
You can find the configmap in git repository:
```
$ oc apply -f animals-cat-configmap.yaml
```
Add the values from configmap to `animals` application pod.

Remove current environment variables (`ANIMAL`, `DB_HOST`, DB_PORT`), and add the new entry in deployment:
```
          envFrom:
            - configMapRef:
                name: cat
```
#Lab 10 - Deployment strategies

Increase the replica count of `animals` app to 3.
```
$ oc scale deployment cat --replicas 3
```
Then start new rollout of deployment.
```
$ oc rollout restart deployment cat
```
Observe pods and notice how for each new pod starting, an old one is destroyed.

Now change strategy to Recreate.
Edit the `cat` deployment:
```
$ oc edit deployment cat
```
Replace:
```
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
```
With:
```
  strategy:
    type: Recreate
```
Start another rollout.
```
$ oc rollout restart deployment cat
```
Notice that this time all of the old pods get destroyed before new pods are created.

Scale the `cat` pod replicas to 1.
```
$ oc scale deployment cat --replicas 1
```