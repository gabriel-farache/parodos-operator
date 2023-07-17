# parodos-operator


## Creation
```bash
 make -f Makefile_cpy download-manifest generate-helmchart 
```

Please note that the parodos version is hold by the parameters `PARODOS_VERSION` and `PARODOS_VERSION_DNS` in the `Makefile` file.

This will create the folder helm-charts/parodos-{PARODOS_VERSION_DNS}. We have to DNSify the name to avoid error while deploying the operator.

`generate-helmchart` target is remplacing strings in templates and `values.yaml` and adding `read` rights to the `helm-charts` folder, to change this behaviour take a look in the `Makefile` and `Makefile_cpy` files. 
`Makefile_cpy` is used to replace the generated `Makefile` when initializing the operator (see below)

Then, 
```bash
 operator-sdk init --plugins helm --domain redhat.com --kind Parodos --helm-chart helm-charts/parodos-v1-0-19 && cp Makefile_cpy Makefile
```

```bash
>> operator-sdk init --plugins helm --domain redhat.com --kind Parodos --helm-chart helm-charts/parodos-v1-0-19
Writing kustomize manifests for you to edit...
Creating the API:
$ operator-sdk create api --kind Parodos --helm-chart helm-charts/parodos-v1-0-19
Writing kustomize manifests for you to edit...
Created helm-charts/parodos-v1-0-19
Generating RBAC rules
WARN[0000] Using default RBAC rules: failed to generate RBAC rules: failed to get server resources: Get "https://127.0.0.1:46431/api?timeout=32s": dial tcp 127.0.0.1:46431: connect: connection refused 
```

## Deploy

To push the docker image
```bash
make docker-build docker-push
```

Then, 
```bash
make install deploy
```

Check the parodos-operator-controller-manager is running:
```bash
kubectl -n parodos-operator-system get pods
NAME                                                   READY   STATUS    RESTARTS   AGE
parodos-operator-controller-manager-68f776c5fb-nvl2t   2/2     Running   0          19m
```

Once it is running, you can deploy the sample (here in the `default` namespace):
```bash
kubectl apply -f config/samples/charts_v1alpha1_parodos.yaml
```

Wait until the pods are up and running. It may take some times and some restarts as the posgresql needs to be available before anything else can start
```bash
kubectl get pods
NAME                                    READY   STATUS    RESTARTS      AGE
backstage-5468f68f5b-pmfn6              1/1     Running   2 (54s ago)   3m10s
notification-service-649656d6cb-btv4j   1/1     Running   0             3m10s
postgres-7857bf5cf-vb2qj                1/1     Running   0             3m10s
workflow-service-d5c4884c-t8zl4         1/1     Running   2 (50s ago)   3m10s
```

Now we can port forward the backstage port (`7007` by default) in order to access it from `localhost`:
```bash
kubectl port-forward parodos-backstage-7f46f49fb9-n4gx2 7007:7007 &
Forwarding from 127.0.0.1:7007 -> 7007
Forwarding from [::1]:7007 -> 7007
Handling connection for 7007
```

You should be able to access `http://localhost:7007/` with `test/test` credentials.

## Push to OperatorHub
See https://k8s-operatorhub.github.io/community-operators/packaging-operator/

First, we need to create a bundle, see https://sdk.operatorframework.io/docs/olm-integration/quickstart-bundle/#creating-a-bundle for more details

Make sure to update the `VERSION` in the Makefile or to export it first
```bash
make bundle
/usr/local/bin/operator-sdk generate kustomize manifests -q

Display name for the operator (required): 
> parodos-controller

Description for the operator (required): 
> Parodos controller to create parodos environment

Provider's name for the operator (required): 
> parodos-dev

Any relevant URL for the provider name (optional): 
> https://github.com/parodos-dev

Comma-separated list of keywords for your operator (required): 
> parodos,postgresql,backstage

Comma-separated list of maintainers and their emails (e.g. 'name1:email1, name2:email2') (required): 
> gabriel:gfarache@redhat.com,gloria:gciavarr@redhat.com,richard:ricwang@redhat.com,annel:aketcha@redhat.com                       
cd config/manager && /home/gabriel/git/parados_stuff/parodos-operator/bin/kustomize edit set image controller=quay.io/parodos-dev/parodos-operator:latest
/home/gabriel/git/parados_stuff/parodos-operator/bin/kustomize build config/manifests | /usr/local/bin/operator-sdk generate bundle -q --overwrite --version 0.0.1  
INFO[0000] Creating bundle.Dockerfile                   
INFO[0000] Creating bundle/metadata/annotations.yaml    
INFO[0000] Bundle metadata generated successfully       
/usr/local/bin/operator-sdk bundle validate ./bundle
INFO[0000] All validation tests have completed successfully 
```

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

