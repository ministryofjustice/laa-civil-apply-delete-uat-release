# Delete UAT release

Github action to encapsulate the reusable steps required
for deleting an apply service PRs UAT release.

## Prerequsites and assumptions:

- github secrets containing kubernetes credentials for the cluster
- assumes a certain naming mechanism for UAT release (see deploy script)

## Inputs

| Input               | Description                                                                     |
|---------------------|---------------------------------------------------------------------------------|
| release_name_prefix | custom release prefix for the release name to be deleted                        |
| delete_all_pvc      | `true` to delete all persistenvolumeclaims (PVCs) for any installed helm charts |
| k8s_cluster         | k8s cluster name                                                                |
| k8s_cluster_cert    | k8s authentication certificate                                                  |
| k8s_namespace       | k8s namespace in cluster                                                        |
| k8s_token           | k8s authentication token                                                        |

## Workflow example: delete a UAT release when PR on merge

```yml
# minimal example that deletes only on branch merge
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
        uses: ministryofjustice/laa-civil-apply-delete-uat-release@v1.0.0
        with:
          k8s_cluster: ${{ secrets.K8S_CLUSTER }}
          k8s_cluster_cert: ${{ secrets.K8S_CLUSTER_CERT }}
          k8s_namespace: ${{ secrets.K8S_NAMESPACE }}
          k8s_token: ${{ secrets.K8S_TOKEN }}
```

```yml
# Real world example for deletes on branch merge and close, with output
# This also supplies:
#   - a the custom release prefix that will be used to identify the release to delete
#   - flag to delete all persistenvolumeclaims (PVCs) for any installed helm charts
#
on:
  pull_request:
    types:
      - closed

jobs:
  delete_uat_job:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Delete UAT release action
        id: delete_uat
        uses: ministryofjustice/laa-civil-apply-delete-uat-release@v1.0.0
        with:
          release_name_prefix: "apply-"
          delete_all_pvc: true
          k8s_cluster: ${{ secrets.K8S_GHA_UAT_CLUSTER_NAME }}
          k8s_cluster_cert: ${{ secrets.K8S_GHA_UAT_CLUSTER_CERT }}
          k8s_namespace: ${{ secrets.K8S_GHA_UAT_NAMESPACE }}
          k8s_token: ${{ secrets.K8S_GHA_UAT_TOKEN }}
      - name: Result
        shell: bash
        run: echo ${{ steps.delete_uat.outputs.delete-message }}
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

