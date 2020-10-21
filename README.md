# munkicon
A collection of conditional items scripts for munki.

## DISCLAIMER
This is experimental. Please test that they work for you in your environment.

## Building a package
After cloning, run `./build.sh`, the resulting package is generated in `./dist/pkg/`.

## Munki Python Symlink
The `postinstall` script re-writes the shebang in the processor files if `/usr/local/munki-python` is found.

## Disable processors
Delete any processor you don't want to have run. An installer pkg that has a choices XML capability may be built in the future.

## Binaries
All binaries called by the processors exist in macOS 10.15.7+. No intention to make these backwards compatible with older macOS versions.

## Issues
Raise an issue here https://github.com/carlashley/munkicon/issues.
To get any errors that might end up in `stdout`/`stderr`, manually execute the processors and copy+paste any output into the issue:
```
for i in /usr/local/munki/conditions/com.github.carlashley.munkicon.*.py; do $i; done
```

### com.github.carlashley.munkicon.filevault.py
Usefule in determining various states/results pertaining to FileVault.
- Generates eight conditions:
- - `filevault_active` is a boolean `True`/`False` based on FileVault being active or inactive. This differs from `filevault_status`. Use this to determine if the disk has been encrypted and therefore FileVault is on.
- - `filevault_decryption_in_progress` is a boolean `True`/`False` if decryption is in progress (`True`), or not in progress (`False`).
- - `filevault_deferral` returns `not_found` if there are no defferals, but will return `active` if deferrals are found.
- - `filevault_encryption_in_progress` is a boolean `True`/`False` if encryption is in progress (`True`), or not in progress (`False`).
- - `filevault_institution_key` is a boolean `True`/`False` based on whether an institutional recovery key is in use.
- - `filevault_personal_key` is a boolean `True`/`False` based on whether an personal recovery key is in use.
- - `filevault_status` returns a string of either `on` or `off`. This differes from `filevault_active`. Use this to determine if FileVault has been turned on or off, and the disk is in the process of being encrypted or decrypted.
- - `filevault_users` returns an array of usernames. The UUID for the user is not returned as this is not necessarily predictable.
- Usage (on their own or combine):
- - `filevault_active == TRUE`
- - `filevault_decryption_in_progress == TRUE`
- - `filevault_deferral == 'active'`
- - `filevault_encryption_in_progress == FALSE`
- - `filevault_institution_key == TRUE`
- - `filevault_personal_key == FALSE`
- - `filevault_status == 'on'`
- - `ANY filevault_users == 'jappleseed'`

### com.github.carlashley.munkicon.kext.py
Useful in determining if a package in a manifest should be made available (or not) based on if the KEXT is whitelisted. For example, an anti-virus application might run in a heavily crippled state if KEXTs are not whitelisted before installation.
- Generates three conditions:
- - `kext_bundles` contains an array of Bundle ID's for KEXTs which are whitelisted (user and/or MDM).
- - - Example: `'com.vmware.kext.vmioplug.18.1.2'`
- - `kext_teams` contains an array of Team ID's for KEXTs which have been whitelisted (user and/or MDM).
- - - Example: `'EG7KH642X6'`
- - `kext_team_bundle` contains an array of Team ID's and Bundle ID's (as a comma seperated string) for any whitelisted KEXTs (user and/or MDM) which have _both_ the Team ID and Bundle ID present.
- - - Example: `'EG7KH642X6,com.vmware.kext.vmioplug.18.1.2'`
- Usage (on their own or combine):
- - `ANY kext_bundles == 'com.vmware.kext.vmioplug.18.1.2'`
- - `ANY kext_teams == 'EG7KH642X6'`
- - `ANY kext_team_bundle == 'EG7KH642X6,com.vmware.kext.vmioplug.18.1.2'`

### com.github.carlashley.munkicon.mdm-enrolled.py
Useful in determining if a package in a manifest should be made available (or not) if a client is or is not enrolled in an MDM, For example, a particular set of apps might only need to be deployed to MDM supervised devices.
- Generates two conditions:
- - `enrolled_via_dep` will be `yes` or `no`.
- - `mdm_enrollment` will be `yes_user_approved` or `no`.
- Usage (on their own or combine):
- - `enrolled_via_dep == 'yes'`
- - `mdm_enrollment == 'yes_user_approved'`

### com.github.carlashley.munkicon.pppcp.py
Useful in determining if a package in a manifest should be made available based on whether any MDM deployed PPPCP payloads exist for a specifc Bundle ID or path. For example, the user experience for a particular app might be cumbersome if it is installed before a PPPCP payload is pushed to the client via MDM.

Each condition generates an array of strings that indicate if the PPPCP payload is `allow`, `deny`, or `allow_user`. The format returned is either:
- `allow,org.example.foo` or `allow,/Applications/Example.app`
- `deny,org.example.foo` or `deny,/Applications/Example.app`
- `allow_user,org.example.foo` this is only for `ListenEvent` and `ScreenCapture`, macOS 11+ and is the equivalent of the macOS 11+ `AllowStandardUserToSetSystemService` key.

