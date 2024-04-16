# misty

This script checks for availability of new releases of macOS, starting from macOS Monterey, using [mist-cli](https://github.com/ninxsoft/mist-cli) (not included in the installert of this project). It also requires a [munki](https://github.com/munki/munki/) repo already being set up.

## Goals of this script

If a new update for any major version is found, it will import into munki repo, creating the following installers:
- Apple Silicon: *stage_os_installer* and preloader for the installer
- Intel: *startosinstall* with precache key set to true
- Both architectures: Place installer in /Applications directory

The item’s *name* keys will not include a major version. Scoping is done using *board_id* and *device_id* in the plists as *installable_condition*. By using this logic, only the latest available macOS upgrade will be offered to the individual client.

*misty* is not intended for updating clients (minor updates within a major macOS version). There are other solutions available for that task.

## Running the script

The script does not have any command line options. You specify them in the override files.

### First run: customization

Please run the script using `sudo misty`. The script will then create a `usr/` subfolder in `/Users/Shared/Mist/`, that is already present after installing mist-cli. An override file will be created that you should customize to suite your needs. The script will also ask if you require localizations. If you do, localization templates for the relevant plist files will be copied to the `usr/` subfolder that you should also adjust to your language(s).

You can also place a script called `postinstall.sh` into your usr folder that will be executed after each run. Make sure it is executable by running `chmod +x /Users/Shared/Mist/usr/postinstall.sh` in terminal.

During the first run, a LaunchDaemon (`/Library/LaunchDaemons/de.wycomco.misty.plist`) will be enabled if not running yet. You will be asked at what time *misty* should run.

### Subsequent runs

During each run, the values specified in `/Users/Shared/Mist/usr/override.txt` will be used. Note: When changing the start time for the LaunchDaemon in the override file, the value will be changed during the next run of *misty*.

Each run of *misty* compares the full versions of each major version in the `Logs/` subdirectory. If no new updates are available, the script will terminate without downloading or packaging any item.

## Folder structure

As mentioned above, *mist-cli* is required for running this script, since the search for and download of full installers is done using this tool. *mist-cli* creates a folder `Mist` inside `/Users/Shared`. We are using this folder. Inside that folder, we have several subfolders:

* `skel/`: These are the template files. Do not edit files in here. *misty* may not work properly if the files in that directory are edited, and updates may overwrite some files, too. Don't change anything in there.
*  `usr/`: This is your place to edit configuration files to suit your needs.
*  `Logs/`: A changelog will be created and updated each time updates get imported. Also, there are files for each major version called `previous_state_[major_version].txt`. These are looked up by *misty* on each run. If you want to create installers for a major version already present in the repo, simply remove all plists and the dmg of that version, run makecatalogs, and lower the version in the `previous_state_[major_version].txt` file. Then run *misty* again.

During the packaging process, the installer for the respective major version will be present inside the `/Users/Shared/Mist` folder. It is deleted after all plists have been created.

The script *misty* itself is located in `/usr/local/wycomo`.

## System Requirements

This script was tested with macOS 14 Sonoma. It should also work with prior macOS versions, but this is has not been tested.

You should at least have 60 GB of free disk space available during first run.

## To do

This is a pre-release. It is working, but we have some tasks on our to do list:

- Testing in different environments.
- Check for space available. We need to check the space on the munki repo, but more importantly, the space on the system disk. If not enough space is available, the resulting installer .app will not be complete, resulting in unusable plists and payloads being offered to clients. There exists a check with hard-coded values, but more testing needs to be done if the values are appropriate.
- Proper redirection of echo messages, depending on interactive run or launchd job. Time stamps would be enlightening, too.
- Function `rm_previous_files` can be minimized a lot.
- Harmonization of variable names.
- Readability of code in general.
