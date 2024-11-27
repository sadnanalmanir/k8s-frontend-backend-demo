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

```sh
$ docker compose -f docker-compose.dev.yaml up -d --build
```
