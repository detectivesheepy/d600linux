# d600Linux | Linux support for Kogan Atlas D600

This guide documents the fix that worked on a Kogan Atlas 10.1" D600
running CachyOS (MATE/X11), adapted for Debian. It may also help other
Intel Gemini Lake devices with an inverted cursor and incorrect pointer
coordinates.

## Notes

- Touchscreen support is yet to be fixed as there is no documentation online about the touchscreen's manufacturer.
- This guide is about Debian! Other distros *might* not work but could possibly, I haven't tried them.

## Symptoms

-   The display is rotated upright using the desktop display settings.
-   The mouse pointer icon itself appears upside down.
-   Pointer movement and hover locations are inverted or misaligned.
-   Rotating the display again is undesirable because the display
    orientation is already correct.

## 1. Confirm you're using X11

This fix uses Xorg configuration and `xinput`, so make sure you're
logged into an X11 session rather than Wayland.

Check:

``` bash
echo "$XDG_SESSION_TYPE"
```

Expected: `x11`.

If it says `wayland`, this guide's Xorg steps won't apply as written.

## 2. Install the required tools

``` bash
sudo apt update
sudo apt install xinput xserver-xorg
```

## 3. Identify the mouse

``` bash
xinput list --short
```

Find the physical mouse or touchpad in the list and note its ID. On my
original D600 setup, the mouse was listed as:

`SIGMACHIP USB Keyboard Mouse`

with ID `10`.

**Use the ID shown on your own system.** Device IDs can change after
reboot or when devices are reconnected.

## 4. Correct the mouse mapping

For the D600's mouse, a 180-degree coordinate transformation corrected
pointer movement:

``` bash
xinput set-prop 10 "Coordinate Transformation Matrix" -1 0 1 0 -1 1 0 0 1
```

Replace `10` with your mouse's actual ID.

This is a temporary setting. It usually resets when you log out or
reboot.

## 5. Force software cursor rendering (the main fix)

The D600 uses Intel Gemini Lake graphics with the `i915` driver. Forcing
software cursor rendering fixed the upside-down cursor icon and the
hover-coordinate mismatch.

Create an Xorg configuration file:

``` bash
sudo mkdir -p /etc/X11/xorg.conf.d
sudo nano /etc/X11/xorg.conf.d/20-intel-cursor.conf
```

Put the following in the file:

``` conf
Section "Device"
    Identifier "Intel Graphics"
    Driver "modesetting"
    Option "SWCursor" "true"
EndSection
```

Save in nano: - `Ctrl + O`, then Enter - `Ctrl + X`

## 6. Reboot and test

Save any open work, then:

``` bash
sudo reboot
```

After reboot, check: - Cursor icon is upright. - Mouse movement is
correct. - Hover highlights match the actual cursor location. - Display
orientation remains unchanged.

The software-cursor setting is persistent because it is stored in
`/etc/X11/xorg.conf.d/`.

## 7. Make the mouse mapping persistent (optional)

The `xinput` command from Step 4 is temporary. To reapply it
automatically at login, use MATE's Startup Applications:

1.  Open **Startup Applications**.
2.  Add a new startup program.
3.  Name it `Mouse coordinate fix`.
4.  Command (replace `10` with the current mouse ID):

``` bash
sh -c 'xinput set-prop 10 "Coordinate Transformation Matrix" -1 0 1 0 -1 1 0 0 1'
```

This runs after login. If the mouse ID changes, update the command.

## Undo the software cursor setting

If the change causes problems, remove the config file:

``` bash
sudo rm /etc/X11/xorg.conf.d/20-intel-cursor.conf
```

Then reboot.

## Important

-   Do **not** rotate the display with `xrandr` as part of this fix. The
    original D600 display orientation was set correctly through MATE
    Display Settings, with the rotation set to `right`.
-   Do not apply the coordinate matrix to the virtual XTEST pointer.
    Apply it to the actual mouse device.
-   If the physical mouse is not ID `10`, use its actual ID.
-   The exact mouse mapping may differ on other devices. The matrix
    above was the one that restored correct movement on the original
    D600 setup.
-   The successful software-cursor fix was tested on CachyOS/MATE/X11.
    Debian uses the same Xorg configuration location, but different
    graphics/session setups may need adjustments.
