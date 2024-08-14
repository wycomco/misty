# misty

This script checks for the availability of new macOS releases, starting from macOS Monterey, using *mist-cli* (not included in this project's installer). It also requires a *munki* repo to be already set up. Please see the [System Requirements](#system-requirements) section below for links.

## Goals of this Script

If a new update for any major version is found, it will be imported into the munki repo, creating the following installers:
- Apple Silicon: *stage_os_installer* and preloader for the installer
- Intel: *startosinstall* with precache key set to true
- Both architectures: Place installer in the `/Applications` directory

The item’s *name* keys will not include a major version. Scoping is done using *board_id* and *device_id* in the plists as *installable_condition*. By using this logic, only the latest available macOS upgrade will be offered to the individual client.

*misty* is not intended for updating clients (minor updates within a major macOS version). There are other solutions available for that task.

## Running the Script

The script does not have any command line options. You specify them in the configuration files (see [Folder Structure](#folder-structure)).

### First Run: Customization

Please run the script using `sudo misty`. The script will then create a `usr/` subfolder in `/Users/Shared/Mist/`, which should already be present after installing *mist-cli*. A configuration file will be created that you should customize to suit your needs. The script will also ask if you require localizations. If you do, localization templates for the relevant plist files will be copied to the `usr/` subfolder, which you should adjust to your language(s).

You can also place a script called `postinstall.sh` into your usr folder that will be executed after each new import of at least one major version. Make sure it is executable by running `chmod +x /Users/Shared/Mist/usr/postinstall.sh` in the terminal. The script will run after the misty run has finished, so you may find it useful for `sed`ing plists or altering anything else that is not covered by *misty* itself.

During the first run, a LaunchDaemon (`/Library/LaunchDaemons/de.wycomco.misty.plist`) will be enabled if not running yet. You will be asked at what time *misty* should run.

#### Config Variables

- `RepoPath`: This is the `Repo URL` configured in munki. Run `munkiimport --configure` to see what is set there if unsure.
- `Is_SMB`: Set to `yes` if your `RepoPath` lives on an `SMB` share.
- `RepoUser`: Only applicable if `Is_SMB` is set to `yes`. Must contain the munki user login name for the repo share.
- `RepoPass`: Only applicable if `Is_SMB` is set to `yes`. Must contain the munki user password for the repo share. A better implementation is planned, as `/Users/Shared/` is not the best place for keeping secrets.
- `RepoName`: Only applicable if `Is_SMB` is set to `yes`. DNS name or IP address of the server holding the repo’s share. Not both.
- `munki_path`: Folder path below the `pkgsinfo` folder inside the munki repo. Can contain slashes for structured repos.
- `munki_name`: Item name for the Apple Silicon and Intel munki items. The Apple Silicon preloader is handled as described in the *Additional info* section of the [munki wiki](https://github.com/munki/munki/wiki/Staging-macOS-Installers#additional-info) and is suffixed by `_arm`.
- `deploy_name`: Set a custom name for deployment scenarios. No `force_install_after_date` is set by intention. If you plan to use this key, please set it manually.
- `munki_catalog_[major_version]`: In the first 90 days, major updates (upgrades) to a new macOS version can be omitted using MDM. To allow scoping of different staging groups, each major version can be set to a different munki catalog.
- `munki_category`: Sets the category for all resulting munki items.
- `localization`: Set to `yes` if you support different languages other than English in your munki repository.

### Subsequent Runs

During each run, the values specified in `/Users/Shared/Mist/usr/config.txt` will be used. Note: When changing the start time for the LaunchDaemon in the configuration file, the value will be changed during the next run of *misty*.

Each run of *misty* compares the full versions of each major version in the `Logs/` subdirectory. If no new updates are available, the script will terminate without downloading or packaging any items.

## Folder Structure

As mentioned above, *mist-cli* is required for running this script since the search for and download of full installers are done using this tool. *mist-cli* creates a folder `Mist` inside `/Users/Shared`. We are using this folder. Inside that folder, we have several subfolders:

- `skel/`: These are the template files. Do not edit files in here. *misty* may not work properly if the files in this directory are edited, and updates may overwrite some files too. Don't change anything in there.
- `usr/`: This is your place to edit configuration files to suit your needs.
- `Logs/`: A changelog will be created and updated each time updates get imported. Also, there are files for each major version called `previous_state_[major_version].txt`. These are looked up by *misty* on each run. If you want to recreate installers for a major version already present in the repo, please look at the [Testing Methods](testing-methods) section below.

During the packaging process, the installer for the respective major version will be present inside the `/Users/Shared/Mist` folder. It is deleted after all related plists have been created.

The script *misty* itself is located in `/usr/local/wycomo` and has an alias in `/usr/local/bin/`.

## Icons

The resulting payloads expect png files in the `icon` subfolder of the munki repo following the naming conventions `%munki_name%_version.png` with `%munki_name%` being the name you have specified in the `/Users/Shared/Mist/usr/config.txt` file (standard = *macos*) and `version` being the major macOS version like *monterey*, *ventura* and *sonoma*. Example: `macos_sonoma.png`.

## System Requirements

This script was tested with macOS 14 Sonoma. It should also work with prior macOS versions, but this has not been tested.

You should have at least 60 GB of free disk space available during the first run in order to package all three major versions.

## Prerequisites

You need to install the following dependencies:

- [mist-cli](https://github.com/ninxsoft/mist-cli)
- [munki](https://github.com/munki/munki/)

The munki repo needs to be set up by you. In a vanilla repo, you first need to `munkiimport` any item and do a `makecatalogs` to initialize the `all` catalog.

If you are running *misty* via its LaunchDaemon (advised workflow) and the munki repository is being accessed via `SMB`, you need to give full disk access (FDA) to *Zsh*. This is due to restrictions in macOS. You can do so by first using the *Finder*’s *Go To …* command and enter `/bin`. Scroll down to `zsh` and leave the window open. Then, go to *System Settings* => *Privacy & Security* => *Full Disk Access* and drag and drop the `zsh` binary into the *System Settings* window. After that, also grant FDA to `/usr/bin/hdiutil` using the same way described for `/bin/zsh`.

## Testing Methods

This is a pre-release. Being said that, you are highly encouraged to test it in your environment and provide feedback using issues or pull requests.

*misty* will do nothing if it thinks all is up to date. If you really are up to date with all three major versions, you can do the following:

1. In the repo, rename the dmg file of the macOS installer to an earlier version. Then, rename all plist files of that version, too. Edit each plist file so the `item_installer_location` is updated to the name of the renamed dmg file. Don’t forget to also adjust the `version` string at the bottom of the files. This will ensure that the removal of older versions (more than two) will get tested.
2. Alternatively, just delete all plists and the corresponding dmg. If you followed step 1, ignore this step.
3. Do a `makecatalogs` on the repo.
4. For each major version that you altered or deleted, you need to edit the file `/Users/Shared/Mist/previous_state_[major_version].txt`. Just change the current version key to another number. You need root permissions for that.
5. Then you can run *misty* manually or wait for the LaunchDaemon to do its job.

## To Do

This is a pre-release. It is working, but we have some tasks on our to-do list:

- Testing in different environments, preferably with SMB shares.
- Ensure all items that require FDA are mentioned.
- Check for available space. We need to check the space on the munki repo, but more importantly, the space on the system disk. If not enough space is available, the resulting installer .app will not be complete, resulting in unusable plists and payloads being offered to clients. There exists a check with hard-coded values that stops the import process for each major version, but more testing needs to be done to ensure the values are appropriate.
- Improve message output.
- Harmonize variable names.
- Improve code readability in general.
