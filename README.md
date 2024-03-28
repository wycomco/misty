# misty

This script checks for availability of new releases of macOS, starting from macOS Monterey, using [mist-cli](https://github.com/ninxsoft/mist-cli). It requires a [munki](https://github.com/munki/munki/) repo already being set up.

If a new update for any major version is found, it will import into munki repo, creating installers for Apple Silicon and Intel architectures. Also, a preloader for Apple Silicon will be created, as well as a plist for deploying macOS at a later time using MDM solutions. By using this logic, only the latest available macOS upgrade will be offered to the individual client.

`misty` is not intended for updating clients (minor updates within a major version). There are other solutions available for that task.

## First run: customization

The script will create a `usr/` subfolder in `/Users/Shared/Mist/`, which is already present by installing mist-cli. An override file will be created that you should customize to suite your needs. The script will also ask if you require localizations. If you do, localization templates for the relevant plist files will be copied to the `usr/` subfolder that you should also adjust to your language(s).

You can also place a script called "postinstall.sh" into your usr folder that will get executed after each run. Make sure to make it executable.

## Subsequent runs

After you have edited the files in the `usr/` subdirectory, you can start another run of `misty`. It will then download the current full installers and create the respective plist files.

The next run of `misty` compares the full versions of each major version in the `Logs/` subdirectory. If no new updates are available, the script will terminate without downloading or packaging any item. You may want to create a LaunchAgent that will run `misty` every day. It will automatically create new munki items each time a new full installer is available.

## Folder structure

As mentioned above, `mist-cli` is required for running this script, since the search for and download of full installers is done using this tool. `mist-cli` creates a folder `Mist` inside `/Users/Shared`. We are using this folder. Inside that folder, we have several subfolders:

* `skel/`: These are the template files. Do not edit files in here. `misty` may not work properly if the wrong files are edited, and updates may overwrite some files, too.
*  `usr/`: This is your place to edit files to suit your needs.
*  `Logs/`: A changelog will be created and updated each time updates get imported. Also, there are files for each major version called `previous_state_[major_version].txt`. These are looked up by `misty` on each run. If you want to create installers for a major version already present in the repo, simply remove all plists and the dmg of that version, run makecatalogs, and lower the version in the `previous_state_[major_version].txt` file. Then run `misty` again.

During the packaging process, the installer for the respective major version will be present inside the `/Users/Shared/Mist` folder. It will get deleted after all plists have been created.

The script `misty` itself is located in `/usr/local/wycomo`.

## System Requirements

This script was tested with macOS 14 Sonoma. It should work with prior macOS versions, but this is has not been tested.

You should at least have 40 GB of free disk space available during each run.

## Known issues

This is a pre-release. It is working, but we have some tasks on our to do list:

*  Automatically remove older versions of macOS from the repo. There is only one dmg that is used for all plists; however, a full installer requires space. Since we want to keep at least one version prior to the current one (testing vs production, ~~bugs~~ unwanted features in new release), we have to parse the directory for every specific major version and keep the two highest version numbers.
*  Check for space left. We need to check the space on the munki repo, but more importantly, the space on the system disk.