Let's build and validate the bundle image:
```bash
make bundle-build bundle-push
docker build -f bundle.Dockerfile -t quay.io/parodos-dev/parodos-operator-bundle:v0.0.1 .
[+] Building 0.1s (7/7) FINISHED                                                                                                                             
 => [internal] load build definition from bundle.Dockerfile                                                                                             0.0s
 => => transferring dockerfile: 975B                                                                                                                    0.0s
 => [internal] load .dockerignore                                                                                                                       0.0s
 => => transferring context: 2B                                                                                                                         0.0s
 => [internal] load build context                                                                                                                       0.0s
 => => transferring context: 718B                                                                                                                       0.0s
 => CACHED [1/3] COPY bundle/manifests /manifests/                                                                                                      0.0s
 => CACHED [2/3] COPY bundle/metadata /metadata/                                                                                                        0.0s
 => CACHED [3/3] COPY bundle/tests/scorecard /tests/scorecard/                                                                                          0.0s
 => exporting to image                                                                                                                                  0.0s
 => => exporting layers                                                                                                                                 0.0s
 => => writing image sha256:f30da8634124a3d97ef7ea95d243e8a8ef37ff8069ee8ac66060fec0ccb7ceb7                                                            0.0s
 => => naming to quay.io/parodos-dev/parodos-operator-bundle:v0.0.1                                                                                     0.0s
make docker-push IMG=quay.io/parodos-dev/parodos-operator-bundle:v0.0.1
make[1]: Entering directory '/home/gabriel/git/parados_stuff/parodos-operator'
docker push quay.io/parodos-dev/parodos-operator-bundle:v0.0.1
The push refers to repository [quay.io/parodos-dev/parodos-operator-bundle]
a7714f77c6bb: Pushed 
dd8d2d4a35ad: Pushed 
2c140f320731: Pushed 
v0.0.1: digest: sha256:e6d1b970eb7d38264ace62e49316be475cb77cd64f30d241e33581f367bf5cec size: 939
make[1]: Leaving directory '/home/gabriel/git/parados_stuff/parodos-operator'
```
```bash
operator-sdk bundle validate quay.io/parodos-dev/parodos-operator-bundle:v0.0.1
INFO[0000] Unpacking image layers                       
INFO[0000] running /usr/bin/docker pull quay.io/parodos-dev/parodos-operator-bundle:v0.0.1 
INFO[0002] running docker create                        
INFO[0002] running docker cp                            
INFO[0002] running docker rm                            
INFO[0002] All validation tests have completed successfully 
```

You can now test it by following: https://k8s-operatorhub.github.io/community-operators/testing-operators/

```bash
operator-sdk bundle validate ./bundle --select-optional suite=operatorframework
WARN[0000] Warning: Value : (parodos-operator.v0.0.1) csv.Spec.minKubeVersion is not informed. It is recommended you provide this information. Otherwise, it would mean that your operator project can be distributed and installed in any cluster version available, which is not necessarily the case for all projects. 
WARN[0000] Warning: Value parodos-operator.v0.0.1: owned CRD "parodos.charts.redhat.com" has an empty description 
WARN[0000] Warning: Value parodos-operator.v0.0.1: owned CRD "parodosv1019s.charts.redhat.com" has an empty description 
INFO[0000] All validation tests have completed successfully 
```

