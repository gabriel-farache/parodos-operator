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
 operator-sdk create api --version v1 --helm-chart helm-charts/parodos-v1-0-19
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


```bash
>> operator-sdk create api --version v1 --helm-chart helm-charts/parodos-v1-0-19 

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

Once it is running, you can deploy the sample (here for v1.0.19 and on the `default` namespace):
```bash
kubectl apply -f config/samples/charts_v1_parodosv1019.yaml
```

Wait until the pods are up and running. It may take some times and some restarts as the posgresql needs to be available before anything else can start
```bash
kubectl get pods
NAME                                                    READY   STATUS    RESTARTS      AGE
parodos-v1-0-19-backstage-7f46f49fb9-n4gx2              1/1     Running   1 (15m ago)   17m
parodos-v1-0-19-notification-service-548f5544f8-v28k7   1/1     Running   0             17m
parodos-v1-0-19-postgres-684d57544d-dl2td               1/1     Running   0             17m
parodos-v1-0-19-workflow-service-7b9cfc8cb5-tksgn       1/1     Running   1 (16m ago)   17m
```

Now we can port forward the backstage port (`7007` by default) in order to access it from `localhost`:
```bash
kubectl port-forward parodos-v1-0-19-backstage-7f46f49fb9-n4gx2 7007:7007 &
Forwarding from 127.0.0.1:7007 -> 7007
Forwarding from [::1]:7007 -> 7007
Handling connection for 7007
```

You should be able to access `http://localhost:7007/` with `test/test` credentials.