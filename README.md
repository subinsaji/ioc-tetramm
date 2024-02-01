Generic IOC Template Repository
===============================

This repository is a template for creating a generic IOC in the epics-containers
framework. See https://epics-containers.github.io for tutorials and documentation.

Making a new generic IOC
------------------------

1.  Go to the [GitHub page](https://github.com/epics-containers/ioc-template)
    and click **Use this template** -> **Create a new repository** Then choose a
    name and organization for your new generic IOC.

1. Change the `README.md` file to describe your IOC.

1. Change the `Dockerfile` to compile in the support modules you need in
   your generic IOC Container.

1. Once you have it working you can update `tests/runtests.sh` to run your IOC
   in a container and test it. We recommend creating a Tickit simulation of the
   device you are controlling and using that to test your IOC. See
   https://dls-controls.github.io/tickit/typing_extensions/index.html


Working with the IOC
--------------------

1. A generic IOC with a working Dockerfile can be built with the build script in
   the root folder. This builds the developer target by default. This must be
   run outside the dev container.

1. Pushing the repo to github and tagging it will release a container image to
   the GitHub container registry. Gitlab CI is also supported.

1. Launching the project in vscode or other devcontainer IDE will allow you to
   open a devcontainer and test the IOC locally (including IOCs with system
   dependencies not available on your host).

1. A Basic test that verifies you have the correct libraries installed in
   the runtime container is provided in tests/runtests.sh. This must be
   run outside the dev container. These tests can be updated to be more
   specific to your IOC when you have it working.

Developing inside the devcontainer
----------------------------------

A few points to note when developing inside the devcontainer:

1. The project folder is mounted at `/workspaces/ioc-template` (or equivalent).
   This is the same as any other project folder in vscode.

   The project folder is also mounted at `/epics/generic-source` which is
   where `ibek` expects to find it. Because the Dockerfile also creates
   the ioc source at `/epics/generic-source/ioc`, the
   devcontainer container looks almost identical to the runtime container.

   Because the project source is mounted over the top of the ioc source location
   in the devcontainer the compiled binaries are not visible in the
   devcontainer. On first opening the devcontainer you must build the IOC
   binaries as follows:

   - `cd /epics/ioc`
   - `make`

1. The ibek tool is pre-installed in the devcontainer. It is used by Dockerfiles
   to build the IOC but you can also use it on the command line. See
   `ibek --help` for more information.

1. You can add additional projects to your workspace that are peers to the
   `ioc-template` project. The folder `/workspaces` is mounted to the parent of
   your project folder for this purpose. It is often useful to add your beamline
   repository to the workspace so that you can work on IOC instances at the same
   time as the Generic IOC.

1. You can customize bash configuration and also customize a profile that can
   install additional packages at devcontainer creation time. These are
   customizations are for developer personalization and can apply to all
   epics-containers ioc projects. See [README.md](user_examples/README.md).

Updating projects that use this template
----------------------------------------

When there have been useful changes to this template you can update your project
to use them. The simplest way to do this is to simply run a comparison of your
`ioc-xxx` root folder against a local copy of the latest `ioc-template`. The
only files that need to be changed usually are the `Dockerfile` and `README.md`.
So you can adopt the latest version of all other file. Also any Dockerfile
changes at the top and bottom of the file should be copied over, your changes in
the middle should only replace the block comment.

For an easy folder comparison you can use the vscode extension
[Compare Folders](https://marketplace.visualstudio.com/items?itemName=moshfeu.compare-folders).

Quick Update to latest ioc-template
-----------------------------------

If you are confident you have only changed the Dockerfile and README.md you can
use the following script to update your project to the latest ioc-template.
You will still need to merge any changes to the header/footer of the Dockerfile.

```bash
#!/bin/bash

set -xe

ioc_template="$(realpath ${1})"
ioc_for_update="$(realpath ${2})"

cd ${ioc_for_update}
if [ -n "$(git status --porcelain)" ]; then
      echo "You have uncommitted changes, this script is not safe to run"
      exit 1
fi

git checkout -b update-to-latest-template
rm -fr $(ls -A -I .git -I ibek-support)

cd ${ioc_template}
cp -r $(ls -A -I .git -I ibek-support) ${ioc_for_update}

cd ${ioc_for_update}
git checkout Dockerfile README.md .gitmodules
git add .

tests/run-tests.sh
```


Once you are happy with the updates you can do:

```bash
cd ioc-xxx
git commit -m'update to latest ioc-template'
git push -u origin update-to-latest-template
# now go to github and create a pull request
```

If you want to revert the changes you can do:

```bash
cd ioc-xxx
git reset --hard
git checkout main
git branch -d update-to-latest-template
```


Jan 2024 Changes
----------------

After a review we have decided to simplify the approach to managing the IOC
source code. It is now included in the ioc-template repository and is
no longer templated in `ibek`.

The Makefile is the only dynamic thing in the ioc folder and it now uses
the files /epics/support/lib_list and dbd_list which `ibek` generates.

We also changed where the project is mounted in the devcontainer. It is now
in `/workspaces/ioc-template` which makes it the same as any other. But note
that it is *also* mounted in `/epics/generic-source` so that there is a
consistent location for `ibek` to find it.

To update a pre-existing generic ioc to the new scheme:

- update requirements.txt to `ibek==1.6.2`
- update ibek-support sub module to the latest version in `main`
- copy in the ioc folder from ioc-template verbatim
- update Dockerfile top and bottom to match ioc-template (use a diff to see)
- compare you ioc-xxx folder against ioc-template and update any other files
  that have changed, especially in .github/workflows