```bash
operator-sdk scorecard bundle
--------------------------------------------------------------------------------
Image:      quay.io/operator-framework/scorecard-test:v1.30.0
Entrypoint: [scorecard-test olm-crds-have-validation]
Labels:
        "test":"olm-crds-have-validation-test"
        "suite":"olm"
Results:
        Name: olm-crds-have-validation
        State: fail

        Suggestions:
                Add CRD validation for spec field `notificationService` in ParodosV1019/v1
                Add CRD validation for spec field `notificationServiceConfig` in ParodosV1019/v1
                Add CRD validation for spec field `workflowService` in ParodosV1019/v1
                Add CRD validation for spec field `config` in ParodosV1019/v1
                Add CRD validation for spec field `kubernetesClusterDomain` in ParodosV1019/v1
                Add CRD validation for spec field `fullnameOverride` in ParodosV1019/v1
                Add CRD validation for spec field `postgres` in ParodosV1019/v1
                Add CRD validation for spec field `postgresSecretsTh27274644` in ParodosV1019/v1
                Add CRD validation for spec field `pvc` in ParodosV1019/v1
                Add CRD validation for spec field `appConfig4Gh2B9Ghf4` in ParodosV1019/v1
                Add CRD validation for spec field `backstage` in ParodosV1019/v1
                Add CRD validation for spec field `postgres` in Parodos/v1alpha1
                Add CRD validation for spec field `workflowService` in Parodos/v1alpha1
                Add CRD validation for spec field `backstage` in Parodos/v1alpha1
                Add CRD validation for spec field `config` in Parodos/v1alpha1
                Add CRD validation for spec field `kubernetesClusterDomain` in Parodos/v1alpha1
                Add CRD validation for spec field `notificationService` in Parodos/v1alpha1
                Add CRD validation for spec field `notificationServiceConfig` in Parodos/v1alpha1
                Add CRD validation for spec field `postgresSecretsTh27274644` in Parodos/v1alpha1
                Add CRD validation for spec field `pvc` in Parodos/v1alpha1
                Add CRD validation for spec field `appConfig4Gh2B9Ghf4` in Parodos/v1alpha1
                Add CRD validation for spec field `fullnameOverride` in Parodos/v1alpha1
        Log:
                Loaded 2 Custom Resources from alm-examples
                Loaded CustomresourceDefinitions: [&CustomResourceDefinition{ObjectMeta:{parodos.charts.redhat.com      0 0001-01-01 00:00:00 +0000 UTC <nil> <nil> map[] map[] [] [] []},Spec:CustomResourceDefinitionSpec{Group:charts.redhat.com,Names:CustomResourceDefinitionNames{Plural:parodos,Singular:parodos,ShortNames:[],Kind:Parodos,ListKind:ParodosList,Categories:[],},Scope:Namespaced,Versions:[]CustomResourceDefinitionVersion{CustomResourceDefinitionVersion{Name:v1alpha1,Served:true,Storage:true,Schema:&CustomResourceValidation{OpenAPIV3Schema:&JSONSchemaProps{ID:,Schema:,Ref:nil,Description:Parodos is the Schema for the parodos API,Type:object,Format:,Title:,Default:nil,Maximum:nil,ExclusiveMaximum:false,Minimum:nil,ExclusiveMinimum:false,MaxLength:nil,MinLength:nil,Pattern:,MaxItems:nil,MinItems:nil,UniqueItems:false,MultipleOf:nil,Enum:[]JSON{},MaxProperties:nil,MinProperties:nil,Required:[],Items:nil,AllOf:[]JSONSchemaProps{},OneOf:[]JSONSchemaProps{},AnyOf:[]JSONSchemaProps{},Not:nil,Properties:map[string]JSONSchemaProps{apiVersion: {  <nil> APIVersion defines the versioned schema of this representation of an object. Servers should convert recognized schemas to the latest internal value, and may reject unrecognized values. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources string   nil <nil> false <nil> false <nil> <nil>  <nil> <nil> false <nil> [] <nil> <nil> [] nil [] [] [] nil map[] nil map[] map[] nil map[] nil nil false <nil> false false [] <nil> <nil> []},kind: {  <nil> Kind is a string value representing the REST resource this object represents. Servers may infer this from the endpoint the client submits requests to. Cannot be updated. In CamelCase. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds string   nil <nil> false <nil> false <nil> <nil>  <nil> <nil> false <nil> [] <nil> <nil> [] nil [] [] [] nil map[] nil map[] map[] nil map[] nil nil false <nil> false false [] <nil> <nil> []},metadata: {  <nil>  object   nil <nil> false <nil> false <nil> <nil>  <nil> <nil> false <nil> [] <nil> <nil> [] nil [] [] [] nil map[] nil map[] map[] nil map[] nil nil false <nil> false false [] <nil> <nil> []},spec: {  <nil> Spec defines the desired state of Parodos object   nil <nil> false <nil> false <nil> <nil>  <nil> <nil> false <nil> [] <nil> <nil> [] nil [] [] [] nil map[] nil map[] map[] nil map[] nil nil false 0xc0009ed6f1 false false [] <nil> <nil> []},status: {  <nil> Status defines the observed state of Parodos object   nil <nil> false <nil> false <nil> <nil>  <nil> <nil> false <nil> [] <nil> <nil> [] nil [] [] [] nil map[] nil map[] map[] nil map[] nil nil false 0xc0009ed6f0 false false [] <nil> <nil> []},},AdditionalProperties:nil,PatternProperties:map[string]JSONSchemaProps{},Dependencies:JSONSchemaDependencies{},AdditionalItems:nil,Definitions:JSONSchemaDefinitions{},ExternalDocs:nil,Example:nil,Nullable:false,XPreserveUnknownFields:nil,XEmbeddedResource:false,XIntOrString:false,XListMapKeys:[],XListType:nil,XMapType:nil,XValidations:[]ValidationRule{},},},Subresources:&CustomResourceSubresources{Status:&CustomResourceSubresourceStatus{},Scale:nil,},AdditionalPrinterColumns:[]CustomResourceColumnDefinition{},Deprecated:false,DeprecationWarning:nil,},},Conversion:nil,PreserveUnknownFields:false,},Status:CustomResourceDefinitionStatus{Conditions:[]CustomResourceDefinitionCondition{},AcceptedNames:CustomResourceDefinitionNames{Plural:,Singular:,ShortNames:[],Kind:,ListKind:,Categories:[],},StoredVersions:[],},} &CustomResourceDefinition{ObjectMeta:{parodosv1019s.charts.redhat.com      0 0001-01-01 00:00:00 +0000 UTC <nil> <nil> map[] map[] [] [] []},Spec:CustomResourceDefinitionSpec{Group:charts.redhat.com,Names:CustomResourceDefinitionNames{Plural:parodosv1019s,Singular:parodosv1019,ShortNames:[],Kind:ParodosV1019,ListKind:ParodosV1019List,Categories:[],},Scope:Namespaced,Versions:[]CustomResourceDefinitionVersion{CustomResourceDefinitionVersion{Name:v1,Served:true,Storage:true,Schema:&CustomResourceValidation{OpenAPIV3Schema:&JSONSchemaProps{ID:,Schema:,Ref:nil,Description:ParodosV1019 is the Schema for the parodosv1019s API,Type:object,Format:,Title:,Default:nil,Maximum:nil,ExclusiveMaximum:false,Minimum:nil,ExclusiveMinimum:false,MaxLength:nil,MinLength:nil,Pattern:,MaxItems:nil,MinItems:nil,UniqueItems:false,MultipleOf:nil,Enum:[]JSON{},MaxProperties:nil,MinProperties:nil,Required:[],Items:nil,AllOf:[]JSONSchemaProps{},OneOf:[]JSONSchemaProps{},AnyOf:[]JSONSchemaProps{},Not:nil,Properties:map[string]JSONSchemaProps{apiVersion: {  <nil> APIVersion defines the versioned schema of this representation of an object. Servers should convert recognized schemas to the latest internal value, and may reject unrecognized values. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources string   nil <nil> false <nil> false <nil> <nil>  <nil> <nil> false <nil> [] <nil> <nil> [] nil [] [] [] nil map[] nil map[] map[] nil map[] nil nil false <nil> false false [] <nil> <nil> []},kind: {  <nil> Kind is a string value representing the REST resource this object represents. Servers may infer this from the endpoint the client submits requests to. Cannot be updated. In CamelCase. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds string   nil <nil> false <nil> false <nil> <nil>  <nil> <nil> false <nil> [] <nil> <nil> [] nil [] [] [] nil map[] nil map[] map[] nil map[] nil nil false <nil> false false [] <nil> <nil> []},metadata: {  <nil>  object   nil <nil> false <nil> false <nil> <nil>  <nil> <nil> false <nil> [] <nil> <nil> [] nil [] [] [] nil map[] nil map[] map[] nil map[] nil nil false <nil> false false [] <nil> <nil> []},spec: {  <nil> Spec defines the desired state of ParodosV1019 object   nil <nil> false <nil> false <nil> <nil>  <nil> <nil> false <nil> [] <nil> <nil> [] nil [] [] [] nil map[] nil map[] map[] nil map[] nil nil false 0xc0009ed6f2 false false [] <nil> <nil> []},status: {  <nil> Status defines the observed state of ParodosV1019 object   nil <nil> false <nil> false <nil> <nil>  <nil> <nil> false <nil> [] <nil> <nil> [] nil [] [] [] nil map[] nil map[] map[] nil map[] nil nil false 0xc0009ed6f3 false false [] <nil> <nil> []},},AdditionalProperties:nil,PatternProperties:map[string]JSONSchemaProps{},Dependencies:JSONSchemaDependencies{},AdditionalItems:nil,Definitions:JSONSchemaDefinitions{},ExternalDocs:nil,Example:nil,Nullable:false,XPreserveUnknownFields:nil,XEmbeddedResource:false,XIntOrString:false,XListMapKeys:[],XListType:nil,XMapType:nil,XValidations:[]ValidationRule{},},},Subresources:&CustomResourceSubresources{Status:&CustomResourceSubresourceStatus{},Scale:nil,},AdditionalPrinterColumns:[]CustomResourceColumnDefinition{},Deprecated:false,DeprecationWarning:nil,},},Conversion:nil,PreserveUnknownFields:false,},Status:CustomResourceDefinitionStatus{Conditions:[]CustomResourceDefinitionCondition{},AcceptedNames:CustomResourceDefinitionNames{Plural:,Singular:,ShortNames:[],Kind:,ListKind:,Categories:[],},StoredVersions:[],},}]


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
                parodosv1019s.charts.redhat.com does not have status spec. Note thatAll objects that represent a physical resource whose state may vary from the user's desired intent SHOULD have a spec and a status. More info: https://github.com/kubernetes/community/blob/master/contributors/devel/sig-architecture/api-conventions.md#spec-and-status
                parodos.charts.redhat.com does not have status spec. Note thatAll objects that represent a physical resource whose state may vary from the user's desired intent SHOULD have a spec and a status. More info: https://github.com/kubernetes/community/blob/master/contributors/devel/sig-architecture/api-conventions.md#spec-and-status
        Log:
                Loaded ClusterServiceVersion: parodos-operator.v0.0.1
                Loaded 2 Custom Resources from alm-examples


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
                Loaded ClusterServiceVersion: parodos-operator.v0.0.1


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
                time="2023-07-12T12:40:13Z" level=debug msg="Found manifests directory" name=bundle-test
                time="2023-07-12T12:40:13Z" level=debug msg="Found metadata directory" name=bundle-test
                time="2023-07-12T12:40:13Z" level=debug msg="Getting mediaType info from manifests directory" name=bundle-test
                time="2023-07-12T12:40:13Z" level=debug msg="Found annotations file" name=bundle-test
                time="2023-07-12T12:40:13Z" level=debug msg="Could not find optional dependencies file" name=bundle-test


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
Entrypoint: [scorecard-test olm-spec-descriptors]
Labels:
        "suite":"olm"
        "test":"olm-spec-descriptors-test"
Results:
        Name: olm-spec-descriptors
        State: fail

        Suggestions:
                Add a spec descriptor for postgres
                Add a spec descriptor for postgresSecretsTh27274644
                Add a spec descriptor for pvc
                Add a spec descriptor for appConfig4Gh2B9Ghf4
                Add a spec descriptor for kubernetesClusterDomain
                Add a spec descriptor for notificationService
                Add a spec descriptor for notificationServiceConfig
                Add a spec descriptor for backstage
                Add a spec descriptor for config
                Add a spec descriptor for fullnameOverride
                Add a spec descriptor for workflowService
                Add a spec descriptor for appConfig4Gh2B9Ghf4
                Add a spec descriptor for kubernetesClusterDomain
                Add a spec descriptor for notificationService
                Add a spec descriptor for pvc
                Add a spec descriptor for workflowService
                Add a spec descriptor for backstage
                Add a spec descriptor for config
                Add a spec descriptor for fullnameOverride
                Add a spec descriptor for notificationServiceConfig
                Add a spec descriptor for postgres
                Add a spec descriptor for postgresSecretsTh27274644
        Errors:
                postgres does not have a spec descriptor
                postgresSecretsTh27274644 does not have a spec descriptor
                pvc does not have a spec descriptor
                appConfig4Gh2B9Ghf4 does not have a spec descriptor
                kubernetesClusterDomain does not have a spec descriptor
                notificationService does not have a spec descriptor
                notificationServiceConfig does not have a spec descriptor
                backstage does not have a spec descriptor
                config does not have a spec descriptor
                fullnameOverride does not have a spec descriptor
                workflowService does not have a spec descriptor
                appConfig4Gh2B9Ghf4 does not have a spec descriptor
                kubernetesClusterDomain does not have a spec descriptor
                notificationService does not have a spec descriptor
                pvc does not have a spec descriptor
                workflowService does not have a spec descriptor
                backstage does not have a spec descriptor
                config does not have a spec descriptor
                fullnameOverride does not have a spec descriptor
                notificationServiceConfig does not have a spec descriptor
                postgres does not have a spec descriptor
                postgresSecretsTh27274644 does not have a spec descriptor
        Log:
                Loaded ClusterServiceVersion: parodos-operator.v0.0.1
                Loaded 2 Custom Resources from alm-examples
```

You can now fork https://github.com/k8s-operatorhub/community-operators/ then
* create a new folder in `operators/parodos-operator` with the new operator's version as name (ie: 0.0.1)
* copy the folder `bundle` previously generated into the newly created folder
* commit the changes and push them
    * you need to sign the commit: https://github.com/operator-framework/community-operators/blob/master/docs/contributing-prerequisites.md#sign-your-work
* create a PR to have the new operator version available on OperatorHub