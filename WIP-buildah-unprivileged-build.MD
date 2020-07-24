# How to Run Containers as Unprivileged (Rootless) User with Buildah
### Running on Fedora 32 OS

### Documentation:

#### Buildah:
[Rootless User Configuration in Buildah Tutorial #1 Doc](https://github.com/containers/buildah/blob/13f190e764230377037bf465ccbc307f809d748c/docs/tutorials/01-intro.md#rootless-user-configuration)
#### Podman:
[Basic Setup and Use of Podman in a Rootless environment](https://github.com/containers/podman/blob/master/docs/tutorials/rootless_tutorial.md#basic-setup-and-use-of-podman-in-a-rootless-environment)

---

### Github Issues:

##### github.com/containers/buildah
[How to use buildah bud inside of unprivileged container #1335  closed](https://github.com/containers/buildah/issues/1335)

[Unprivileged buildah bud in Kube: Overlay fails, but vfs does not #2084 open](https://github.com/containers/buildah/issues/2084)

##### github.com/redhat-developer/build
[Track the buildah support and plan for build v2 #134 open](https://github.com/redhat-developer/build/issues/134)

[How to use Buildah bud in unprivileged container #1335  closed](https://github.com/containers/buildah/issues/1335)

---

### Articles:

[How rootless Buildah works: Building containers in unprivileged environments](https://opensource.com/article/19/3/tips-tricks-rootless-buildah#:~:text=It%20contains%20fields%20like%20the,is%20expected%20to%20be%20used)

[How Does Rootless Podman Work?](https://opensource.com/article/19/2/how-does-rootless-podman-work) By Dan Walsh on opensource.com

[Running containers as root or rootless](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux_atomic_host/7/html/managing_containers/finding_running_and_building_containers_with_podman_skopeo_and_buildah#running_containers_as_root_or_rootless) on Offical Red Hat Documentation 

---

### Prep/Installation:
1. Install [Podman](https://podman.io/getting-started/installation.html)

  Fedora(CoreOS) comes with Podman installed already. 
  
 2. Install [Buildah]
