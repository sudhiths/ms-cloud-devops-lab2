# Lab 2 — hello-app on Kubernetes

Your copy of this repository is your submission. The lab sheet is on the course site; this file is where you record what you did.

What's here:

| File | What it is |
|---|---|
| `src/`, `index.html` | a small React page that shows which Pod answered |
| `nginx.conf` | serves the page, and answers `/whoami` with the Pod name |
| `Dockerfile` | builds the page, then serves it with nginx on port 80 |

You don't change any of these. You add `hello-app.yaml` and fill in the sections below.

## Before you push, check

- [ ] `hello-app.yaml` is in the root of this repository
- [ ] `screenshot.png` is in the root of this repository
- [ ] every section below is filled in

---

## 1. My image

Docker Hub image I deployed (for example `ravi123/hello-app:v1`):

```
sudhiths/hello-app:v1
```

## 2. Pods across nodes

Output of `kubectl get pods -o wide` after you applied `hello-app.yaml`:

```

NAME                         READY   STATUS    RESTARTS   AGE   IP           NODE
hello-app-5877877cbf-8j8rd   1/1     Running   0          19m   10.244.2.2   desktop-worker
hello-app-5877877cbf-cvmvv   1/1     Running   0          19m   10.244.2.3   desktop-worker
hello-app-5877877cbf-qnbd9   1/1     Running   0          19m   10.244.1.2   desktop-worker2
hello-app-5877877cbf-v4wxk   1/1     Running   0          11m   10.244.1.3   desktop-worker2

```

## 3. The Service and its endpoints

Output of `kubectl get service hello-app` and `kubectl get endpointslices -l kubernetes.io/service-name=hello-app`:

```

$ kubectl get service hello-app

NAME        TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
hello-app   LoadBalancer   10.96.75.237   172.18.0.7    8080:32371/TCP   20m

$ kubectl get endpointslices -l kubernetes.io/service-name=hello-app

NAME              ADDRESSTYPE   PORTS   ENDPOINTS
hello-app-zd8jw   IPv4          80      10.244.2.2,10.244.1.2,10.244.2.3 + 1 more...

```

## 4. Browser

Add `screenshot.png`: the page at `http://localhost:8080` with at least two Pods in the "Answers so far" table.

## 5. The four traps

For each trap, write what you saw and why it happened, in two or three lines of your own words.

**Trap 1 — The local image**

I built the Docker image locally using `hello-app:v1` and created a Deployment with that image. Unlike the expected `ImagePullBackOff`, all three Pods started successfully because Docker Desktop Kubernetes was able to access the local image. I then tagged and pushed the image to Docker Hub and used `sudhiths/hello-app:v1` for the final Deployment.

**Trap 2 — The selector typo**

I changed the Service selector from `app: hello-app` to `app: helo-app`. The Service still existed, but its EndpointSlice became empty because no Pod had the incorrect label. After restoring `app: hello-app` and applying the manifest, the EndpointSlice listed all four Pods again.

**Trap 3 — The wrong targetPort**

I changed `targetPort` from 80 to 8080. The EndpointSlice still listed all four Pods, but the port changed to 8080. The application stopped responding because nginx listens on port 80. Restoring `targetPort: 80` fixed the issue.

**Trap 4 — The EXTERNAL-IP address**

Initially, the LoadBalancer remained pending because the EDB Postgres Enterprise Manager HTTP service (PEMHTTPD-x64) was occupying host port 8080. After stopping that service, the LoadBalancer was successfully provisioned with external IP 172.18.0.7. The React application then loaded successfully at http://localhost:8080.

## 6. Your YAML vs the generated YAML

Name two fields in the `--dry-run=client -o yaml` output that were not in your file. Why does your file still work without them?

Two fields present in the generated Deployment YAML but not in my manifest are:

1. **`strategy: {}`** — Defines the Deployment update strategy. When omitted, Kubernetes uses the default `RollingUpdate` strategy.
2. **`resources: {}`** — Represents container resource requests and limits. Since no resource requests or limits were specified, the container uses no explicitly configured CPU or memory requests or limits.

My YAML still works because these fields are optional. Kubernetes applies default behavior for the Deployment strategy, while resource requests and limits can remain unspecified.