woocommerce-extension-release
=============================

`woorelease` is a helper utility which follows the release process for The Extendables to release an extension on GitHub, and then deploy it to WooCommerce.com or WordPress.org. It allows for bulk deployments.

# Documentation

The documentation is available at https://fieldguide.automattic.com/woorelease-script/

# Development documentation

Additional information related to the development of WooRelease can be found at [docs/DEVELOPER.md](docs/DEVELOPER.md).

# Branches

This repo uses the GitFlow branching model. Meaning the default branch is `develop` where we add any feature work to. `master` branch will be the branch that is in circulation (public release). Any bug fixes will be derived from `master` branch.

# Installation

If you are using `woorelease` from the GitHub repository it is necessary to install the dependencies. To do that execute `composer install` from inside the `woorelease` folder. You can also execute `npm run build` command. Besides triggering the installation this command also creates a release ready .zip file.

# Prerequisites for configuration

1. Git is configured so that you are able to clone repositories
1. Supply env variable `WOORELEASE_GH_TOKEN`, with scope of at least `Full control of private repositories`.
1. Supply env variable `WOORELEASE_WCCOM_REST_KEY` in the format of `ck_xxx:cs_xxx` providing your WooCommerce.com REST credentials.
1. For WordPress.org released, extensions, `svn` must be installed and configured with credentials that have commit access to the plugin repository.

# Requirements

A `package.json` file must exist and must have a build step to at least create the archive zip file. If you're releasing a wp.org extenion additional items must be listed in `package.json`. Please refer to `WordPress.org deployments` section below.

**Example `package.json` file.**
```
{
  "name": "YOUR-EXTENSION",
  "title": "YOUR EXTENSION",
  "version": "1.5.3",
  "scripts": {
    "build": "git archive --format zip HEAD | cat > $npm_package_name.zip"
  }
}
```
 Of course that is just an example. You can use whatever method of creating this archive you want as long as its in your `build` step where this script will run.
 
# Logging

WooRelease logs release flow to the `woorelease-{date}.log` files located in the `logs` folder in the main WooRelease folder.
The `{date}` in `woorelease-{date}.log` is substituted by the date when the script is executed. For example `woorelease-2020-04-03.log`.
Up to 10 newest log files are kept. If more then 10 files are present in the logs folder the oldest ones are deleted.
To change where the logs files are located please specify the `WOORELEASE_LOGS_FOLDER` env variable with the desired location.

# Steps to release an extension

There are different ways you can do a release. Some will have automations which may be useful.

**Method 1**
This method is used when everything is updated and ready to go as is. No changes need to be made for you.

1. Make sure that the version is bumped everywhere (changelog.txt, plugin file header, readme.txt)
1. Make sure changelog contains latest update entries.
1. Run `php woorelease.php extension-name` for simulation mode and confirm everything is ok!
1. If everything looks ok, then run `php woorelease.php --release extension-name` and go get some coffee :)

**Method 2**
This method is used when you want the script to bump versions for you and commit to GitHub.

1. Using the `--version` means you want the script to bump versions/date for you. This requires specific placeholder style.
1. Make sure in your changelog.txt you have an unfinished date entry format like this `2020-xx-xx - version 1.5.3` or if it is a wp.org extension like this `= 1.5.3 - 2020-xx-xx =`. This lets the script know this is the entry line where it should bump the date for you.
1. If the extension is a wp.org extension, make sure in your readme.txt file changelog section, you have an unfinished date entry format like this `= 1.5.3 - 2020-xx-xx =`. This lets the script know this is the entry line where it should bump the date for you.
1. Run `php woorelease.php extension-name` for simulation mode and confirm everything is ok!
1. If everything looks ok, then run `php woorelease.php --release extension-name` and go get some coffee :)

# Command line examples

Examples:
- `WOORELEASE_GH_TOKEN=ck_xxx:cs_xxx WOORELEASE_WCCOM_REST_KEY=xxx php woorelease.php https://github.com/woocommerce/woocommerce-bookings`
- `php woorelease.php https://github.com/woocommerce/woocommerce-bookings` (assuming `WOORELEASE_GH_TOKEN` and `WOORELEASE_WCCOM_REST_KEY` persist in e.g. `bash_profile`)
- `php woorelease.php https://github.com/woocommerce/woocommerce-bookings/tree/develop` (use `develop` branch to create release from)
- `php woorelease.php --no-deploy --prerelease https://github.com/woocommerce/woocommerce-bookings/tree/release/1.14.0` (create a release from `release/1.14.0` branch, marked as a prerelease, and skip deploy)
- `php woorelease.php https://github.com/woocommerce/woocommerce-bookings` (see what steps are taken to release/deploy without actually releasing and deploying)
- `php woorelease.php --release https://github.com/woocommerce/woocommerce-bookings` (passing the `--release` flag will REALLY release/deploy the extension)

