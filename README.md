Jenkins CI/CD Pipeline to build a React APP and deploy to a Kubernetes deployment
Jenkins CI/CD Pipeline to orchestrate building a React application with the Node Package Manager (npm) and deploy it to a Kubernetes deployment.

We will

1. Pull the code from github.
2. Use NPM to build it.
3. Use a Dockerfile to copy the built files to an NGINX image.
4. Push the image to docker
4. Update the kubernetes deployment with the new image and trigger a rolling update
