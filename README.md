# Magento 2 Deployment tool

Deployment tool for Magento 2 created with [PHing](https://www.phing.info/). This tool builds a new project version into a separate directory and switches live version at the end.

Workflow:

```
1. Get new Project version (i.e git clone, curl, ...)
2. Build Project (i.e composer install, untar, ...)
3. Symlinks to shared content in server
4. Generate Magento files (Skipped if deploying `.tar`)
5. Set folder/files permissions
6. Set Maintenance
7. Database backup
8. Magento setup:upgrade
9. Replace live version with new one
10. Unset maintenance
11. Clean up old releases and backups
```

## Demos

* Deploying git repo

<a href="http://www.youtube.com/watch?feature=player_embedded&v=JFDen6iXMko
" target="_blank"><img src="docs/images/youtube/deploy_git.png"
alt="Magento2 deploy from git" width="240" height="180" border="10" /></a>

* Deploying .tar archive

<a href="http://www.youtube.com/watch?feature=player_embedded&v=JqmZTjbmDwo
" target="_blank"><img src="docs/images/youtube/deploy_tar.png"
alt="Magento2 deploy from tar" width="240" height="180" border="10" /></a>

## Installation

Global installation using composer is required.

0. Composer require:

	```
	composer global require "staempfli/magento2-deployment-tool":"dev-master"
	```

0. Check you global composer `bin-dir` configuration:

	```
	composer global config -l | grep "bin-dir"
	```

0. Add path from previous step into your `$PATH` configuration:
0. Open a new console tab and check that `mg2-deployer` tool is found

	```
	which mg2-deployer
	```

## Setup

0. File `config.php` is required to come within the project cloned.

	* You can follow the following documentation if you do not have your project configured like that yet:
		* [docs/setup/config-php.md](docs/setup/config-php.md)

0. Create Database:

	```
	 CREATE DATABASE <database_name>;
	 CREATE USER `<database_user>`@`<database_host>` IDENTIFIED BY "<user_password>";
     GRANT ALL ON <database_name>.* TO `<database_user>`@`<database_host>`;
	```

0. Run Setup: (this might take several minutes because of magento compilation)

	```
	mg2-deployer setup
	```

0. At the end of the process you should get a folder structure similar to this:

```
  | - backups
  | - deployment-settings
  | - public_html (Symlink)
  | - releases
  | - tmp-downloads
  | - shared
    | - magento
    	| - app (etc/env.php)
    	| - pub
    		| - media
    	| - var
    		| - log
```

## Usage

Tool must be executed at the path where the the project will be deployed.

* Deploy new releases:

	```
	mg2-deployer release [-Drelease.version=<version_number>]
	```

* Other available commands:

	```
	mg2-deployer -list

	Main targets:
	------------
 	release               Options -> [skipCronInstall|skipDatabaseBackup|finishWithMaintenance]
 	maintenance:set       Set maintenance window
 	maintenance:unset     Replace maintenance window with a specific released version
 	release:live:replace  Set a specific release as live version
    cache:clean:all       Clear all caches (Magento, OPcache, Varnish)
	```


## Custom Configuration

### Properties

You can customise properties according to your needs:

**On the Server:**

* `deployment-settings/project.properties`
* Properties added in that file, overwrite default ones

**On the Project:**

* `{{PROJECT_ROOT}}/config/project.properties`
* If you want to share properties within your project, you can add them into that file.
* Properties added here have the highest priority and will overwrite `deployment-settings/project.properties` and default ones.

You can check all default properties that can be customised on:

* [build/config/default.properties](build/config/default.properties)

### Maintenance Window

You can edit the maintenance file with your own design:

```
vim deployment-settings/templates/maintenance/${magento.dir}/pub/index.php
```

### Shared content & Symlinks

* Static content that is only relevant on the server will be kept into the `shared` folder.
* Symlinks are created automatically in the released project during every deployment.
* You can add your custom symlinks by editing the file `deployment-settings/shared.symlinks`

	```
	shared/magento/app/etc/env.php=>{{MAGENTO_DIR}}/app/etc/env.php
	shared/magento/pub/media=>{{MAGENTO_DIR}}/pub/media
	shared/magento/var/log=>{{MAGENTO_DIR}}/var/log
	```
	* Note that you must replace `{{MAGENTO_DIR}}` with your magento dir or `.` if same as root project.

### Scripts

You can set custom scripts to run at the end of the release process on the following file:

```
cp deployment-settings/scripts/release-after.sh.dist deployment-settings/scripts/release-after.sh
vim deployment-settings/scripts/release-after.sh
```

## Build project from Artifact
If you select `artifact` build type for your deployments, these are the default properties to get and build the project files:

```
command.get.project.artifact=
command.build.project.artifact=mkdir -p ${release.target} && tar -xzf ${downloads.target}.tar.gz -C ${release.target}
```

This configuration expects the `artifact` to be available inside `tmp-downloads` with `.tar.gz` extension.

If you need to get the `artifact` from a different server, you can modify the property like that:

```
command.get.project.version=scp <user>@<server_domain>:<path>${release.version}.tar.gz ${downloads.target}.tar.gz
```

**NOTE** `${}` variables are automatically replaced during deployment.

## Tips:

### Speed up deployment process on dev-servers:

* Release most recent version of develop branch `-Drelease.version=develop`
* Skip database backup step `-DskipDatabaseBackup`
* You can even set these options by default on `deployment-settings/project.properties`:

    ```
    vim deployment-settings/project.properties
    release.version=develop
    skipDatabaseBackup=1
    ```

## Troubleshooting

#### Js translations missing (magento versions >=2.1.3 <2.2.1)

*  **Problem**: Known Magento issue when executing `setup:static-content:deploy` for several languages.

* **Github Issues**:
	* [7862](https://github.com/magento/magento2/issues/7862)
	* [10673](https://github.com/magento/magento2/issues/10673)

* **Solution**: Until that gets fixed in `2.2.1`, the only workaround is to execute `setup:static-content:deploy` individually for each language. To run that automatically with `mg2-deployer release`, you need to edit the following: 

	1. `vim deployment-settings/project.properties`
	2. Set only 1 language on `static-content.languages`:
		* `static-content.languages=en_US`
		 
	3. Add the rest of your languages on `command.static-content.deploy.options` with following command for each language:
		* `&& ${bin.n98-magerun2} --root-dir=${release.target}/${magento.dir} setup:static-content:deploy de_CH`

	4. The result will be something like in this example:
	
		```
		static-content.languages=en_US
		command.static-content.deploy.options=--exclude-theme Magento/luma --exclude-theme Magento/blank && ${bin.n98-magerun2} --root-dir=${release.target}/${magento.dir} setup:static-content:deploy de_CH --exclude-theme Magento/luma --exclude-theme Magento/blank && ${bin.n98-magerun2} --root-dir=${release.target}/${magento.dir} setup:static-content:deploy fr_FR --exclude-theme Magento/luma --exclude-theme Magento/blank
		```

#### Compilation error

* **Solution**: Increase php `memory_limit` configuration to 728M o 1024M

#### Static deploy error when setting a new template

* **Problems**:
    * [LogicException] Unable to load theme by specified key: 'Template'
    * @variable is undefined in file
* **Reason**: If a new template is set, running `setup:upgrade` is required before executing `setup:static-content:deploy`
* **Solution**: Skip `setup:static-content:deploy` first time you deploy the new template. Do a release following these steps:

    1. `mg2-deployer release -DfinishWithMaintenance -DskipStaticContentDeploy`
    2. `<latest_release>/<magento_bin> setup:static-content:deploy <language1> <language2> ...`
    3. `mg2-deployer maintenance:unset`
    
    After that, future deployments will work without issues


## Prerequisites

- PHP >= 7.0.8
- MAGENTO >= 2.1.1

## Developers

* [Juan Alonso](https://github.com/jalogut)

Licence
-------
[GNU General Public License, version 3 (GPLv3)](http://opensource.org/licenses/gpl-3.0)

Copyright
---------
(c) 2016 Staempfli AG
