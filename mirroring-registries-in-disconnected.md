In a disconnected cluster you can’t pull container images outside from the cluster. 

First mirror the source application container image into the disconnected cluster mirror registry

**Mirror registry** : A dedicated registry that disconnect cluster can access within the cluster network
__________

### Mirror container images into mirror registry without certificate

STEP 1:
Login into the mirror registry
`$ podman login --tls-verify=false <mirror_registry_host>:<port>`

STEP 2:
Mirror the registry

`$ oc image mirror <external_registry_path>/<namespace>/<image:tag> <mirror_registry_host>:<port>/<namespace>/<image:tag>`
__________


### Mirror container images into mirror registry with certificate

STEP 1:
Get certificate from : https://gitlab.cee.redhat.com/tsze/flexy-templates/-/blob/master/functionality-testing/certs/all-in-one.1/client_ca.crt

STEP 2:
Save a copy of the cert from above on your host machine.

STEP 3:
copy the certificate to `/etc/pki/ca-trust/source/anchors` (as root)

STEP 4:
Then run $ update-ca-trust

STEP 5:
Login into the mirror registry
` $ podman login <mirror_registry_host>:<port>`

STEP 8:

`$ oc image mirror <external_registry_path>/<namespace>/<image:tag> <mirror_registry_host>:<port>/<namespace>/<image:tag>`
  
Go to the application source and change the image path to the mirror path, then create an application (edited) 
