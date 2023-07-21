# parodos-operator


## Creation
When you want to delete it
You can either run `make -f Makefile_reference init` to create the helm charts from the manifests and initiliaze the helm operator 
```bash
$ make -f Makefile_reference init
curl -LO https://github.com/parodos-dev/parodos/releases/download/v1.0.19/manifests.yaml
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
100 10040  100 10040    0     0  34476      0 --:--:-- --:--:-- --:--:-- 34476
cat manifests.yaml | helmify helm-charts/parodos-v1-0-19
find helm-charts/parodos-v1-0-19/templates/*.yaml -type f -exec sed -i "s/{{ include \"parodos-v1-0-19.fullname\" . }}-//g" {} \; 
yq -iY '.postgresSecretsTh27274644.pgdata="/var/lib/postgresql/data/mydata"' helm-charts/parodos-v1-0-19/values.yaml 
yq -iY '.postgresSecretsTh27274644.postgresDb="parodos"' helm-charts/parodos-v1-0-19/values.yaml 
yq -iY '.postgresSecretsTh27274644.postgresPassword="parodos"' helm-charts/parodos-v1-0-19/values.yaml 
yq -iY '.postgresSecretsTh27274644.postgresUser="parodos"' helm-charts/parodos-v1-0-19/values.yaml 
chmod -R +r helm-charts
/usr/local/bin/operator-sdk init --plugins helm --domain redhat.com --kind Parodos --helm-chart helm-charts/parodos-v1-0-19
Writing kustomize manifests for you to edit...
Creating the API:
$ /usr/local/bin/operator-sdk create api --kind Parodos --helm-chart helm-charts/parodos-v1-0-19
Writing kustomize manifests for you to edit...
Created helm-charts/parodos-v1-0-19
Generating RBAC rules
WARN[0000] The RBAC rules generated in config/rbac/role.yaml are based on the chart's default manifest. Some rules may be missing for resources that are only enabled with custom values, and some existing rules may be overly broad. Double check the rules generated in config/rbac/role.yaml to ensure they meet the operator's permission requirements. 
cp Makefile_reference Makefile
```

Or the following manual steps.

### Manual steps

```bash
$ make -f Makefile_reference download-manifest generate-helmchart 
```

Please note that the parodos version is hold by the parameters `PARODOS_VERSION` and `PARODOS_VERSION_DNS` in the `Makefile` file.

This will create the folder helm-charts/parodos-{PARODOS_VERSION_DNS}. We have to DNSify the name to avoid error while deploying the operator.

`generate-helmchart` target is replacing strings in templates and `values.yaml` and it is also granting `read` rights to the `helm-charts` folder, to change this behaviour take a look in the `Makefile` and `Makefile_reference` files. 
`Makefile_reference` is used to replace the generated `Makefile` when initializing the operator (see below)

Then initialize the helm chart by executing `perator-sdk init --plugins helm --domain redhat.com --kind `
```bash
$ operator-sdk init --plugins helm --domain redhat.com --kind Parodos --helm-chart helm-charts/parodos-v1-0-19 && cp Makefile_reference Makefile
    Writing kustomize manifests for you to edit...
    Creating the API:
    $ operator-sdk create api --kind Parodos --helm-chart helm-charts/parodos-v1-0-19
    Writing kustomize manifests for you to edit...
    Created helm-charts/parodos-v1-0-19
    Generating RBAC rules
    WARN[0000] Using default RBAC rules: failed to generate RBAC rules: failed to get server resources: Get "https://127.0.0.1:46431/api?timeout=32s": dial tcp 127.0.0.1:46431: connect: connection refused 
```

## Deploy
The target `kind_create` creates a local K8S cluster with Kind. It uses the `PARODOS_VERSION_DNS` to set the cluster's name. It is run when executing `make kind_deploy`.
The target `kind_delete` deletes this local cluster.

### Quick deploy
You can either execute `make kind_deploy`
```bash
$ make kind_deploy
kind create cluster --name parodos-v1-0-19
Creating cluster "parodos-v1-0-19" ...
 ✓ Ensuring node image (kindest/node:v1.25.3) 🖼
 ✓ Preparing nodes 📦  
 ✓ Writing configuration 📜 
 ✓ Starting control-plane 🕹️ 
 ✓ Installing CNI 🔌 
 ✓ Installing StorageClass 💾 
Set kubectl context to "kind-parodos-v1-0-19"
You can now use your cluster with:

kubectl cluster-info --context kind-parodos-v1-0-19

Not sure what to do next? 😅  Check out https://kind.sigs.k8s.io/docs/user/quick-start/
sudo sysctl fs.inotify.max_user_watches=524288
Place your finger on the fingerprint reader
fs.inotify.max_user_watches = 524288
sudo sysctl fs.inotify.max_user_instances=512
fs.inotify.max_user_instances = 512
docker build -t quay.io/parodos-dev/parodos-operator:v1.0.19 .
[+] Building 0.8s (9/9) FINISHED                                                                                                                             
 => [internal] load .dockerignore                                                                                                                       0.1s
 => => transferring context: 2B                                                                                                                         0.0s
 => [internal] load build definition from Dockerfile                                                                                                    0.0s
 => => transferring dockerfile: 233B                                                                                                                    0.0s
 => [internal] load metadata for quay.io/operator-framework/helm-operator:v1.30.0                                                                       0.6s
 => [1/4] FROM quay.io/operator-framework/helm-operator:v1.30.0@sha256:e82703bd9ce640b048c00d0c2e860d784cd0c5bfb724869c1aba85e92a13ce58                 0.0s
 => [internal] load build context                                                                                                                       0.0s
 => => transferring context: 1.20kB                                                                                                                     0.0s
 => CACHED [2/4] COPY watches.yaml /opt/helm/watches.yaml                                                                                               0.0s
 => CACHED [3/4] COPY helm-charts  /opt/helm/helm-charts                                                                                                0.0s
 => CACHED [4/4] WORKDIR /opt/helm                                                                                                                      0.0s
 => exporting to image                                                                                                                                  0.0s
 => => exporting layers                                                                                                                                 0.0s
 => => writing image sha256:b7aef943fdf67bd166c41c2a3a1038a5e4806c104a11f3dd25df30d055ba0c84                                                            0.0s
 => => naming to quay.io/parodos-dev/parodos-operator:v1.0.19                                                                                           0.0s
/home/gabriel/git/parados_stuff/parodos-operator/bin/kustomize build config/crd | kubectl apply -f -
customresourcedefinition.apiextensions.k8s.io/parodos.charts.redhat.com created
cd config/manager && /home/gabriel/git/parados_stuff/parodos-operator/bin/kustomize edit set image controller=quay.io/parodos-dev/parodos-operator:v1.0.19
/home/gabriel/git/parados_stuff/parodos-operator/bin/kustomize build config/default | kubectl apply -f -
namespace/parodos-operator-system created
customresourcedefinition.apiextensions.k8s.io/parodos.charts.redhat.com unchanged
serviceaccount/parodos-operator-controller-manager created
role.rbac.authorization.k8s.io/parodos-operator-leader-election-role created
clusterrole.rbac.authorization.k8s.io/parodos-operator-manager-role created
clusterrole.rbac.authorization.k8s.io/parodos-operator-metrics-reader created
clusterrole.rbac.authorization.k8s.io/parodos-operator-proxy-role created
rolebinding.rbac.authorization.k8s.io/parodos-operator-leader-election-rolebinding created
clusterrolebinding.rbac.authorization.k8s.io/parodos-operator-manager-rolebinding created
clusterrolebinding.rbac.authorization.k8s.io/parodos-operator-proxy-rolebinding created
service/parodos-operator-controller-manager-metrics-service created
deployment.apps/parodos-operator-controller-manager created
Deploying Parodos operator sample...
kubectl -n parodos-operator-system wait pod/$(kubectl get pods --no-headers -o custom-columns=":metadata.name" -n parodos-operator-system) --for=condition=Ready --timeout=60s
pod/parodos-operator-controller-manager-5548fc4ff6-bt94s condition met
kubectl create namespace local-test || true
namespace/local-test created
kubectl apply -f config/samples/charts_v1alpha1_parodos.yaml -n local-test
parodos.charts.redhat.com/parodos-sample created
It may take up to 5min for all pods to be ready
kubectl -n local-test wait pod/$(kubectl get pods -n local-test --no-headers -o custom-columns=":metadata.name" -l app=postgres) --for=condition=Ready --timeout=300s
pod/postgres-6dcfdf7b58-rwvtw condition met
kubectl -n local-test wait pod/$(kubectl get pods -n local-test --no-headers -o custom-columns=":metadata.name" -l app=backstage) --for=condition=Ready --timeout=300s
pod/backstage-7f78dbdc5f-6jz45 condition met
kubectl -n local-test wait pod/$(kubectl get pods -n local-test --no-headers -o custom-columns=":metadata.name" -l app=workflow-service) --for=condition=Ready --timeout=300s
pod/workflow-service-97697495b-bdckq condition met
kubectl -n local-test wait pod/$(kubectl get pods -n local-test --no-headers -o custom-columns=":metadata.name" -l app=notification-service) --for=condition=Ready --timeout=300s
pod/notification-service-64bc65f7c9-czq72 condition met
All set!
Run 'kubectl -n local-test port-forward backstage-7f78dbdc5f-6jz45 7007:7007 &' to access backstage UId
```

