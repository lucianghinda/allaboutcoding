## How to open rails console in safe mode on GCP k8s

Here is how to open rails console in safe mode on GCP k8s.

First you need to list the pods: 

```bash
kubectl get pods | grep <name>
```
replace <name> with the name of the pod that you want to connect with rails console.
In the list that you see copy the name of one of the pods. 

Then you can execute

```bash
kubectl exec <pod_name> -i -t -- rails c --sandbox
```

This will open your rails console in sandbox mode. 

**Futher read: **

1. Read  [here](https://guides.rubyonrails.org/command_line.html#bin-rails-console)  more about rails console with sandbox.

2. Directly execute locally `rails console --help` to see all available options.

---
*This works so far up until and including Rails 6. Did not tested this yet on Rails 7.*



