## Troubleshooting Bluetooth

For the totem layout we have bluetooth channel select buttons mapped to each of the outer pinky bonus keys.

The left for channel 0, currently linked to the personal laptop, and the right for channel 1, currently linked to the work laptop.

Activate the utility layer by pressing both thumb layer keys at the same time and then press one of the pinky buttons to select the host.

If the connection isn't working you may need to do the following:

1. On the host, go to bluetooth settings and forget the Totem
2. On the keyboard, go select the bluetooth channel for the computer you want to disconnect from
3. Activate the utility layer as described earlier and then press the leftmost thumb key to clear the bt connection
4. Reconnect the Totem on the host in bluetooth settings

## Setup Local Builds

[Setup Instructions](https://zmk.dev/docs/development/local-toolchain/setup/container?container=podman)

## Dependencies

Install `podman` cli: https://podman.io/docs/installation

## Configuration

Clone `zmk` repo:

```sh
git clone https://github.com/zmkfirmware/zmk.git
```

### Create Volumes

```sh
podman volume create --driver local -o o=bind -o type=none \
 -o device="/Users/robertreed/dev/keyboards/zmk/keyboards/totem/zmk-config-totem" zmk-config
```

If you are using modules, you would need to create a volume for them too.

### Initialize the Container

```sh
podman run -it --rm \
  --security-opt label=disable \
  --workdir /workspaces/zmk \
  -v /Users/robertreed/dev/keyboards/zmk/zmk:/workspaces/zmk \
  -v /Users/robertreed/dev/keyboards/zmk/keyboards/totem/zmk-config-totem:/workspaces/zmk-config \   # Removeable
  -p 3000:3000 \
  <container-name> /bin/bash
```

If you are using modules, you would need to mount that directory too.

### Configure Zephyr Workspace

From within the container, make sure you are in `/workspaces/zmk` and then run the following:

```sh
west init -l app/ # Initialization
west update       # Update modules
```

## Building

From within the container, `cd` into the `app` directory within `/workspaces/zmk`:

```sh
cd app
```

Build the shield one half at a time:

```sh
west build -d build/totem/left -b seeeduino_xiao_ble -- -DSHIELD=totem_left -DZMK_CONFIG=/workspaces/zmk-config/config -DSHIELD_DIR=/workspaces/zmk-config/boards/shields
```

Let's break this down:

- `-d build/totem/left` we are specifying the output directory, which needs to have `left/right` as each firmware will be simply named `zmk.uf2`
- `-b seeeduino_xiao_ble` we are specifying that the actual MCU is a `seeeduino_xiao_ble` board
- `--` anything after the `--` will be passed directly to the `CMake` command
- `-DSHIELD=totem_left` the shield name needs to match one of the `siblings` in `zmk-config-totem/config/boards/shields/totem/totem.zmk.yml`
- `-DZMK_CONFIG=/workspaces/zmk-config/config` we are saying that we should use the keymap and settings in that `config` dir, rather than the defaults for the shield
- `-DSHIELD_DIR=/workspaces/zmk-config/boards/shields` since the `totem` isn't integrated into `zmk` we have to tell it where to find the shield definition
- `-p` we didn't specify it, but pass `-p` after `build` in order to generate a `--pristine` build, not using cached artifacts

For follow on builds, you can use the build directory as a shortcut and it will use all of the same settings, but use the new artifacts.

For example, to rebuild the above you would run:

```sh
west build -d build/totem/left
```

Then to build the right half:

```sh
west build -d build/totem/right -b seeeduino_xiao_ble -- -DSHIELD=totem_right -DZMK_CONFIG=/workspaces/zmk-config/config -DSHIELD_DIR=/workspaces/zmk-config/boards/shields
```

And then afterwards:

```sh
west build -d build/totem/right
```

## Flashing

Now that you have a build, you need to flash it to the board. Make sure you flash the correct left/right half.

The build file will be found as `/zephyr/zmk.uf2` within the build directory you specified.

For example, `/workspaces/zmk/app/build/totem/left/zephyr/zmk.uf2`.

You should be able to find the file within the `zmk` folder on your computer as well as within podman.
