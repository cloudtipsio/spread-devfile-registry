# https://github.com/eclipse-che/che-devfile-registry.git

./build.sh -t v1 -r cloudtips/devfile-registry -o docker.io

# create
oc new-app -f deploy/openshift/devfile-registry.yaml \
             -p IMAGE="cloudtips/devfile-registry" \
             -p IMAGE_TAG="latest"

# delete
oc delete -f deploy/openshift/che-devfile-registry.yaml

# new
oc process \
    -f devfile-registry.yaml \
    -p IMAGE="cloudtips/devfile-registry" \
    -p IMAGE_TAG="v3" | oc apply -f-


## oc patch checlusters.org.eclipse.che devspaces --type=merge --patch-file devspacescrd.json


# change files and folder permission
find devspaces -type d -exec chmod 755 {} \;
find devspaces -type f -exec chmod 644 {} \;
find . -name "*.sh" -exec chmod +x {} \;





# build, push and publish devfile-registry

IMAGE_TAG="v1"

./build.sh -t ${IMAGE_TAG}

oc patch checlusters devspaces \
    --type=merge \
    --patch='{"spec":{"components":{"devfileRegistry":{"deployment":{"containers":[{"image":"docker.io/cloudtips/devfile-registry:'${IMAGE_TAG}'"}]}}}}}' \
    -n eclipse-che

