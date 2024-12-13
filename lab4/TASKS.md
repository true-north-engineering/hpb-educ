# Lab 1 - CI, CD and GitOps

Before starting with lab exercises connect to the linux system hpb1.tn.hr with ssh or putty as userX where X is a number designated to you by the presenter.
Example:

```ssh userX@hpb1.tn.hr```

## Task 1 - S2I build

1. Clone the hpb-educ-src in your home folder.

```git clone https://github.com/true-north-engineering/hpb-educ-src.git```

2. Generate Containerfile for source in ```hpb-educ-src/todo-frontend``` based on the builder image ```registry.access.redhat.com/ubi8/nodejs-16:latest```

3. Build the container image from the generated Containerdile

## Task 2 - Tekton builds

1. Create an Openshift project called by your username, e.g. user1

2. Connect to the Openshift on ```https://console-openshift-console.ocp.hpb.tn.hr/```. In project ```hpb-cicd``` you have Tekton pipeline examples. Copy them to your project.

3. Create ```nexus-docker-auth``` secret that containes authentication data for ```nexus-docker.ocp.hpb.tn.hr```. Link that secret with ```pipeline``` service account.

4. Adjust the values in Tekton pipelines in your project:
    * Java version for build should be 11.
    * Location to push the builded image sould be ```nexus-docker.ocp.hpb.tn.hr/todo-api:<your_username>```
    * Correct any other mistakes in ```todo-api``` pipeline, ```s2i-java``` task.

5. Build ```todo-api``` and ```todo-frontend``` applications using Tekton pipelines. Use the VolumeClaimTemplate for persistence.

6. Check the images in Nexus.

## Task 3 - Deploy the solution

1. Check the ```https://github.com/true-north-engineering/hpb-educ-src``` source and the Helm template for the solution in ```helm/todo```folder.

2. In Argocd ```https://gitops.ocp.hpb.tn.hr``` create a new application with following properties:
    * Application name should be ```<your_username>-todo```
    * Application project should be ```default```
    * Repository URL should be ```https://github.com/true-north-engineering/hpb-educ-src.git```
    * Repository revision should be ```main```
    * Repository path should be ```helm/todo```
    * Deployment target should be local Openshift and namespace that you have creates in Task 2, Step 1
    * Select values files ```values.yaml```
    * Override the following properties:
        * fe.image.tag should contain the tag of your frontend image
        * api.image.tag should contain the tag of your api image
        * route.host should be ```todo-<your_username>.ocp.hpb.tn.hr```

3. Observe the applications in Openshift. Check the application logs and test the application.

4. Connect to mysql pod and create the necessary tables:

```
mysql -h localhost -utodo -ptodopassw0rd todo

create table Item (id bigint not null auto_increment, description varchar(255), done bool, primary key (id));
CREATE TABLE hibernate_sequence ( next_val BIGINT );
insert into hibernate_sequence values (1);
```

5. Restart the applications and test it.