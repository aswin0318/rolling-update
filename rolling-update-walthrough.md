# 🚀 Kubernetes Rolling Update – Step-by-Step Walkthrough

---

## Step 1: Create Deployment

>>> COMMAND:
kubectl apply -f rolling-demo.yaml

- Deployment object is created
- Desired state is stored in etcd
- Deployment controller detects new object
- ReplicaSet is created
- ReplicaSet creates Pods
- Scheduler assigns Pods to nodes
- Kubelet starts containers

---

## Step 2: Verify Resources

>>> COMMAND:
kubectl get deploy
kubectl get rs
kubectl get pods

- Deployment exists with desired replicas
- ReplicaSet maintains replica count
- Pods are running containers

---

## Step 3: Watch Cluster Changes

>>> COMMAND:
kubectl get pods -w
kubectl get rs -w

- Continuous watch on Pod lifecycle changes
- Continuous watch on ReplicaSet scaling

---

## Step 4: Trigger Rolling Update

>>> IMPERATIVE COMMAND (UPDATE IMAGE):
kubectl set image deployment/rolling-demo app=nginx:1.26

- Deployment spec is updated in etcd
- Deployment generation increments
- Deployment controller detects change
- New ReplicaSet is created with new image
- Old ReplicaSet is retained

---

## Step 5: Rolling Update Execution

- New ReplicaSet starts with 0 Pods
- Scale up new ReplicaSet (maxSurge limit)
- Create new Pod
- Kubelet pulls image and starts container

---

## Step 6: Readiness Check

- Kubelet executes readiness probe
- If probe succeeds → Pod marked READY
- Endpoints controller adds Pod to Service
- Pod starts receiving traffic

---

## Step 7: Scale Down Old ReplicaSet

- Old ReplicaSet scaled down (maxUnavailable limit)
- One old Pod selected for deletion

---

## Step 8: Pod Termination Flow

- Pod removed from Service (no new traffic)
- Kubelet sends SIGTERM to container
- terminationGracePeriodSeconds countdown starts
- Application completes ongoing requests
- If not stopped → SIGKILL sent
- Pod removed from etcd

---

## Step 9: Loop Continues

- Repeat:
  - Scale up new ReplicaSet
  - Wait for readiness
  - Scale down old ReplicaSet

- Continue until:
  - Old ReplicaSet = 0 Pods
  - New ReplicaSet = desired replicas

---

## Step 10: Failure Scenario

>>> IMPERATIVE COMMAND (BREAK DEPLOYMENT):
kubectl set image deployment/rolling-demo app=nginx:wrong

>>> COMMAND:
kubectl get pods

- New Pods fail to start (ImagePullBackOff)
- Readiness probe never succeeds
- Pods not marked READY
- Endpoints controller does not route traffic
- Deployment controller stops rollout
- Old Pods continue running

---

## Step 11: Rollback

>>> IMPERATIVE COMMAND (ROLLBACK):
kubectl rollout undo deployment rolling-demo

- Deployment spec restored in etcd
- Previous ReplicaSet scaled up
- Failed ReplicaSet scaled down
- System returns to stable state

---

## Step 12: Internal Flow Summary

kubectl command
→ API Server
→ etcd updated
→ Deployment controller reacts
→ ReplicaSet created/updated
→ Pods created/deleted
→ Scheduler assigns nodes
→ Kubelet runs containers
→ Probes determine readiness
→ Traffic routed only to healthy Pods

---

## Final State

- New ReplicaSet fully active
- Old ReplicaSet scaled to zero
- Application running without downtime
- System consistent with desired state in etcd
