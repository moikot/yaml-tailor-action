# yaml-tailor-action

GitHub action for customizing YAML to your needs.

### Example

The pipeline modifies  `Chart.yaml` and `values.yaml` files providing a version and an image name. In this way, the image name and the version can depend on the branch or tag which triggered the build.

Assumptions:
* The Helm chart directory in the current repository is `chart`.

```yaml
name: ci

on:
  push:

jobs:
  buildx:
    runs-on: ubuntu-latest
    steps:

     - name: Prepare
        id: prep
        run: |
          ... // Logic for extracting IMAGE and VERSION
          echo ::set-output name=image::${IMAGE}
          echo ::set-output name=version::${VERSION}

      - name: Change the Helm chart
        uses: moikot/yaml-tailor-action@v1
        with:
          yaml_file: chart/Chart.yaml
          values: |- 
            version=${{ steps.prep.version }}
            appVersion=${{ steps.prep.version }}

      - name: Change the Helm chart values
        uses: moikot/yaml-tailor-action@v1
        with:
          yaml_file: chart/values.yaml
          values: |- 
            image.name=${{ steps.prep.image }}
            image.tag=${{ steps.prep.version }}

      // Steps for publishing the modified Helm chart      

```

## Customization

### Inputs

Following inputs can be used as `step.with` keys:

| Name               | Type    | Description                       |
|--------------------|---------|-----------------------------------|
| `tailor_image`           | String  | The Docker image containing [`yaml-tailor`](https://github.com/moikot/yaml-tailor) (default: 'moikot/yaml-tailor:1.0') |
| `yaml_file`      | String     | The relative path under $GITHUB_WORKSPACE to a YAML file |
| `values`      | String     | The YAML Tailor values, e.g. foo=bar |
| `strings`  | String  | The YAML Tailor strings, e.g. baz=1 |

**Note:** For more information about `values` and `strings` see [yaml-tailor](https://github.com/moikot/yaml-tailor) and [DJSON library documentation](https://github.com/moikot/djson).

## Limitation

This action is available for Linux [virtual environments](https://docs.github.com/en/actions/reference/virtual-environments-for-github-hosted-runners#supported-virtual-environments-and-hardware-resources) only.