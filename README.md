# LibreCoach Dashboard

A modern, responsive, and touch-friendly Home Assistant dashboard designed for RVs. It uses a combination of built-in HA cards and community cards from HACS to provide granular light control, shade and lock management, climate control, energy monitoring, and tank level tracking — all from a single dashboard with a navigation bar on every view.

---

## Dashboard Views

This dashboard has 10 views. Each view has a navigation bar at the bottom (using HA badges) to jump between sections.

| View         | Path        | What it does                                                   | Cards used                         |
| :----------- | :---------- | :------------------------------------------------------------- | :--------------------------------- |
| **Lights**   | `/lights`   | Dimmer sliders for every light by room                         | Mushroom Light Card                |
| **Lighting** | `/lighting` | Quick on/off panels grouped by room                            | Button Card + Card Mod             |
| **Shades**   | `/shades`   | Night and day shade controls by location                       | Mushroom Cover Card                |
| **Doors**    | `/doors`    | Open/close controls for powered doors                          | Built-in Button                    |
| **Locks**    | `/locks`    | Lock/unlock controls                                           | Built-in Button                    |
| **Heat**     | `/heat`     | Aqua-Hot burner switches, floor heat thermostats, temp sensors | Mushroom Light Card, Thermostat    |
| **AC**       | `/ac`       | Micro-Air EasyTouch thermostat zones                           | Built-in Thermostat                |
| **Energy**   | `/energy`   | Battery state of charge, 120V and 12V power flow diagrams      | Gauge, Power Flow Card Plus        |
| **Tanks**    | `/tanks`    | Water pump, autofill switch, fresh and black tank levels       | Mushroom Entity Card, Gauge        |
| **Misc**     | `/misc`     | Location, time, weather, NWS alerts                            | Markdown, Sensor, Weather Forecast |

---

## Prerequisites & Installation

### 1. Install HACS

**HACS (Home Assistant Community Store)** is required to install the custom cards used in this dashboard.

