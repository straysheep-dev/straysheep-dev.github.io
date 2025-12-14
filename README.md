# straysheep-dev.github.io

A site to document, maintain, and share notes with demonstrations or visual components, cross-platform

## Development

These steps will clone the site's source, install Zensical, and serve a working version locally for development. The entire snippet can be copied and pasted into a terminal.

### Zensical

```bash
mkdir ~/src
mkdir ~/venv
python3 -m venv ~/venv
source ~/venv/bin/activate
python3 -m pip install zensical
# cd ~/src; git clone https://github.com/straysheep-dev/straysheep-dev.github.io
cd ~/src/straysheep-dev.github.io/
# Preview as you write
# https://zensical.org/docs/create-your-site/#preview-as-you-write
zensical serve
```

### mkdocs-material

> [!WARNING]
> As of November 2025, mkdocs-material has entered [maintenance mode](https://github.com/squidfunk/mkdocs-material/issues/8523), and will be fully deprecated in November 2026. This site has been migrated to Zensical, but can still be built locally on mkdocs-material via the `straysheep-dev/mkdocs-archival-snapshot` branch.

```bash
mkdir ~/src
mkdir ~/venv
python3 -m venv ~/venv
source ~/venv/bin/activate
python3 -m pip install "mkdocs-material[imaging]"
# cd ~/src; git clone https://github.com/straysheep-dev/straysheep-dev.github.io
cd ~/src/straysheep-dev.github.io/
# Preview as you write
# https://squidfunk.github.io/mkdocs-material/creating-your-site/?h=mkdocs+serve#previewing-as-you-write
mkdocs serve
```

## Licenses

Copyright (c) 2023-2024 straysheep-dev

- [Creative Commons Attribution-NonCommercial 4.0 International (CC BY-NC 4.0)](https://creativecommons.org/licenses/by-nc/4.0/)
- [MIT License](https://github.com/squidfunk/mkdocs-material?tab=MIT-1-ov-file)

[The straysheep-dev icon was built from the Material Design Icons](https://github.com/Templarian/MaterialDesign/blob/master/svg/sheep.svg).

- [Pictogrammers Free License](https://github.com/Templarian/MaterialDesign/tree/master?tab=License-1-ov-file)
