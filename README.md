# AutoPress

AutoPress is a fork from [Bedrock](http://roots.io/wordpress-stack/) to simplify setting up a module wordpress development stack

# ToC

* [Quick Start](#quick-start)
* [Features](#features)
* [Required](#requirements)
* [Installation/Usage](#installationusage)
  * [via Composer](#using-create-project)
  * [Manually](#manually)
* [Deploying with Capistrano](#deploying-with-capistrano)
  * [Steps](#deployment-steps)
* [Documentation](#deploying-with-capistrano)
  * [Folder Structure](#folder-structure)
  * [Configuration Files](#configuration-files)
  * [Environment Variables](#environment-variables)
  * [Composer](#composer)
  * [Capistrano](#capistrano)
  * [WP-CLI](#wp-cli)
  * [Vagrant/Ansible](#vagrantansible)
  * [mu-plugins Autoloader](#mu-plugins-autoloader)
* [Contributing](#contributing)
* [Support](#support)

## Quick Start

Use [bedrock-ansible](https://github.com/roots/bedrock-ansible) to get started with a development VM customized for Bedrock.

Or run `composer create-project roots/bedrock <path>` (see [Installation/Usage](#installationusage) for more details) to just get a new copy of Bedrock locally.

## Features

* Dependency management with [Composer](http://getcomposer.org)
* Automated deployments with [Capistrano](http://www.capistranorb.com/)
* Better folder structure
* Easy WordPress configuration with environment specific files
* Environment variables with [Dotenv](https://github.com/vlucas/phpdotenv)
* Easy server environments with [Vagrant](http://www.vagrantup.com/) and [Ansible](http://www.ansible.com/home) - [bedrock-ansible](https://github.com/roots/bedrock-ansible) on GitHub
* Autoloader for mu-plugins (let's you use regular plugins as mu-plugins)

Bedrock is meant as a base for you to fork and modify to fit your needs. It is delete-key friendly and you can strip out or modify any part of it. You'll also want to customize Bedrock with settings specific to your sites/company.

Much of the philosphy behind Bedrock is inspired by the [Twelve-Factor App](http://12factor.net/) methodology including the [WordPress specific version](http://roots.io/twelve-factor-wordpress/).

Note: While this is a project from the guys behind the [Roots starter theme](http://roots.io/starter-theme/), Bedrock isn't tied to Roots in any way and works with any theme.

## Requirements

* Git
* PHP >= 5.3.2 (for Composer)
* Ruby >= 1.9 (for Capistrano)

If you aren't interested in using a part, then you don't need its requirements either. Not deploying with Capistrano? Then don't worry about Ruby for example.

## Installation/Usage

See [Documentation](#documentation) for more details on the steps below.

### Using `create-project`

Composer's `create-project` command will automatically install the Bedrock project to a directory and run `composer install`.

The post-install script will automatically copy `.env.example` to `.env` and you'll be prompted about generating salt keys and appending them to your `.env` file.

Note: To generate salts without a prompt, run `create-project` with `-n` (non-interactive). You can also change the `generate-salts` setting in `composer.json` under `config` in your own fork. The default is `true`.

To skip the scripts completely, `create-project` can be run with `--no-scripts` to disable it.

1. Run `composer create-project roots/bedrock <path>` (`path` being the folder to install to)
2. Edit `.env` and update environment variables:
  * `DB_NAME` - Database name
  * `DB_USER` - Database user
  * `DB_PASSWORD` - Database password
  * `DB_HOST` - Database host (defaults to `localhost`)
  * `WP_ENV` - Set to environment (`development`, `staging`, `production`, etc)
  * `WP_HOME` - Full URL to WordPress home (http://example.com)
  * `WP_SITEURL` - Full URL to WordPress including subdirectory (http://example.com/wp)
3. Add theme(s)
4. Set your Nginx or Apache vhost to `/path/to/site/web/` (`/path/to/site/current/web/` if using Capistrano)
5. Access WP Admin at `http://example.com/wp/wp-admin`


### Manually

1. Clone/Fork repo
2. Run `composer install`
3. Copy `.env.example` to `.env` and update environment variables:
  * `DB_NAME` - Database name
  * `DB_USER` - Database user
  * `DB_PASSWORD` - Database password
  * `DB_HOST` - Database host (defaults to `localhost`)
  * `WP_ENV` - Set to environment (`development`, `staging`, `production`, etc)
  * `WP_HOME` - Full URL to WordPress home (http://example.com)
  * `WP_SITEURL` - Full URL to WordPress including subdirectory (http://example.com/wp)
4. Add theme(s)
4. Set your Nginx or Apache vhost to `/path/to/site/web/` (`/path/to/site/current/web/` if using Capistrano)
5. Access WP Admin at `http://example.com/wp/wp-admin`

Using Capistrano for deploys?

### Deploying with Capistrano

Required Gems:

* capistrano (> 3.1.0)
* capistrano-composer

These can be installed manually with `gem install <gem name>` but it's highly suggested you use [Bundler](http://bundler.io/) to manage them. Bundler is basically the Ruby equivalent to PHP's Composer. Just as Composer manages your PHP packages/dependencies, Bundler manages your Ruby gems/dependencies. Bundler itself is a Gem and can be installed via `gem install bundler` (sudo may be required).

The `Gemfile` in the root of this repo specifies the required Gems (just like `composer.json`). Once you have Bundler installed, run `bundle install` to install the Gems in the `Gemfile`. When using Bundler, you'll need to prefix the `cap` command with `bundle exec` as seen below (this ensures you're not using system Gems which can cause conflicts).

See http://capistranorb.com/documentation/getting-started/authentication-and-authorisation/ for the best way to set up SSH key authentication to your servers for password-less (and secure) deploys.

### Deployment Steps

1. Edit your `config/deploy/` stage/environment configs to set the roles/servers and connection options.
2. Before your first deploy, run `bundle exec cap <stage> deploy:check` to create the necessary folders/symlinks.
3. Add your `.env` file to `shared/` in your `deploy_to` path on the remote server for all the stages you use (ex: `/srv/www/example.com/shared/.env`)
4. Run the normal deploy command: `bundle exec cap <stage> deploy`
5. Enjoy one-command deploys!

* Edit stage/environment configs in `config/deploy/` to set the roles/servers and connection options.

## Documentation

### Folder Structure

```
├── Capfile
├── composer.json
├── config
│   ├── application.php
│   ├── deploy
│   │   ├── staging.rb
│   │   └── production.rb
│   ├── deploy.rb
│   ├── environments
│   │   ├── development.php
│   │   ├── staging.php
│   │   └── production.php
│   └── application.php
├── Gemfile
├── vendor
└── web
    ├── app
    │   ├── mu-plugins
    │   ├── plugins
    │   └── themes
    ├── wp-config.php
    ├── index.php
    └── wp
```

The organization of Bedrock is similar to putting WordPress in its own subdirectory but with some improvements.

* In order not to expose sensitive files in the webroot, Bedrock moves what's required into a `web/` directory including the vendor'd `wp/` source, and the `wp-content` source.
* `wp-content` (or maybe just `content`) has been named `app` to better reflect its contents. It contains application code and not just "static content". It also matches up with other frameworks such as Symfony and Rails.
* `wp-config.php` remains in the `web/` because it's required by WP, but it only acts as a loader. The actual configuration files have been moved to `config/` for better separation.
* Capistrano configs are also located in `config/` to make it consistent.
* `vendor/` is where the Composer managed dependencies are installed to.
* `page/` is where the WordPress core lives. It's also managed by Composer but can't be put under `vendor` due to WP limitations.


### Configuration Files

The root `web/wp-config.php` is required by WordPress and is only used to load the other main configs. Nothing else should be added to it.

`config/application.php` is the main config file that contains what `wp-config.php` usually would. Base options should be set in there.

For environment specific configuration, use the files under `config/environments`. By default there's is `development`, `staging`, and `production` but these can be whatever you require.

The environment configs are required **before** the main `application` config so anything in an environment config takes precedence over `application`.

Note: You can't re-define constants in PHP. So if you have a base setting in `application.php` and want to override it in `production.php` for example, you have a few options:

* Remove the base option and be sure to define it in every environment it's needed
* Only define the constant in `application.php` if it isn't already defined.

#### Don't want it?

You will lose the ability to define environment specific settings.

* Move all configuration into `wp-config.php`
* Manually deal with environment specific options
* Remove `config` directory

### Environment Variables

Bedrock tries to separate config from code as much as possible and environment variables are used to achieve this. The benefit is there's a single place (`.env`) to keep settings like database or other 3rd party credentials that isn't committed to your repository.

[PHP dotenv](https://github.com/vlucas/phpdotenv) is used to load the `.env` file. All variables are then available in your app by `getenv`, `$_SERVER`, or `$_ENV`.

Currently, the following env vars are required:

* `DB_USER`
* `DB_NAME`
* `DB_PASSWORD`
* `WP_HOME`
* `WP_SITEURL`

#### Don't want it?

You will lose the separation between config and code and potentially put secure credentials at risk.

* Remove `dotenv` from `composer.json` requires
* Remove `.env.example` file from root
* Remove `require_once('vendor/autoload.php');` from `wp-config.php`
* Replace all `getenv` calls with whatever method you want to set those values

### Composer

[Composer](http://getcomposer.org) is used to manage dependencies.

#### Plugins

[WordPress Packagist](http://wpackagist.org/) is already registered in the `composer.json` file so any plugins from the [WordPress Plugin Directory](http://wordpress.org/plugins/) can easily be required.

To add a plugin, add it under the `require` directive or use `composer require <namespace>/<packagename>` from the command line. If it's from WordPress Packagist then the namespace is always `wpackagist-plugin`.

Example: `"wpackagist-plugin/akismet": "dev-trunk"`

Whenever you add a new plugin or update the WP version, run `composer update` to install your new packages.

`plugins`, and `mu-plugins` are Git ignored by default since Composer manages them. If you want to add something to those folders that *isn't* managed by Composer, you need to update `.gitignore` to whitelist them:

`!web/app/plugins/plugin-name`

Sample PLugins added in

Note: Some plugins may create files or folders outside of their given scope, or even make modifications to `wp-config.php` and other files in the `app` directory. These files should be added to your `.gitignore` file as they are managed by the plugins themselves, which are managed via Composer. Any modifications to `wp-config.php` that are needed should be moved into `config/application.php`.

#### Updating WP and plugin versions

Updating your WordPress version (or any plugin) is just a matter of changing the version number in the `composer.json` file.

Then running `composer update` will pull down the new version.

#### Themes

A starter wp themes as a part of post load script in composer. Can be switched to any custom theme.

#### Don't want it?

Composer integration is the biggest part of Bedrock, so if you were going to remove it there isn't much point in using Bedrock.

#### DB Syncing

Bedrock doesn't come with anything by default to do DB syncing yet. The best option is to use WP-CLI.

[@lavmeiker](https://github.com/lavmeiker) has a nice Capistrano WP-CLI wrapper plugin to make this even easier. [capistrano-wpcli](https://github.com/lavmeiker/capistrano-wpcli) offers the following commands (and more):

* Sync DB: `cap production wpcli:db:push` and `cap production wpcli:db:pull`
* Sync uploads: `cap production wpcli:uploads:rsync:push` and `cap production wpcli:uploads:rsync:pull`

### wp-cron

Bedrock disables the internal WP Cron via `define('DISABLE_WP_CRON', true);`. If you keep this setting, you'll need to manually set a cron job like the following in your crontab file:

`*/5 * * * * curl http://example.com/wp/wp-cron.php`

## WP-CLI

Bedrock works with [WP-CLI](http://wp-cli.org/) just like any other WordPress project would. Previously we required WP-CLI in our `composer.json` file as a dependency. This has been removed since WP-CLI now recommends installing it globally with a `phar` file. It also caused conflicts if you tried using a global install.

The `wp` command will automatically pick up Bedrock's subdirectory install as long as you run commands from within the project's directory (or deeper). Bedrock includes a `wp-cli.yml` file that sets the `path` option to `web/wp`. Use this config file for any further [configuration](http://wp-cli.org/config/).

## mu-plugins autoloader

Bedrock includes an autoloader that enables standard plugins to be required just like must-use plugins.
The autoloaded plugins are included after all mu-plugins and standard plugins have been loaded.
An asterisk (*) next to the name of the plugin designates the plugins that have been autoloaded.
To remove this functionality, just delete `web/app/mu-plugins/bedrock-autoloader.php`.

This enables the use of mu-plugins through Composer if their package type is `wordpress-muplugin`. You can also override a plugin's type like the following example:

```json
"installer-paths": {
  "web/app/mu-plugins/{$name}/": ["type:wordpress-muplugin", "roots/soil"],
  "web/app/plugins/{$name}/": ["type:wordpress-plugin"],
  "web/app/themes/{$name}/": ["type:wordpress-theme"]
},
```

[Soil](https://github.com/roots/soil) is a package with its type set to `wordpress-plugin`. Since it implements `composer/installers` we can override its type.

