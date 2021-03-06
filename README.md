<img src="https://raw.githubusercontent.com/elbb/bb-gitversion/master/.assets/logo.png" height="200">

# (e)mbedded (l)inux (b)uilding (b)locks - containerized GitVersion (with some improvements)

This building block integrates GitVersion ("GitVersion looks at your git history and works out the semantic version of the commit being built") and adds the following features to GitVersion:

-   Output of the version numbers as json file.
-   Output of the version numbers, divided into plain text files
-   Output of the version numbers as linux environment file.

By default, only the default configuration of GitVersion is supported, and thus any GitHub flow-compatible workflow.

For more information, see <https://gitversion.net/docs/>

# Build

_There is normally no need to build this building block manually. For the integration
into your project it is sufficient to use the image on hub.docker.com as described
in the Usage section._

The corresponding image can be created manually or e.g. via dobi (<https://github.com/dnephin/dobi>), the way described here.

## Prerequisites

-   dobi (<https://github.com/dnephin/dobi>)
-   docker (<https://docs.docker.com/install/>)

## Build and publish

To build the image, simply call the appropriate dobi resource:

```bash
./dobi.sh build
```

To publish the image in a registry, the image resource is called with the option push.

```bash
./dobi.sh image-gitversion:push
```

The image name and registry can be adjusted via environment variable $GITVERSION_IMAGE. The default repository is <https://hub.docker.com/r/elbb/bb-gitversion>.

```bash
export GITVERSION_IMAGE=REPO_NAME/IMAGE_NAME ./dobi.sh image-gitversion:push
```

# Usage and Integrate into your own project

There are several ways to integrate this building block into your project. The preferred way is via dockerfile.

## Integration via dockerfile

All stable versions of the building block are published in the elbb project on docker.io and can be obtained from there:

`docker pull elbb/bb-gitversion:dev`

To generate a version number for the current git branch docker is called as follows:

```bash
docker run -v $(pwd):/git -v $(pwd)/gen:/gen elbb/bb-gitversion:dev
docker run -v $(pwd):/git -v $(pwd)/gen:/gen -e USERID=$(id -u) elbb/bb-gitversion:dev
```

The repository to be scanned is mounted under `/git`. The generated files are stored under `/gen`.
To use the generated files, it is recommended to pass the current userid per environment variable to the container.

After a successful scan the gen directory looks like this:

```bash
gen/json/gitversion.json
gen/env/gitversion.env
gen/plain/Minor
gen/plain/BuildMetaDataPadded
gen/plain/MajorMinorPatch
gen/plain/AssemblySemVer
gen/plain/LegacySemVerPadded
...
```

The generated files can now be used and evaluated by other applications/ci-systems.

## Integration via dobi

Wrapped in a dobi script the previous example looks like this:

```yaml
mount=mount-git:
  bind: .
  path: /git
  read-only: false # needs to be read/write to work correctly!

mount=mount-gen:
  bind: ./gen/
  path: /gen
  read-only: false

image=image-gitversion:
  image: elbb/bb-gitversion
  tags: ["dev"]
  pull: once

job=generate-version:
  use: image-gitversion
  mounts:
  - mount-git
  - mount-gen
  env:
  - "USERID={user.uid}"
```

## Customization of working directories

The user of this building block can specify the place where the `git` and the `gen` directories are created. 
This is done by setting the environment variables `GIT_PATH` and `GEN_PATH`. Usually this is not needed for local builds, because docker is able to make mounts to the default directories like `/git` and `/gen`. However some systems like concourse CI aren't able to do something like that. In those cases it might be useful to be able to set those paths.

## Integration via git mechanisms

**_Currently there is no really nice way to integrate dobi scripts from other projects into your own. This is mainly due to the fact that first of all there are no namespaces or similars, which means that uniqueness of dobi resources has to be created manually. Secondly, and this makes things even more complicated, paths always refer to the root document. This means that when a subscript is included in a project, all path specifications may need to be adjusted.If you keep both in mind, you can easily integrate dobi-based projects into other projects with a little effort._**

_Integration via git mechanisms is only recommended if changes are made to the building block and these are to be managed together with the project. Integration via docker image is much easier._

For git based integration it is first necessary to integrate the source code so that the dobi script of the project can access the scripts of the building block.

Recommended methods for integration are either per `git submodule` (if no adjustments to the building block are necessary) or per `git subtree` (if it should be adjusted).

In the dobi meta section of your own project you have to include all dobi files integrated in meta.yaml. This way all resources defined by bb-gitversion will be included in your project and can be executed.

To work correctly it is also necessary to adapt all paths to the new root and if necessary to make the naming of the dobi resources unique (see note).

Furthermore, it is possible that you need to adapt the mount-points listed in dobi.yaml to your projects needs.

After that, you can use the building block to generate version informations for you:

```yaml
mount=mount-git:
  bind: .
  path: /git
  read-only: false # needs to be read/write to work correctly!

mount=mount-gen:
  bind: ./gen/
  path: /gen
  read-only: false

job=generate-version:
  use: image-gitversion
  mounts:
  - mount-git
  - mount-gen
  env:
  - "USERID={user.uid}"
```

## Using concourse CI for a CI/CD build

The pipeline file must be uploaded to concourse CI via `fly`. 
Enter the build users ssh private key into the file `ci/credentials.template.yaml` and rename it to `ci/credentials.yaml`. 

**Note: `credentials.yaml` is ignored by `.gitignore` and will not be checked in.**

In further releases there will be a key value store to keep track of the users credentials.
Before setting the pipeline you might login first to your concourse instance `fly -t <target> login --concourse-url http://<concourse>:<port>`. See the [fly documentation](https://concourse-ci.org/fly.html) for more help.
Upload the pipeline file with fly:

    $ fly -t <target> set-pipeline -n -p bb-buildingblock -l ci/config.yaml -l ci/credentials.yaml -c pipeline.yaml

After successfully uploading the pipeline to concourse CI login and unpause it. After that the pipeline should be triggered by new commits on the master branch (or new tags if enabled in `pipeline.yaml`).

# What is embedded linux building blocks

embedded linux building blocks is a project to create reusable and
adoptable blueprints for highly recurrent issues in building an internet
connected embedded linux system.

# License

Licensed under either of

-   Apache License, Version 2.0, (./LICENSE-APACHE or <http://www.apache.org/licenses/LICENSE-2.0>)
-   MIT license (./LICENSE-MIT or <http://opensource.org/licenses/MIT>)

at your option.

# Contribution

Unless you explicitly state otherwise, any contribution intentionally
submitted for inclusion in the work by you, as defined in the Apache-2.0
license, shall be dual licensed as above, without any additional terms or
conditions.

Copyright (c) 2020 conplement AG