Then once validated, you can release the version in the official repo by pushing the image: 
```bash
$ make docker-push
docker push quay.io/parodos-dev/parodos-operator:v1.0.19
The push refers to repository [quay.io/parodos-dev/parodos-operator]
5f70bf18a086: Layer already exists 
784e9b92b0c0: Layer already exists 
cf8333ebf2f5: Layer already exists 
9c9e5071d24a: Layer already exists 
f10ab75b4fc1: Layer already exists 
0d5fc27d682c: Layer already exists 
14946186767b: Layer already exists 
v1.0.19: digest: sha256:f03e402213488b44e2279210177541fc2b0e737f2661596537b98961439fe0dc size: 1776
```

or follow the step by step instructions
### Manual deploy
To push the docker image to the remote repo (release)
```bash
$ make docker-build
```

Let's create a local K8S cluster with Kind:
```bash
$ make kind_create 
kind create cluster --name parodos-v1-0-19
Creating cluster "parodos-v1-0-19" ...
 ✓ Ensuring node image (kindest/node:v1.25.3) 🖼
 ✓ Preparing nodes 📦  
 ✓ Writing configuration 📜 
 ✓ Starting control-plane 🕹️ 
 ✓ Installing CNI 🔌 
 ✓ Installing StorageClass 💾 
Set kubectl context to "kind-parodos-v1-0-19"
You can now use your cluster with:

kubectl cluster-info --context kind-parodos-v1-0-19

Have a nice day! 👋
```

Then, 
```bash
$ make install deploy
```

Check the `parodos-operator-controller-manager` is running:
```bash
$ kubectl -n parodos-operator-system get pods
NAME                                                   READY   STATUS    RESTARTS   AGE
parodos-operator-controller-manager-68f776c5fb-nvl2t   2/2     Running   0          19m
```

Once it is running, you can deploy the sample (here in the `default` namespace):
```bash
$ kubectl apply -f config/samples/charts_v1alpha1_parodos.yaml
```

Wait until the pods are up and running. It may take some times and some restarts as the posgresql needs to be available before anything else can start
```bash
$ kubectl get pods
NAME                                    READY   STATUS    RESTARTS      AGE
backstage-5468f68f5b-pmfn6              1/1     Running   2 (54s ago)   3m10s
notification-service-649656d6cb-btv4j   1/1     Running   0             3m10s
postgres-7857bf5cf-vb2qj                1/1     Running   0             3m10s
workflow-service-d5c4884c-t8zl4         1/1     Running   2 (50s ago)   3m10s
```

Now we can port forward the backstage port (`7007` by default) in order to access it from `localhost`:
```bash
$ kubectl port-forward parodos-backstage-7f46f49fb9-n4gx2 7007:7007 &
Forwarding from 127.0.0.1:7007 -> 7007
Forwarding from [::1]:7007 -> 7007
Handling connection for 7007
```

You should be able to access `http://localhost:7007/` with `test/test` credentials.

To test, you can also run
```bash
$ kubectl port-forward backstage-7f78dbdc5f-dr5cl 7007:7007 &
$ make test_deploy
```

Then once validated, you can release the version in the official repo by pushing the image: 
```bash
$ make docker-push
docker push quay.io/parodos-dev/parodos-operator:v1.0.19
The push refers to repository [quay.io/parodos-dev/parodos-operator]
5f70bf18a086: Layer already exists 
784e9b92b0c0: Layer already exists 
cf8333ebf2f5: Layer already exists 
9c9e5071d24a: Layer already exists 
f10ab75b4fc1: Layer already exists 
0d5fc27d682c: Layer already exists 
14946186767b: Layer already exists 
v1.0.19: digest: sha256:f03e402213488b44e2279210177541fc2b0e737f2661596537b98961439fe0dc size: 1776
```

## Push to OperatorHub

### Create the bundle

See https://k8s-operatorhub.github.io/community-operators/packaging-operator/

First, we need to create a bundle, see https://sdk.operatorframework.io/docs/olm-integration/quickstart-bundle/#creating-a-bundle for more details

Make sure to update the `VERSION` in the Makefile or to export it first
```bash
$ make bundle
cp -r bundle_manifests/* config/manifests/.
/usr/local/bin/operator-sdk generate kustomize manifests -q
cd config/manager && /home/gabriel/git/parados_stuff/parodos-operator/bin/kustomize edit set image controller=quay.io/parodos-dev/parodos-operator:v1.0.19
/home/gabriel/git/parados_stuff/parodos-operator/bin/kustomize build config/manifests | /usr/local/bin/operator-sdk generate bundle -q --overwrite --version 0.0.3  
INFO[0000] Creating bundle.Dockerfile                   
INFO[0000] Creating bundle/metadata/annotations.yaml    
INFO[0000] Bundle metadata generated successfully       
/usr/local/bin/operator-sdk bundle validate ./bundle
INFO[0000] All validation tests have completed successfully 
Completing CSV...
/usr/local/bin/operator-sdk bundle validate ./bundle
INFO[0000] All validation tests have completed successfully 
Bundle ready
```

Please be aware that no prompts will appear as the requested values are already saved in the `bundle_manifests` folder, which will be automatically moved.

The `bundle` target is also doing the following automatically, no need to do it, it's kept there in order to keep history.

#### Steps done automatically by `make bundle` (no need to execute)

