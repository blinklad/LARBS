# Luke's Auto-Rice Bootstraping Scripts (LARBS)


## Installation:

On an Arch based distribution as root, run the following:

```
curl -LO larbs.xyz/larbs.sh
sh larbs.sh
```

That's it.

## What is LARBS?

LARBS is a script that autoinstalls and autoconfigures a fully-functioning
and minimal terminal-and-vim-based Arch Linux environment.

LARBS was originally intended to be run on a fresh install of Arch Linux, and
provides you with a fully configured diving-board for work or more
customization. But LARBS also works on already configured systems *and* other
Arch-based distros such as Manjaro, Antergos and Parabola (although Parabola,
which uses slightly different repositories might miss one or two minor
programs).

## Customization

By default, LARBS uses the programs [here in progs.csv](progs.csv) and installs
[my dotfiles repo (voidrice) here](https://github.com/lukesmithxyz/voidrice),
but you can easily change this by either modifying the default variables at the
beginning of the script or giving the script one of these options:

- `-r`: custom dotfiles repository (URL)
- `-p`: custom programs list/dependencies (local file or URL)
- `-a`: a custom AUR helper (must be able to install with `-S` unless you
  change the relevant line in the script

### The `progs.csv` list

LARBS will parse the given programs list and install all given programs. Note
that the programs file must be a three column `.csv`.

The first column is a "tag" that determines how the program is installed, ""
(blank) for the main repository, `A` for via the AUR or `G` if the program is a
git repository that is meant to be `make && sudo make install`ed.

The second column is the name of the program in the repository, or the link to
the git repository, and the third comment is a description (should be a verb
phrase) that describes the program. During installation, LARBS will print out
this information in a grammatical sentence. It also doubles as documentation
for people who read the csv or who want to install my dotfiles manually.

Depending on your own build, you may want to tactically order the programs in
your programs file. LARBS will install from the top to the bottom.

If you include commas in your program descriptions, be sure to include double quotes around the whole description to ensure correct parsing.

### The script itself

The script is broken up extensively into functions for easier readability and
trouble-shooting. Most everything should be self-explanatory.

The main work is done by the `installationloop` function, which iterates
through the programs file and determines based on the tag of each program,
which commands to run to install it. You can easily add new methods of
installations and tags as well.

Note that programs from the AUR can only be built by a non-root user. What
LARBS does to bypass this by default is to temporarily allow the newly created
user to use `sudo` without a password (so the user won't be prompted for a
password multiple times in installation). This is done ad-hocly, but
effectively with the `newperms` function. At the end of installation,
`newperms` removes those settings, giving the user the ability to run only
several basic sudo commands without a password (`shutdown`, `reboot`,
`pacman -Syu`).

### TODO

1. You will want to enable networking services:
```
# systemctl enable NetworkManager
# systemctl start NetworkManager
```

2. Xorg will have tearing under xcompmgr, especially noticable when scrolling
text documents. You will want to create a file for your graphics card (or
integrated graphics in the case of Intel) to remove the tearing.
Assuming vim is your text editor, and you are using Intel graphics:
```
# vim /etc/X11/xorg-conf.d/20-intel.conf

Section "Device"
	"Intel Graphics"
	"intel"
	"TearFree" "true"
EndSection
```
3. If you wish to adjust your backlight settings using eg ```xbacklight```, add
an additional line under the "Device" section:
```
"Backlight" "intel_backlight"
```

4. If you want to set up eduroam for UTas, run the included script (it should be
under your $PATH). Make sure to use your _full_ username, including email:
```
$ eduroam-linux-UoT.py
```

5. Nextly, if you want to use the provided VPN, you will need to download the
networkmanager-vpnc plugin _and_ nm-applet. Technically you won't need the applet,
but vanilla nmtui does not contain checks for the vpnc options, and it's slightly
easier to wrestle with nmgui to get things set up before enabling automation through
nmcli.
networkmanager-vpnc (sp.?) should be on the AUR:
```
$ yay -Syu networkmanager-vpnc
```
Pre-shared keys / group names would be silly to list here, and change often enough to not write them down.
Ask your system administrator or someone else who has this setup for them.

6. ```pacman``` mirror lists often need updating by most recent synchronization status.
The ```reflector``` package contains a script to run a hook to update the mirror list
after a successful transaction; it just needs to be enabled.
An easy way is to update the list once a week.
```
# nvim /etc/systemd/system/reflector.service

[Unit]
Description=Pacman mirrorlist update
Wants=network-online.target
After=network-online.target

[Service]
Type=oneshot
ExecStart=/usr/bin/reflector --protocol https --latest 30 --number 20 --sort rate --save /etc/pacman.d/mirrorlist

[Install]
RequiredBy=multi-user.target

# systemctl enable reflector.service
# systemctl start reflector.service
```
