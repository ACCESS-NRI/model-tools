# model-deployment-template

A template repository for the deployment of `spack`-based models.

> [!NOTE]
> Feel free to replace this README with information on the model once the TODOs have been ticked off.

## Things TODO to get your model deployed

### Settings

#### Repository Settings

Branch protections should be set up on `main` and the special `backport/*.*` branches, which are used for backporting of fixes to major releases (the `YEAR.MONTH` portion of the `YEAR.MONTH.MINOR` version) of models.

#### Repository Secrets/Variables

There are a few secrets and variables that must be set at the repository level.

##### Repository Secrets

* `GH_COMMIT_CHECK_TOKEN`: GitHub Token that allows workflows to run based on workflow-authored commits (in  the case where a user uses `!bump` commands in PRs that bumps the version of the model)

##### Repository Variables

* `BUILD_DB_PACKAGES`: List of `spack` packages that are model components that will be uploaded to the release provenance database
* `NAME`: which corresponds to the model name - which is usually the repository name
* `CONFIG_VERSIONS_SCHEMA_VERSION`: Version of the [`config/versions.json` schema](https://github.com/ACCESS-NRI/schema/tree/main/au.org.access-nri/model/deployment/config/versions) used in this repository
* `SPACK_YAML_SCHEMA_VERSION`: Version of the [ACCESS-NRI-style `spack.yaml` schema](https://github.com/ACCESS-NRI/schema/tree/main/au.org.access-nri/model/spack/environment/deployment) used in this repository
* `RELEASE_DEPLOYMENT_TARGETS`: Space-separated list of deployment targets when doing release deployments. These are often the names of [keys under the `deployment` key of `build-cd`s `config/settings.json`](https://github.com/ACCESS-NRI/build-cd/blob/09cdf100eefc58f06900e8e9145e77b4caf5a39d/config/settings.json#L3), such as `Gadi` or `Setonix`. As noted [below](#environment-secretsvariables), it is the same as the GitHub Environment name. For example: `Gadi Setonix`
* `PRERELEASE_DEPLOYMENT_TARGETS`: Space-separated list of deployment targets when doing prerelease deployments, similar to the above. For example: `Gadi Setonix` - note the lack of a `Prerelease` specifier!

#### Environment Secrets/Variables

GitHub Environments are sets of variables and secrets that are used specifically to deploy software, and hence have more security requirements for their use.

Currently, we have two Environments per deployment target - one for `Release` and one for `Prerelease`. Our current list of deployment targets and Environments can be found in this [deployment configuration file in `build-cd`](https://github.com/ACCESS-NRI/build-cd/blob/main/config/deployment-environment.json).

In order to deploy to a given deployment target:

* Environments with the name of the deployment target and the type must be created _in this repository_ and have the associated secrets/variables set ([see below](#environment-secrets))
* There must be a `Prerelease` Environment associated with the `Release` Environment. For example, if we are deploying to `SUPERCOMPUTER`, we require Environments with the names `SUPERCOMPUTER Release`, `SUPERCOMPUTER Prerelease`.

When setting the environment up, remember to require sign off by a member of ACCESS-NRI when deploying as a `Release`.

Regarding the secrets and variables that must be created:

##### Environment Secrets

* `HOST`: The deployment location SSH Host
* `HOST_DATA`: The deployment location SSH Host for data transfer (may be the same as `HOST`)
* `SSH_KEY`: A SSH Key that allows access to the above `HOST`/`HOST_DATA`
* `USER`: A Username to login to the above `HOST`/`HOST_DATA`
* (Required only for `Release` Environment) `TRACKING_SERVICES_POST_TOKEN`: A token used to post build information about the packages in `secrets.BUILD_DB_PACKAGES` to the release provenance database as part of tracking services. 

##### Environment Variables

* `DEPLOYED_MODULES_DIR`: Directory that will contain the modules created during the installation of the model. This can be virtual modules created by a [`.modulerc` file](https://github.com/ACCESS-NRI/build-cd/tree/main/tools/modules) in the directory.
* `DEPLOYMENT_TARGET`: Name of the deployment target. It is exported to the deployment target and used for variations in `spack.yaml` build processes - seen most prominently in mutually-exclusive 'when' clauses like `spack.definitions[].when = env['DEPLOYMENT_TARGET'] == 'gadi'`. Also used for logging purposes.
* `SPACK_INSTALLS_ROOT_LOCATION`: Path to the directory that contains all versions of a deployment of `spack`. For example, if `/some/apps/spack` is the `SPACK_INSTALLS_ROOT_LOCATION`, that directory will contain directories like `0.20`, `0.21`, `0.22`, which in turn contain an install of `spack`, `spack-packages` and `spack-config`
* `SPACK_YAML_LOCATION`: Path to a directory that will contain the `spack.yaml` from this repository during deployment
* (Optional) `SPACK_INSTALL_ADDITIONAL_ARGS`: Additional flags outside of `--fresh --fail-fast` to add to the `spack install` command. For advanced users who need to tailor the installation options in their repository.
* (Required only for `Release` Environment) `TRACKING_SERVICES_POST_URL`: A url to the API of the release provenance database as part of tracking services. 

### File Modifications

#### In `.github/workflows`

* Reminder that these workflows use `vars.NAME` (as well as inherit the above environment secrets) and hence these must be set.
* If the name of the root SBD for the model (in [`spack-packages`](https://github.com/ACCESS-NRI/spack-packages/tree/main/packages)) is different from the model name (for example, `ACCESS-ESM1.5`s root SBD is `access-esm1p5`), you must uncomment and set the `jobs.[pr-ci|pr-comment|pr-closed].with.root-sbd` line to the appropriate SBD name.

#### In `config/versions.json`

* `.spack` must be given a version. For example, it will clone the associated `releases/vVERSION` branch of `ACCESS-NRI/spack` if you give it `VERSION`.
* `.spack-packages` should also have a CalVer-compliant tag as the version. See the [associated repo](https://github.com/ACCESS-NRI/spack-packages/tags) for a list of available tags.

#### In `spack.yaml`

There are a few TODOs for the `spack.yaml`:

* Only do this step if there are variations in compiler/etc across deployment targets:
  * `spack.definitions`: Use the `ROOT_PACKAGE` to define the root SBD. The `ROOT_SPEC` simply combines the `ROOT_PACKAGE` with the other, mutually-exclusive `compiler_target` definition.
  * `spack.specs`: Set the only element in the spec list to `$ROOT_SPEC` - it will be filled in at install time.

Otherwise:

* `spack.specs`: Set the root SBD as the only element of `spack.specs`. This must also have an `@git.YEAR.MONTH.MINOR` version as it is the version of the entire deployment (and indeed will be a tag in this repository).
* `spack.packages.*`: In this section, you can specify the versions and variants of dependencies. Note that the first element of the `spack.packages.*.require` must be only a version. Variants and other configuration can be done on subsequent lines.
* `spack.packages.all`: Can set configuration for all packages. For example, the compiler used, or the target architecture.
* `spack.modules.default.tcl.include`: List of package names that will be explicitly included and available to `module load`.
* `spack.modules.default.tcl.projections`: For included modules, you must set the name of the module to be the same as the `spack.packages.*.require[0]` version, without the `@git.`.