Then open `bundle/manifests/parodos-operator.clusterserviceversion.yaml` and 
* add the following in `metadata.annotations`:
```
categories: "Modernization & Migration,Application Runtime,Developer Tools"
containerImage: quay.io/parodos-dev/parodos-operator:v1.0.19
repository: https://github.com/parodos-dev/parodos
```
* set `metadata.namespace` to `parodos-operator` 
* set `spec.icon`:
```
 icon:
  - base64data: iVBORw0KGgoAAAANSUhEUgAAADAAAAAwCAYAAABXAvmHAAADpklEQVR4nOyYT2hcRRzHvzPvz741abu7TTe2LCvWtLU1S/1TVm2RqqAgqKcKVkGlKAhVEMnBgngo4kFzKEJ78SIeRPRkixAQmqqItoYc2m40Fo3bhK6wzZ/Nn3V333sz8maTHAKuM7NJ3wvkA5OZwPx+7/slmfnNjAkAR/J25q0nrRMP77HuAEAQffhPo27x1EDtA5JJUfLXqc1XQMg9YatSh1+lD/YYmfUpPoD0UgJYYctoBxq2gHbZMBA2psrk6lPfgjtblT9CWANw50GqJdDKKIzyLzD+/gHEqyrnWomSAdibAXuL8kd48MPZBr7pTrDug/B2vQxaHoI59iXM4jkQ7innXELJQPzsIYBo1Dliib8cS+yFv/0wvOzTYOk8Guk83N3H4Px4HHThunreIPWRvLXzizc7/9CK1oRbm+DufR3unlcBaoLUbiJ24SUYlVHlXKEsYuLOwb78EZzzR4HaFLjThfrhT8GcbuVcoe5CxuQw4oMvAPUKeDyN+kP9yjlC30bp7DXELvYBnIsF7mafUYtfM2UKmKXzMCYGxNjd94ZSbCQMBFiF06LnW3rgb71POi4yBozKryAzv4uxv/0x6TilOlDPfwgeFDMVYcVzsMa/kZtb/hleYjf8VE46v5KBoAgFW54KdHpEei6ZHRM978hIx6gdJRYxSt+BzraufV73QfDE3Up5iTe/qKpDOkbLgFn8WrRW8APvw1M0ACPe7P2adEhkFnEA68yKPji1yhItA9sOiJ7OFKRjImPAT+bAUvvF4du8LrdrISoGuKjAx8VRncz8Bjp1RTo2Ega8Xa/AzzwhzkOx4ZMgYNKxoRvwdjyOxr0nxNgc+wpG+ZJSfKgG3LueR/3QGXGpoVNXYQ+fVM6hVQfaxU/0orG/D+z2R8TvdLoA5/tjIP4/yrm0DLCODFiyt+UcHks2eycNv+sB8FgXWLJ5J2bJ3PLd2iieRWzoXRBvQUeKngE397ZoMng9R0VbSbDb2Jf7YZYGdSQsc+v+hdwF0Lk/m88pEwMwbg6tSlotA/ald/632NTvfw/+zudgjpyBXfgYhLm6GluiZYCwBojf+lVt6bEqEL5W4hH2NroabBgImw0DYaO1C3nZZ8ES+1rOUXnbaQctA/6OR0WLAkoGzGufKb0YBKxWxf0vlAzYI6fXTokm634Rr38DHJB/RYog9OIN98b4JLsQthAdxif9QXEtyqRIZ/+Lt70WDMMWJQ3HRN/n1U/+DQAA///cXxHcbqTXhQAAAABJRU5ErkJggg==
    mediatype: image/png
```
* add the following in the spec.description:
```
[Parodos](https://github.com/parodos-dev/parodos) is a Java toolkit to help enterprise's with a legacy footprint build internal developer platforms (IDP) that enable their developers to get access to the tools and environments required to start coding with less time, friction and frustration.
    ## About this Operator
    This Operator is based on a [Helm chart](https://github.com/parodos-dev/parodos-operator) for Parodos. It supports all the parameters from the helm chart.
    ## Using the cluster
    Parodos operator deploys all components needed to have a fully functionnal platform.
    ```
    kubectl get pods
    NAME                                    READY   STATUS    RESTARTS      AGE
    backstage-5468f68f5b-pmfn6              1/1     Running   2 (54s ago)   3m10s
    notification-service-649656d6cb-btv4j   1/1     Running   0             3m10s
    postgres-7857bf5cf-vb2qj                1/1     Running   0             3m10s
    workflow-service-d5c4884c-t8zl4         1/1     Running   2 (50s ago)   3m10s

    kubectl get service
    NAME                   TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
    backstage              ClusterIP   10.109.227.212   <none>        7007/TCP   3m57s
    notification-service   ClusterIP   10.108.203.52    <none>        8080/TCP   3m57s
    postgres               ClusterIP   10.99.8.76       <none>        5432/TCP   3m57s
    workflow-service       ClusterIP   10.103.97.20     <none>        8080/TCP   3m57s
    ```
```
* add a description in `spec.owned.customresourcedefinitions.description`
* set spec.minKubeVersion to v1.18.0
* below `spec.version`, add `replaces: parodos-operator.v<previous_version>` with `<previous_version>` being the previous version (ie: 0.0.1)

### Prepare the operator

You can now fork https://github.com/k8s-operatorhub/community-operators. To fork the repo, if you have the github CLI installed and configured you can run `https://github.com/k8s-operatorhub/community-operators`

