# Testing action-awx-template Github Action

Currently using the Github action workflow files to execute the merging branch for integration testing. Trying to test with a local AWX instance is still under works.

## Integration Instance
In order to do the integration testing there needs to be a publicly accessible version of AWX that Github actions can use. If doing this with private runners, then the runner needs access to AWX in your environment.

Included is some prebuilt k8s kustomize templates for various cloud providers in order to build an AWX instance that is publicly exposed.

Be sure to follow the [AWX Operator documentation](https://ansible.readthedocs.io/projects/awx-operator/en/latest/index.html) if there are any questions to deploying an AWX instance.

### General Actions
These are general actions that are needed for any AWX Instance setup.

Clone and switch to the branch with the version of operator to install
```bash
git clone git@github.com:ansible/awx-operator.git
cd awx-operator
git tag
git checkout tags/<tag>
```

Copy in `kubernetes/**/kustomization.yaml` and `kubernetes/**/awx-demo.yaml` files into the `awx-operator` directory.

Replace `<TAG>` in `kustomization.yaml` file with the version of the operator to install.

Apply the current config:
```bash
kubectl apply -k .
```

Update the conxt to use awx namespace as a default.
```bash
kubectl config set-context --current --namespace=awx
```

You can use `kubectl get pods` in order to see the awx-operator pod. Once its deployed you can move on.

Update `kustomization.yaml` and remove the comment under resources for the awx-demo.yaml file.
```yaml
...
resources:
  - github.com/ansible/awx-operator/config/default?ref=<TAG>
  - awx-demo.yaml
...
```

Apply the new updates.
```bash
kubectl apply -k .
```

To get pods deployed by the awx-operator:
```bash
kubectl get pods -l "app.kubernetes.io/managed-by=awx-operator"
```

To get the LoadBalancer service
```bash
kubectl get svc -l "app.kubernetes.io/managed-by=awx-operator"
```

To get the logs
```bash
kubectl logs -f deployments/awx-operator-controller-manager -c awx-manager
```

To get the admin password:
```bash
kubectl get secret awx-demo-admin-password -o jsonpath="{.data.password}" | base64 --decode ; echo
```

### Digital Ocean:
Currently my preferred method since its cheap and easy to use.

There are some consdiertions when trying to setup the load balancer. The files in `kubernetes/digitalocean/` should be sufficient enough to setup a basic one without SSL.

Review the [Digital Ocean LoadBalancer setup](https://docs.digitalocean.com/products/kubernetes/how-to/configure-load-balancers/) documents for more details.

## Prebuilt Templates
Included is some basic exports of some builtin templates in order to do some simple integration testing. You can import the templates using [awxcli](https://docs.ansible.com/ansible-tower/latest/html/towercli/usage.html#installation). Otherwise, you can create them manually or use your own pre-made templates.

Import Job Templates:
```bash
awx --conf.host ${AWX_HOST} \  
    --conf.username ${AWX_USERNAME} \  
    --conf.password ${AWX_PASSWORD} \  
    import < exports/job_templates.json
```

Import Workflow Job Templates:
```bash
awx --conf.host ${AWX_HOST} \  
    --conf.username ${AWX_USERNAME} \  
    --conf.password ${AWX_PASSWORD} \  
    import < exports/workflow_job_templates.json
```
