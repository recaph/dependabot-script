# Dependabot Update Script

This repo contains two scripts that demonstrates [Dependabot Core][dependabot-core]. 
It is intended to give you a feel for how Dependabot Core works so that you can use it in your own project.

If you're looking for a hosted, feature-rich dependency updater then you probably want [Dependabot][dependabot] itself.

## Setup and usage

* `rbenv install` (Install Ruby version from `.ruby-version`)
* `bundle install`

### Native helpers

Languages that require native helpers to be installed: Terraform, Python, Go (Dep & Modules), Elixir, PHP, JS

To install the native helpers, export an environment variable that points to the
directory into which the helpers should be installed and add the relevant bins
to your PATH:

* `export DEPENDABOT_NATIVE_HELPERS_PATH="$(pwd)/native-helpers"`
* `mkdir -p $DEPENDABOT_NATIVE_HELPERS_PATH/{terraform,python,dep,go_modules,hex,composer,npm_and_yarn}`
* `export PATH="$PATH:$DEPENDABOT_NATIVE_HELPERS_PATH/terraform/bin:$DEPENDABOT_NATIVE_HELPERS_PATH/python/bin:$DEPENDABOT_NATIVE_HELPERS_PATH/go_modules/bin:$DEPENDABOT_NATIVE_HELPERS_PATH/dep/bin"`
* `export MIX_HOME="$DEPENDABOT_NATIVE_HELPERS_PATH/hex/mix"`

Copy the relevant helpers from the gem source to the new install location

| Language   | Command                                                                                                  |
| ---------- | -------------------------------------------------------------------------------------------------------- |
| Terraform  | `cp -r $(bundle show dependabot-terraform)/helpers $DEPENDABOT_NATIVE_HELPERS_PATH/terraform/helpers`       |
| Python     | `cp -r $(bundle show dependabot-python)/helpers $DEPENDABOT_NATIVE_HELPERS_PATH/python/helpers`             |
| Go Dep     | `cp -r $(bundle show dependabot-dep)/helpers $DEPENDABOT_NATIVE_HELPERS_PATH/dep/helpers`                   |
| Go Modules | `cp -r $(bundle show dependabot-go_modules)/helpers $DEPENDABOT_NATIVE_HELPERS_PATH/go_modules/helpers`     |
| Elixir     | `cp -r $(bundle show dependabot-hex)/helpers $DEPENDABOT_NATIVE_HELPERS_PATH/hex/helpers`                   |
| PHP        | `cp -r $(bundle show dependabot-composer)/helpers $DEPENDABOT_NATIVE_HELPERS_PATH/composer/helpers`         |
| JS         | `cp -r $(bundle show dependabot-npm_and_yarn)/helpers $DEPENDABOT_NATIVE_HELPERS_PATH/npm_and_yarn/helpers` |

Build the helpers you want to use (you'll also need the corresponding language installed)

| Language   | Command                                                                                                   |
| ---------- | --------------------------------------------------------------------------------------------------------- |
| Terraform  | `$DEPENDABOT_NATIVE_HELPERS_PATH/terraform/helpers/build $DEPENDABOT_NATIVE_HELPERS_PATH/terraform`       |
| Python     | `$DEPENDABOT_NATIVE_HELPERS_PATH/python/helpers/build $DEPENDABOT_NATIVE_HELPERS_PATH/python`             |
| Go Dep     | `$DEPENDABOT_NATIVE_HELPERS_PATH/dep/helpers/build $DEPENDABOT_NATIVE_HELPERS_PATH/dep`                   |
| Go Modules | `$DEPENDABOT_NATIVE_HELPERS_PATH/go_modules/helpers/build $DEPENDABOT_NATIVE_HELPERS_PATH/go_modules`     |
| Elixir     | `$DEPENDABOT_NATIVE_HELPERS_PATH/hex/helpers/build $DEPENDABOT_NATIVE_HELPERS_PATH/hex`                   |
| PHP        | `$DEPENDABOT_NATIVE_HELPERS_PATH/composer/helpers/build $DEPENDABOT_NATIVE_HELPERS_PATH/composer`         |
| JS         | `$DEPENDABOT_NATIVE_HELPERS_PATH/npm_and_yarn/helpers/build $DEPENDABOT_NATIVE_HELPERS_PATH/npm_and_yarn` |

### Running `update-script.rb`

* `bundle exec irb`
* Edit the variables at the top of the script you're using, or set the corresponding environment variables.
* Copy and paste the script into the Ruby session to see how Dependabot works.

If you run into any trouble with the above please create an issue!

#### Running script with Docker

If you don't want to setup the machine where the script will be executed, you could run the script within
a `dependabot/dependabot-core` container.
In order to do that, you'll have to pull the image from Docker Hub and mount your working directory into the container.
You'll also have to set several environment variables to make the script work with your configuration, 
as specified in the documentation.

Steps:

1. Pull dependabot-core Docker image

```shell
$ docker pull dependabot/dependabot-core
```

2. Install dependencies

```shell
docker run -v "$(pwd):/home/dependabot/dependabot-script" -w /home/dependabot/dependabot-script dependabot/dependabot-core bundle install -j 3 --path vendor
```

3. Run dependabot

```shell
docker run -v "$(pwd):/home/dependabot/dependabot-script" -w /home/dependabot/dependabot-script -e ENV_VARIABLE=value dependabot/dependabot-core bundle exec ruby ./generic-update-script.rb
```

You'll have to pass the right environment variables to make the script work with your configuration. You can find how to pass environment variables to your container in [Docker run reference](https://docs.docker.com/engine/reference/run/#env-environment-variables).

You'll have to set some mandatory variables like `PROJECT_PATH` and `PACKAGE_MANAGER` (see [script](https://github.com/dependabot/dependabot-script/blob/master/generic-update-script.rb) to know more).
There are other variables that you must pass to your container that will depend on the Git source you use:

* Github (required to allow detection of latest versions of dependencies)
    * GITHUB_USERNAME (OPTIONAL) 
    * GITHUB_ACCESS_TOKEN (required)
* Github Enterprise
    * GITHUB_ENTERPRISE_HOSTNAME  (in the format user:token@ghe.company.com or github.com)
    * GITHUB_ENTERPRISE_ACCESS_TOKEN (required)

If everything goes well you should be able to see something like:

```shell
/home/dependabot/dependabot-script# ./generic-update-script.rb
Fetching gradle dependency files for myorganisation/project
Parsing dependencies information
...
```

## The scripts

* [Single dependency update script][github-script]
* [GHE update script][generic-script]

[github-script]: update-script.rb
[generic-script]: generic-update-script.rb
[dependabot-core]: https://github.com/dependabot/dependabot-core
[dependabot]: https://dependabot.com
