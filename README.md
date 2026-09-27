# discord_desktop_core

Este repositório contém o bypass do `index.js` do módulo `discord_desktop_core`
do Discord, que permite carregar um payload (`anon.js`) sem crashar o Discord.

**Lê o [Bypass.md](Bypass.md) para as instruções completas** — paths por
sistema operativo, o loader de 3 linhas, e como sobreviver a updates do Discord.

---

## Ficheiros

| ficheiro | o que é |
|---|---|
| `Bypass.md` | instruções do bypass (paths, loader, updates) |
| `index.js` | loader de 3 linhas que substitui o original |
| `anon.js` | payload carregado pelo loader |
| `index.js_ORIGINAL` | arquivo original do index.js |
