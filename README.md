# devops-docs
## k8s update strategies
- canary : some new version pods with keeping old versions
- recreate : depricate all old pods while deploying new pods (used when system is not compatible with two versions at once)
- rolling : partially update pods and after health check depricate preivious ones.
## deployments vs statefulsets
in statefulset pods have states and can not easily be replacible. they have unique netvork ID and Storage ID
example:
