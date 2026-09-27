```markdown
# Bypass no discord_desktop_core

Este projeto substitui o `index.js` do módulo `discord_desktop_core` do Discord
por um loader que carrega um payload (`anon.js`) sem crashar o Discord.

O `index.js` original só faz isto:

```js
module.exports = require('./core.asar');
```

Substituímos por uma versão que faz o mesmo **e** carrega o `anon.js` a seguir,
num tick separado (`setImmediate`). O `module.exports` devolve o core original
antes de qualquer patch — por isso o Discord arranca normalmente, sem o erro
`core.startup is not a function`, e o Vencord pode patchear por cima sem colidir.

Funciona em Discord Stable, PTB e Canary, em Windows, Linux e macOS.

---

## Paths

A pasta muda de nome a cada update (`app-1.0.1222`, `app-1.0.1223`, ...).
O que interessa é a pasta `discord_desktop_core` lá dentro. Confirma o nome
exato do `app-*` e do `discord_desktop_core-*` com `dir` ou `ls` na pasta pai.

### Windows

**Stable**
```
C:\Users\USER\AppData\Local\Discord\app-1.0.xxxx\modules\discord_desktop_core-1\discord_desktop_core
```

**PTB**
```
C:\Users\USER\AppData\Local\DiscordPTB\app-1.0.xxxx\modules\discord_desktop_core-1\discord_desktop_core
```

**Canary**
```
C:\Users\USER\AppData\Local\DiscordCanary\app-1.0.xxxx\modules\discord_desktop_core-1\discord_desktop_core
```

### Linux — Flatpak

**Stable**
```
~/.var/app/com.discordapp.Discord/config/discord/app-1.0.xxxx/modules/discord_desktop_core-1/discord_desktop_core
```

**PTB**
```
~/.var/app/com.discordapp.DiscordPTB/config/discordptb/app-1.0.xxxx/modules/discord_desktop_core-1/discord_desktop_core
```

**Canary**
```
~/.var/app/com.discordapp.DiscordCanary/config/discordcanary/app-1.0.xxxx/modules/discord_desktop_core-1/discord_desktop_core
```

### Linux — Snap

**Stable**
```
~/snap/discord/current/.config/discord/app-1.0.xxxx/modules/discord_desktop_core-1/discord_desktop_core
```

**PTB**
```
~/snap/discordptb/current/.config/discordptb/app-1.0.xxxx/modules/discord_desktop_core-1/discord_desktop_core
```

**Canary**
```
~/snap/discordcanary/current/.config/discordcanary/app-1.0.xxxx/modules/discord_desktop_core-1/discord_desktop_core
```

### macOS

**Stable**
```
~/Library/Application Support/discord/app-1.0.xxxx/modules/discord_desktop_core-1/discord_desktop_core
```

**PTB**
```
~/Library/Application Support/discordptb/app-1.0.xxxx/modules/discord_desktop_core-1/discord_desktop_core
```

**Canary**
```
~/Library/Application Support/discordcanary/app-1.0.xxxx/modules/discord_desktop_core-1/discord_desktop_core
```

### Encontrar a pasta automaticamente

**Windows — PowerShell (PTB)**
```powershell
Get-ChildItem "$env:LOCALAPPDATA\DiscordPTB\app-*\modules\discord_desktop_core-*\discord_desktop_core" | Select-Object -First 1 -ExpandProperty FullName
```

**Linux — Flatpak (PTB)**
```bash
ls -d ~/.var/app/com.discordapp.DiscordPTB/config/discordptb/app-*/modules/discord_desktop_core-*/discord_desktop_core
```

**Linux — Snap (PTB)**
```bash
ls -d ~/snap/discordptb/current/.config/discordptb/app-*/modules/discord_desktop_core-*/discord_desktop_core
```

**macOS — PTB**
```bash
ls -d ~/Library/Application\ Support/discordptb/app-*/modules/discord_desktop_core-*/discord_desktop_core
```

Troca `DiscordPTB` / `discordptb` por `Discord` / `discord` (Stable) ou
`DiscordCanary` / `discordcanary` (Canary) conforme o canal que usas.

---

## Bypass — `index.js`

Fecha o Discord. Entra na pasta `discord_desktop_core` (um dos paths acima).
Abre o `index.js` num editor de texto. O conteúdo original é uma linha:

```js
module.exports = require('./core.asar');
```

Apaga tudo e cola isto:

```js
const path = require('path');
module.exports = require(path.join(__dirname, 'core.asar'));
setImmediate(() => { try { require('./anon.js'); } catch (_) {} });
```

Grava.

**O que isto faz:**
- Linha 1: importa o `path`.
- Linha 2: exporta o core original — o Discord arranca normalmente.
- Linha 3: carrega o `anon.js` a seguir, num tick separado. Se o `anon.js`
  não existir, o `try/catch` apanha o erro e o Discord abre na mesma.

Coloca o `anon.js` ao lado do `index.js`, na mesma pasta. Abre o Discord.

---

## Update do Discord

O update apaga a pasta `app-*` antiga e cria uma nova. O `index.js` e o `anon.js`
que tinhas lá desaparecem. Volta a aplicar o bypass na pasta nova.

Para evitar isto, move o `anon.js` para fora do `app-*` e usa este `index.js`:

```js
const path = require('path');
module.exports = require(path.join(__dirname, 'core.asar'));
setImmediate(() => {
  try { require(path.join(process.env.APPDATA || process.env.HOME, 'anon', 'anon.js')); } catch (_) {}
});
```

- Windows: guarda o `anon.js` em `%APPDATA%\anon\anon.js`.
- Linux/macOS: guarda em `~/anon/anon.js`.

Assim, quando o Discord atualizar, só reescreves o `index.js` (3 linhas) — o
`anon.js` fica onde está e continua a ser carregado.
```
