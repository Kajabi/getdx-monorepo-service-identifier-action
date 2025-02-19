# 📖 GitHub Action - Identify Mono(repo/lith) Services to send to GetDX

This GitHub Action is used to identify monorepository services as well as services as part of a monolith and send `setPullServices` requests to the [GetDX](https://getdx.com) platform. 

> Based on this [help article](https://help.getdx.com/en/articles/7669458-deployments#h_7d23ab886c).

## Versions

* [`main`](https://github.com/Kajabi/getdx-monorepo-service-identifier-action)

## Features

This action will utilize various methods for identifying services as part of a monorepository or monolith to send to GetDX for consumption. Three methods are available to identify the services:

* `backstage` (default)
* `service-file`
* `codeowners`

### `backstage` service association method

This association method leverages `catalog-info.yaml` documents (or the name given with the `catalog-filename` parameter) utilized in the [Backstage](https://backstage.io) framework to identify the service name based on the folder-tree for the file(s) being augmented in a pull request. It will pull a list of the files that have been altered, traverse up the directory tree to find the closeset `catalog-info.yaml` document, and then utilize the `metadata.name` attribute of all `kind: Component` entities.

It then will send a `setPullServices` request with the affected services to [GetDX](https://getdx.com).

#### Usage

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
          service-association-type: 'backstage' # Optional. Is the default
```

### `service-file` service association method

This association method leverages a custom `.service-associations` document located in the root of the repository. It utilizes the mapping in that file to associate specific files/folders/or naming globs to identify the service name of files being augmented in a pull request. 

It then will send a `setPullServices` request with the affected services to [GetDX](https://getdx.com).

#### `.service-assocations` file rules

The rules map almost identically to the rules associated with Github CODEOWNER files (which in-turn follows `.gitignore` rules), the ruleset can be found [here](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-code-owners#example-of-a-codeowners-file).

Additionally, if you want a "default" service to be associated with files that do not match other rules in the list you can provide a `/ <service>` mapping at the top of the file for that association.

Differing from the CODEOWNERS file, which uses Github User/Group & Emails for ownership, you can provide targetted strings to identify the service impacted (e.g. `/app/foo/bar.txt   bar-service).

##### Example `.service-associations` file

```shell
/                           app-service
apps/emails/tsconfig.json    email-service
apps/bot/                   bot-service
apps/bot/src/foo.txt        foo-service
apps/web*                   web-service
```

#### Usage

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
          service-association-type: 'service-file'
```

### `.github/CODEOWNERS` service association method

This association method leverages the `.github/CODEOWNERS` document. It utilizes the mapping in that file to associate specific files/folders/or naming globs to identify the service name of files being augmented in a pull request. 

It then will send a `setPullServices` request with the affected services to [GetDX](https://getdx.com).

#### `.github/CODEOWNERS` file rules

The rules map to the Github CODEOWNERS rules (which in-turn follows `.gitignore` rules), the ruleset can be found [here](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-code-owners#example-of-a-codeowners-file).

Because CODEOWNERS files utilize Github Groups/Users and emails to identify ownership there's not really a direct way to associate "custom" service names to these file sets. In order to circumvent this the script will generate dash-separated/lowercase names based on the entries. For instance a `@MyOrg/myTeam` handle would be converted to `group-myorg-myteam`. Likewise an email of `foo@bar.com` would be converted to `email-foo-at-bar-dot-com`, and finally a user `@foobar` would be converted to `user-foobar`

##### Example `.github/CODEOWNERS` file

```shell
/                                                                @MyOrg/catch-all # group-myorg-catach-all
/db/                                                             @MyOrg/db-migration-reviewers # group-myorg-db-migration-reviewers
/app/constraints/checkout*                                       @awesome-dev # user-awesome-dev
/app/models/commerce/*.rb                                        @MyOrg/commerce # group-myorg-commerce
/app/models/email_delivery.rb                                    foo@bar.com # email-foo-at-bar-dot-com
/.env-sample                                                     @MyOrg/platform # group-myorg-platform
```

#### Usage

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
          service-association-type: 'codeowners'
```

## Configuration

| Name                       | Description                                                                            | Required | Default             |
| -------------------------- | ---------------------------------------------------------------------------------------| -------- | ------------------- |
| `getdx-instance-name`      | Instance name for getdx (e.g. {instance-name}.getdx.net)                               | `true`   |                     |
| `getx-token`               | Token for GetDX API Calls (use Github Secrets for security)                            | `true`   |                     |
| `service-association-type` | The association strategy for your services (`backstage`, `service-file`, `codeowners` ) | `false`  | `backstage`         |
| `catalog-filename`          | The name of the Backstage catalog file to search for in the tree                        | `false`  | `catalog-info.yaml` |
| `fetch-depth`              | The depth of history to fetch in the repo                                              | `false`  | `0`                 |
| `debug`                    | Enable debug mode and don't send API requests                                          | `false`  | `false`             |