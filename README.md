# Crossplane Service

![GitHub release (latest by date)](https://img.shields.io/github/v/release/keptn-sandbox/crossplane-service)
[![Go Report Card](https://goreportcard.com/badge/github.com/keptn-sandbox/crossplane-service)](https://goreportcard.com/report/github.com/keptn-sandbox/crossplane-service)


This service interacts with [Crossplane](https://crossplane.io) to manage infrastructure via declarative YAML files. 
The current implementation aims to create Kubernetes clusters on demand via Crossplane (triggered by Keptn) and to delete the clusters (also triggered via Keptn).
Therefore, the services makes us of two kind of Keptn cloud events:

- `sh.keptn.event.environment-setup.triggered`: to initiate the setup/creation of a new environment (cluster)
- `sh.keptn.event.environment-teardown.triggered`: to initiate the teardown/deletion of an environment (cluster)

A simple shipyard that makes use of this:
```
apiVersion: "spec.keptn.sh/0.2.0"
kind: "Shipyard"
metadata:
  name: "keptn-crossplane"
spec:
  stages:
    - name: "perf-test"
      sequences:
        - name: "delivery"
          tasks:
            - name: "environment-setup"
              properties:
                size: "medium"
            - name: "deployment"
              properties:
                deploymentstrategy: "user_managed"
            - name: "test"
              properties:
                teststrategy: "performance"
            - name: "evaluation"
             - name: "environment-teardown"
```

The `crossplane-service` will look for a resource `crossplane/cluster.yaml` in the Keptn managed git-repository and will basically execute a `kubectl apply` or `kubectl delete` on this resource to either create or delete the cluster.
In the example, the created cluster is already equipped with additional Keptn services, such as the [job-executor](https://github.com/keptn-sandbox/job-executor-service) and the [helm-service](https://github.com/keptn/keptn/tree/master/helm-service). 

## Demo

Please find demo resources in the `demo/` folder of this repo.



## Compatibility Matrix

*Please fill in your versions accordingly*

| Keptn Version    | [crossplane-service Docker Image](https://hub.docker.com/r/keptnsandbox/crossplane-service/tags) |
|:----------------:|:----------------------------------------:|
|       0.9.2      | keptnsandbox/crossplane-service:0.1.0 under development |


## Installation

The *crossplane-service* can be installed as a part of [Keptn's uniform](https://keptn.sh).

### Deploy in your Kubernetes cluster

To deploy the current version of the *crossplane-service* in your Keptn Kubernetes cluster, apply the [`deploy/service.yaml`](deploy/service.yaml) file:

```console
kubectl apply -f deploy/service.yaml
```

This should install the `crossplane-service` together with a Keptn `distributor` into the `keptn` namespace, which you can verify using

```console
kubectl -n keptn get deployment crossplane-service -o wide
kubectl -n keptn get pods -l run=crossplane-service
```

### Up- or Downgrading

Adapt and use the following command in case you want to up- or downgrade your installed version (specified by the `$VERSION` placeholder):

```console
kubectl -n keptn set image deployment/crossplane-service crossplane-service=keptnsandbox/crossplane-service:$VERSION --record
```

### Uninstall

To delete a deployed *crossplane-service*, use the file `deploy/*.yaml` files from this repository and delete the Kubernetes resources:

```console
kubectl delete -f deploy/service.yaml
```

## Development

Development can be conducted using any GoLang compatible IDE/editor (e.g., Jetbrains GoLand, VSCode with Go plugins).

It is recommended to make use of branches as follows:

* `master` contains the latest potentially unstable version
* `release-*` contains a stable version of the service (e.g., `release-0.1.0` contains version 0.1.0)
* create a new branch for any changes that you are working on, e.g., `feature/my-cool-stuff` or `bug/overflow`
* once ready, create a pull request from that branch back to the `master` branch

When writing code, it is recommended to follow the coding style suggested by the [Golang community](https://github.com/golang/go/wiki/CodeReviewComments).

### Where to start

If you don't care about the details, your first entrypoint is [eventhandlers.go](eventhandlers.go). Within this file 
 you can add implementation for pre-defined Keptn Cloud events.
 
To better understand all variants of Keptn CloudEvents, please look at the [Keptn Spec](https://github.com/keptn/spec).
 
If you want to get more insights into processing those CloudEvents or even defining your own CloudEvents in code, please 
 look into [main.go](main.go) (specifically `processKeptnCloudEvent`), [deploy/service.yaml](deploy/service.yaml),
 consult the [Keptn docs](https://keptn.sh/docs/) as well as existing [Keptn Core](https://github.com/keptn/keptn) and
 [Keptn Contrib](https://github.com/keptn-contrib/) services.

### Common tasks

* Build the binary: `go build -ldflags '-linkmode=external' -v -o crossplane-service`
* Run tests: `go test -race -v ./...`
* Build the docker image: `docker build . -t keptnsandbox/crossplane-service:dev` (Note: Ensure that you use the correct DockerHub account/organization)
* Run the docker image locally: `docker run --rm -it -p 8080:8080 keptnsandbox/crossplane-service:dev`
* Push the docker image to DockerHub: `docker push keptnsandbox/crossplane-service:dev` (Note: Ensure that you use the correct DockerHub account/organization)
* Deploy the service using `kubectl`: `kubectl apply -f deploy/`
* Delete/undeploy the service using `kubectl`: `kubectl delete -f deploy/`
* Watch the deployment using `kubectl`: `kubectl -n keptn get deployment crossplane-service -o wide`
* Get logs using `kubectl`: `kubectl -n keptn logs deployment/crossplane-service -f`
* Watch the deployed pods using `kubectl`: `kubectl -n keptn get pods -l run=crossplane-service`
* Deploy the service using [Skaffold](https://skaffold.dev/): `skaffold run --default-repo=your-docker-registry --tail` (Note: Replace `your-docker-registry` with your DockerHub username; also make sure to adapt the image name in [skaffold.yaml](skaffold.yaml))


### Testing Cloud Events

We have dummy cloud-events in the form of [RFC 2616](https://ietf.org/rfc/rfc2616.txt) requests in the [test-events/](test-events/) directory. These can be easily executed using third party plugins such as the [Huachao Mao REST Client in VS Code](https://marketplace.visualstudio.com/items?itemName=humao.rest-client).

## Automation

### GitHub Actions: Automated Pull Request Review

This repo uses [reviewdog](https://github.com/reviewdog/reviewdog) for automated reviews of Pull Requests. 

You can find the details in [.github/workflows/reviewdog.yml](.github/workflows/reviewdog.yml).

### GitHub Actions: Unit Tests

This repo has automated unit tests for pull requests. 

You can find the details in [.github/workflows/tests.yml](.github/workflows/tests.yml).

### GH Actions/Workflow: Build Docker Images

This repo uses GH Actions and Workflows to test the code and automatically build docker images.

Docker Images are automatically pushed based on the configuration done in [.ci_env](.ci_env) and the two [GitHub Secrets](https://github.com/keptn-sandbox/crossplane-service/settings/secrets/actions)
* `REGISTRY_USER` - your DockerHub username
* `REGISTRY_PASSWORD` - a DockerHub [access token](https://hub.docker.com/settings/security) (alternatively, your DockerHub password)

## How to release a new version of this service

It is assumed that the current development takes place in the master branch (either via Pull Requests or directly).

To make use of the built-in automation using GH Actions for releasing a new version of this service, you should

* branch away from master to a branch called `release-x.y.z` (where `x.y.z` is your version),
* write release notes in the [releasenotes/](releasenotes/) folder,
* check the output of GH Actions builds for the release branch, 
* verify that your image was built and pushed to DockerHub with the right tags,
* update the image tags in [deploy/service.yaml], and
* test your service against a working Keptn installation.

If any problems occur, fix them in the release branch and test them again.

Once you have confirmed that everything works and your version is ready to go, you should

* create a new release on the release branch using the [GitHub releases page](https://github.com/keptn-sandbox/crossplane-service/releases), and
* merge any changes from the release branch back to the master branch.

## License

Please find more information in the [LICENSE](LICENSE) file.
