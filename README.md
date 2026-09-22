# Niklas Plugins

Personlig marketplace for ChatGPT/Codex plugins.

## Included plugin

`aqc-konsult-hantering` ar en skills-only plugin for att strukturera AQC-konsultbehov, profiler, matchningar och uppfoljning pa svenska.

## Installera lokalt

Fran detta repo:

```sh
codex plugin marketplace add .
```

Installera sedan `aqc-konsult-hantering` fran marketplace `Niklas Plugins`.

## Publicera pa GitHub

Skapa ett privat repo och pusha denna mapp:

```sh
git init
git add .
git commit -m "Initial AQC consultant management plugin"
gh repo create chatgpt-plugins --private --source=. --push
```

Pa varje dator:

```sh
codex plugin marketplace add <din-github-anvandare>/chatgpt-plugins
```

Efter andringar:

```sh
codex plugin marketplace upgrade niklas-plugins
```

Starta om ChatGPT desktop-appen efter installation eller uppdatering.
