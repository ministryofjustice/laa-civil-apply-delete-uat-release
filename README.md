# Delete UAT release

Github action to encapsulate the reusable steps required
for deleting an apply service PRs UAT release.

## Prerequsites and assumptions:

- github secrets containing kubernetes credentials for the cluster
- assumes a certain naming mechanism for UAT release (see deploy script)

## Workflow example: delete a UAT release when PR on merge

```yml
name: Delete UAT release on PR merge

on:
  pull_request:
    types:
      - closed

  my_job:
    if: github.event.pull_request.merged == true
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Delete UAT release action
        uses: uses: ministryofjustice/laa-civil-apply-delete-uat-release@v1.0.0
        with:
          k8s_cluster: ${{ secrets.K8S_CLUSTER }}
          k8s_cluster_cert: ${{ secrets.K8S_CLUSTER_CERT }}
          k8s_namespace: ${{ secrets.K8S_NAMESPACE }}
          k8s_token: ${{ secrets.K8S_TOKEN }}
```

## Secrets
You will need to provide Github action secrets, `settings > secrets > actions` for accessing the kubernetes cluster. You can name them
whatever you like and then supply them to the `action` using the `with` option:

```
  with:
    k8s_cluster: ${{ secrets.MY_CLUSTER }}
    k8s_cluster_cert: ${{ secrets.MY_CLUSTER_CERT }}
    k8s_namespace: ${{ secrets.MY_NAMESPACE }}
    k8s_token: ${{ secrets.MY_TOKEN }}
```

Note that none of them are expected to be base64 encoded

