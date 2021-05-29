# Application Life Cycle Management

## Rollout and Versioning in a Deployment
when you first create deployment, it will create a rollout which trigger deployment ( e.g v1), future when the application is upgraded meaning when the container version is updated to a new one a new rollout is triggered and a new deployment revision is created named revision (v2).

## Deployment Strategies

There are 2 types of deployment strategies

- Recreate
- RollingUpdate (Default Strategy)

### Recreate
One way to upgrade these to a newer version is to destroy all of these and then create newer versions of application instances meaning first destroy the running instances and then deploy the new instances of the new application version.

The problem with this as you can imagine is that during the period after the older versions are down and before any newer version is up the application is down and inaccessible to users this strategy is known as the Recreate strategy.

### RollingUpdate

Instead we take down the older version and bring up a newer version one by one. This way the application never goes down and the upgrade is seamless. In other words rolling update is the default deployment strategy so we talked about upgrades.

How exactly do you update your deployment when you say update.

It could be different things such as updating your application version by updating the version of docker containers used, updating their labels or updating the number of replicas etc. Since we already have a deployment definition file it is easy for us to modify this file once we make the necessary changes.

we run the kubectl apply command to apply the changes. A new rollout is triggered and a new revision after deployment is created.

```
$ kubectl create -f mywebapp.yaml
$ kubectl get deployments
$ kubectl apply -f mywebapp.yaml
$ kubectl describe deployment mywebapp
$ kubectl rollout status deployment/mywebapp
$ kubectl rollout history deployment/mywebapp
$ kubectl rollout undo deployment/mywebapp
```
This is imperative way of editing using kubectl. This will not be edited using the yaml definition.
Suggested would be to use the declarative way to use this.

```
$ kubectl set image deployment/myapp-deployment nginx=nginx:1.9.1
```

## Configuring Applications

- Configuring Command and Arguments on applications
- Configuring Environment Variables
- Configuring Secrets

### Configuring Command and Arguments on applications
Anything that is appended to the *docker run* command will go into the *args* property of the pod defination file in the form of an array. The *command* field corresponds to the *entrypoint* instruction in the Dockerfile

```
containers:
- name: ubuntu-sleeper
  image: ubuntu-sleeper
  command: ["sleep2.0"]
  args: ["10"]
```

### Configuring Environment Variables

To set an environment variable set an *env* property in pod defination file.

There are other ways of setting the environment variables such as,
- ConfigMaps
- Secrets

```
containers:
- name: ubuntu-sleeper
  image: ubuntu-sleeper
  command: ["sleep2.0"]
  args: ["10"]
env:
- name: APP_COLOR
  value: pink
```

## ConfigMaps

There are 2 phases involved in configuring ConfigMaps.
  First, create the configMaps, Second, Inject then into the pod.

There are 2 ways of creating a configmap.
#### Imperative

```
$ kubectl create configmap dbconfig --from-literal=DB_HOST="mysql_db" --from-literal=DB_PASSWORD= "mysql" --from-literal=DB_PORT=3306

$ kubectl create configmap dbconfig --from-file=dbconfig.properties (Another way)
```

#### Declarative

```
apiVersion: v1
kind: ConfigMap
metadata:
    name: dbconfig
data:
  DB_HOST: "mysql_db"
  DB_PASSWORD: "mysql"
  DB_PORT: "3306"
```

reference these config maps to the pods and then check by logging into the pod, you must be able to see the env variables.

```
containers:
  - name: env-configmap
    image: nginx
    envFrom:
      - configMapRef:
          name: dbconfig
```

```
kubectl get pods -o wide
kubectl exec -it pod/pod-name sh
env
```
There are other ways to inject configuration variables into pod

- You can inject it as a single environment variable.
- You can inject it as a file in a Volume.

## Secrets

How Kubernetes handles secrets ?

- A secret is only sent to a node if a pod on that node requires it.
- Kubelet stores the secret into a tmpfs so that the secret is not written to disk storage.
- Once the Pod that depends on the secret is deleted, kubelet will delete its local copy of the secret data as well.

## Multi Container Pod
There are at times you need to have an container which has two or more services required to be available ( e.g webserver & log agent). They need to co-exists and hence they are create/deleted at the same time. Such cases we are required to have multi container pods. They would share same namespaces, network isolations etc .

There are 3 common patterns, when it comes to designing multi-container PODs. The first and what we just saw with the logging service example is known as a *side car pattern*. The others are the adapter and the ambassador pattern.


```
spec:
  containers:
      - name: nginx
        image: nginx:latest
      - name: redis
        image: redis
```
what incase if you require any container to be run only once in the multi pod container ?
i.e to pull a code or binary from a repository that will be used by the main web application. That is a task that will be run only  one time when the pod is first created. Or a process that waits  for an external service or database to be up before the actual application starts

Those can be configured as *initContainers*.

## initContainers
An initContainer is configured in a pod like all other containers, except that it is specified inside a initContainers section,  like this:

```
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox
    command: ['sh', '-c', 'git clone <some-repository-that-will-be-used-by-application> ; done;']
```

When a POD is first created the initContainer is run, and the process in the initContainer must run to a completion before the real container hosting the application starts.

You can configure multiple such initContainers as well, like how we did for multi-pod containers. In that case each init container is run one at a time in sequential order.

```
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox:1.28
    command: ['sh', '-c', 'until nslookup myservice; do echo waiting for myservice; sleep 2; done;']
  - name: init-mydb
    image: busybox:1.28
    command: ['sh', '-c', 'until nslookup mydb; do echo waiting for mydb; sleep 2; done;']
```

# Liveness, Readiness, Startup probes

Many applications running for long periods of time eventually transition to broken states, and cannot recover except by being restarted. Kubernetes provides liveness probes to detect and remedy such situations.

kubelet uses readiness probes to know when a container is ready to start accepting traffic. one use of this signal is to control which pods are used as backend for services. when pod is not ready, its removed from service LB. Sometimes, applications are temporarily unable to serve traffic. For example, an application might need to load large data or configuration files during startup, or depend on external services after startup. In such cases, you don't want to kill the application, but you don't want to send it requests either. Kubernetes provides readiness probes to detect and mitigate these situations

kubelet uses startup probes to know when a container application has started. if any such probe is configured, it disables liveness and readiness checks until it succeeds. Sometimes, you have to deal with legacy applications that might require an additional startup time on their first initialization. In such cases, it can be tricky to set up liveness probe parameters without compromising the fast response to deadlocks that motivated such a probe



References:
https://kubernetes.io/docs/tasks/debug-application-cluster/get-shell-running-container/
