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

1.) Intstall Openshift Pipelines Operator

2.) Install Sealed Secrets Operator (in `cicd` namespace) and then create `sealedsecretscontroller`

3.) Install ArgoCD Operator (in `argocd` namespace)

4.) Run `oc adm policy add-cluster-role-to-user cluster-admin system:serviceaccount:argocd:argocd-application-controller`

5.) Run `oc replace -f https://raw.githubusercontent.com/redhat-developer/kam/master/docs/updates/buildah.yaml`

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

9.) Commit local changes to gitops repo created by kam and push to main github branch. 
```
$ git init
$ git add .
$ git commit -m "Initial commit"
$ git remote add origin https://github.com/reginapizza/gitops.git
$ git push -u origin master
```

10.) Go to `argocd` namespace and go to `argocd-server` deployment pod to get link to ArgoCD web console, then sign in with `admin` as username and token from running
```
kubectl get secret argocd-cluster -n argocd -ojsonpath='{.data.admin\.password}' | base64 --decode
```

11.) Run 
```
oc apply -k config/cicd/base
```

12.) Create the webhook with
```
 kam webhook create \
    --access-token {github token} \
    --env-name dev \
    --service-name taxi
```

13.) Run
```
kubectl create namespace pipelines-kubeadmin-github
```

14.) Run
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
$ git push origin master
```
