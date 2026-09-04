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
