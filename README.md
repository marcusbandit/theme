# theme

One palette, everywhere.

## The contract

**Nothing reads the theme name.** Every application includes a fixed path under
`current/`, and that include line is never edited again. Switching a theme
rewrites what lives behind that path, atomically.

    themes/<name>/colors.toml   the source of truth, hand authored
    templates/*.tpl             one per output file, Omarchy {{ key }} syntax
    current/                    symlink to the active build, what apps read
    builds/                     rendered output, garbage collected

## Use

    theme-set              re-render the active theme
    theme-set greensteel   switch
    theme-set --list       what exists
    theme-get              which theme is active
    theme-get accent       one resolved value
    theme-reload           poke running apps (theme-set calls this for you)
    theme-mode             flip light and dark (SUPER+SHIFT+L)
    theme-mode light|dark  pick one; theme-mode status prints it

## Light and dark

The mode is not part of a theme. `theme-mode` writes it to `mode` (gitignored)
and re-runs `theme-set`, which turns the dark palette light with the OKLCH curve
from banditshell's `config/scheme.js`, so the shell and the apps match.
`palette.json` stays dark on purpose: banditshell lightens it itself.
`theme-mode` also flips what is not a colour: the GTK3 theme name, the icon set,
the GTK and Qt ini files, and banditshell's `colour.mode`.

The light curve is designed, not a plain inversion:

- The lightest colour is OKLCH L 0.96 and the page ground sits at 0.936, a soft
  cool grey. Nothing is pure white.
- Every text colour keeps at least its dark-mode APCA contrast against the
  ground. Body text goes up to about Lc 87, ANSI normals land around 65 and
  brights around 78, so all six hues are equally readable.
- Hue chroma goes up 1.35x, because a pale ground mutes colour.
- Light terminals are fully opaque. At 90% the wallpaper and desktop widgets
  show through a pale ground and fight the text.

`kdeglobals` is rendered too, and
`~/.config/kdeglobals` and `~/.local/share/color-schemes/ThemePalette.colors`
link to. KDE apps only use it once their rc names it: `[UiSettings]
ColorScheme=ThemePalette`.

## Adding an app

1. Add `templates/<output-name>.tpl` using `{{ key }}` placeholders.
2. Point the app's own config at `~/.config/theme/current/<output-name>` with
   whatever include mechanism it has.
3. Add its reload command to `bin/theme-reload`, guarded on the process
   actually running.

Never edit anything in `current/` or `builds/`; it is overwritten on every
switch.

## Omarchy compatibility

`themes/<name>/colors.toml` is a valid Omarchy 4.x theme. All 26 keys Omarchy's
`omarchy-theme-color` resolver expects are present; the `ramp_*`, `accent_*`
and `metal_*` keys are extensions that Omarchy's line parser carries along
harmlessly. To publish one, put the directory in a repo named
`omarchy-<name>-theme` with a `preview.png` and a `backgrounds/` directory.

Templates use Omarchy's placeholder syntax, so they also drop into
`~/.config/omarchy/themed/` unchanged.

## Placeholders

    {{ key }}          #RRGGBB          {{ key_strip }}   RRGGBB
    {{ key_rgb }}      R,G,B            {{ key_argb }}    #ffrrggbb  (Qt)
    {{ key_r/g/b }}    one channel      {{ key_lower }}   lowercased
    {{ mix a b 25% }}  blend, 25% of b. Also mix_strip and mix_rgb.

A template referencing an unknown key is a hard error, not a blank.
