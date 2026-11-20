### Full-stack developer · PHP · Laravel · Angular · React · Node · architecture · AI agents

Hi, I'm Jakub. At work I build e-contract, recruitment and appointment booking systems in PHP (Laravel, Yii2) and Angular. I run my own clients from the first call to deployment: I work out what they need, estimate it, write the analysis, design the architecture and then build it.

I also work a lot with AI agents. I build harnesses for them in which an agent gets a file scope, going out to the network needs consent, and tests decide whether a task is finished. Two of them are described below.

I have a master's in AI from Adam Mickiewicz University. Most of my code is in private repos. If you're hiring and want to see some of it, message me and I'll give you access or show it on a call.

## How I design

I start with who is responsible for what and the contracts between the parts of a system. Code comes after that. I treat AI agents the same way: the model does the work, and code decides whether the work is done.

I design all of it and I approve it. AI does the work inside a process I designed. I learned how to build that process through a lot of trial and error and from people with more experience than me.

This is the harness I wrote for [Kokona](#kokona). It is about 9,500 lines of Node.js with no dependencies and 258 tests of its own.

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="img/harness-dark.png">
  <img src="img/harness-light.png" alt="The Kokona harness: a task goes from goal to done through a plan, task packages, a router, an executor, verification and review. Every tool call passes a hook, a classifier and a consent ledger. State is kept in files that survive a crash." width="100%">
</picture>

- **Task packages.** The main session splits a goal into packages. Each one names the files it may touch and the commands that have to pass. A package can't pick its own model. A router does that by role, with Sonnet for routine work and Opus for the hard parts and for diagnosis.
- **Verification.** The harness runs the acceptance commands itself and fingerprints the files in scope. An agent saying "done" changes nothing. A task that fails the same way twice with the same fingerprint gets blocked, so it can't loop.
- **Gates.** A hook classifies every shell, web, MCP and agent call as local, external API, remote SSH or protected state. Anything that leaves the machine needs a grant from a consent ledger. Grants expire and have use limits, and an agent can't grant one to itself. A second layer stops Node processes from opening sockets outside the allowlist.
- **Recovery.** State is one JSON file written atomically under a lock, with an event log next to it. Runs hold leases, so after a crash unfinished work is picked up again and finished work is never sent twice.
- **Known limits.** The docs include a threat model and a list of what the harness does not protect against, for example a hostile agent that builds commands at runtime outside Node.

So far it has handled over 270 tasks, 230 verifications and 130 reviews. The biggest job was moving Kokona's API and panel from PHP to Rust.

The Domed Realms has its own harness with the same idea and different mechanics. An orchestrator process starts headless workers, snapshots the files before every attempt and rolls them back after a failure. Network traffic goes through a local proxy that only lets it out with consent, and changes to the harness itself land in a staging area until I approve them. It has 267 tests.

## Projects

### Kokona

A Discord bot with an API and a web panel, in production for communities I'm in. Admin panel: [kokona.thedomedrealms.usermd.net](https://kokona.thedomedrealms.usermd.net)

It has about 90 commands in Polish and English: moderation with automod and raid protection, tickets and modmail, XP levels, an economy, gacha, trading between players, giveaways, tournaments with brackets, voice rooms and TTS. NovaQ is ranked 5v5 matchmaking for Strinova with team Elo, captain votes, map bans and score reports with disputes. In progress: a link to a match analytics site, so the matchmaking can balance teams on full player stats.

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="img/kokona-architecture-dark.png">
  <img src="img/kokona-architecture-light.png" alt="Kokona architecture: Discord talks to a Node.js process, which talks to a Rust API over signed HTTP. Only the API touches MySQL. A Rust web panel uses the same API with its own key." width="100%">
</picture>

- The Discord process never touches the database. It talks to the API with HMAC-signed requests (method, path, body hash, timestamp, nonce), and the answers are signed too, because on shared hosting another tenant can bind the loopback port.
- Trades, pulls and prizes run in one transaction with a fixed lock order and a retry after a deadlock. A giveaway that crashes halfway announces the same winners later and never draws twice.
- In November I moved the API and the panel from PHP to Rust (tokio, hyper, sqlx, axum). Agents wrote the code under my harness, and over 2,000 contract tests run against both implementations to keep them identical. In my measurements the Rust API answers 3.5 to 7 times faster than the PHP one.
- Profiles, pulls, rankings and brackets come out as images. They are SVG templates drawn by a Rust addon (napi-rs, resvg) off the main thread, with WebAssembly and plain text as fallbacks. A profile card takes 99 ms, down from 330 ms.
- Deploys go through my own deployer: a build on the server, smoke tests, a database backup before migrations, a directory swap, and a return to the previous version if the bot doesn't come up.

Over 4,000 tests (Node.js, Rust, PHP) and 57 SQL migrations.

Two more bots follow the same ideas. Si shares Kokona's architecture and is still being built for an Arknights: Endfield community. Rosmontis does AI chat through Groq and Gemini, and I'm now rewriting it in Rust.

`Node.js` `discord.js` `Rust` `PHP` `MySQL` `resvg` `napi-rs`

<p>
  <img src="img/kokona-profile-card.jpg" alt="Kokona: profile card with stats, skin collection and achievement badges" width="49%">
  <img src="img/kokona-ranking-card.jpg" alt="Kokona: NovaQ ranking card, top 100 players of one mode (demo data)" width="49%">
</p>
<p>
  <img src="img/kokona-bracket-card.jpg" alt="Kokona: tournament bracket card" width="49%">
  <img src="img/kokona-kokodle-card.jpg" alt="Kokona: Kokodle, a daily character quiz" width="49%">
</p>
<p>
  <img src="img/kokona-panel-automod.jpg" alt="Kokona panel: automod and raid protection (demo data)" width="49%">
  <img src="img/kokona-panel-leveling.jpg" alt="Kokona panel: levels and rewards (demo data)" width="49%">
</p>

<details>
<summary>More screenshots</summary>
<br>
<p>
  <img src="img/kokona-panel.jpg" alt="Kokona panel: server overview" width="49%">
  <img src="img/kokona-pull-card.jpg" alt="Kokona: gacha pull x10" width="49%">
</p>
<p>
  <img src="img/kokona-slots-card.jpg" alt="Kokona: slot machine" width="49%">
</p>
</details>

### The Domed Realms

A platform for running tabletop RPG campaigns, in Polish and English. I use it for my own campaigns. Live at [thedomedrealms.usermd.net](https://thedomedrealms.usermd.net)

Players keep their character sheets here, arrange the panels however they want and can search through a whole sheet. Sheets save live. There's a dice simulator that reads formulas from character stats, a world map with submaps and pins, guilds with expeditions and a shared scheme, a wiki with a relation map, my rulebook and choose-your-path stories.

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="img/tdr-realtime-dark.png">
  <img src="img/tdr-realtime-light.png" alt="The Domed Realms: the browser asks Laravel for a signed ticket and connects with it to the WebSocket relay. Laravel is the only writer to MySQL. The relay sends each player only their own view." width="100%">
</picture>

- Everything live runs on one WebSocket server in Node.js with 6 kinds of rooms: battle map, dungeons, guild scheme, GM table, character sheets and locations.
- Only Laravel checks permissions and writes to the database. It signs a ticket with HMAC, the WebSocket server verifies it and sends each player only their own view. Hidden HP, GM layers and fog of war never leave the server.
- Rules that have to give the same result on both sides exist as a JavaScript core and a PHP mirror, tested against the same fixtures.
- Character sheets keep an outbox in the browser, so edits survive a dropped connection.

What I added this autumn:

- Locations, a map maker for towns, villages, dungeons, caves and terrain. 12 generators build a whole map from a seed, the GM edits it by hand and players watch it live from a link.
- A GM table. The GM sets up scenes with images, text, drawings and characters, and players watch it live on their own screens or on a TV.
- A guild scheme, which is a shared grid map up to 3 km wide. Several people can draw on it at once and the server checks every change against the user's role.
- Guild expeditions with character fatigue, camp actions and a food and water calculator.
- A rebuilt battle map with walls, buildings and a library of saved layouts.
- A relation map in the wiki, where every player sees only what their character knows.

About 690 application tests (PHPUnit, node:test) and 267 harness tests. Google and Discord login, 3 themes.

`Laravel 12` `React 18` `Inertia.js` `Node.js` `MySQL` `Tailwind CSS`

<p>
  <img src="img/tdr-home.jpg" alt="The Domed Realms: home page with the player's characters and the places to go next" width="99%">
</p>
<p>
  <img src="img/tdr-locations-editor.jpg" alt="The Domed Realms: location editor with a sewer dungeon on three levels, in blueprint style" width="99%">
</p>
<p>
  <img src="img/tdr-locations-generator.jpg" alt="The Domed Realms: location generator with 12 map types" width="60%">
  <img src="img/tdr-locations-dungeon.jpg" alt="The Domed Realms: the same dungeon shown as a stack of levels" width="37.5%">
</p>
<p>
  <img src="img/tdr-gm-table.jpg" alt="The Domed Realms: GM table, the GM's view" width="99%">
</p>
<p>
  <img src="img/tdr-character-sheets.jpg" alt="The Domed Realms: character sheet" width="99%">
</p>

<details>
<summary>More screenshots</summary>
<br>
<p>
  <img src="img/tdr-dungeon-session.jpg" alt="The Domed Realms: running a dungeon, with GM notes and what the players see" width="99%">
</p>
<p>
  <img src="img/tdr-battle-map.jpg" alt="The Domed Realms: hex battle map with rounds and initiative" width="99%">
</p>
<p>
  <img src="img/tdr-guild-scheme.jpg" alt="The Domed Realms: guild scheme editor" width="49%">
  <img src="img/tdr-guild-overview.jpg" alt="The Domed Realms: guild overview with roles and permissions" width="49%">
</p>
<p>
  <img src="img/tdr-sheet-combat.jpg" alt="The Domed Realms: character sheet: combat stats, skills and summary" width="99%">
</p>
<p>
  <img src="img/tdr-gm-table-player.jpg" alt="The Domed Realms: GM table, the player's view with a dice panel" width="99%">
</p>
<p>
  <img src="img/tdr-world-map.jpg" alt="The Domed Realms: hand-drawn world map with pins" width="99%">
</p>
<p>
  <img src="img/tdr-dice-lab.jpg" alt="The Domed Realms: dice simulator with rolls from the active character sheet" width="49%">
  <img src="img/tdr-wiki.jpg" alt="The Domed Realms: wiki article about a character" width="49%">
</p>
<p>
  <img src="img/tdr-handbook.jpg" alt="The Domed Realms: rulebook in the app" width="49%">
  <img src="img/tdr-sheet-search.jpg" alt="The Domed Realms: search inside a character sheet" width="49%">
</p>
</details>

### Strinova Lore

A local, searchable archive of the lore of the game Strinova. The story is spread over Chinese wikis, official accounts, events and comics, so I wrote a Python tool that collected about 3,600 public sources in 4 languages.

AI agents pulled about 5,200 claims out of them. Each claim has a word-for-word quote and a category such as confirmed, unverified or theory. The harness throws out any quote it can't find in the source, and nothing counts as confirmed without an official source.

I browse it in Lore Explorer, a FastAPI app with semantic search (bge-m3 on the GPU), full-text search (SQLite FTS5), articles for 69 characters and a timeline.

`Python` `FastAPI` `SQLite FTS5` `sentence-transformers` `bge-m3`

<p>
  <img src="img/lore-explorer-search.jpg" alt="Lore Explorer: hybrid search" width="49%">
  <img src="img/lore-explorer-character.jpg" alt="Lore Explorer: character article with sources" width="49%">
</p>

### Marketplace platform (in progress)

A marketplace with a social side: listings with applications, profiles with portfolios, a feed, a calendar and chat. The name and details are confidential for now, so I only describe the technical side.

It's a pnpm and Turborepo monorepo. The Laravel 13 API is split into 10 modules and the web app is React 19 with TanStack Router and Query. The TypeScript client is generated from an OpenAPI spec with 174 paths, and chat and notifications go through Laravel Reverb. There's an Expo mobile app too, but it's only a skeleton so far.

Over 2,200 tests (Pest on real PostgreSQL, Vitest), Larastan, and the whole environment starts with one command in Docker Compose. The repo has one contract for AI agents (AGENTS.md), hooks that block edits to generated files and a check that fails when the docs mention files that don't exist.

`Laravel 13` `React 19` `TypeScript` `PostgreSQL` `Valkey` `Reverb` `Expo` `Docker`

### Rosmontis Assistant

A desktop pet with Rosmontis from Arknights, built in Electron. She walks around the desktop, climbs onto windows and sits on their edges, reminds me about dailies and deadlines, launches programs and rolls RPG dice.

She can also pick up real desktop icons, fly them around and put them back where they were. That goes through Windows APIs from a small C# helper, and if the app crashes, the next start restores the icons from a backup.

Chat goes to the ChatGPT app or to an API (Ollama, LM Studio, OpenAI). It also works as a wallpaper and can export one for Wallpaper Engine. In November I ported it to Linux and added installers for both systems. The repo is private because the character art isn't mine.

`Electron` `JavaScript` `Spine` `WebGL` `C#` `PowerShell` `Python`

### Browser extensions

**X Media Downloader** saves videos and GIFs from X with a right-click. When X only serves HLS, it fetches video and audio separately and joins them into an MP4 with my own code. GIFs are made in the browser with WebCodecs.

**YouTube downloader** adds a download option to YouTube's own player menu. The extension talks through Native Messaging to a small Python host that runs yt-dlp and FFmpeg, so there's no open port and nothing running in the background.

**Bilibili downloader** works the same way with a bigger host. It measures every stream variant with ffprobe, picks video and audio separately, merges them without re-encoding and checks the result with a decode test. The host accepts a closed set of operations and never runs a shell.

`TypeScript` `React` `Vite` `WebCodecs` `Vitest` `Python` `pytest`

<p>
  <img src="img/x-media-downloader-settings.jpg" alt="X Media Downloader: settings" width="40%">
</p>

### Smaller things

- **TierList**: a drag & drop tier list for 282 skins, with data collected by a Python scraper
- **Browser games** in plain JavaScript: a turn-based strategy game and a logic puzzle
- **Training AI models**: fine-tuning LLMs and training LoRAs for image models

## What I use

**Every day:** `PHP` `Laravel` `Yii2` `Angular` `TypeScript` `React` `MySQL` `PostgreSQL`

**Also know well:** `Node.js` `Next.js` `Python` `FastAPI` `Electron` `Redis` `Docker` `WebSocket` `OpenAPI`

**AI:** `Claude Code` `GPT Codex` `agent harnesses` `hooks and consent gates` `semantic search` `LLM fine-tuning` `LoRA`

**Learning:** `Rust` (tokio, hyper, axum, sqlx)

## What's next

I want to move towards architecture and leading projects technically. After hours I'm learning Rust on my own code and finding out how much work can safely be handed to agents. I'm looking for a team where both are useful and where I can learn from others.

## Outside of code

I run tabletop RPG sessions as a GM and I'm writing my own game system (the rulebook is at 281 pages so far). I also give talks at fan conventions, for example Hikari.

## Contact

[LinkedIn](https://www.linkedin.com/in/jakubzareba-dev/) · jakubzareba99@gmail.com
