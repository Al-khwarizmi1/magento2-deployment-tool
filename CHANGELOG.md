# Changelog
All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](http://keepachangelog.com/en/1.0.0/)
and this project adheres to [Semantic Versioning](http://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [2.0] - 14-12-2017
### Added
- Zero Downtime using new features in `Magento >= 2.2`
	- `setup:db:status`
	- `config:import`
	- `cron:install` 
- Add out of the box functionality to deploy artifacts already built.
	- `build.project.type=artifact`

### Changed
- Set files permissions only when generating magento files. 
	- If `build.project.type=artifact`, permission are already set on the built archive.
- `${release.target}/config/project.properties` have highest priority. That is to be able to share configurations with [magento2-builder-tool](https://github.com/staempfli/magento2-builder-tool)
- Update composer install command options:
	- `composer install --no-dev --prefer-dist --optimize-autoloader`


### Removed
- Remove `snapshots` option. Release version `develop` should be used instead for `CD`
