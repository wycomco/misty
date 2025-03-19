# Changelog


## Unreleased

### Added
- Clean up *mist-cli*’s tmp folder 
- Exit *misty* run if invoked by launchd without config files present

### Fixed
- When `exit`ing `1`: do a `makecatalogs | grep warning` and display correct file name of error log – [PR#25](https://github.com/wycomco/misty/pull/25)
- If the version to be processed is already present in repo, do not remove the prior version – [PR#26](https://github.com/wycomco/misty/pull/26)

### Changed
- Use *munki*’s recommended `minimum_munki_version`, also for x86_64 plists – [PR#24](https://github.com/wycomco/misty/pull/24)


## [0.2.3](https://github.com/wycomco/misty/releases/tag/v0.2.3) – 2024-10-24 (Public pre-release)

### Added
- Check for presence of repo if remote – [PR#17](https://github.com/wycomco/misty/pull/17)
- Create workflow for signed and notarized installer – [PR#20](https://github.com/wycomco/misty/pull/20)
- Check owner and group of subdirectories defined in `$munki_path` – [PR#22](https://github.com/wycomco/misty/pull/22)

### Changed
- Checking order, output of launchd logs on exit 1 – [PR#18](https://github.com/wycomco/misty/pull/18)
- Increased storage requirement for installers  – [PR#19](https://github.com/wycomco/misty/pull/19)
- Handling of special characters in password for repo share  – [PR#21](https://github.com/wycomco/misty/pull/21)


## [0.2.2](https://github.com/wycomco/misty/releases/tag/v0.2.2) – 2024-09-17 (Pre-release)

### Added
- Add support for macOS Sequoia, remove macOS Monterey routine; update config.txt if present – [PR#15](https://github.com/wycomco/misty/pull/15)


## [0.2.1](https://github.com/wycomco/misty/releases/tag/v0.2.1) – 2024-09-04 (Pre-release)

### Security
- Sanity check for launchdaemon’s start time – [PR#1](https://github.com/wycomco/misty/pull/1), [PR#10](https://github.com/wycomco/misty/pull/10)
- Move *misty* dirs (usr, skel, Logs) from `/Users/Shared/Mist` to `/var/root/misty` – [PR#4](https://github.com/wycomco/misty/pull/4), [PR#13](https://github.com/wycomco/misty/pull/13)

### Fixed
- Error messages only get prepended by a timestamp if *misty* is running as a launchd job – [PR#11](https://github.com/wycomco/misty/pull/11)
- Removed initial start value for LaunchDaemon in config.txt (fix for [0.1.7](https://github.com/wycomco/misty/releases/tag/v0.1.7)) – [PR#8](https://github.com/wycomco/misty/pull/8)


## [0.2.0](https://github.com/wycomco/misty/releases/tag/v0.2.0) – 2024-08-22 (Pre-release)

### Added 
- Check and restart of LaunchDaemon (only works in interactive run at the moment)
- Export `/bin` to `$PATH`
- Explicitly set `minimum_munki_version` for macOS 12 (was not evaluated if newer major version were not available)
- Claim *misty* as author of plists (main improvement)

### Changed
- Single logging function with timestamps also for error messages of invoked commands (like `munkiimport`)

### Info
- Successful test on two different repo shares


## [0.1.7](https://github.com/wycomco/misty/releases/tag/v0.1.7) – 2024-08-14 (Pre-release)

### Changed
- Re-added initial start value for LaunchDaemon in config.txt


## [0.1.6](https://github.com/wycomco/misty/releases/tag/v0.1.6) – 2024-08-14 (Pre-release)

### Added 
- New variables in `config.txt` for setup using an SMB share
- Timestamps in log files include milliseconds

### Fixed
- Order of declaration of variables

### Changed
- Improved output


## [0.1.5](https://github.com/wycomco/misty/releases/tag/v0.1.5) – 2024-07-05 (Pre-release)

### Added 
- Improved output for removal of previous versions (enhancement)
- Use of master function for less code

### Fixed
- `munkiimport` creates a different payload when run as stage_os_installer or as startosinstall. Since we only want one payload, `installer_item_hash` and `installer_item_size` have to be copied from the stage_os_installer
- Proper checking of required space
- Use different `minimum_munki_versions` for Intel and Apple Silicon
- Use of single continuous strings for variables in paths where possible for better readability


## [0.1.4](https://github.com/wycomco/misty/releases/tag/v0.1.4) – 2024-04-26 (Pre-release)

### Added 
- Prepend timestamps to messages sent to log files when *misty* is invoked by the LaunchDaemon.

### Fixed
- Do cleanup for each major version separately.

### Changed
- License was changed to MIT.
- File `override.txt` was renamed to `config.txt`. If you have tested *misty* before, please rename the `usr` folder in `/Users/Shared/Mist` and let the folder be created again on first run. No changes have been made to other files in the `usr` folder, they can be replaced by your definitions.
- LaunchDaemon will not be loaded, since smb shares do not work yet. Start date in LaunchDaemon will be set on first run nevertheless.
- Alphabetic ordering of functions.
- Soft link of script to `/usr/bin` instead of handling multiple PATH variables in postinstall script.
- Update of README.


## [0.1.3](https://github.com/wycomco/misty/releases/tag/v0.1.3) – 2024-04-16 (Pre-release)

### Security
- Quote variables.

### Fixed
- Do not exit 1 the script if no previous dmg’s are found.
- Import major version if no previous version was present.

### Changed
- LaunchDaemon’s `StartCalendarInterval` will be set by the user at first run and can be edited in override (config) file.
- Re-ordering of functions and definitions.
- Update of README.

### Info
- First successful run on vanilla munki repo environment.


## [0.1.2](https://github.com/wycomco/misty/releases/tag/v0.1.2) – 2024-04-11 (Pre-release)

### Added 
- LaunchDaemon to run at 22:00 local time.
- Keep only highest version of each major upgrade in repo if new specific major version is available. After the run, only the new and previous version are present in repo.
- Differentiate output if run interactively or by launchd.
- Ignore changed timestamp in `mist list`.
- Ignore environment variables when running the script.
- Run a user-specific postinstall only if present and if any new major version have been processed.

### Changed
- Put *misty* in `PATH` for bash and zsh instead of taking care of different shell environments.
- Corrected file permissions.
- Order of information in preupgrade_alert for Intel installers.
- Restructuring of README.


## [0.1.1](https://github.com/wycomco/misty/releases/tag/v0.1.1) – 2024-04-01 (Pre-release)

### Added 
- Improve explanation texts for preloader and deployment plists


## [0.1.0](https://github.com/wycomco/misty/releases/tag/v0.1.0) – 2024-03-28 (Pre-release)

### Added 
- Initial release. Tested successfully on two munki repositories.

### Info
- The version number in the buildinfo.plist is 0.1 without a patch number. Works nevertheless.