This repo contains scripts for Artix to start pipewire, pipewire-pulse and wireplumber from openrc.
This is just for my own user and I am not going to support it. But you are free to fork it to make it better.

EDIT: Thank you very much for the scripts! I found an issue regarding KDE (SDDM) enviroments that caused instability (pipewire cannot start before the login manager starts) and since open-rc starts processes in parallel, you can see where this is going. The changes are very simple, add the `sddm` service as a dependency to the `pipewire` service, and that should make everything work smoothly.

If you are encountering similar issues, try adding your login manager (SDDM, LXDM, GDM, etc) as a dependency and that should make it work flawlessly.

**Prerequisites**

Have the following packages installed
```
pipewire
pipewire-pulse
wireplumber
```
NOTE: Out of these, you should have `pipewire` and `wireplumber` already installed. Install `pipewire-pulse`; If it complains about pulseaudio packages, remove those.
**Mechanism**

After login, PAM would change the runlevel to user runlevel by UID.

It will await the session to be started and export `DBUS_SESSION_BUS_ADDRESS` to a file. (timeout is 30s). runlevel switching would be completed after the dbus session file is found so that the pipewire service can retrieve `DBUS_SESSION_BUS_ADDRESS` to work.


**File Structure**
NOTE: All files except user-dbus.desktop should be executable.

`/etc/init.d/pipewire`

`/etc/init.d/pipewire-pulse`

`/etc/init.d/wireplumber`

`/usr/local/bin/await-user-dbus`

`/usr/local/bin/user-dbus`

`/usr/local/bin/user-service-start`

`~/.config/autostart/user-dbus.desktop`

**Setup**
*create runlevel by user ID (e.g. default UID is 1000, can check it by `$ echo $UID`)*

`# mkdir /etc/runlevels/1000`

*Stack the run level*

`# rc-update -s add default 1000`

*Add service to user runlevel*

`# rc-update add pipewire 1000`

`# rc-update add pipewire-pulse 1000`

`# rc-update add wireplumber 1000`

*Append PAM to change runlevel after login*

`# echo "session   optional  pam_exec.so /usr/local/bin/user-service-start" >> /etc/pam.d/system-local-login`

NOTE: You cannot echo into this file with a `sudo` command, you have to be `root` in order to do this command.
sample:

```
#%PAM-1.0

auth      include   system-login
account   include   system-login
password  include   system-login
session   include   system-login
session   optional  pam_exec.so /usr/local/bin/user-service-start
```