Then
* create a new folder in `operators/parodos-operator` with the new operator's version as name (ie: 0.0.1)
* copy the folder `bundle` and the file `bundle.Dockerfile` previously generated into the newly created folder
  * you do not need to do that if you run the [quick test](#quick-testing) as a new folder folr the `VERSION` will be created with all the files needed

### Testing the operator

You can now test it by following: https://k8s-operatorhub.github.io/community-operators/testing-operators/ and https://sdk.operatorframework.io/docs/olm-integration/tutorial-bundle/

Here we will mostly leverage `operator sdk`

You can either run `make prepare_push_to_operatorhub` from this repo and then copy the generated folder into your fork. At the end of this section you will then copy the generated folder into operators/parodos-operator of your fork.
Or you can follow the [step by step instructions](#step-by-step).

#### Quick testing

/!\ Remember to either set `LOCAL_REPO` with you own repo in the makefile or override it while executing the target.

```bash
$ LOCAL_REPO=quay.io/gfarache make prepare_push_to_operatorhub
cp -r bundle/* 0.0.3/.
cp bundle.Dockerfile 0.0.3/.
sed -i 's/bundle\///g' 0.0.3/bundle.Dockerfile
operator-sdk bundle validate 0.0.3 --select-optional suite=operatorframework || operator-sdk bundle validate 0.0.3 --select-optional suite=operatorframework
INFO[0000] All validation tests have completed successfully 
operator-sdk scorecard 0.0.3 || true
--------------------------------------------------------------------------------
Image:      quay.io/operator-framework/scorecard-test:v1.30.0
Entrypoint: [scorecard-test olm-spec-descriptors]
Labels:
        "suite":"olm"
        "test":"olm-spec-descriptors-test"
Results:
        Name: olm-spec-descriptors
        State: fail

        Suggestions:
                Add a spec descriptor for postgres
                Add a spec descriptor for pvc
                Add a spec descriptor for appConfig4Gh2B9Ghf4
                Add a spec descriptor for backstage
                Add a spec descriptor for kubernetesClusterDomain
                Add a spec descriptor for notificationService
                Add a spec descriptor for notificationServiceConfig
                Add a spec descriptor for config
                Add a spec descriptor for postgresSecretsTh27274644
                Add a spec descriptor for workflowService
        Errors:
                postgres does not have a spec descriptor
                pvc does not have a spec descriptor
                appConfig4Gh2B9Ghf4 does not have a spec descriptor
                backstage does not have a spec descriptor
                kubernetesClusterDomain does not have a spec descriptor
                notificationService does not have a spec descriptor
                notificationServiceConfig does not have a spec descriptor
                config does not have a spec descriptor
                postgresSecretsTh27274644 does not have a spec descriptor
                workflowService does not have a spec descriptor
        Log:
                Loaded ClusterServiceVersion: parodos-operator.v0.0.3
                Loaded 1 Custom Resources from alm-examples


--------------------------------------------------------------------------------
Image:      quay.io/operator-framework/scorecard-test:v1.30.0
Entrypoint: [scorecard-test olm-crds-have-validation]
Labels:
        "suite":"olm"
        "test":"olm-crds-have-validation-test"
Results:
        Name: olm-crds-have-validation
        State: fail

        Suggestions:
                Add CRD validation for spec field `pvc` in Parodos/v1alpha1
                Add CRD validation for spec field `workflowService` in Parodos/v1alpha1
                Add CRD validation for spec field `appConfig4Gh2B9Ghf4` in Parodos/v1alpha1
                Add CRD validation for spec field `config` in Parodos/v1alpha1
                Add CRD validation for spec field `kubernetesClusterDomain` in Parodos/v1alpha1
                Add CRD validation for spec field `notificationService` in Parodos/v1alpha1
                Add CRD validation for spec field `postgres` in Parodos/v1alpha1
                Add CRD validation for spec field `backstage` in Parodos/v1alpha1
                Add CRD validation for spec field `notificationServiceConfig` in Parodos/v1alpha1
                Add CRD validation for spec field `postgresSecretsTh27274644` in Parodos/v1alpha1
        Log:
                Loaded 1 Custom Resources from alm-examples
                Loaded CustomresourceDefinitions: [&CustomResourceDefinition{ObjectMeta:{parodos.charts.redhat.com      0 0001-01-01 00:00:00 +0000 UTC <nil> <nil> map[] map[] [] [] []},Spec:CustomResourceDefinitionSpec{Group:charts.redhat.com,Names:CustomResourceDefinitionNames{Plural:parodos,Singular:parodos,ShortNames:[],Kind:Parodos,ListKind:ParodosList,Categories:[],},Scope:Namespaced,Versions:[]CustomResourceDefinitionVersion{CustomResourceDefinitionVersion{Name:v1alpha1,Served:true,Storage:true,Schema:&CustomResourceValidation{OpenAPIV3Schema:&JSONSchemaProps{ID:,Schema:,Ref:nil,Description:Parodos is the Schema for the parodos API,Type:object,Format:,Title:,Default:nil,Maximum:nil,ExclusiveMaximum:false,Minimum:nil,ExclusiveMinimum:false,MaxLength:nil,MinLength:nil,Pattern:,MaxItems:nil,MinItems:nil,UniqueItems:false,MultipleOf:nil,Enum:[]JSON{},MaxProperties:nil,MinProperties:nil,Required:[],Items:nil,AllOf:[]JSONSchemaProps{},OneOf:[]JSONSchemaProps{},AnyOf:[]JSONSchemaProps{},Not:nil,Properties:map[string]JSONSchemaProps{apiVersion: {  <nil> APIVersion defines the versioned schema of this representation of an object. Servers should convert recognized schemas to the latest internal value, and may reject unrecognized values. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources string   nil <nil> false <nil> false <nil> <nil>  <nil> <nil> false <nil> [] <nil> <nil> [] nil [] [] [] nil map[] nil map[] map[] nil map[] nil nil false <nil> false false [] <nil> <nil> []},kind: {  <nil> Kind is a string value representing the REST resource this object represents. Servers may infer this from the endpoint the client submits requests to. Cannot be updated. In CamelCase. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds string   nil <nil> false <nil> false <nil> <nil>  <nil> <nil> false <nil> [] <nil> <nil> [] nil [] [] [] nil map[] nil map[] map[] nil map[] nil nil false <nil> false false [] <nil> <nil> []},metadata: {  <nil>  object   nil <nil> false <nil> false <nil> <nil>  <nil> <nil> false <nil> [] <nil> <nil> [] nil [] [] [] nil map[] nil map[] map[] nil map[] nil nil false <nil> false false [] <nil> <nil> []},spec: {  <nil> Spec defines the desired state of Parodos object   nil <nil> false <nil> false <nil> <nil>  <nil> <nil> false <nil> [] <nil> <nil> [] nil [] [] [] nil map[] nil map[] map[] nil map[] nil nil false 0xc0007d28b0 false false [] <nil> <nil> []},status: {  <nil> Status defines the observed state of Parodos object   nil <nil> false <nil> false <nil> <nil>  <nil> <nil> false <nil> [] <nil> <nil> [] nil [] [] [] nil map[] nil map[] map[] nil map[] nil nil false 0xc0007d28b1 false false [] <nil> <nil> []},},AdditionalProperties:nil,PatternProperties:map[string]JSONSchemaProps{},Dependencies:JSONSchemaDependencies{},AdditionalItems:nil,Definitions:JSONSchemaDefinitions{},ExternalDocs:nil,Example:nil,Nullable:false,XPreserveUnknownFields:nil,XEmbeddedResource:false,XIntOrString:false,XListMapKeys:[],XListType:nil,XMapType:nil,XValidations:[]ValidationRule{},},},Subresources:&CustomResourceSubresources{Status:&CustomResourceSubresourceStatus{},Scale:nil,},AdditionalPrinterColumns:[]CustomResourceColumnDefinition{},Deprecated:false,DeprecationWarning:nil,},},Conversion:nil,PreserveUnknownFields:false,},Status:CustomResourceDefinitionStatus{Conditions:[]CustomResourceDefinitionCondition{},AcceptedNames:CustomResourceDefinitionNames{Plural:,Singular:,ShortNames:[],Kind:,ListKind:,Categories:[],},StoredVersions:[],},}]


--------------------------------------------------------------------------------
Image:      quay.io/operator-framework/scorecard-test:v1.30.0
Entrypoint: [scorecard-test olm-bundle-validation]
Labels:
        "suite":"olm"
        "test":"olm-bundle-validation-test"
Results:
        Name: olm-bundle-validation
        State: pass

        Log:
                time="2023-07-20T12:33:36Z" level=debug msg="Found manifests directory" name=bundle-test
                time="2023-07-20T12:33:36Z" level=debug msg="Found metadata directory" name=bundle-test
                time="2023-07-20T12:33:36Z" level=debug msg="Getting mediaType info from manifests directory" name=bundle-test
                time="2023-07-20T12:33:36Z" level=debug msg="Found annotations file" name=bundle-test
                time="2023-07-20T12:33:36Z" level=debug msg="Could not find optional dependencies file" name=bundle-test


--------------------------------------------------------------------------------
Image:      quay.io/operator-framework/scorecard-test:v1.30.0
Entrypoint: [scorecard-test olm-status-descriptors]
Labels:
        "suite":"olm"
        "test":"olm-status-descriptors-test"
Results:
        Name: olm-status-descriptors
        State: pass

        Suggestions:
                parodos.charts.redhat.com does not have status spec. Note thatAll objects that represent a physical resource whose state may vary from the user's desired intent SHOULD have a spec and a status. More info: https://github.com/kubernetes/community/blob/master/contributors/devel/sig-architecture/api-conventions.md#spec-and-status
        Log:
                Loaded ClusterServiceVersion: parodos-operator.v0.0.3
                Loaded 1 Custom Resources from alm-examples


--------------------------------------------------------------------------------
Image:      quay.io/operator-framework/scorecard-test:v1.30.0
Entrypoint: [scorecard-test olm-crds-have-resources]
Labels:
        "suite":"olm"
        "test":"olm-crds-have-resources-test"
Results:
        Name: olm-crds-have-resources
        State: fail

        Errors:
                Owned CRDs do not have resources specified
        Log:
                Loaded ClusterServiceVersion: parodos-operator.v0.0.3


--------------------------------------------------------------------------------
Image:      quay.io/operator-framework/scorecard-test:v1.30.0
Entrypoint: [scorecard-test basic-check-spec]
Labels:
        "suite":"basic"
        "test":"basic-check-spec-test"
Results:
        Name: basic-check-spec
        State: pass



kubectl delete pods -l app=scorecard-test
No resources found
docker build -f 0.0.3/bundle.Dockerfile -t quay.io/gfarache/parodos:v0.0.3 0.0.3/
[+] Building 0.1s (7/7) FINISHED                                                                                                                             
 => [internal] load .dockerignore                                                                                                                       0.0s
 => => transferring context: 2B                                                                                                                         0.0s
 => [internal] load build definition from bundle.Dockerfile                                                                                             0.0s
 => => transferring dockerfile: 954B                                                                                                                    0.0s
 => [internal] load build context                                                                                                                       0.0s
 => => transferring context: 24.97kB                                                                                                                    0.0s
 => CACHED [1/3] COPY manifests /manifests/                                                                                                             0.0s
 => CACHED [2/3] COPY metadata /metadata/                                                                                                               0.0s
 => CACHED [3/3] COPY tests/scorecard /tests/scorecard/                                                                                                 0.0s
 => exporting to image                                                                                                                                  0.0s
 => => exporting layers                                                                                                                                 0.0s
 => => writing image sha256:f2b5cd855d0d3ee6273a5319ababa386201af3d74b42224b18193ca89fde570f                                                            0.0s
 => => naming to quay.io/gfarache/parodos:v0.0.3                                                                                                        0.0s
docker push quay.io/gfarache/parodos:v0.0.3
The push refers to repository [quay.io/gfarache/parodos]
6541beb78fe2: Layer already exists 
2e50e8de3d07: Layer already exists 
5a3e566f3b16: Layer already exists 
v0.0.3: digest: sha256:4cf17b2e8d27361c29d608792f24722b4798854ef097676a95efb5bb0bafe3f4 size: 939
operator-sdk olm uninstall || true
INFO[0000] Fetching CRDs for version "v0.25.0"          
INFO[0000] Fetching resources for resolved version "v0.25.0" 
INFO[0001] Uninstalling resources for version "v0.25.0" 
INFO[0001]   Deleting CustomResourceDefinition "catalogsources.operators.coreos.com" 
INFO[0001]   Deleting CustomResourceDefinition "clusterserviceversions.operators.coreos.com" 
INFO[0001]   Deleting CustomResourceDefinition "installplans.operators.coreos.com" 
INFO[0001]   Deleting CustomResourceDefinition "olmconfigs.operators.coreos.com" 
INFO[0001]   Deleting CustomResourceDefinition "operatorconditions.operators.coreos.com" 
INFO[0002]   Deleting CustomResourceDefinition "operatorgroups.operators.coreos.com" 
INFO[0002]   Deleting CustomResourceDefinition "operators.operators.coreos.com" 
INFO[0002]   Deleting CustomResourceDefinition "subscriptions.operators.coreos.com" 
INFO[0002]   Deleting Namespace "olm"                   
INFO[0012]   Deleting Namespace "operators"             
INFO[0017]   Deleting ServiceAccount "olm/olm-operator-serviceaccount" 
INFO[0017]     ServiceAccount "olm/olm-operator-serviceaccount" does not exist 
INFO[0017]   Deleting ClusterRole "system:controller:operator-lifecycle-manager" 
INFO[0017]   Deleting ClusterRoleBinding "olm-operator-binding-olm" 
INFO[0017]   Deleting OLMConfig "cluster"               
INFO[0017]     OLMConfig "cluster" does not exist       
INFO[0017]   Deleting Deployment "olm/olm-operator"     
INFO[0017]     Deployment "olm/olm-operator" does not exist 
INFO[0017]   Deleting Deployment "olm/catalog-operator" 
INFO[0017]     Deployment "olm/catalog-operator" does not exist 
INFO[0017]   Deleting ClusterRole "aggregate-olm-edit"  
INFO[0017]   Deleting ClusterRole "aggregate-olm-view"  
INFO[0017]   Deleting OperatorGroup "operators/global-operators" 
INFO[0017]     OperatorGroup "operators/global-operators" does not exist 
INFO[0017]   Deleting OperatorGroup "olm/olm-operators" 
INFO[0017]     OperatorGroup "olm/olm-operators" does not exist 
INFO[0017]   Deleting ClusterServiceVersion "olm/packageserver" 
INFO[0017]     ClusterServiceVersion "olm/packageserver" does not exist 
INFO[0017]   Deleting CatalogSource "olm/operatorhubio-catalog" 
INFO[0017]     CatalogSource "olm/operatorhubio-catalog" does not exist 
INFO[0017] Successfully uninstalled OLM version "v0.25.0" 
operator-sdk olm install
INFO[0000] Fetching CRDs for version "latest"           
INFO[0000] Fetching resources for resolved version "latest" 
I0720 14:34:05.379853  259265 request.go:690] Waited for 1.048795192s due to client-side throttling, not priority and fairness, request: GET:https://127.0.0.1:43029/apis/certificates.k8s.io/v1?timeout=32s
INFO[0006] Creating CRDs and resources                  
INFO[0006]   Creating CustomResourceDefinition "catalogsources.operators.coreos.com" 
INFO[0006]   Creating CustomResourceDefinition "clusterserviceversions.operators.coreos.com" 
INFO[0006]   Creating CustomResourceDefinition "installplans.operators.coreos.com" 
INFO[0006]   Creating CustomResourceDefinition "olmconfigs.operators.coreos.com" 
INFO[0006]   Creating CustomResourceDefinition "operatorconditions.operators.coreos.com" 
INFO[0006]   Creating CustomResourceDefinition "operatorgroups.operators.coreos.com" 
INFO[0006]   Creating CustomResourceDefinition "operators.operators.coreos.com" 
INFO[0006]   Creating CustomResourceDefinition "subscriptions.operators.coreos.com" 
INFO[0006]   Creating Namespace "olm"                   
INFO[0006]   Creating Namespace "operators"             
INFO[0006]   Creating ServiceAccount "olm/olm-operator-serviceaccount" 
INFO[0006]   Creating ClusterRole "system:controller:operator-lifecycle-manager" 
INFO[0006]   Creating ClusterRoleBinding "olm-operator-binding-olm" 
INFO[0006]   Creating OLMConfig "cluster"               
INFO[0008]   Creating Deployment "olm/olm-operator"     
INFO[0008]   Creating Deployment "olm/catalog-operator" 
INFO[0008]   Creating ClusterRole "aggregate-olm-edit"  
INFO[0008]   Creating ClusterRole "aggregate-olm-view"  
INFO[0008]   Creating OperatorGroup "operators/global-operators" 
INFO[0008]   Creating OperatorGroup "olm/olm-operators" 
INFO[0008]   Creating ClusterServiceVersion "olm/packageserver" 
INFO[0008]   Creating CatalogSource "olm/operatorhubio-catalog" 
INFO[0008] Waiting for deployment/olm-operator rollout to complete 
INFO[0008]   Waiting for Deployment "olm/olm-operator" to rollout: 0 of 1 updated replicas are available 
INFO[0012]   Deployment "olm/olm-operator" successfully rolled out 
INFO[0012] Waiting for deployment/catalog-operator rollout to complete 
INFO[0012]   Deployment "olm/catalog-operator" successfully rolled out 
INFO[0012] Waiting for deployment/packageserver rollout to complete 
INFO[0012]   Waiting for Deployment "olm/packageserver" to rollout: 0 of 2 updated replicas are available 
INFO[0015]   Deployment "olm/packageserver" successfully rolled out 
INFO[0015] Successfully installed OLM version "latest"  

NAME                                            NAMESPACE    KIND                        STATUS
catalogsources.operators.coreos.com                          CustomResourceDefinition    Installed
clusterserviceversions.operators.coreos.com                  CustomResourceDefinition    Installed
installplans.operators.coreos.com                            CustomResourceDefinition    Installed
olmconfigs.operators.coreos.com                              CustomResourceDefinition    Installed
operatorconditions.operators.coreos.com                      CustomResourceDefinition    Installed
operatorgroups.operators.coreos.com                          CustomResourceDefinition    Installed
operators.operators.coreos.com                               CustomResourceDefinition    Installed
subscriptions.operators.coreos.com                           CustomResourceDefinition    Installed
olm                                                          Namespace                   Installed
operators                                                    Namespace                   Installed
olm-operator-serviceaccount                     olm          ServiceAccount              Installed
system:controller:operator-lifecycle-manager                 ClusterRole                 Installed
olm-operator-binding-olm                                     ClusterRoleBinding          Installed
cluster                                                      OLMConfig                   Installed
olm-operator                                    olm          Deployment                  Installed
catalog-operator                                olm          Deployment                  Installed
aggregate-olm-edit                                           ClusterRole                 Installed
aggregate-olm-view                                           ClusterRole                 Installed
global-operators                                operators    OperatorGroup               Installed
olm-operators                                   olm          OperatorGroup               Installed
packageserver                                   olm          ClusterServiceVersion       Installed
operatorhubio-catalog                           olm          CatalogSource               Installed

operator-sdk run bundle quay.io/gfarache/parodos:v0.0.3
INFO[0012] Creating a File-Based Catalog of the bundle "quay.io/gfarache/parodos:v0.0.3" 
INFO[0013] Generated a valid File-Based Catalog         
INFO[0016] Created registry pod: quay-io-gfarache-parodos-v0-0-3 
INFO[0016] Created CatalogSource: parodos-operator-catalog 
INFO[0016] OperatorGroup "operator-sdk-og" created      
INFO[0016] Created Subscription: parodos-operator-v0-0-3-sub 
INFO[0028] Approved InstallPlan install-lm45j for the Subscription: parodos-operator-v0-0-3-sub 
INFO[0028] Waiting for ClusterServiceVersion "default/parodos-operator.v0.0.3" to reach 'Succeeded' phase 
INFO[0028]   Waiting for ClusterServiceVersion "default/parodos-operator.v0.0.3" to appear 
INFO[0031]   Found ClusterServiceVersion "default/parodos-operator.v0.0.3" phase: Pending 
INFO[0033]   Found ClusterServiceVersion "default/parodos-operator.v0.0.3" phase: Installing 
INFO[0044]   Found ClusterServiceVersion "default/parodos-operator.v0.0.3" phase: Succeeded 
INFO[0044] OLM has successfully installed "parodos-operator.v0.0.3" 
kubectl  wait pod/$(kubectl get pods --no-headers -o custom-columns=":metadata.name" | grep parodos-operator-controller-manager) --for=condition=Ready --timeout=300s
pod/parodos-operator-controller-manager-cdf9df47c-nht4m condition met
kubectl create namespace operator-bundle-test || true
Error from server (AlreadyExists): namespaces "operator-bundle-test" already exists
kubectl apply -f config/samples/charts_v1alpha1_parodos.yaml -n operator-bundle-test
parodos.charts.redhat.com/parodos-sample unchanged
It may take up to 5min for all pods to be ready
kubectl -n operator-bundle-test wait pod/$(kubectl get pods -n operator-bundle-test --no-headers -o custom-columns=":metadata.name" -l app=postgres | head -n 1) --for=condition=Ready --timeout=300s
pod/postgres-6dcfdf7b58-6rvw8 condition met
kubectl -n operator-bundle-test wait pod/$(kubectl get pods -n operator-bundle-test --no-headers -o custom-columns=":metadata.name" -l app=backstage | head -n 1) --for=condition=Ready --timeout=300s
pod/backstage-7f78dbdc5f-8cvkq condition met
kubectl -n operator-bundle-test wait pod/$(kubectl get pods -n operator-bundle-test --no-headers -o custom-columns=":metadata.name" -l app=workflow-service | head -n 1) --for=condition=Ready --timeout=300s
pod/workflow-service-97697495b-2h62v condition met
kubectl -n operator-bundle-test wait pod/$(kubectl get pods -n operator-bundle-test --no-headers -o custom-columns=":metadata.name" -l app=notification-service | head -n 1) --for=condition=Ready --timeout=300s
pod/notification-service-64bc65f7c9-lstxs condition met
Operator bundle prepared and tested successfuly!
You can now copy 0.0.3 folder into operators/parodos-operator of your fork of https://github.com/k8s-operatorhub/community-operators
```

You can now copy the generated folder into operators/parodos-operator of your fork of https://github.com/k8s-operatorhub/community-operators


#### Step by step

##### Static tests

From your fork folder of community-operators, in operators/parodos-operator/\<version\>: (here version = 0.0.2)

```bash
$ operator-sdk bundle validate . --select-optional suite=operatorframework
INFO[0000] All validation tests have completed successfully 
```

You may need to run the following twice.

```bash
$ operator-sdk scorecard .
--------------------------------------------------------------------------------
Image:      quay.io/operator-framework/scorecard-test:v1.30.0
Entrypoint: [scorecard-test olm-status-descriptors]
Labels:
        "suite":"olm"
        "test":"olm-status-descriptors-test"
Results:
        Name: olm-status-descriptors
        State: pass

        Suggestions:
                parodos.charts.redhat.com does not have status spec. Note thatAll objects that represent a physical resource whose state may vary from the user's desired intent SHOULD have a spec and a status. More info: https://github.com/kubernetes/community/blob/master/contributors/devel/sig-architecture/api-conventions.md#spec-and-status
        Log:
                Loaded ClusterServiceVersion: parodos-operator.v0.0.2
                Loaded 1 Custom Resources from alm-examples


--------------------------------------------------------------------------------
Image:      quay.io/operator-framework/scorecard-test:v1.30.0
Entrypoint: [scorecard-test basic-check-spec]
Labels:
        "suite":"basic"
        "test":"basic-check-spec-test"
Results:
        Name: basic-check-spec
        State: pass



--------------------------------------------------------------------------------
Image:      quay.io/operator-framework/scorecard-test:v1.30.0
Entrypoint: [scorecard-test olm-bundle-validation]
Labels:
        "suite":"olm"
        "test":"olm-bundle-validation-test"
Results:
        Name: olm-bundle-validation
        State: pass

        Log:
                time="2023-07-18T08:43:21Z" level=debug msg="Found manifests directory" name=bundle-test
                time="2023-07-18T08:43:21Z" level=debug msg="Found metadata directory" name=bundle-test
                time="2023-07-18T08:43:21Z" level=debug msg="Getting mediaType info from manifests directory" name=bundle-test
                time="2023-07-18T08:43:21Z" level=debug msg="Found annotations file" name=bundle-test
                time="2023-07-18T08:43:21Z" level=debug msg="Could not find optional dependencies file" name=bundle-test


--------------------------------------------------------------------------------
Image:      quay.io/operator-framework/scorecard-test:v1.30.0
Entrypoint: [scorecard-test olm-crds-have-validation]
Labels:
        "suite":"olm"
        "test":"olm-crds-have-validation-test"
Results:
        Name: olm-crds-have-validation
        State: fail

        Suggestions:
                Add CRD validation for spec field `workflowService` in Parodos/v1alpha1
                Add CRD validation for spec field `appConfig4Gh2B9Ghf4` in Parodos/v1alpha1
                Add CRD validation for spec field `config` in Parodos/v1alpha1
                Add CRD validation for spec field `notificationServiceConfig` in Parodos/v1alpha1
                Add CRD validation for spec field `postgres` in Parodos/v1alpha1
                Add CRD validation for spec field `postgresSecretsTh27274644` in Parodos/v1alpha1
                Add CRD validation for spec field `pvc` in Parodos/v1alpha1
                Add CRD validation for spec field `backstage` in Parodos/v1alpha1
                Add CRD validation for spec field `fullnameOverride` in Parodos/v1alpha1
                Add CRD validation for spec field `kubernetesClusterDomain` in Parodos/v1alpha1
                Add CRD validation for spec field `notificationService` in Parodos/v1alpha1
        Log:
                Loaded 1 Custom Resources from alm-examples
                Loaded CustomresourceDefinitions: [&CustomResourceDefinition{ObjectMeta:{parodos.charts.redhat.com      0 0001-01-01 00:00:00 +0000 UTC <nil> <nil> map[] map[] [] [] []},Spec:CustomResourceDefinitionSpec{Group:charts.redhat.com,Names:CustomResourceDefinitionNames{Plural:parodos,Singular:parodos,ShortNames:[],Kind:Parodos,ListKind:ParodosList,Categories:[],},Scope:Namespaced,Versions:[]CustomResourceDefinitionVersion{CustomResourceDefinitionVersion{Name:v1alpha1,Served:true,Storage:true,Schema:&CustomResourceValidation{OpenAPIV3Schema:&JSONSchemaProps{ID:,Schema:,Ref:nil,Description:Parodos is the Schema for the parodos API,Type:object,Format:,Title:,Default:nil,Maximum:nil,ExclusiveMaximum:false,Minimum:nil,ExclusiveMinimum:false,MaxLength:nil,MinLength:nil,Pattern:,MaxItems:nil,MinItems:nil,UniqueItems:false,MultipleOf:nil,Enum:[]JSON{},MaxProperties:nil,MinProperties:nil,Required:[],Items:nil,AllOf:[]JSONSchemaProps{},OneOf:[]JSONSchemaProps{},AnyOf:[]JSONSchemaProps{},Not:nil,Properties:map[string]JSONSchemaProps{apiVersion: {  <nil> APIVersion defines the versioned schema of this representation of an object. Servers should convert recognized schemas to the latest internal value, and may reject unrecognized values. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources string   nil <nil> false <nil> false <nil> <nil>  <nil> <nil> false <nil> [] <nil> <nil> [] nil [] [] [] nil map[] nil map[] map[] nil map[] nil nil false <nil> false false [] <nil> <nil> []},kind: {  <nil> Kind is a string value representing the REST resource this object represents. Servers may infer this from the endpoint the client submits requests to. Cannot be updated. In CamelCase. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds string   nil <nil> false <nil> false <nil> <nil>  <nil> <nil> false <nil> [] <nil> <nil> [] nil [] [] [] nil map[] nil map[] map[] nil map[] nil nil false <nil> false false [] <nil> <nil> []},metadata: {  <nil>  object   nil <nil> false <nil> false <nil> <nil>  <nil> <nil> false <nil> [] <nil> <nil> [] nil [] [] [] nil map[] nil map[] map[] nil map[] nil nil false <nil> false false [] <nil> <nil> []},spec: {  <nil> Spec defines the desired state of Parodos object   nil <nil> false <nil> false <nil> <nil>  <nil> <nil> false <nil> [] <nil> <nil> [] nil [] [] [] nil map[] nil map[] map[] nil map[] nil nil false 0xc000a1ae60 false false [] <nil> <nil> []},status: {  <nil> Status defines the observed state of Parodos object   nil <nil> false <nil> false <nil> <nil>  <nil> <nil> false <nil> [] <nil> <nil> [] nil [] [] [] nil map[] nil map[] map[] nil map[] nil nil false 0xc000a1ae61 false false [] <nil> <nil> []},},AdditionalProperties:nil,PatternProperties:map[string]JSONSchemaProps{},Dependencies:JSONSchemaDependencies{},AdditionalItems:nil,Definitions:JSONSchemaDefinitions{},ExternalDocs:nil,Example:nil,Nullable:false,XPreserveUnknownFields:nil,XEmbeddedResource:false,XIntOrString:false,XListMapKeys:[],XListType:nil,XMapType:nil,XValidations:[]ValidationRule{},},},Subresources:&CustomResourceSubresources{Status:&CustomResourceSubresourceStatus{},Scale:nil,},AdditionalPrinterColumns:[]CustomResourceColumnDefinition{},Deprecated:false,DeprecationWarning:nil,},},Conversion:nil,PreserveUnknownFields:false,},Status:CustomResourceDefinitionStatus{Conditions:[]CustomResourceDefinitionCondition{},AcceptedNames:CustomResourceDefinitionNames{Plural:,Singular:,ShortNames:[],Kind:,ListKind:,Categories:[],},StoredVersions:[],},}]


--------------------------------------------------------------------------------
Image:      quay.io/operator-framework/scorecard-test:v1.30.0
Entrypoint: [scorecard-test olm-crds-have-resources]
Labels:
        "suite":"olm"
        "test":"olm-crds-have-resources-test"
Results:
        Name: olm-crds-have-resources
        State: fail

        Errors:
                Owned CRDs do not have resources specified
        Log:
                Loaded ClusterServiceVersion: parodos-operator.v0.0.2


--------------------------------------------------------------------------------
Image:      quay.io/operator-framework/scorecard-test:v1.30.0
Entrypoint: [scorecard-test olm-spec-descriptors]
Labels:
        "test":"olm-spec-descriptors-test"
        "suite":"olm"
Results:
        Name: olm-spec-descriptors
        State: fail

        Suggestions:
                Add a spec descriptor for appConfig4Gh2B9Ghf4
                Add a spec descriptor for config
                Add a spec descriptor for fullnameOverride
                Add a spec descriptor for notificationServiceConfig
                Add a spec descriptor for postgresSecretsTh27274644
                Add a spec descriptor for backstage
                Add a spec descriptor for kubernetesClusterDomain
                Add a spec descriptor for notificationService
                Add a spec descriptor for postgres
                Add a spec descriptor for pvc
                Add a spec descriptor for workflowService
        Errors:
                appConfig4Gh2B9Ghf4 does not have a spec descriptor
                config does not have a spec descriptor
                fullnameOverride does not have a spec descriptor
                notificationServiceConfig does not have a spec descriptor
                postgresSecretsTh27274644 does not have a spec descriptor
                backstage does not have a spec descriptor
                kubernetesClusterDomain does not have a spec descriptor
                notificationService does not have a spec descriptor
                postgres does not have a spec descriptor
                pvc does not have a spec descriptor
                workflowService does not have a spec descriptor
        Log:
                Loaded ClusterServiceVersion: parodos-operator.v0.0.2
                Loaded 1 Custom Resources from alm-examples
```

You can run
```bash
$ kubectl delete pods -l app=scorecard-test
pod "scorecard-test-4xlm" deleted
pod "scorecard-test-hcv4" deleted
pod "scorecard-test-pbr7" deleted
pod "scorecard-test-tln2" deleted
pod "scorecard-test-xdhc" deleted
pod "scorecard-test-zlv8" deleted
```

To clean the pods list.

##### Testing operator with OLM

To test the operator we will follow https://sdk.operatorframework.io/docs/olm-integration/tutorial-bundle/

Start by generating the bundle image: from your fork folder of community-operators, in operators/parodos-operator/\<version\>: (here version = 0.0.2)

/!\ Remember to change the quay repository to one you have pull and push rights.
 
```bash
$ docker build -f 0.0.2/bundle.Dockerfile -t quay.io/gfarache/parodos:v0.0.2 0.0.2/
[+] Building 0.1s (7/7) FINISHED                                                                                                                                        
 => [internal] load build definition from bundle.Dockerfile                                                                                                        0.0s
 => => transferring dockerfile: 954B                                                                                                                               0.0s
 => [internal] load .dockerignore                                                                                                                                  0.0s
 => => transferring context: 2B                                                                                                                                    0.0s
 => [internal] load build context                                                                                                                                  0.0s
 => => transferring context: 549B                                                                                                                                  0.0s
 => CACHED [1/3] COPY manifests /manifests/                                                                                                                        0.0s
 => CACHED [2/3] COPY metadata /metadata/                                                                                                                          0.0s
 => CACHED [3/3] COPY tests/scorecard /tests/scorecard/                                                                                                            0.0s
 => exporting to image                                                                                                                                             0.0s
 => => exporting layers                                                                                                                                            0.0s
 => => writing image sha256:053a265c6225dc3440fa3ca95657333bee4a7a2613fdb8ceee71e95a5e33664f                                                                       0.0s
 => => naming to quay.io/gfarache/parodos:v0.0.2                                                                                                                   0.0s

$ docker push quay.io/gfarache/parodos:v0.0.2
The push refers to repository [quay.io/gfarache/parodos]
dc0bfc49ce4f: Pushed 
452227c146a9: Pushed 
670c815e7e40: Pushed 
v0.0.2: digest: sha256:f9a3e964b6d8f701f50f34fe8d09377b9491e5ce2e574453196f7a5911db1928 size: 939
```

Then run

```bash
$ operator-sdk olm install
INFO[0000] Fetching CRDs for version "latest"           
INFO[0000] Fetching resources for resolved version "latest" 
I0718 10:02:13.370804 1442283 request.go:690] Waited for 1.046429415s due to client-side throttling, not priority and fairness, request: GET:https://192.168.49.2:8443/apis/authorization.k8s.io/v1?timeout=32s
INFO[0008] Creating CRDs and resources                  
INFO[0008]   Creating CustomResourceDefinition "catalogsources.operators.coreos.com" 
INFO[0008]   Creating CustomResourceDefinition "clusterserviceversions.operators.coreos.com" 
INFO[0008]   Creating CustomResourceDefinition "installplans.operators.coreos.com" 
INFO[0008]   Creating CustomResourceDefinition "olmconfigs.operators.coreos.com" 
INFO[0008]   Creating CustomResourceDefinition "operatorconditions.operators.coreos.com" 
INFO[0008]   Creating CustomResourceDefinition "operatorgroups.operators.coreos.com" 
INFO[0008]   Creating CustomResourceDefinition "operators.operators.coreos.com" 
INFO[0008]   Creating CustomResourceDefinition "subscriptions.operators.coreos.com" 
INFO[0008]   Creating Namespace "olm"                   
INFO[0008]   Creating Namespace "operators"             
INFO[0008]   Creating ServiceAccount "olm/olm-operator-serviceaccount" 
INFO[0008]   Creating ClusterRole "system:controller:operator-lifecycle-manager" 
INFO[0008]   Creating ClusterRoleBinding "olm-operator-binding-olm" 
INFO[0008]   Creating OLMConfig "cluster"               
INFO[0012]   Creating Deployment "olm/olm-operator"     
INFO[0012]   Creating Deployment "olm/catalog-operator" 
INFO[0012]   Creating ClusterRole "aggregate-olm-edit"  
INFO[0012]   Creating ClusterRole "aggregate-olm-view"  
INFO[0012]   Creating OperatorGroup "operators/global-operators" 
INFO[0012]   Creating OperatorGroup "olm/olm-operators" 
INFO[0012]   Creating ClusterServiceVersion "olm/packageserver" 
INFO[0012]   Creating CatalogSource "olm/operatorhubio-catalog" 
INFO[0012] Waiting for deployment/olm-operator rollout to complete 
INFO[0012]   Waiting for Deployment "olm/olm-operator" to rollout: 0 of 1 updated replicas are available 
INFO[0035]   Deployment "olm/olm-operator" successfully rolled out 
INFO[0035] Waiting for deployment/catalog-operator rollout to complete 
INFO[0035]   Waiting for Deployment "olm/catalog-operator" to rollout: 0 of 1 updated replicas are available 
INFO[0038]   Deployment "olm/catalog-operator" successfully rolled out 
INFO[0038] Waiting for deployment/packageserver rollout to complete 
INFO[0038]   Waiting for Deployment "olm/packageserver" to rollout: 0 of 2 updated replicas are available 
INFO[0042]   Deployment "olm/packageserver" successfully rolled out 
INFO[0042] Successfully installed OLM version "latest"  

NAME                                            NAMESPACE    KIND                        STATUS
catalogsources.operators.coreos.com                          CustomResourceDefinition    Installed
clusterserviceversions.operators.coreos.com                  CustomResourceDefinition    Installed
installplans.operators.coreos.com                            CustomResourceDefinition    Installed
olmconfigs.operators.coreos.com                              CustomResourceDefinition    Installed
operatorconditions.operators.coreos.com                      CustomResourceDefinition    Installed
operatorgroups.operators.coreos.com                          CustomResourceDefinition    Installed
operators.operators.coreos.com                               CustomResourceDefinition    Installed
subscriptions.operators.coreos.com                           CustomResourceDefinition    Installed
olm                                                          Namespace                   Installed
operators                                                    Namespace                   Installed
olm-operator-serviceaccount                     olm          ServiceAccount              Installed
system:controller:operator-lifecycle-manager                 ClusterRole                 Installed
olm-operator-binding-olm                                     ClusterRoleBinding          Installed
cluster                                                      OLMConfig                   Installed
olm-operator                                    olm          Deployment                  Installed
catalog-operator                                olm          Deployment                  Installed
aggregate-olm-edit                                           ClusterRole                 Installed
aggregate-olm-view                                           ClusterRole                 Installed
global-operators                                operators    OperatorGroup               Installed
olm-operators                                   olm          OperatorGroup               Installed
packageserver                                   olm          ClusterServiceVersion       Installed
operatorhubio-catalog                           olm          CatalogSource               Installed

$ operator-sdk run bundle quay.io/gfarache/parodos:v0.0.2
INFO[0013] Creating a File-Based Catalog of the bundle "quay.io/gfarache/parodos:v0.0.2" 
INFO[0014] Generated a valid File-Based Catalog         
INFO[0023] Created registry pod: quay-io-gfarache-parodos-v0-0-2 
INFO[0023] Created CatalogSource: parodos-operator-catalog 
INFO[0023] OperatorGroup "operator-sdk-og" created      
INFO[0023] Created Subscription: parodos-operator-v0-0-2-sub 
INFO[0036] Approved InstallPlan install-zg6g5 for the Subscription: parodos-operator-v0-0-2-sub 
INFO[0036] Waiting for ClusterServiceVersion "default/parodos-operator.v0.0.2" to reach 'Succeeded' phase 
INFO[0036]   Waiting for ClusterServiceVersion "default/parodos-operator.v0.0.2" to appear 
INFO[0039]   Found ClusterServiceVersion "default/parodos-operator.v0.0.2" phase: Pending 
INFO[0040]   Found ClusterServiceVersion "default/parodos-operator.v0.0.2" phase: Installing 
INFO[0071]   Found ClusterServiceVersion "default/parodos-operator.v0.0.2" phase: Succeeded 
INFO[0071] OLM has successfully installed "parodos-operator.v0.0.2" 
```

You should have something similar to:
```bash
$ kubectl get pods
NAME                                                              READY   STATUS      RESTARTS   AGE
d3da320f9796cf289a37153926cbb0642215d45b17437c0930e1bf282a5q9ls   0/1     Completed   0          6m49s
parodos-operator-controller-manager-7b9fbcfdc5-8fkd2              2/2     Running     0          6m35s
quay-io-gfarache-parodos-v0-0-2                                   1/1     Running     0          7m
```

Your operator is now running, try to deploy the sample to see the pods coming into life:
```bash
$ kubectl apply -f sample.yaml 
parodos.charts.redhat.com/parodos-sample created

$ kubectl get pods
NAME                                                              READY   STATUS      RESTARTS       AGE
backstage-5468f68f5b-p67lz                                        1/1     Running     3 (2m9s ago)   4m51s
d3da320f9796cf289a37153926cbb0642215d45b17437c0930e1bf282amfprw   0/1     Completed   0              6m10s
notification-service-649656d6cb-rs5d4                             1/1     Running     0              4m51s
parodos-operator-controller-manager-5c498467d-jl597               2/2     Running     0              5m56s
postgres-7857bf5cf-2kb7f                                          1/1     Running     0              4m51s
quay-io-gfarache-parodos-v0-0-2                                   1/1     Running     0              6m21s
workflow-service-d5c4884c-xllqt                                   1/1     Running     4 (89s ago)    4m51s
```

Note that until the `postgresql` pod is running, `backstage` and `workflow-service` will be in error.

### Push new version to OperatorHub

From your fork folder of community-operators:
* commit the changes and push them
    * you need to sign the commit: https://github.com/operator-framework/community-operators/blob/master/docs/contributing-prerequisites.md#sign-your-work
      * use `git commit -s`
* create a PR (see [guildines](https://github.com/operator-framework/community-operators/blob/master/docs/contributing-via-pr.md) to have the new operator version available on OperatorHub
* Prior to create the PR you can execute the CI tests locally from the root path of the fork:
  1. `curl -sL https://raw.githubusercontent.com/redhat-openshift-ecosystem/community-operators-pipeline/ci/latest/ci/scripts/opp.sh > opp.sh`
  2. `sudo bash opp.sh all operators/parodos-operator/<version>`
  3. `cat /tmp/op-test/log.out` to see the result