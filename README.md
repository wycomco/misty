# misty

This script checks for availability of new releases of macOS, starting from macOS Monterey, using [mist-cli](https://github.com/ninxsoft/mist-cli) (not included in the installert of this project). It also requires a [munki](https://github.com/munki/munki/) repo already being set up.

## Goals of this script

If a new update for any major version is found, it will import into munki repo, creating the following installers:
- Apple Silicon: `stage_os_installer` and preloader for the installer
- Intel: `startosinstall` with precache key set to true
- Both architectures: Place installer in /Applications directory

The item’s `name` keys will not include a major version. Scoping is done using `board_id` and `device_id` in the plists as `installable_condition`. By using this logic, only the latest available macOS upgrade will be offered to the individual client.

`misty` is not intended for updating clients (minor updates within a major macOS version). There are other solutions available for that task.

## Running the script

- The script requires root privileges.
- The script does not have any options. You need to specify them in the override files.
- Currently, after installation a restart is required. After that, you can call it using `misty` or via full path `usr/local/wycomco/misty`.

### First run: customization

The script will create a `usr/` subfolder in `/Users/Shared/Mist/`, which is already present by installing mist-cli. An override file will be created that you should customize to suite your needs. The script will also ask if you require localizations. If you do, localization templates for the relevant plist files will be copied to the `usr/` subfolder that you should also adjust to your language(s).

You can also place a script called `postinstall.sh` into your usr folder that will get executed after each run. Make sure to make it executable.

### Subsequent runs

After you have edited the files in the `usr/` subdirectory, you can start another run of `misty`. It will then download the current full installers and create the respective plist files.

The next run of `misty` compares the full versions of each major version in the `Logs/` subdirectory. If no new updates are available, the script will terminate without downloading or packaging any item. You may want to create a LaunchAgent that will run `misty` every day. It will automatically create new munki items each time a new full installer is available.

## Folder structure

As mentioned above, `mist-cli` is required for running this script, since the search for and download of full installers is done using this tool. `mist-cli` creates a folder `Mist` inside `/Users/Shared`. We are using this folder. Inside that folder, we have several subfolders:

* `skel/`: These are the template files. Do not edit files in here. `misty` may not work properly if the wrong files are edited, and updates may overwrite some files, too.
*  `usr/`: This is your place to edit files to suit your needs.
*  `Logs/`: A changelog will be created and updated each time updates get imported. Also, there are files for each major version called `previous_state_[major_version].txt`. These are looked up by `misty` on each run. If you want to create installers for a major version already present in the repo, simply remove all plists and the dmg of that version, run makecatalogs, and lower the version in the `previous_state_[major_version].txt` file. Then run `misty` again.

During the packaging process, the installer for the respective major version will be present inside the `/Users/Shared/Mist` folder. It will be deleted after all plists have been created.

The script `misty` itself is located in `/usr/local/wycomo`.

## System Requirements

This script was tested with macOS 14 Sonoma. It should work with prior macOS versions, but this is has not been tested.

You should at least have 60 GB of free disk space available during each run.

## To do

This is a pre-release. It is working, but we have some tasks on our to do list:

- Check for space left. We need to check the space on the munki repo, but more importantly, the space on the system disk. After each run involving an installation, files are written to `/private/tmp/msu-target*/` that cannot be deleted by root. If not enough space is available, the resulting installer .app will not be complete, resulting in unusable plists and payloads. More testing needs to be done.
- LaunchAgent or LaunchDaemon. If the munki repo is not a local storage, we need a LaunchAgent to ensure that the user running this script can access the repo. On the other hand, a LaunchDaemon will run also if no user is logged in.
- Add `/usr/local/wycomco` to logged in user's PATH
