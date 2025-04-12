# ![Lakeside Mutual Logo](./resources/logo-32x32.png) Kubernetes

## Kind

This is an experimental deployment tested on Kind. The following steps assume that you have created a Kind cluster and have all required CLI tools set up.

### Building

1. **Build the images:**

   ```bash
   docker-compose build
   ```

   _(First run can take about 15-20 minutes.)_

1. **Create a Kind cluster:**

   ```bash
   kind create cluster --name lakeside-mutual
   ```

1. **Load all local images to the Kind cluster:**

   ```zsh
   for image in $(docker images --format '{{.Repository}}:{{.Tag}}' | grep '^lakesidemutual/'); do
     kind load docker-image "$image" --name lakeside-mutual
   done
   ```

   > **Note**: You can get a list of images on a cluster node by running `docker exec -it my-node-name crictl images`, where `my-node-name` is the Docker container name (for example, `lakeside-mutual-control-plane`).

1. **Create deployments and services:**

   ```bash
   kubectl apply -f manifests
   ```

### Accessing the Frontend Locally

To view the frontend in your local environment, you can use port forwarding:

```bash
kubectl port-forward service/customer-management-frontend 3020:3020
```

Then, you can access the frontend at `http://localhost:3020`.
```

### Updating a Deployment

If you update an image, you can restart the deployment with:

```bash
kubectl rollout restart deployment <deployment-name>
```