Example `.bash_profile` configuration:

```
export WOORELEASE_GH_TOKEN="xxx"
export WOORELEASE_WCCOM_REST_KEY="ck_xxx:cs_xxx"
export WOORELEASE_EDITOR=vi
export WOORELEASE_GENERATE_CHANGELOG=true
alias wr="php ~/dev/woocommerce-extension-release/woorelease.php $1"
```

Parameters that are not properly formatted, or extensions that cannot be cloned, or versions that do not have their specific tag in the Git remote repo will be skipped for release/deployment.

# WordPress.org deployments

Extensions that are deployed to WordPress.org instead of WooCommerce.com, must be checked in to the WordPress.org svn repository. For such extensions, `woocommerce-extension-release` automatically creates a GitHub release, checks out the applicable svn folder, copies the files into the working copy of the the trunk. If there are any changes such as added or deleted files, you will get a prompt to asking you how you want to proceed. If you answered `yes` to make the changes for you, then the script will continue to release your extension to wp.org. You don't have to do anything else. If you answered `no` however, then you must correct the changes in the prompt and re-run this script.

The file `package.json` must exist, and it must have a build step configured. The build step must generate an output zip file of the `[extension-slug].zip` containing contents that will be deployed. Extensions that are deployed to WordPress.org require a `config.wp_org_slug` value that is set to extension's slug on WordPress.org (this does not always match the GitHub repo slug).

An example `package.json` file would look like:

```
{
	...
	"config": {
		"wp_org_slug": "woocommerce-google-analytics-integration"
	}
	...
}
```

# Using an editor for generating the changelog

When generating the changelog, it's possible to use an editor to correct any changelog entries. Set either the env variable `WOORELEASE_EDITOR` or pass the editor through the option `--editor`. This can be set to your favourite editor, ex:

- `WOORELEASE_EDITOR=vi`
- `WOORELEASE_EDITOR=nano`
- `WOORELEASE_EDITOR="code --wait"`

To enable the changelog generator, set the environment variable `WOORELEASE_GENERATE_CHANGELOG=true` or use the option `--generate-changelog`.

# Changelog formats

The expected changelog.txt format depends on how the extension is deployed.

# Changelog entries for .org extensions

The changelog within the readme.txt is assumed to be filled prior to running this script. The script however will bump the `Stable tag` in the header along with `Tested up to` if this parameter is used.

# Defined constants

Some extensions have defined constants such as `define( 'WC_EXTENSION_VERSION', '1.2.3' );`. This release script if using the `version` parameter will try and update this constant version for the extension as well. To be able to correctly read this line of code, you must make sure there is a correct comment available inline with the declaration. For example it has to look something like this `define( 'WC_EXTENSION_VERSION', '1.2.3' ); // WRCS: DEFINED_VERSION.` Without this commented line, the script will not be able to pick this version up and will not bump it.

# WooCommerce.com deployed extensions

```
2019-09-03 - version 1.6.13
* Fix - A changelog note
```

# WordPress.org deployed extensions

The changelog entries for WordPress.org extensions, follow a certain format so they can be directly copied into the readme.txt without change:
```
= 1.6.13 - 2019-09-03 =
* Fix - A changelog note
```

# Options

## Parameters

1. `--version` (optional) Sets the version to bump extension with. Acceptable values are for example `1.1.20` or `3.20.1`. If version parameter is not provided, it will make the release as is within the GitHub repo.
1. `--wc_tested` Sets the `WC tested up to` version.
1. `--wp_tested` Sets the `Tested up to` version. ( for WordPress version ).

## Flags

1. `--prerelease` This flag causes the created GitHub release to be marked as a prerelease. No other steps are modified by this flag (i.e. extension will still be deployed).
1. `--no-deploy` Skips deploying to WooCommerce.com.
1. `--release` By default this tool will be in simulation mode. Meaning it will skip creating release branches on GitHub, skip uploading of assets to GitHub, skip commiting changes to GitHub and won't deploy to woocommerce.com. After running it in simulation mode and you are ready, then pass this flag to release/deploy for real.
1. `--generate-changelog` This flag will enable the generation of changelog entries (retrieved from merged PR's).
