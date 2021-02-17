##### BEFORE STARTING: 
1.) delete gitops repo in github if previously created, should be uninitialized. (https://github.com/reginapizza/gitops)

2.) removed the gitops repo locally
```
rm -rf output
```
3.) Sign into OpenShift cluster. 
```
oc login --token={token} --server={server}
```

##### If doing development locally, sign in to cluster and run:
```
$ source ./contrib/oc-environment.sh
$ ./build.sh
$ ./bin/bridge
```

Now with your locally running dev console:

2.) Install Sealed Secrets Operator (in `cicd` namespace) and then create `sealedsecretscontroller`

6.) Install Gitops Operator

7.) Run
```
kam bootstrap \
  --service-repo-url https://github.com/reginapizza/taxi.git \
  --gitops-repo-url https://github.com/reginapizza/gitops.git \
  --image-repo quay.io/reginapizza/taxi \
  --dockercfgjson ~/Downloads/reginapizza-robot-auth.json \
  --git-host-access-token {github token} \
  --output ~/output \
  --push-to-git=true
```

8.) Run
```
cd output
oc apply -k config/argocd/
```

10.) 10.) Switch to Developer view and go to `openshift-gitops` namespace, click on Secrets tab, and then find `argocd-cluser-cluster`. Click on it and then copy the password at the bottom of the page. Go to Console Application Launcher and click on the ArgoCD link, then log in with username `admin` and password from clipboard.

11.) Run (sometimes I run this if it isn't synced properly- not sure if I need to?)
```
oc apply -k config/cicd/base
```

12.) Create the webhook with
```
 kam webhook create \
    --git-host-access-token {github token} \
    --env-name dev \
    --service-name taxi
```

13.) Run (?)
```
kubectl create namespace pipelines-kubeadmin-github
```

14.) Run (?)
```
kubectl create secret -n pipelines-kubeadmin-github generic kubeadmin-github-token --from-literal=token={github token}
```

Application Stages should now be populated; clicking on an application should take you to the Application Details page.
____________________________________
DAY 2 Kam Tasks:

1.) Run
```
kam environment add \
  --env-name new-env \
  --pipelines-folder ~/output
```

2.) Run
```
kam service add \
  --env-name new-env \
  --app-name app-bus \
  --service-name bus \
  --git-repo-url http://github.com/reginapizza/bus.git \
  --pipelines-folder ~/output
```

3.) Open ~/output in code editor. 
```
code .
```

4.) Navigate to  `~/output/environments/new-env/apps/app-bus/services/bus/base/config` and in that config folder add a three new files called 
- `100-deployment.yaml`
- `200-service.yaml` 
- `kustomization.yaml`

5.) In `100-deployment.yaml`, add:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  name: bus
  namespace: new-env
spec:
  replicas: 1
  selector:
    matchLabels:
      app.kubernetes.io/name: bus
      app.kubernetes.io/part-of: app-bus
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app.kubernetes.io/name: bus
        app.kubernetes.io/part-of: app-bus
    spec:
      containers:
      - image: nginxinc/nginx-unprivileged:latest
        imagePullPolicy: Always
        name: bus
        ports:
        - containerPort: 8080
        resources: {}
      serviceAccountName: default
status: {}
```

6.) In `200-service.yaml` add:
```
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app.kubernetes.io/name: bus
    app.kubernetes.io/part-of: app-bus
  name: bus
  namespace: dev
spec:
  ports:
  - name: http
    port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    app.kubernetes.io/name: bus
    app.kubernetes.io/part-of: app-bus
status:
  loadBalancer: {}
```

7.) In `kustomization.yaml` add:
```
resources:
- 100-deployment.yaml
- 200-service.yaml
```

8.) Now save changes and run
```
$ git add .
$ git commit -m "Add new service"
$ git push origin main
```
