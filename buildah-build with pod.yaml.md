### Steps to Create an Image using Buildah and Push it to Dockerhub:

1.) Make new folder. For this demo it will just be called `demo`.

2.) Create your repository on dockerhub.io. For this example it will be called `rescott/buildah-demo` where rescott is my dockerhub username. 

3.) Add pod.yaml file to `demo` folder
```
apiVersion: v1
kind: Pod
metadata:
  name: buildah
spec:
  volumes:
  - name: build-context
    emptyDir: {}
  - name: buildah-secret
    secret:
      secretName: regcred
      items:
        - key: .dockerconfigjson
          path: config.json
  initContainers:
    - name: buildah-init
      image: ubuntu
      args:
        - "sh"
        - "-c"
        - "while true; do sleep 1; if [ -f /tmp/complete ]; then break; fi done"
      volumeMounts:
        - name: build-context
          mountPath: /buildah/build-context
  containers:
  - name: buildahcontainer
    image: quay.io/buildah/stable:latest
    resources:
      limits:
        memory: "1Gi"
        cpu: "500m"
    securityContext:
        privileged: true
    args:
        - buildah
        - bud
        - --isolation 
        - chroot
        - -t
        - mybuildahdemo
        - buildah/build-context
        - buildah
        - push
        - mybuildahdemo
        - docker://docker.io/rescott/buildah-demo:latest
    volumeMounts:
        - name: build-context
          mountPath: /buildah/build-context
        - name: buildah-secret
          mountPath: /buildah/.docker
```
4.) Add buildah-build-context.tar.gz to `demo` folder.

5.) Log into OCP cluster `oc login`

6.) Create and check into new namespace `oc new-project projectname`

7.) Make your secrets. Create this Secret, naming it regcred:

```
kubectl create secret docker-registry regcred --docker-server=<your-registry-server> --docker-username=<your-name> --docker-password=<your-pword> --docker-email=<your-email>
```
`<your-registry-server>` is your Private Docker Registry FQDN. (https://index.docker.io/v1/ for DockerHub)
`<your-name>` is your Docker username.
`<your-pword>` is your Docker password.
`<your-email>` is your Docker email.

For example (with fake inputs), mine would be: ```kubectl create secret docker-registry regcred --docker-server=https://index.docker.io/v1/ --docker-username=myusername --docker-password=fakepassword!123 --docker-email=myemail@redhat.com```
This secret will be used in pod.yaml config.

8.) Run `oc apply -f pod.yaml` (can also run `kubectl apply -f pod.yaml`)  will start up your init container, output will be `pod/buildah created`

9.) Run `oc get po` to see running pods, you will see your init container listed there.

10.) To see logs for your pod, run `oc logs buildah`. Right now you only have an init pod so it will say `Error from server (BadRequest): container "buildahcontainer" in pod "buildah" is waiting to start: PodInitializing`

11.) Run `oc cp $HOME/demo/buildah-build-context.tar.gz buildah:/tmp/buildah-build-context.tar.gz -c buildah-init` (no output expected)

12.) Run `oc exec buildah -c buildah-init -- tar -zxf /tmp/buildah-build-context.tar.gz -C /buildah/build-context` (no output expected)

13.) Run `oc exec buildah -c buildah-init -- touch /tmp/complete` (no output expected)

14.) Now if you run `oc logs buildah` (or look in your OpenShift console) you should be able to see your pod's logs now.


To make changes to your pod.yaml after this, delete your pod by running `oc delete po buildah`, make your changes in pod.yaml and save, and then restart at step #8.
