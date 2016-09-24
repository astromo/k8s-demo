Kubernetes Demo
---

This projects requires a configured and running Kubernetes cluster.

# Services

- [Ping Service](https://github.com/astromo/astromo-ping)
- [Auth Service](https://github.com/astromo/astromo-auth)

# Setting up the deployments and services

---

**NOTE:** Kubernetes pre 1.2 requires "Replication Controllers" instead of "Deployments". Use the `-controller.json` files instead of the `-deployment.json` files if you have an older Kubernetes cluster running.

---

- Setup the Ping deployment:
`kubectl create -f ./ping/astromo-ping-deployment.json`
- Setup the Ping Service:
`kubectl create -f ./ping/astromo-ping-service.json`

- Setup the Auth deployment:
`kubectl create -f ./auth/astromo-auth-deployment.json`
- Setup the Auth Service:
`kubectl create -f ./auth/astromo-auth-service.json`

# Ping Service
The ping service is a simple example of a stateless application that simply responds with "pong" if you're authenticated and authorised.

This service runs on port `30002`.

It has 3 different routes:

## - `GET /ping`
This route will simply reply with `{ "message": "pong!" }` even if you're not authenticated.

## - `GET /pong`
This route requires authentication as well as authorisation (from the auth service) and will include the user's identity information in the response.

```
{
  "message": "pong!",
  "identity": {
    "email": "michiel@demey.io",
    "permissions": [
      "ping"
    ],
    "iat": 1470499312,
    "exp": 1470585712,
    "aud": "astromo",
    "iss": "auth.astromo.io",
    "sub": "michiel@demey.io"
  }
}
```

## - `GET /foo`

This route requires the `foo` permission to be available to the user. Currently only the `ping` permission is granted in this demo and thus the response will always be a `403 Forbidden`.

# Auth Service
The auth service provides both authentication and authorisation.

This service runs on port `30001`.

Only one route is available:

## - `POST /auth`
This service has the following hard-coded credentials:

```
{
  "email": "michiel@demey.io",
  "password": "test123"
}
```
Use these to authenticate with the Ping Service. The response will be a JWT token, to authenticate with the Ping Service, simply send the token in the `Authorization` header.

e.g `Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzUxMiJ9...`

# Debug information
The services are setup to include the pod that responded to the request so you can debug the load balancing.

Check the `server` response header to see which pod responded to the request. To see which node (server) the pod runs on, check the output of `kubectl get pods -o wide`.
