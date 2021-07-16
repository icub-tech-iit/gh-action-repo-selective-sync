Repo Selective Sync
===================

Automatically synchronize target from source repos by specifying what to copy.

Compared with other actions devoted to synching repos, this action has been designed
specifically to allow for **publishing a subset of files** stored within a
**large private repository** (tens of GB) into a **public repository**.

To this end, the action is purposely deviced to take advantage of self-hosted runners,
although it can be certainly used in a standard fashion with GitHub-hosted runners.
With a self-hosted runner, in fact, one can save precious GitHub actions minutes quota
that instead would be employed in a substantial amount to transfer tons of large files from
the source to the target repositories.

Moreover, the source and target repositories cloned and updated during the job are
retained in a self-hosted runner so that pulls made in subsequent action calls are
much faster.

## Installation

### Self-hosted runner
To enable the action simply create the `.github/workflows/publish.yml`
file with the following content:

```yml
name: Publish Repo

on:
  push:
    branches:
    - 'master'
  workflow_dispatch:

jobs:
  publish:
    name: "Publish"
    runs-on: [self-hosted, Linux, X64]
    steps:
    - uses: icub-tech-iit/gh-action-repo-selective-sync@v1.0.0
      with:
        recipe-file: '.publish/recipe.yml'
        token-source: ${{ secrets.GITHUB_TOKEN }}
        token-target: ${{ secrets.TARGET_PAT }}
        sudo-passwd: ${{ secrets.SELF_HOSTED_RUNNER_PASSWD }}
```

This action will be triggered upon any push on the `master` branch
or by a manual request and will copy out files from the current (source)
repository according to the rules detailed in the file specified via the 
parameter `recipe-file`, among which there is the target repository. 

### GitHub-hosted runner
For a GitHub-hosted runner the parameter `sudo-passwd` is simply
not required, whereas the key `runs-on` needs to be something like
`ubuntu-latest`, clearly.

## Parameters

### `recipe-file`
Points to the file located within the source repository that
contains the rules to copy out data from the source to the target repo.

Here's an example:
```yml
source:
  copy_from:
    - ".publish/README.md"
    - ".gitattributes"
    - ".gitignore"
    - "common"
    - "project_A/pub"
    - "project_B/pub"
    - "project_C/pub/README.md"
target:
  repo: "target-owner/target-repo"
  maxsize_commit_MB: 1000
  commit_message: "just publishing"
  copy_to:
    - "README.md"
    - ".gitattributes"
    - ".gitignore"
    - "common"
    - "project_A"
    - "project_B"
    - "project_C/README.md"
  remove_after_copy:
    - "project_B/secret_gadgets"
```

The content of the recipe is quite self-explainig, although
a few more words are certainly needed.

The action will copy out data (files and directories) specified 
in the paths under `source.copy_from` into the paths specified under
`target.copy_to` of the target repo determined by the field `target.repo`.

Optionally, one can also specify a target branch different from the source
by means of the field `target.branch`.

After this copy, a clean up can be requested by listing down the
paths to be removed under `target.remove_after_copy` (can be left empty).

When the copy and (possibly) the subsequent clean up are done,
a push will be made to the target repo, actually executing the 
data transfer. With this regards, the push may contain several
commits in a row in case the data amount overcomes the limit
for a single commit specified via `target.maxsize_commit_MB`
given in MB. The latter often happens at the first publication 
of large repositories containing LFS files.

### `token-source`
The Personal Access Token (PAT) that allows accessing the
source repository. It may be the standard `GITHUB_TOKEN`.

### `token-target`
The Personal Access Token (PAT) that allows accessing the
target repository.

### `sudo-passwd`
The `sudo` password for the user of the self-hosted runner that
is responsible of executing the workflow. This parameter is not
required for workflows running on GitHub.

The `sudo` access is needed for installing the dependencies.
With this respect, a Docker container action would have avoided
this need. On the other hand, installing and maintaing Docker
on a self-hosted runner might be not always ideal as Docker
would need to be configured with reasonably big volumes to deal
with the data involved in this process.

## Examples
Here's below a list of public repositories that are managed
leveraging on workflows built on this action:
- [`icub-tech-iit/cad-mechanics-public`](https://github.com/icub-tech-iit/cad-mechanics-public)
- [`icub-tech-iit/electronics-boards-public`](https://github.com/icub-tech-iit/electronics-boards-public)
- [`icub-tech-iit/electronics-wiring-public`](https://github.com/icub-tech-iit/electronics-wiring-public)

## Maintainers
This action is maintained by:

| | |
|:---:|:---:|
| [<img src="https://github.com/pattacini.png" width="40">](https://github.com/pattacini) | [@pattacini](https://github.com/pattacini) |