The `tcc_apple_events` condition additionally returns the `AEReceiverIdentifier` if it exists. For example:
```
allow,org.example.foo,/Applications/Microsoft Remote Desktop.app
```
- Generates twenty-one conditions:
- - `tcc_accessibility` 
- - `tcc_address_book`
- - `tcc_apple_events`
- - `tcc_calendar`
- - `tcc_camera`
- - `tcc_file_provider_presence`
- - `tcc_listen_event`
- - `tcc_media_library`
- - `tcc_microphone`
- - `tcc_photos`
- - `tcc_post_event`
- - `tcc_reminders`
- - `tcc_screen_capture`
- - `tcc_speech_recognition`
- - `tcc_all_files`
- - `tcc_desktop_folder`
- - `tcc_documents_folder`
- - `tcc_downloads_folder`
- - `tcc_network_volumes`
- - `tcc_removable_volumes`
- - `tcc_sys_admin_files`
- Usage (on their own or combine):
- - `ANY tcc_all_files == 'allow,com.apple.Terminal'`
- - `ANY tcc_screen_capture == 'allow_user,us.zoom.xos'`
- - `ANY tcc_desktop_folder == 'deny,us.zoom.xos'`
- - `ANY tcc_apple_events == 'allow,net.pulsesecure.Pulse-Secure,net.pulsesecure.Pulse-Secure'`

### com.github.carlashley.munkicon.python.py
Useful in determing basic version information about various Python versions.
- Generates six conditions:
- - `mac_os_python_path` is the real path (symlinks followed) of the Python that ships with macOS.
- - `mac_os_python_ver` is the version string of the Python that ships with macOS.
- - `munki_python_path`is the real path (symlinks followed) of the Python used by munki.
- - `munki_python_ver` is the version string of the Python used by munki.
- - `official_python3_path` is the real path (symlinks followed) of the offical Python installation.
- - `official_python3_ver` is the version string of the official Python installation.
- Usage (on their own or combine):
- - `mac_os_python_path == '/usr/bin/python'`
- - `mac_os_python_ver BEGINSWITH '2.7'`
- - `munki_python_path CONTAINS 'Framework'`
- - `munki_python_ver >= 3.7.4`
- - `official_python3_path LIKE '/Library/Frameworks/Python.framework/Versions/3.*/bin/python3.*'`
- - `official_python3_ver LIKE '3.7.*'`

### com.github.carlashley.munkicon.system-extensions.py
Useful in determining if a package in a manifest should be made available (or not) based on if a System Extension is whitelisted. For example, a VPN might not run if its System Extensions are not whitelisted before installation.
- Generates four conditions:
- - `sys_ext_bundles` contains an array of Bundle ID's for System Extensions which are whitelisted (user and/or MDM).
- - - Example: `'com.microsoft.wdav.netext'`
- - `sys_ext_teams` contains an array of Team ID's for System Extensions which have been whitelisted (user and/or MDM).
- - - Example: `'UBF8T346G9'`
- - `sys_ext_team_bundle` contains an array of Team ID's and Bundle ID's (as a comma seperated string) for any whitelisted System Extensions (user and/or MDM) which have _both_ the Team ID and Bundle ID present.
- - - Example: `'UBF8T346G9,com.microsoft.wdav.netext'`
- - `sys_ext_types` contains an array of Team ID's and Extension Types (`DriverExtension`, `NetworkExtension`, `EndpointSecurityExtension`) that have been whitelisted (MDM).
- Usage (on their own or combine):
- - `ANY sys_ext_bundles == 'com.microsoft.wdav.netext'`
- - `ANY sys_ext_teams == 'UBF8T346G9'`
- - `ANY sys_ext_team_bundle == 'UBF8T346G9,com.microsoft.wdav.netext'`
- - `ANY sys_ext_types == 'UBF8T346G9,EndpointSecurityExtension'`

### com.github.carlashley.munkicon.system-setup.py
Useful for obtaining various bits of system setup information.
- Generates 11 conditions:
- - `ard_enabled` is either `True` or `False`. Relies on `/usr/libexec/mdmclient` existing.
- - `cups_web_interface_enabled` is either `True` or `False`.
- - `efi_password_enabled` is either `True` or `False`. Relies on `/usr/libexec/mdmclient` existing.
- - `ntp_enabled` is either `True` or `False`.
- - `ntp_servers` returns an array of all servers found in `/etc/ntp.conf`.
- - `printer_sharing_enabled` is either `True` or `False`.
- - `remote_apple_events_enabled` is either `True` or `False`.
- - `sip_enabled` is either `True` or `False`.
- - `ssh_enabled` is either `True` or `False`.
- - `timezone` returns the timezone as a string in the format `"Australia/Melbourne"`.
- - `wake_on_lan` is either `True` or `False`.
- Usage (on their own or combine):
- - `ard_enabled == TRUE`
- - `cups_web_interface_enabled == FALSE`
- - `efi_password_enabled == TRUE`
- - `ntp_enabled == TRUE`
- - `ANY ntp_servers == 'time.apple.asia.com'`
- - `printer_sharing_enabled == FALSE`
- - `remote_apple_events_enabled == FALSE`
- - `sip_enabled == TRUE`
- - `ssh_enabled == TRUE`
- - `timezone "Australia/Melbourne"`
- - `wake_on_lan == FALSE`

### com.github.carlashley.munkicon.user-accounts.py
Useful in determining if a package in a manifest should be made available based on whether a local user account exists on a client. For example, customised profile settings for a local user should only be installed if that user exists.
- Generates one condition:
- - `user_home_path` that contains an array of username and home path locations (as a comma seperated string) for _local_ accounts only (ignoring all inbuilt accounts except for `root`). This combination is used as home paths do not necessarily have the username forming part of the path.
- Usage (on their own or combine):
- - `ANY user_home_path == 'administrator,/Users/admin'`