- [Follow the official HACS installation guide.](https://hacs.xyz/docs/setup/download)

> **Note:** You will need a GitHub account to set up HACS, as it downloads files directly from GitHub repositories.

### 2. Install Required Cards

Once HACS is installed, go to **HACS** in the sidebar. For each card below, use the **Search** box and click **Download**.

| Card                                                                        | Where It's Used                       | Why it's needed                                                                                                       |
| :-------------------------------------------------------------------------- | :------------------------------------ | :-------------------------------------------------------------------------------------------------------------------- |
| **[Mushroom](https://github.com/piitaya/lovelace-mushroom)**                | Lights, Shades, Heat, Tanks, AC views | Provides the rounded, touch-friendly cards for lights, covers, thermostats, and entities.                             |
| **[Button Card](https://github.com/custom-cards/button-card)**              | Lighting view                         | Creates the square, icon-free on/off tiles in the room panels. Uses a shared template so all buttons look consistent. |
| **[Card Mod](https://github.com/thomasloven/lovelace-card-mod)**            | Lighting view                         | **Required.** Applies the CSS glass/blur styling that makes each room's buttons appear as a distinct floating panel.  |
| **[Power Flow Card Plus](https://github.com/flixlix/power-flow-card-plus)** | Energy view                           | Visualizes energy flow between shore power, inverter, batteries, and loads in a live animated diagram.                |

After installing all four, **reload your browser** before proceeding.

### 3. Create the Dashboard and Paste the YAML

> **Important:** The navigation links inside the YAML are hard-coded to `/dashboard-librecoach/...`. You must use `librecoach` as the URL when creating the dashboard or all navigation badges will be broken.

1. Go to **Settings** → **Dashboards**.
2. Click **Add Dashboard** (bottom right).
3. Select **New dashboard from scratch**.
4. Set the **Name** (e.g., `LibreCoach`) and set the **URL** to exactly `librecoach`.
5. Click **Create**.
6. Click on your new dashboard to open it.
7. Click the **pencil icon** (top right) to enter edit mode.
8. Click the **three-dot menu** (⋮, top right) → **Raw configuration editor**.
9. **Select all** the existing text in the editor and **delete it**.
10. Paste the entire contents of `dashboard.yml` from this repository.
11. Click **Save**.

---

## Understanding the Code

The YAML file has two top-level sections:

```yaml
button_card_templates:   # Reusable button styles defined once at the top
  panel_light: ...

views:                   # The list of dashboard pages (Lights, Shades, etc.)
  - title: Lights
    ...
```

### Card Types at a Glance

| Pattern                       | HACS Card            | Used In                                  |
| :---------------------------- | :------------------- | :--------------------------------------- |
| `custom:mushroom-light-card`  | Mushroom             | Lights, Heat views                       |
| `custom:mushroom-cover-card`  | Mushroom             | Shades view                              |
| `custom:mushroom-entity-card` | Mushroom             | Tanks view (pump/autofill switches)      |
| `custom:mushroom-title-card`  | Mushroom             | Section headers in all views             |
| `custom:button-card`          | Button Card          | Lighting view only                       |
| `custom:mod-card`             | Card Mod             | Lighting view only (the floating panels) |
| `custom:power-flow-card-plus` | Power Flow Card Plus | Energy view                              |
| `type: gauge`                 | Built-in HA          | Energy and Tanks views                   |
| `type: thermostat`            | Built-in HA          | Heat and AC views                        |

---

### 1. Button Card Template (`button_card_templates`)

At the very top of the YAML, we define one reusable button style called `panel_light`. This is like a blueprint — instead of writing the same 25 lines of styling for every single light switch, we define it once here and reference it by name later.

```yaml
# This block lives at the top of the file, before "views:"
button_card_templates:
  panel_light:
    # "name_state" layout shows the entity name and its state (on/off)
    layout: name_state

    # 1/1 makes the button a perfect square
    aspect_ratio: 1/1

    # We don't want an icon — just the name text
    show_icon: false
    show_name: true

    styles:
      card:
        # Use HA's built-in color variables so the card adapts to dark/light mode
        - background-color: var(--ha-card-background, var(--card-background-color))
        - border-radius: 12px
        - border: 1px solid var(--divider-color)
        - margin: 1px
        # Cap the button size so they don't grow too large on tablet screens
        - max-height: 82px
        - max-width: 82px
        - box-shadow: 0 2px 4px rgba(0,0,0,0.15)
        # Smooth animation when state changes
        - transition: all 0.15s ease-in-out
      name:
        - font-size: 18px
        - font-weight: 500
        - text-align: center
        - white-space: normal # Allow the name to wrap to a second line
        - word-break: break-word
        - line-height: 1.2

    state:
      # --- STYLE WHEN THE LIGHT IS ON ---
      - value: "on"
        styles:
          card:
            - background-color: rgba(0, 95, 204, 0.25) # Blue tint
            - box-shadow: 0px 0px 8px rgba(0, 95, 204, 0.4) # Blue glow
            - transform: scale(1.05) # Slightly larger when on
          name:
            - color: "#005fcc" # Blue text when on
            - font-weight: 600

      # --- STYLE WHEN THE LIGHT IS OFF ---
      - value: "off"
        styles:
          card:
            - background-color: rgba(255, 59, 59, 0.2) # Red tint when off
            - border: 1px solid rgba(255, 59, 59, 0.3)
          name:
            - color: "#ff3b3b" # Red text when off
```

---

### 2. Mushroom Light Cards (Lights View)

The **Lights** view uses `mushroom-light-card` which gives you a brightness slider directly on the card. Each room is a `vertical-stack` with a title followed by a `grid` of light cards.

```yaml
# A section for one room — the same pattern repeats for every room
- type: vertical-stack
  cards:
    # The room title (from Mushroom)
    - type: custom:mushroom-title-card
      title: Living Room
      alignment: center

    # A grid of light cards, 2 per row
    - type: grid
      columns: 2
      square: false
      cards:
        - type: custom:mushroom-light-card
          entity: light.switch_6 # Replace with your entity ID
          name: Ceiling # Override the display name
          secondary_info: none # Hides the secondary info line for a cleaner look
          show_brightness_control: true # Shows the brightness slider

        - type: custom:mushroom-light-card
          entity: light.switch_9
          secondary_info: none
          show_brightness_control: true
          # If "name" is omitted, the entity's friendly name is used
```

---

### 3. The Floating Lighting Panels (Card Mod + Button Card)

The **Lighting** view is different from the Lights view. Instead of dimmer sliders, it shows compact square on/off buttons grouped into "floating panels" — one panel per room. This is built with two HACS cards working together.

**How it works:**

- `custom:mod-card` (Card Mod) wraps a group of buttons and applies CSS to give them the glass panel background
- Inside the mod-card, a `vertical-stack` holds a title and a grid of `custom:button-card` tiles
- Each `button-card` uses `template: panel_light` — the template we defined at the top of the file

```yaml
# Card Mod creates the styled container (the floating glass panel)
- type: custom:mod-card
  card_mod:
    style: |
      ha-card {
        border: 1px solid var(--divider-color);
        border-radius: 12px;
        padding: 8px;
        height: fit-content;

        /* Dark mode: semi-transparent dark background with a blur behind it */
        background: rgba(20, 20, 20, 0.85);
        backdrop-filter: blur(10px);
      }

      /* Light mode override: switch to a white, shadowed look */
      @media (prefers-color-scheme: light) {
        ha-card {
          background: rgba(255, 255, 255, 0.9) !important;
          border: 1px solid rgba(0, 0, 0, 0.1);
          box-shadow: 0 4px 10px rgba(0, 0, 0, 0.05);
        }
      }

  # Everything inside "card:" goes inside the styled container
  card:
    type: vertical-stack
    cards:
      # The room label at the top of the panel (from Mushroom)
      - type: custom:mushroom-title-card
        title: Entry
        alignment: center

      # A 4-column grid of square on/off buttons
      - type: grid
        columns: 4
        square: false
        cards:
          # Each button uses the "panel_light" template defined at the top of the file.
          # You only need to specify the entity — all styling comes from the template.
          - type: custom:button-card
            template: panel_light
            entity: light.switch_4 # Replace with your entity ID

          - type: custom:button-card
            template: panel_light
            entity: light.switch_3

          # You can also override the tap action if needed (e.g., to call a script)
          - type: custom:button-card
            template: panel_light
            entity: light.accents
            tap_action:
              action: perform-action
              perform_action: script.accents_smart_toggle
```

---

### 4. Mushroom Cover Cards (Shades View)

The **Shades** view uses `mushroom-cover-card` for window shades. The view is laid out in two columns — one for night shades, one for day shades — so you can control both types side by side.

```yaml
# Two columns side by side: night shades (left) and day shades (right)
- type: grid
  cards:
    # Night shades column — mdi:moon icon
    - type: custom:mushroom-cover-card
      entity: cover.shade_7 # Replace with your shade entity
      name: Windshield # Display name
      icon: mdi:moon-waning-crescent # Icon to show on the card
      secondary_info: none
      fill_container: true
      primary_info: name

    # Day shades column — mdi:sun icon
    - type: custom:mushroom-cover-card
      entity: cover.shade_2
      name: Windshield
      icon: mdi:white-balance-sunny
      secondary_info: none
  columns: 1
  grid_options:
    columns: 6 # Each column takes up half the 12-column grid
    rows: auto
```

---

### 5. Energy Flow Cards (Energy View)

The **Energy** view uses the built-in `gauge` card for battery state of charge and `custom:power-flow-card-plus` for animated power flow diagrams.

#### Battery Gauge

```yaml
# A needle-style gauge for battery state of charge
- type: gauge
  entity: sensor.victron_system_0_dc_battery_soc # Replace with your battery SoC sensor
  name: House
  needle: true # Shows a needle pointer instead of filling the arc
  min: 0
  max: 100
  severity:
    green: 75 # Green zone: 75–100%
    yellow: 25 # Yellow zone: 25–75%
    red: 0 # Red zone: 0–25%
```

#### Power Flow Diagram

The `power-flow-card-plus` card shows a live animated diagram of power moving between grid (shore power), solar, battery, and loads.

```yaml
- type: custom:power-flow-card-plus
  entities:
    # "battery" — your house battery bank
    battery:
      icon: mdi:car-battery
      invert_state: true # Positive value = charging; set false if your sensor is reversed
      entity:
        production: sensor.victron_multiplus_ii_276_dc_0_power # Power going into battery

    # "grid" — shore power connection (goes to zero when off-grid)
    grid:
      entity: sensor.victron_system_0_ac_grid_total_power
      display_state: one_way_no_zero # Hides the line when zero watts

    # "home" — the RV as the load icon
    home:
      icon: mdi:rv-truck
      circle_animation: true

    # "individual" — optional breakdown of specific loads
    # Each entry adds a labeled branch to the diagram
    individual:
      - entity: sensor.victron_energy_meter_0_ac_power
        name: Discretionary # High-draw items like AC
        icon: mdi:air-conditioner
      - entity: sensor.victron_multiplus_ii_276_ac_out_total_p
        name: Essential # Low-draw items like lights
        icon: mdi:lightbulb

    # "solar" — solar panel input (omit or leave blank if no solar)
    solar:
      icon: ""

  # Display settings
  use_new_flow_rate_model: true
  w_decimals: 0 # Show whole watts (no decimal)
  kw_decimals: 1 # Show one decimal for kilowatts
  watt_threshold: 1000 # Switch from W to kW display above this value
  display_zero_lines:
    mode: hide # Hide flow lines when power is zero
```

---

### 6. Navigation Badges

Every view has a row of navigation badges at the bottom that link to other views. These use the built-in HA `entity` badge type. The `zone.home` entity is used as a placeholder — it's always available in every HA installation and shows an icon without displaying any state value.

```yaml
badges:
  # Each badge navigates to a different view when tapped
  - type: entity
    show_name: true
    show_state: false # Hides the state value — we only want the icon and label
    show_icon: true
    name: Lights # The label shown under the badge
    icon: mdi:light-recessed # The icon shown on the badge
    entity: zone.home # Placeholder entity (always exists, shows no useful state)
    tap_action:
      action: navigate
      # The path format is: /dashboard-{url-slug}/{view-path}
      # "librecoach" must match the URL you set when creating the dashboard
      navigation_path: /dashboard-librecoach/lights

  - type: entity
    show_name: true
    show_state: false
    show_icon: true
    name: Shades
    icon: mdi:roller-shade
    entity: zone.home
    tap_action:
      action: navigate
      navigation_path: /dashboard-librecoach/shades

  # Repeat for each additional view...
```

---

### 7. Customizing for Your RV

When you paste this dashboard, the entity IDs (like `light.switch_4`, `cover.shade_7`, `sensor.tank_fresh`) will refer to entities in the original RV. You'll need to update them to match your own entity names.

**To find your entity IDs:**

1. Go to **Settings** → **Devices & Services** → **Entities**
2. Search by device name or domain (e.g., search `light.` to see all lights)
3. Click an entity to see its full entity ID

**To edit a card's entity:**

1. Open the dashboard and click the **pencil icon**
2. Hover over the card you want to change and click the **pencil icon** on that card
3. Update the entity field and save

Or, edit the YAML directly in the **Raw configuration editor** and find-and-replace entity IDs in bulk.

---

## Support LibreCoach

LibreCoach is free and open source. If it's useful to you, you can support its development in either of these ways:

[![GitHub Sponsors](https://img.shields.io/badge/Sponsor-GitHub-EA4AAA?logo=github-sponsors&logoColor=white)](https://github.com/sponsors/Backroads4Me)
[![Buy Me a Coffee](https://img.shields.io/badge/Support-Buy%20Me%20a%20Coffee-FFDD00?logo=buy-me-a-coffee&logoColor=000000)](https://buymeacoffee.com/Backroads4Me)
