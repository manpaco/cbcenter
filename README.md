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
