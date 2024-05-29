# cbcenter

Manage your favorite rooms.

WARNING: Do not distribute the content obtained with this program without the user's consent.

## Usage

Use `cbcenter help` for more info.

You need the following dependencies:

- `curl` to get previews, programs/channels (240p, 360p, 480p, ...), and do searches.

- `wget` to extract playlist (.m3u8 files) from pages.

- `sed` to manage favorites.

- `jq` to manage JSON files (server responses when searching or fetching rooms).

- `ffmpeg` (with `ffplay`) to save/play streams.

- `imv` (or another image viewer) to open previews or captures.

You can use `cbcenter __check_deps` to check the list:

![Checking dependencies](/resources/check_deps_screenshot)

## Notifications

You can use cbcenter as a basic (one user) notifier, using the `cbcenter __is_online user` command every 1 hour (or whatever you want). Something like that:

    #!/bin/bash
    #
    # cbnoti
    #

    # Environment variables used by notify-send
    export DISPLAY=:0
    export XAUTHORITY="/home/gabriel/.Xauthority" # Only on X11
    export XDG_RUNTIME_DIR="/run/user/$(id -u)"
    export DBUS_SESSION_BUS_ADDRESS="unix:path=/run/user/$(id -u)/bus"

    verifile="/tmp/notified.cb"
    user="null" # edit this

    if ! [[ -e "$verifile" ]]; then
        cbcenter __is_online "$user" && notify-send "cbcenter" "$user is online" && touch "$verifile"
    else
        ! cbcenter __is_online "$user" && rm "$verifile"
    fi

    exit 0

Then we must execute that script using cron or systemd services.

### Cron

Execute `crontab -e` (as normal user) and add the following line:

    0 * * * * /path/to/your/script

That line indicates that the script will be executed every time the clock marks 0 in the minutes (that is, when an hour is completed).

### Systemd timer

Put the following file on `$HOME/.config/systemd/user` directory:

    # cbnoti.timer
    [Unit]
    Description=Run CB check every 1 hour
    
    [Timer]
    OnCalendar=*:00:00
    # Every 30 minutes
    #OnCalendar=*:0/30
    
    [Install]
    WantedBy=default.target

Also the service:

    # cbnoti.service
    [Unit]
    Description=Check CB favorites
    
    [Service]
    ExecStart=/path/to/your/script

Finally run `systemctl --user enable --now cbnoti.timer`.

Of course you can also make a multi-user notifier (storing online user in `verifile`, and removing it when it goes offline). Something like that:

    # ----------- shebang and exports -----------
    #                    . . .
    # -------------------------------------------
    verifile="/tmp/notified.cb"
    [[ -e $verifile ]] || touch $verifile
    users="$(cbcenter __favorites_file)" # your favorites file
    [[ -e $users ]] || exit 1
    
    store_user () {
        echo "$1" >> "$verifile"
    }
    
    remove_user () {
        if match="$(grep -xn "$1" "$verifile")"; then # Exact match
            sed -i "${match%%:*}d" "$verifile"
        fi
    }
    
    notified () {
        grep -qx "$1" "$verifile"
    }
    
    while IFS="" read -r current || [ -n "$current" ]; do
        if ! notified "$current"; then
            cbcenter __is_online "$current" && notify-send "cbcenter" "$current is online" && store_user "$current"
        else
            ! cbcenter __is_online "$current" && remove_user "$current"
        fi
    done < "$users"

    exit 0

If you prefer you can add sound, but as a demonstration it's fine.
