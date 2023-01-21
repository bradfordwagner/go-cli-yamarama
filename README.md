# Yamarama
- An argocd configuration flattener. Takes configuration for many clusters, resolves their variable hierarchy by depth first traversal. Then outputs the resolution to leaf level cluster configuration. If we use many git cluster generators instead of this strategy, git io can overwhelm an appset controller and deployments will stall.

## Commands
- `bootstrap` - generates structure for yaml hierarchy based on a `structure.yaml`
- `resolve` - outputs configuration files

## Structure
```yaml
# ~/.yamarama/structure.yaml
# jurisdictions are usually abbrevs for cloud providers
# jurisdictions have different regions
stages:
  - preprod
  - prod
jurisdictions:
  - name: azure
    regions: [west,east]
  - name: gcp
    regions: [north,south]
output:
  dir: ./config
# tells yamarama how to parse a cluster name to handle hierarchy resolution
name:
  # takes the values at a given split and looks upwards for reolution
  # _ signifies that we do not care about this split
  # can parse out [region, jurisdiction]
  format: [_, _, jurisdiction, region]
  # how to split the cluster name
  separator: _

# command bootstrap - creates values files for editing
# ./yamarama/default.yaml
# ./yamarama/preprod.yaml
# ./yamarama/preprod.azure.yaml
# ./yamarama/preprod.azure.west.yaml
# ./yamarama/preprod.azure.east.yaml
# ./yamarama/preprod.gcp.yaml
# ./yamarama/preprod.gcp.north.yaml
# ./yamarama/preprod.gcp.south.yaml
# ./yamarama/preprod.clusters.yaml <-- where cluster level overrides go
#
# ./yamarama/prod.yaml
# ...

```
#### Hierarchy Files
- Examples
```yaml

# preprod.yaml
stage: preprod

# preprod.azure.yaml
jurisdiction: azure
registry: registry.example.com

```

#### Clusters
```yaml
# preprod.clusters.yaml
clusters:
  # matches the name configuration structure, outputs a file ./config/${stage}/${name}.yaml
- name: dev_common_azure_west
  values:
    leaf: dev_azure_west
    registry: registry-west.example.com
- name: dev_common_azure_east # use defaults
```

#### Output
```yaml
# ./config/${stage}/dev_azure_west.yaml
# name is always first
name: dev_common_azure_west
# the rest is alphabetized
jurisdiction: azure
leaf: dev_azure_west
registry: registry-west.example.com
stage: preprod

# ./config/${stage}/dev_azure_east.yaml
# name is always first
name: dev_common_azure_west
# the rest is alphabetized
jurisdiction: azure
registry: registry.example.com
stage: preprod
```

