# DevOps Challenge: Helm deployment on Kubernetes

Helm chart that deploys the Tradebyte demo app (Tornado + Redis) on Kubernetes, with 3+ replicas, CPU-based autoscaling, and configuration kept out of the image.

## What's included

- `Dockerfile`: builds the app image (`python:3.9-slim`, non-root user)
- `devops-challenge/`: Helm chart
  - app Deployment and NodePort Service
  - ConfigMap for plain settings and a Secret for `REDIS_HOST` and any extra sensitive values
  - single-replica Redis Deployment and Service
  - HorizontalPodAutoscaler: 3 to 10 replicas at 70% CPU
- Image published at `soltanahmed1960/devops-challenge-app` on Docker Hub

## Prerequisites

Docker, minikube, kubectl, Helm 3.

```bash
minikube start
minikube addons enable metrics-server   # required for the HPA
```

## Deploy

```bash
helm install my-app ./devops-challenge -n challenge --create-namespace
kubectl get pods,svc,hpa -n challenge
minikube service my-app-devops-challenge -n challenge
```

To build the image yourself instead of pulling it:

```bash
docker build -t <your-user>/devops-challenge-app:0.1.1 .
docker push <your-user>/devops-challenge-app:0.1.1
helm upgrade --install my-app ./devops-challenge -n challenge \
  --set image.repository=<your-user>/devops-challenge-app
```

## Configuration

Non-sensitive settings live under `env` in `values.yaml` and go into a ConfigMap. Sensitive values go under `secretEnv` and are stored in a Kubernetes Secret. Pass them at install time and never commit them:

```bash
helm upgrade my-app ./devops-challenge -n challenge --set secretEnv.MY_KEY=value
```

Pods restart automatically when the ConfigMap or Secret changes (checksum annotations on the pod template).

## Autoscaling test

```bash
kubectl get hpa -n challenge -w
```

In another terminal:

```bash
for i in 1 2 3; do
  kubectl run load-$i -n challenge --image=busybox --restart=Never -- \
    sh -c "while true; do wget -q -O- http://my-app-devops-challenge/ > /dev/null; done"
done
```

In my test, CPU reached 245% of the request and the replicas scaled 3 → 6 → 10. Stop the load with `kubectl delete pod load-1 load-2 load-3 -n challenge`. Scale-down takes about 5 minutes.

## Helm lifecycle

```bash
helm upgrade my-app ./devops-challenge -n challenge
helm history my-app -n challenge
helm rollback my-app <revision> -n challenge
helm uninstall my-app -n challenge
```

## Notes and limitations

- The pinned `tornado==5.1.1` uses `collections.MutableMapping`, removed in Python 3.10. With `python:3.11-slim` the pods crashed on import, so the image uses Python 3.9.
- Redis is a single replica with no persistence, which is fine for a demo but not production.
- The service is a NodePort. A real setup would use an Ingress.

## Original challenge



# TradeByte DevOps Challenge

This repository is meant to be used as a challenge for DevOps candidates at Tradebyte.

You should fork/clone this repository to use as a basis for the challenge.

## The challenge

Subject of this challenge is to setup a robust, production ready and scalable Kubernetes deployment for the given demo application.

The demo application can be found in this repository.

The requirements are as follows:

- Use a container technology of your own choosing for the demo application.
- Setup a local Kubernetes cluster (e.g. [minikube](https://minikube.sigs.k8s.io/docs/)) for the containerized demo application as you would do it in production.
  - It should have at least 3 replicas.
  - It should automatically scale with CPU utilization.
  - It should handle environment variables in a secure way.
- Think about scalability and performance.

## Demo application

### Requirements

#### System

- GNU/Linux
- `python` >= 3.7
- `pip` >= 9.0
- `redis` >= 5.0

`>=` means any version of the package, above or equal to the specified version.

#### Application

- `redis-py`
- `tornado`

You can find them in the `requirements.txt` file and their required version number.
You can install them by using:

```bash
pip install -r requirements.txt
```

### :rocket: Starting the Application

The application uses several environment variables.
You can find them all and their default values in the `.env` file. They need to be avaiable at runtime. Here is an overview about the environment variables:

- `ENVIRONMENT` the environment in which the application is run. Likely `PROD` for production or `DEV` for development context.
- `HOST` the hostname on which the application is running. Locally it is `localhost`.
- `PORT` is the port on which the application is running.
- `REDIS_HOST` is the hostname on which redis is running. Locally it is `localhost`.
- `REDIS_PORT` is the port on which to communicate with redis. Normally it is `6379`.
- `REDIS_DB` which redis db should be used. Normally it is `0`.

Application can be found in `hello.py` file. You can start the application by using:

```bash
export $(cat .env | xargs) && python hello.py
```

Although you don't have to export the environment variables that way. :wink:

### Static files

- Static files are located in `static/` folder.
- Templates are located in `template/` folder.

### Executing Tests

Tests can be found in `tests/test.py` file.
You can run the tests by using:

```bash
python tests/test.py
```

## Contributing

We love contributions from everyone. By participating in this project, you agree to abide by our [code of conduct](https://tradebyte.github.io/Code-of-Conduct/).

We expect everyone to follow the code of conduct anywhere in `DevOps-Challenge`'s project codebases, issue trackers, chatrooms, and mailing lists.<br/>
Thank you, [contributors]!

[contributors]: https://github.com/tradebyte/DevOps-Challenge/graphs/contributors

## License

Copyright (c) 2024 by the Tradebyte Software GmbH.<br/>
`DevOps-Challenge` is free software, and may be redistributed under the terms specified in the [LICENSE] file.

[license]: /LICENSE

## About

`DevOps-Challenge` is maintained and funded by the Tradebyte Software GmbH. <br/>
The names and images for `DevOps-Challenge` are trademarks of the Tradebyte Software GmbH.

We love free software!
