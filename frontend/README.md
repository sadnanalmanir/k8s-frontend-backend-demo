# React + Vite frontend

## Scaffold a React + JS frontend project

Create Vite project called `frontend`

```sh
$ npm create vite@latest

> npx
> create-vite

? Project name: › frontend
? Select a framework: › - Use arrow-keys. Return to submit.
? Select a framework: › - Use arrow-keys. Return to submit.
    Vanilla
    Vue
❯   React
    Preact
    Lit
    Svelte
    Solid
    Qwik
    Angular
    Others
✔ Select a framework: › React
? Select a variant: › - Use arrow-keys. Return to submit.
    TypeScript
    TypeScript + SWC
❯   JavaScript
    JavaScript + SWC
    Remix ↗
```

## Runing the app

### Deveopment mode

```sh
$ cd frontend
$ npm install
$ npm run dev

VITE v6.0.0  ready in 381 ms

  ➜  Local:   http://localhost:5173/
  ➜  Network: use --host to expose
  ➜  press h + enter to show help
```

### Dockerfile.dev

Create the `Dockerfile.dev` file

```yaml
# specify the node base image
FROM node:20

# Set working directory
WORKDIR /app

# copy only the libraries requiring installation
COPY package.json /app/package.json
COPY package-lock.json /app/package-lock.json

# ci allows installing only from lock file
RUN npm ci && npm cache clean --force

RUN npm i -g serve

# copy the source directory
COPY ./ /app/

RUN npm run build

EXPOSE 3000

CMD [ "serve", "-s", "dist" ]
```

Create a docker image

```sh
$ docker build -f Dockerfile.dev -t react-frontend:dev .
```

Run the image in a container

```sh
$ docker run -it --rm -p 3000:3000 react-frontend:dev
```

Open [http://localhost:3000](http://localhost:3000) to view the landing page.

### docker compose

Start the container

```sh
$ docker compose -f docker-compose.dev.yaml up -d --build
```

Stop the container

```sh
$ docker compose down
```

## Minikube deployment

Open a terminal. Start minikube and the dashboard

```sh
$ minikube start
$ minikube dashboard
```

Open another tab in the terminal. To point your terminal to use the docker daemon inside minikube run this:

```sh
$ eval $(minikube docker-env)
```

Now any ‘docker’ command you run in this current terminal will run against the docker inside minikube cluster.

```sh
$ docker images
```

Build the frontend image

```sh
$ cd frontend
$ docker build -f Dockerfile.dev -t react-frontend:dev .
```

Confirm if the new image `react-frontend:dev` is displayed

```sh
$ docker images

REPOSITORY                     TAG        IMAGE ID       CREATED          SIZE
react-frontend                 dev       6e860ad9e9a8   14 seconds ago   1.18GB
...
```

Describe a kubernetes deployment resource `react-deployment.yaml` for the frontend

```sh
$ touch react-deployment.yaml`
```

Paste the description of the deployment

```yaml
kind: Deployment
apiVersion: apps/v1
metadata:
  name: frontend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: react-app
      tier: frontend
  template:
    metadata:
      labels:
        app: react-app
        tier: frontend
    spec:
      containers:
        - name: react-app-image
          image: react-frontend:dev
          imagePullPolicy: Never
          ports:
            - containerPort: 3000
          resources:
            limits:
              memory: "128Mi"
              cpu: "500m"
            requests:
              memory: "64Mi"
              cpu: "250m"
      restartPolicy: Always
```

Create the deployment resource in Minikube

```sh
$ kubectl apply -f react-deployment.yaml
```

The cotainerPort is 3000 beause it is the react web application is running on port 3000.

Describe a kubernetes service resource `react-service.yaml` for the frontend

```sh
$ touch react-service.yaml
```

Paste the description of the service

```yaml
kind: Service
apiVersion: v1
metadata:
  name: frontend
spec:
  type: LoadBalancer # Minikube tunnel command is required
  selector:
    app: react-app
    tier: frontend
  ports:
    - port: 3001 # Enduser access this port
      targetPort: 3000 # containerPort running in the Pod
```

Create the service resource in Minikube that is used to access the containers running in the pods

```sh
$ kubectl apply -f react-service.yaml
```

The service is of type `LoadBalancer`. The targetPort is 3000 which is the target of the service i.e. containerPort in the deployment. Now the port value is where the user is able to access the service provided by the container. Lets set it at 3001 to differentiate it between the targetPort 3000 where the react web app container is running.

```sh
$ kubectl get svc

NAME         TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
frontend     LoadBalancer   10.99.139.133   <pending>     3001:30957/TCP   45s
...
```

In minikube exposing LoadBalancer services require using the tunnel. Without tunneling, the `EXTERNAL-IP` shows `<pending>`. Use the blocking command to tunnel

```sh
$ minikube tunnel
```

As this process will be a blocking command, open another tab in the terminal and run the command to check the `EXTERNAL-IP`

```sh
$ kubectl get svc

NAME         TYPE           CLUSTER-IP    EXTERNAL-IP   PORT(S)          AGE
frontend     LoadBalancer   10.99.33.63   127.0.0.1     3001:31981/TCP   54s
kubernetes   ClusterIP      10.96.0.1     <none>        443/TCP          5m11s
```

Open the newly available `EXTERNAL-IP`:`PORT` which [http://127.0.0.1:3001](http://127.0.0.1:3001) became available to test the webapp.
