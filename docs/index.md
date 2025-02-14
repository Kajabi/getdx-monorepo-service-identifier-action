# 📖 GitHub Action for Identifying Monorepo Services to send to GetDX

This GitHub Action is used to identify monorepository services and send `setPullServices` requests to the GetDX platform. 

> Based on this help article: https://help.getdx.com/en/articles/7669458-deployments#h_7d23ab886c).

## Versions

- [`main`](https://github.com/Kajabi/getdx-monorepo-service-identifier-action)

## Features

This Action leverages `catalog-info.yaml` documents utilized in the [Backstage](https://backstage.io) framework to identify the service name based on the folder-tree for the file(s) being augmented in a pull request. It will pull a list of the files that have been altered upon PR merge, traverse up the directory tree to find the closeset `catalog-info.yaml` document, and then utilize the `metadata.name` attribute of the top-level `kind: Component`.

It then will send a `setPullServices` request with the affected services to GetDX.

## Usage

```
name: "GetDX Service Identification"

on:
  pull_request:
    types:
      - closed
    branches:
      - 'main'

jobs:
  identify-services:
    runs-on: ubuntu-latest
    steps:
      - name: Identify Services
        uses: Kajabi/getdx-monorepo-service-identifier-action@main
        with:
          getdx-instance-name: 'YOUR_INSTANCE_NAME'
          getdx-token: 'YOUR_TOKEN' # Utilize Github Secrets for security
```


## Configuration

| Name                    | Description                                                                        | Required | Default                                  |
| ----------------------- | ---------------------------------------------------------------------------------- | -------- | ---------------------------------------- |
| `getdx-instance-name`   | Instance name for getdx (e.g. <instance-name>.getdx.net)                           | `true`   |                                          |
| `getx-token`            | Token for GetDX API Calls (use Github Secrets for security)                        | `true`   |                                          |
| `catalog-filename`       | The name of the Backstage catalog file to search for in the tree                    | `false`  | `catalog-info.yaml`                      |
| `fetch-depth`           | The depth of history to fetch in the repo                                          | `false`  | `0`                                      |
| `debug`                 | Enable debug mode and don't send API requests                                      | `false`  | `false`                                  |