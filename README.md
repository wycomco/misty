# misty

This script checks for the availability of new macOS releases currently supported by Apple, using *mist-cli* (not included in this project’s installer). It also requires a *munki* repo to be already set up. Please see the [System Requirements](#system-requirements) section below for links.

*misty* can be used with a local repository and with SMB or Samba shares.

Since this is a pre-release with frequent updates, the installer has not yet been signed or notarized. To run the downloaded installer, please right-click the `.pkg` file and select 'Open'.

Sequoia note: If you are running the installer on macOS 15, you need to right-click the installer and choose "Open". macOS will not allow you to run the installer immediately, so you need to click on "Done". After that, allow the execution of the installer in System Settings => Privacy & Security. Scroll down to the "Security" section, where you will find the message *"misty" was blocked to protect your Mac.* Tick "Open Anyway". This will bring up a pop-up window where you need to confirm that you want to "Open Anyway" the *misty* installer. Finally, grant the installer permission to control "System Events.app".

See the [Changelog](./CHANGELOG.md) for details on version history and updates.

## Goals of this Script

If a new update for any major version is found, it will be imported into the munki repo, creating the following installers:
- Apple Silicon: *stage_os_installer* and preloader for the installer
- Intel: *startosinstall* with precache key set to true
- Both architectures: Place installer in the `/Applications` directory for a later deployment

The item’s *name* keys will not include a major version. Scoping is done using *board_id* and *device_id* in the plists as *installable_condition*. By using this logic, only the latest available macOS upgrade will be offered to the individual client.

If present, one prior version will be kept (last known good). Older versions will be deleted before importing the new major version into the munki repo. We will update the script to support each new macOS once it is available to public. Older versions not being supported by Apple anymore (e.g., macOS 12 Monterey) will then not get deleted automatically. You will have to manually remove older items if not needed anymore in your repo.

Although supported by *mist-cli*, we do not offer beta releases. This workflow aims at production environments serving a larger number of clients with different hardware models needing to upgrade to different major versions.

*misty* is not intended for updating clients (minor updates within a major macOS version). There are other solutions available for that task.

## Running the Script

The script does not have any command line options. You specify them in the configuration files (see [Folder Structure](#folder-structure)).

### First Run: Customization

Please run the script using `sudo misty`. The script will then create a `usr/` subfolder in `/var/root/misty/`. The subfolder `misty` will be created automatically during installation. A configuration file will be created that you should customize to suit your needs. You need root privileges to access the folder `/var/root/misty/` and any subfolders or files in it. The script will also ask if you require localizations. If you do, localization templates for the relevant plist files will be copied to the `usr/` subfolder, which you should adjust to your language(s).

You can also place a script called `postinstall.sh` into your usr folder that will be executed after each new import of at least one major version. Make sure it is executable by running `chmod +x /var/root/misty/usr/postinstall.sh` in the terminal. The script will run after the misty run has finished, so you may find it useful for `sed`ing plists or altering anything else that is not covered by *misty* itself.

During the first run, a LaunchDaemon (`/Library/LaunchDaemons/de.wycomco.misty.plist`) will be enabled if not running yet. You will be asked at what time *misty* should run.

#### Config Variables

- `RepoPath`: This is the `Repo URL` configured in munki. Run `munkiimport --configure` to see what is set there if unsure.
- `Is_SMB`: Set to `yes` if your `RepoPath` lives on an `SMB` share.
- `RepoUser`: Only applicable if `Is_SMB` is set to `yes`. Must contain the munki user login name for the repo share.
- `RepoPass`: Only applicable if `Is_SMB` is set to `yes`. Must contain the munki user password for the repo share.
- `RepoName`: Only applicable if `Is_SMB` is set to `yes`. DNS name or IP address of the server holding the repo’s share. Not both.
- `munki_path`: Folder path below the `pkgsinfo` folder inside the munki repo. Can contain slashes for structured repos.
- `munki_name`: Item name for the Apple Silicon and Intel munki items. The Apple Silicon preloader is handled as described in the *Additional info* section of the [munki wiki](https://github.com/munki/munki/wiki/Staging-macOS-Installers#additional-info) and is suffixed by `_arm`.
- `deploy_name`: Set a custom name for deployment scenarios. No `force_install_after_date` is set by intention. If you plan to use this key, please set it manually.
- `munki_catalog_[major_version]`: In the first 90 days, major updates (upgrades) to a new macOS version can be omitted using MDM. To allow scoping of different staging groups, each major version can be set to a different munki catalog.
- `munki_category`: Sets the category for all resulting munki items.
- `localization`: Set to `yes` if you support different languages other than English in your munki repository.
- `time_input`: Enter the desired start time for automated runs.

A note on SMB shares: On an Almalinux Samba share, credentials were not needed (misty invoked by launchd was able to access the repo that was already mounted in user context). On a Windows SMB share, the share had to be mounted no matter if the share was already mounted in user context or not. So you may or may not have to enter `RepoUser`, `RepoPass` and `RepoName` values.

### Subsequent Runs

During each run, the values specified in `/Users/Shared/Mist/usr/config.txt` will be used. Note: When changing the start time for the LaunchDaemon in the configuration file, the value will be changed during the next run of *misty*.

Each run of *misty* compares the full versions of each major version in the `Logs/` subdirectory. If no new updates are available, the script will terminate without downloading or packaging any items.

## Folder Structure

As mentioned above, *mist-cli* is required for running this script. Its installer creates a folder `Mist` inside `/Users/Shared`. New macOS versions are downloaded to that folder and `munkiimport`ed from that location. After successful import, the installers will be deleted.

We also create a folder `misty` under `/var/root/`. Inside that folder, we have several subfolders:

- `skel/`: These are the template files. Do not edit files in here. *misty* may not work properly if the files in this directory are edited, and updates may overwrite some files too. Don’t change anything in there.
- `usr/`: This is your place to edit configuration files to suit your needs.
- `Logs/`: A changelog will be created and updated each time updates get imported. Also, there are files for each major version called `previous_state_[major_version].txt`. These are looked up by *misty* on each run. If you want to recreate installers for a major version already present in the repo, please look at the [Testing Methods](testing-methods) section below.

When run as a launchd job, the log files `/var/log/misty.log` (for stdout) and `/var/log/misty_error.log` (for stderr) will be created.

The script *misty* itself is located in `/usr/local/wycomo` and has an alias in `/usr/local/bin/`.

## Icons

The resulting payloads expect png files in the `icon` subfolder of the munki repo following the naming conventions `%munki_name%_version.png` with `%munki_name%` being the name you have specified in the `/Users/Shared/Mist/usr/config.txt` file (standard = *macos*) and `version` being the major macOS version like *monterey*, *ventura* and *sonoma*. Example: `macos_sonoma.png`.

## System Requirements

This script was tested on macOS 15 Sequoia and macOS 14 Sonoma. It should also work with earlier versions of macOS, but this has not been tested.

You should have at least 60 GB of free disk space available during the first run in order to package all three major versions.

## Prerequisites

You need to install the following dependencies:

- [mist-cli](https://github.com/ninxsoft/mist-cli)
- [munki](https://github.com/munki/munki/)

The munki repo needs to be set up by you. In a vanilla repo, you first need to `munkiimport` any item and do a `makecatalogs` to initialize the `all` catalog.

If you are running *misty* via its LaunchDaemon (advised workflow) and the munki repository is being accessed via `SMB`, you need to give full disk access (FDA) to several programs, for example *Zsh*. This is due to restrictions in macOS. You can do so by first using the *Finder*’s *Go To …* command and enter `/bin`. Scroll down to `zsh` and leave the window open. Then, go to *System Settings* => *Privacy & Security* => *Full Disk Access* and drag and drop the `zsh` binary into the *System Settings* window.

After that, also grant FDA using the same way described for `/bin/zsh` to
- `/usr/bin/hdiutil`
- `/usr/local/munki/munkiimport`
- `/usr/local/munki/Python.framework/Versions/3.12/Resources/Python.app`

If you are still facing issues, also grant `Terminal.app` (located in `/Applications/Utlities`) Full Disk Access.

## Contributing

This is a pre-release. Being said that, you are highly encouraged to test it in your environment and provide feedback using issues or pull requests.

*misty* makes use of [git-flow](https://github.com/nvie/gitflow). Branch name for production releases is *main*, branch name for "next release" development is *dev*. The version tag prefix is *v*. All other values are the default ones. You can install git-flow using [homebrew](https://brew.sh/) by entering `brew install git-flow` in the terminal.

### Testing methods

*misty* will do nothing if it thinks all is up to date. If you are up to date with all three major versions in the repo, you can do the following:

1. In the repo, rename the dmg file of the macOS installer to an earlier version. Then, rename all plist files of that version, too. Edit each plist file so the `item_installer_location` is updated to the name of the renamed dmg file. Don’t forget to also adjust the `version` string at the bottom of the files. This will ensure that the removal of older versions (at least two) will get tested.
2. Alternatively, just delete all plists and the corresponding dmg. If you followed step 1, ignore this step.
3. Do a `makecatalogs` on the repo.
4. For each major version that you altered or deleted, you need to edit the file `/Users/Shared/Mist/previous_state_[major_version].txt`. Just change the current version key to another number. You need root permissions for that.
5. Then you can run *misty* manually or wait for the LaunchDaemon to do its job.

## Known Issues

- The reload of the launchdaemon does not work if *misty* was invoked by its launchdaemon. Please run *misty* in Terminal using `sudo misty` if you have changed the start time in the config file. This will trigger a proper reload of the daemon (and do nothing else unless a new macOS version is found).

## To Do

This is a pre-release. It is working, but we have some tasks on our to-do list:

- Testing in different environments, preferably with SMB and Samba shares.
- Ensure all items that require FDA are mentioned.
- Calculate space requirements based on actual number of available macOS versions
- Function `rm_previous_files` states one previous version when no version is available
- Improve message output.
- Harmonize variable names.
- Improve code readability in general.
