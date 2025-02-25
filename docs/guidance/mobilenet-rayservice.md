# Serve a MobileNet image classifier using RayService

> **Note:** The Python files for the Ray Serve application and its client are in the repository [ray-project/serve_config_examples](https://github.com/ray-project/serve_config_examples).

## Step 1: Create a Kubernetes cluster with Kind.

```sh
kind create cluster --image=kindest/node:v1.23.0
```

## Step 2: Install KubeRay operator

Follow [this document](../../helm-chart/kuberay-operator/README.md) to install the nightly KubeRay operator via 
Helm. Note that the YAML file in Step 3 uses `serveConfigV2`, which is first supported by KubeRay v0.6.0.

## Step 3: Install a RayService

```sh
# path: ray-operator/config/samples/
kubectl apply -f ray-service.mobilenet.yaml
```

* The [mobilenet.py](https://github.com/ray-project/serve_config_examples/blob/master/mobilenet/mobilenet.py) file requires `tensorflow` as a dependency. Hence, the YAML file uses `rayproject/ray-ml:2.5.0` instead of `rayproject/ray:2.5.0`.
* `python-multipart` is required for the request parsing function `starlette.requests.form()`, so the YAML file includes `python-multipart` in the runtime environment.

## Step 4: Forward the port of Serve

```sh
kubectl port-forward svc/rayservice-mobilenet-serve-svc 8000
```

Note that the Serve service will be created after the Serve applications are ready and running. This process may take approximately 1 minute after all Pods in the RayCluster are running.

## Step 5: Send a request to the ImageClassifier

* Step 5.1: Prepare an image file.
* Step 5.2: Update `image_path` in [mobilenet_req.py](https://github.com/ray-project/serve_config_examples/blob/master/mobilenet/mobilenet_req.py)
* Step 5.3: Send a request to the `ImageClassifier`.
  ```sh
  python mobilenet_req.py
  # sample output: {"prediction":["n02099601","golden_retriever",0.17944198846817017]}
  ```
