# Logarithmic Brightness Control

This is my small contribution to the [Omarchy](https://omarchy.org/) community. 

Advanced screen brightness control for Omarchy with logarithmic scaling for ultra-low brightness levels. 

## Features

- **Logarithmic Scaling**: Fine 1% steps in the 0-5% range
- **Standard Steps**: 5% increments from 5-100%

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

The logarithmic mapping enables comfortable viewing in complete darkness while avoiding the "too bright even at minimum" problem.

## Requirements

All dependencies are pre-installed in Omarchy:
- `brightnessctl`
- `swayosd-client`
- `hyprctl`
- `jq`

## Installation

```bash
# Clone and install
git clone https://github.com/mrdrbrdr/logarithmic-brightness-control.git
cd logarithmic-brightness-control
mkdir -p ~/.local/bin
cp brightness-control ~/.local/bin/
chmod +x ~/.local/bin/brightness-control
```

Add to `~/.config/hypr/bindings.conf`:

```conf
unbind = ,XF86MonBrightnessUp
unbind = ,XF86MonBrightnessDown
bindeld = ,XF86MonBrightnessUp, Logarithmic brightness up, exec, ~/.local/bin/brightness-control up
bindeld = ,XF86MonBrightnessDown, Logarithmic brightness down, exec, ~/.local/bin/brightness-control down
```

## How It Works

**The swayosd Problem**: `swayosd-client --brightness X` actively sets hardware brightness to X%, overriding logarithmic values.

**Solution**: Use `--custom-progress` to display OSD without triggering swayosd's brightness management.

**Architecture**:
1. Array of target percentages: `[0, 1, 2, 3, 4, 5, 10, 15, ..., 100]`
2. Logarithmic mapping for 0-5% range (0→0, 1→1, 2→2, 3→4, 4→9, 5→20)
3. Find next/previous target based on current brightness
4. Apply with `brightnessctl`
5. Display custom OSD with mapped percentage
