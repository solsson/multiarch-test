# multiarch-test image

Can be used to test how your container image infra deals with multiarch images.

```
IMAGE=solsson/multiarch-test
TAG=latest

# build
docker buildx build --platform=linux/amd64,linux/arm64/v8 -t $IMAGE:$TAG --push .

# required if registry is Docker Hub
DOCKER_HUB_TOKEN=$(curl -sSL "https://auth.docker.io/token?service=registry.docker.io&scope=repository:$IMAGE:pull" | jq --raw-output .token)

# check that the image tag is a multiarch manifest
curl -H "Authorization: Bearer ${DOCKER_HUB_TOKEN}" -L "https://index.docker.io/v2/$IMAGE/manifests/$TAG" -I | grep -E --color "vnd.oci.image.index.v1|$"
curl -f "Authorization: Bearer ${DOCKER_HUB_TOKEN}" -L https://index.docker.io/v2/$IMAGE/manifests/$TAG | jq '.mediaType'
```

Using [crane](https://github.com/google/go-containerregistry):

```
crane pull --platform all --format oci $IMAGE:$TAG multiarch-test-$TAG.oci
BLOB=$(cat multiarch-test-$TAG.oci/index.json | jq -r '.manifests[0].digest' | sed 's|:|/|')
cat multiarch-test-$TAG.oci/blobs/$BLOB | jq '.mediaType'
```
