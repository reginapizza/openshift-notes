## How to get a local image onto the internal OpenShift registry using Quay.io

Note: you will need to have an account on quay.io and oc client tool installed to begin.

1. Log into OCP cluster

2. In quay.io, create a new blank repository (this will be where you will push your image to). For this example my blank repo's name is "buildpack-test" and my username is rescott.

3. Run `docker login quay.io` . Input username and password for your quay account when promtped.

4. See your local images by running `docker images`, take note of the name of the image you want to push. 

5. Run `docker run my-buildpack echo "fun" > newfile`. This will create a container with a single new file based off of your image. The container will terminate immediately because it's one command is `echo`.

6. Find your container ID by running `docker ps -l`. You should see your container based off of your image and to the left of it you will see that container's ID. Copy it down. Mine is 7cce9db1c5b4.

7. Commit it by running `docker commit 7cce9db1c5b4 quay.io/rescott/buildpack-test`. The output should be a long sha256 id. 

8. Push to your quay repository by running `docker push quay.io/rescott/buildpack-test`

9. To push to openshift registry, run `oc tag quay.io/rescott/buildpack-test:latest buildpack-test:latest`. Your output should say "Tag buildpack-test:latest set to quay.io/rescott/buildpack-test:latest."

10.) Run `oc get images`. You should see your image in that list now. 


#### Managing Your Images

OpenShift Docs on Managing Image Streams: https://docs.openshift.com/container-platform/4.4/openshift_images/image-streams-manage.html#images-imagestreams-external-image-tags_image-streams-managing

If you want to update your image in openshift, you would update it locally, tag it again, push to quay, and then run `oc tag quay.io/rescott/buildpack-test:latest buildpack-test:latest`

If you want to push a new image with a pointer to a previous image you use the format  `oc tag buildpack-test:tag1 buildpack-test:tag2`. So if I wanted to make a new tag that points to my old tag, I would run `oc tag buildpack-test:latest buildpack-test:2.0`. This would configure the image’s latest tag to always refer to the current image for the tag 2.0. 

If you want to update the latest tag, you can also just push to the same `latest` tag in quay by running `oc tag quay.io/rescott/buildpack-test:latest buildpack-test:latest` and then manually re-sync.

To set up auto sync with your quay.io image, see:  https://docs.openshift.com/container-platform/4.4/openshift_images/image-streams-manage.html#images-imagestreams-import_image-streams-managing
