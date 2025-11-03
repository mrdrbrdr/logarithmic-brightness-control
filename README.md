# Logarithmic Brightness Control

Advanced screen brightness control system with logarithmic scaling for ultra-low brightness levels on Linux/Wayland. Perfect for users who need extremely fine brightness control in dark environments while maintaining intuitive control for normal use.

## Features

- **Ultra-Low Brightness Range**: Scale down to 1/400 of maximum brightness (0.25%)
- **Logarithmic Scaling**: Fine-grained 1% increments in the 0-5% range
- **Standard Increments**: Normal 5% steps from 5-100%
- **Visual Feedback**: On-screen display (OSD) with progress bar
- **Zero Interference**: Custom OSD implementation prevents system override of brightness values

## How It Works

The script provides exponential spacing at low brightness levels for precise control in dark environments:

| Display % | Raw Value | Notes |
|-----------|-----------|-------|
| 0%        | 0         | Completely black screen |
| 1%        | 1         | Absolute minimum visible light |
| 2%        | 2         | |
| 3%        | 4         | Logarithmic progression |
| 4%        | 9         | continues to 5% |
| 5%        | 20        | (calculated from max) |
| 10%       | 40        | Standard 5% increments |
| 15%       | 60        | from here onward |
| ...       | ...       | |
| 100%      | 400       | Maximum brightness |

The logarithmic mapping in the 0-5% range enables comfortable viewing in complete darkness while avoiding the "too bright even at minimum" problem common with linear brightness control.

## Requirements

- `brightnessctl` - Backlight control utility
- `swayosd-client` - On-screen display for brightness feedback
- `hyprctl` - Hyprland compositor control (for OSD monitor detection)
- `jq` - JSON processor for parsing monitor info

### Installation on Arch Linux

```bash
sudo pacman -S brightnessctl swayosd hyprland jq
```

### Installation on Other Distributions

```bash
# Debian/Ubuntu
sudo apt install brightnessctl jq

# Fedora
sudo dnf install brightnessctl jq

# For swayosd, you may need to build from source:
# https://github.com/ErikReider/SwayOSD
```

## Installation

1. Clone this repository:
```bash
git clone https://github.com/YOUR_USERNAME/logarithmic-brightness-control.git
cd logarithmic-brightness-control
```

2. Copy the script to your local bin directory:
```bash
mkdir -p ~/.local/bin
cp brightness-control ~/.local/bin/
chmod +x ~/.local/bin/brightness-control
```

3. Ensure `~/.local/bin` is in your PATH (usually already configured):
```bash
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

4. Configure your desktop environment or window manager to bind the script to brightness keys.

### Configuration for Hyprland

Add to your `~/.config/hypr/bindings.conf`:

```conf
# Unbind default brightness controls
unbind = ,XF86MonBrightnessUp
unbind = ,XF86MonBrightnessDown

# Bind logarithmic brightness control
bindeld = ,XF86MonBrightnessUp, Logarithmic brightness up, exec, ~/.local/bin/brightness-control up
bindeld = ,XF86MonBrightnessDown, Logarithmic brightness down, exec, ~/.local/bin/brightness-control down
```

### Configuration for Other Window Managers

#### i3/sway
Add to your config:
```
bindsym XF86MonBrightnessUp exec ~/.local/bin/brightness-control up
bindsym XF86MonBrightnessDown exec ~/.local/bin/brightness-control down
```

#### KDE Plasma
You can bind the script in System Settings > Shortcuts > Custom Shortcuts

#### GNOME
Use gnome-settings or create a custom keyboard shortcut

## Usage

Once configured, simply use your keyboard's brightness keys (usually Fn+F2/F3 or similar):
- **Brightness Up Key**: Increases brightness following logarithmic scale
- **Brightness Down Key**: Decreases brightness following logarithmic scale

You can also run the script directly from the terminal:
```bash
brightness-control up    # Increase brightness
brightness-control down  # Decrease brightness
```

## Technical Details

### The swayosd Interference Problem

The original challenge was that `swayosd-client --brightness X` doesn't just display "X%" - it actively sets the hardware brightness to X% of maximum, overriding the script's carefully calculated logarithmic values.

**Solution**: The script uses `--custom-progress` instead of `--brightness` to display the OSD tooltip without triggering swayosd's brightness management system.

### Architecture

The script uses an array-based target system with declarative brightness levels:
1. Builds a ladder of target percentages: `[0, 1, 2, 3, 4, 5, 10, 15, ..., 100]`
2. Maps percentages to raw brightness values with logarithmic override for 0-5%
3. Finds the next/previous target based on current brightness
4. Applies the change using `brightnessctl`
5. Displays custom OSD with correct percentage mapping

### Customization

You can adjust the brightness curve by modifying the values in the script:

```bash
# In brightness-control, lines 32-38:
case "$p" in
    0)  raw=0 ;;   # Completely black
    1)  raw=1 ;;   # Adjust these values
    2)  raw=2 ;;   # to change the
    3)  raw=4 ;;   # logarithmic curve
    4)  raw=9 ;;   # in the 0-5% range
esac
```

You can also modify the percentage ladder on line 22:
```bash
# Current: 0,1,2,3,4,5, then every 5%
targets_pct=(0 1 2 3 4 5 10 15 20 25 30 35 40 45 50 55 60 65 70 75 80 85 90 95 100)

# Example: Add more steps in middle range
targets_pct=(0 1 2 3 4 5 10 15 20 25 30 32 34 36 38 40 ...)
```

## Use Cases

This brightness control system is perfect for:
- Late-night computing without disturbing sleep patterns
- Light-sensitive individuals who find minimum brightness too bright
- Working in complete darkness (e.g., astronomy, photography)
- Extending laptop battery life with minimal screen brightness
- Anyone who wants more precise control over screen brightness

## Troubleshooting

### Brightness keys not working
1. Check if `brightnessctl` works: `brightnessctl set 50%`
2. Verify the script is executable: `chmod +x ~/.local/bin/brightness-control`
3. Test manually: `~/.local/bin/brightness-control up`

### No OSD display
1. Ensure `swayosd` is running: `pgrep -a swayosd`
2. Start swayosd if needed: `swayosd-server &`
3. Script will work without OSD, just won't show visual feedback

### Brightness changes too slowly/quickly
Adjust the target ladder in the script to add more or fewer steps.

## Contributing

Contributions are welcome! Feel free to:
- Report bugs
- Suggest features
- Submit pull requests
- Share your custom brightness curves

## License

MIT License - see LICENSE file for details

## Credits

Developed as part of the [Omarchy](https://omarchy.org/) project configuration.

Special thanks to the developers of:
- [brightnessctl](https://github.com/Hummer12007/brightnessctl)
- [SwayOSD](https://github.com/ErikReider/SwayOSD)
- [Hyprland](https://hyprland.org/)
