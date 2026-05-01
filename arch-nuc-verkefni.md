# Sérsniðið Linux Distro á NUC8BEH

**Vélbúnaður:** Intel NUC8BEH (Bean Canyon, 2018)
**Grunnur:** Arch Linux + KDE Plasma 6 (Wayland)
**Markmið:** Daglegt notkunarkerfi sem er sérsniðið frá grunni — UI hannað í Claude Design
**Dagsetning skjals:** Maí 2026

---

## Yfirlit

Þetta skjal er lifandi áætlun fyrir verkefnið. Það er hannað til að gefa **Claude Code** allt sem það þarf til að keyra uppsetninguna með notanda í terminal.

UI hönnunarvinnan (Fasi 3) verður unnin sérstaklega í **Claude Design** (claude.ai/design), sem getur svo pakkað hönnuninni í handoff bundle fyrir Claude Code til að innleiða.

Arch Wiki er aðal heimildin og **alltaf á að fletta þar fyrst** ef eitthvað stangast á við þetta skjal, því Arch breytist hratt.

### Heimildir

- [Arch Linux Installation Guide](https://wiki.archlinux.org/title/Installation_guide) — opinbera leiðbeiningin
- [Arch Wiki: KDE](https://wiki.archlinux.org/title/KDE) — KDE Plasma uppsetning á Arch
- [Arch Wiki: Visual Studio Code](https://wiki.archlinux.org/title/Visual_Studio_Code)
- [KDE Plasma 6 Schedule](https://community.kde.org/Schedules/Plasma_6)
- [Claude Design](https://claude.ai/design) — fyrir UI hönnun (Fasi 3)
- [Claude Design Admin Guide](https://support.claude.com/en/articles/14604406-claude-design-admin-guide-for-team-and-enterprise-plans)

---

## Vélbúnaður — staðfest fyrir þessa vél

**Vörunúmer:** `BOXNUC8i3BEH2` (Bean Canyon, revision 03/2019, BOX = barebones)

| Hluti | Staðfest gildi |
|-------|----------------|
| CPU | Intel Core **i3-8109U** (Coffee Lake, 2C/4T, 3.0 GHz base / 3.6 GHz turbo, 4 MB L3, 28 W TDP) |
| GPU | Intel Iris Plus Graphics 655 (integrated, 128 MB eDRAM, HDMI 2.0a 4K@60) |
| RAM slots | 2× DDR4-2400 SO-DIMM, hámark 32 GB *(BOX útgáfa kemur tóm)* |
| Geymsla | 1× M.2 22×80 (NVMe/SATA) + 1× 2.5" SATA bay *(BOX útgáfa kemur tóm)* |
| Wi-Fi | Intel Wireless-AC **9560** (CNVi, Wi-Fi 5, BT 5.0) |
| Ethernet | Intel i219-V Gigabit |
| Útgangar | HDMI 2.0a, Thunderbolt 3 (USB-C, DP 1.2), 4× USB 3.1 Gen 2 Type-A, microSDXC |
| Hljóð | 7.1 yfir HDMI/TB3, 3.5mm combo jack að framan |
| BIOS | Aptio V (Visual BIOS), family `BECFL357` |
| Aflgjafi | 19V DC, 90W external |

**Þarf enn að staðfesta á vélinni (sjá Fasa 0):**
- RAM: stærð og uppstilling (1 vs 2 stafir)
- Storage: M.2 NVMe stærð, og hvort 2.5" bay er notuð
- BIOS útgáfa á vélinni núna

**Linux samhæfni:** Allt studd af opnum drivers í mainline kernel — `i915` (GPU), `iwlwifi` (Wi-Fi), `e1000e` (ethernet), `snd_hda_intel` (audio). Engin proprietary firmware vandamál.

---

## Hugbúnaðarval

### Grunnkerfi

| Hluti | Val | Af hverju |
|-------|-----|-----------|
| Distro base | Arch Linux | Rolling release, cutting edge, lærdómsleið |
| Kernel | `linux` (nýjasti stable, 6.19+ í maí 2026) | Fær allar nýjar Wayland/KDE features |
| Init | systemd | Sjálfgefið í Arch |
| Bootloader | `systemd-boot` | Einfaldara en GRUB á UEFI |
| Filesystem | Btrfs | Snapshots → "ég brotnaði kerfið, rúlla til baka" |
| Network | NetworkManager | Plasma applet virkar með þessu |
| Audio | PipeWire + WirePlumber | Nútíma audio stack, sjálfgefið |

### Desktop: KDE Plasma 6

**Útgáfa við skjölun:** Plasma 6.6.4 (apr 2026), næst Plasma 6.6.5 (12. maí 2026). Plasma 6 fer á 4 mánaða útgáfu-cycle.

**Session:** Wayland (sjálfgefið frá Plasma 6.4, X11 fjarlægt í Plasma 6.8)

**Helstu pakkar:**
- `plasma-meta` — heildarsett (Plasma desktop + helstu KDE applications)
- *eða* `plasma` — minimal Plasma desktop án allra KDE apps (við blöndum eftir smekk)
- `kde-applications-meta` — full KDE app suite (valfrjálst, mikið af pökkum)
- `sddm` — Plasma Login Manager er nýtt, en SDDM er enn stöðugri

**Af hverju KDE en ekki GNOME:**
- KDE er hannað fyrir djúpa sérsníðingu — panels, widgets, themes, allt stillanlegt í gegnum GUI
- Engar "extensions sem brotna við uppfærslur" eins og í GNOME
- Þú getur endurraðað öllu skjáborðinu án þess að snerta config skrár
- Hentar betur þegar UI er hannað í Claude Design og innleitt sérsniðið

### Daglegt notkunarkerfi

| Flokkur | Pakki | Athugasemd |
|---------|-------|------------|
| Vafri | `firefox` | Virkar vel á KDE Wayland |
| Office | `libreoffice-fresh` | "Fresh" = nýjasta útgáfa |
| Email | `kontact` (KDE) eða `thunderbird` | KDE Kontact er heildarpakki |
| File manager | Dolphin | Innifalið í Plasma |
| Media | `mpv`, `gstreamer` codecs | mpv besti spilarinn |
| Skjámyndir | `spectacle` | Innifalið í KDE, með OCR í 6.6 |
| PDF | `okular` | Innifalið í KDE |
| Prentun | `cups` + Plasma print manager | Ef á við |

### Forritunarumhverfi (web dev)

| Tól | Pakki | Uppsetning |
|-----|-------|-----------|
| Git | `git` | `pacman -S git` |
| Node.js | `nodejs`, `npm` | `pacman -S nodejs npm` |
| TypeScript (global) | `typescript` | `pacman -S typescript` |
| VS Code | `visual-studio-code-bin` | **Úr AUR** (sjá að neðan) |
| Terminal | Konsole (sjálfgefið í KDE) | eða `kitty` ef við viljum cooler |

**ATH um VS Code:** Arch hefur tvo möguleika:
1. `code` (úr official repo) — Code-OSS, opinn kóði, en virkar **ekki** með Microsoft Remote-SSH extension
2. `visual-studio-code-bin` (úr AUR) — opinber Microsoft útgáfa, allt virkar

**Ráðlegging:** AUR útgáfan. Þetta þýðir að við þurfum AUR helper (`yay` eða `paru`) sem við setjum upp í Fasa 2.

**React, Next.js, Vite, osfrv.:** Þetta eru *ekki* kerfispakkar. Þeir koma sem npm pakkar í hverju verkefni: `npm create vite@latest`, `npx create-next-app`, osfrv. Ekki blanda saman við pacman.

---

## Verkefnaskipulag — 5 fasar

### Fasi 0: Undirbúningur (Claude Code, 1 kvöld)

**Áður en NUC-inn er snertur:**

- [ ] Backup á öllu mikilvægu af NUC-inum
- [ ] Niðurhal á [nýjasta Arch ISO](https://archlinux.org/download/) (athuga checksum!)
- [ ] Búa til bootable USB með `dd` (eða BalenaEtcher)
- [ ] Lesa [Arch Installation Guide](https://wiki.archlinux.org/title/Installation_guide) einu sinni í gegn — *án þess að framkvæma neitt*
- [ ] Hafa annan tölvu/síma tilbúna til að lesa wiki á meðan uppsetning gengur
- [ ] Tengja NUC við ethernet (Wi-Fi setup á command line í installer er flóknara)

### Fasi 1: Base Arch uppsetning (Claude Code, 1 helgi)

**Markmið:** Bootable kerfi sem kemst á netið. Engin GUI ennþá.

Skref:

1. Boot inn í Arch live USB — **UEFI mode**, ekki Legacy
2. Stilla lyklaborð (`loadkeys is-latin1`)
3. Tengjast neti (ethernet sjálfvirkt, eða `iwctl` fyrir Wi-Fi)
4. Stilla klukku (`timedatectl`)
5. Partitioning með `cfdisk`:
   - EFI partition: 1 GB, FAT32, mount á `/boot`
   - Root partition: rest, Btrfs, mount á `/`
   - **Engin swap partition** — swapfile á Btrfs í staðinn
6. Format og mount
7. Btrfs subvolumes:
   - `@` fyrir root
   - `@home` fyrir /home
   - `@snapshots` fyrir snapshots
   - `@var_log` fyrir logs (utan snapshots)
8. `pacstrap`: base, base-devel, linux, linux-firmware, intel-ucode, networkmanager, sudo, nano, git, btrfs-progs
9. `genfstab` → /etc/fstab
10. `arch-chroot` inn í nýja kerfið
11. Tímabelti: `Atlantic/Reykjavik`
12. Locale: `is_IS.UTF-8` og `en_US.UTF-8`
13. Hostname
14. `mkinitcpio`
15. Root password
16. `systemd-boot` install + config
17. Notandareikningur + sudo setup
18. `systemctl enable NetworkManager`
19. Reboot, fjarlægja USB

**Til hvers að mæta þessari helgi:** Login prompt, internet virkar.

### Fasi 2: KDE Plasma + daglegt notkunarkerfi (Claude Code, 1 helgi)

1. **GPU drivers:** `mesa`, `vulkan-intel`, `intel-media-driver`
2. **PipeWire:** `pipewire`, `pipewire-pulse`, `pipewire-alsa`, `wireplumber`
3. **KDE Plasma:**
   - `plasma-meta` (full Plasma desktop + meginhluti KDE apps)
   - eða `plasma` (minimal, við bætum við apps eftir smekk)
4. **Display manager:** `sddm` + `systemctl enable sddm`
5. **Wayland session:** sjálfgefið í Plasma 6.4+, ekkert sérstakt þarf
6. **Reboot → SDDM login skjár → Plasma desktop**
7. **AUR helper:** `yay` (úr AUR git, byggjum með `makepkg`)
8. **VS Code:** `yay -S visual-studio-code-bin`
9. **Dev tools:** `pacman -S git nodejs npm typescript`
10. **Snapper setup** fyrir Btrfs snapshots — sjálfvirk afrit fyrir hverja `pacman` uppfærslu
11. **`reflector`** fyrir hraðan mirror lista
12. **Fonts:** `noto-fonts`, `noto-fonts-cjk`, `noto-fonts-emoji`, `ttf-jetbrains-mono`
13. **Browser, office, media:** Firefox, LibreOffice, mpv

**Til hvers að mæta þessari helgi:** Fullkomið KDE Plasma 6 desktop, dev tools tilbúnir, VS Code virkar.

### Fasi 3: UI hönnun og sérsníðing (Claude Design + Claude Code, margar helgar)

**Þetta er hjarta verkefnisins.** Hér gerum við kerfið að *þínu*.

#### Verkflæði

1. **Notandi opnar [claude.ai/design](https://claude.ai/design)** og býr til verkefni fyrir Plasma desktop
2. **Hannar saman með Claude Design** — mockups af panel layout, widgets, lit-paletu, ikonum, möppuskipulagi
3. **Claude Design pakkar handoff bundle** þegar hönnun er tilbúin
4. **Notandi gefur Claude Code bundle-ið** og við innleiðum á NUC-inum

#### 3.1 UI hönnun — það sem á að ákveða í Claude Design

**Plasma desktop layout:**
- Panel staðsetning (toppur/botn/hlið?)
- Hvaða widgets eiga að vera þar
- Application launcher style (Kickoff, Application Dashboard, eitthvað custom?)
- System tray innihald
- Activities — KDE er með "Activities" hugtak fyrir mismunandi vinnuumhverfi
- Workspaces (virtual desktops) skipulag

**Þema:**
- Light eða dark (eða sjálfvirkt skipt eftir tíma — Plasma 6.5+ styður þetta sjálfvirkt)
- Litapaletta (KDE hefur Color Schemes)
- Window decorations (titlebar style)
- Icon theme
- Cursor theme
- Splash screen við login
- SDDM login theme
- Plasma styles (sem KDE kallar "Global Themes")

**Möppuskipulag undir `~/`:**
KDE er ekki bundið við default `~/Documents`, `~/Pictures` o.s.frv. — þú getur hannað þitt eigið skipulag og uppfært `~/.config/user-dirs.dirs`.

Dæmi (Claude Design getur teiknað þetta út fyrir þig):
```
~/
├── projects/           # forritun
│   ├── personal/
│   ├── work/
│   └── learning/
├── docs/               # skjöl
├── media/
│   ├── photos/
│   ├── videos/
│   └── music/
├── downloads/          # tímabundið
├── archive/            # gamalt en geymt
└── sandbox/            # tilraunir, má eyða
```

**Workflows:**
- KDE keyboard shortcuts — kortleggja vinnuflæði
- Custom scripts og hraðtakka
- KWin window rules (sjálfvirk hegðun fyrir tiltekin forrit)
- KRunner shortcuts (Plasma "spotlight")

#### 3.2 Claude Code innleiðing

Þegar UI hönnunin er tilbúin í Claude Design og handoff bundle er gefið Claude Code, framkvæmir Claude Code:

1. **Plasma stillingar:** `~/.config/plasmashell/`, `~/.config/plasma-org.kde.plasma.desktop-appletsrc`, `~/.config/kdeglobals`, o.fl.
2. **KWin stillingar:** `~/.config/kwinrc`, `~/.config/kwinrulesrc`
3. **SDDM:** `/etc/sddm.conf.d/` config
4. **GTK þemu:** fyrir non-KDE forrit svo þau passi
5. **Möppuskipulag:** `~/.config/user-dirs.dirs` + möppur búnar til
6. **Custom scripts** í `~/.local/bin/`

#### 3.3 Dotfiles í git

Allar stillingar í git repository (GitHub/GitLab/Codeberg).

Verkfæri valkostir:
- **GNU Stow** — einfaldur symlink manager
- **chezmoi** — sterkari, með templates fyrir mismunandi vélar
- **yadm** — git wrapper fyrir dotfiles

KDE specific dotfiles sem fara í git:
- `~/.config/plasma-*`
- `~/.config/kdeglobals`
- `~/.config/kwinrc`
- `~/.config/kglobalshortcutsrc`
- `~/.config/Code/User/settings.json`
- `~/.bashrc` eða `~/.zshrc`
- `~/.gitconfig`

#### 3.4 Backup strategía

Þrjár lög:
1. **Btrfs snapshots** (snapper) — innan kerfisins
2. **Ytri diskur** — vikulegar fullar afrit með `borg` eða `restic`
3. **Cloud** — `rclone` til Backblaze B2 / S3 / annað

### Fasi 4: Eigin install ISO (Claude Code, valfrjálst, seinna)

Þegar þú ert kominn með stöðugt kerfi sem þér líkar, getum við pakkað því sem **þínu eigin distro** með [`archiso`](https://wiki.archlinux.org/title/Archiso). Niðurstaðan er ISO sem þú getur sett upp á annarri vél og hún verður strax með þínu skipulagi og UI.

Þetta er þar sem það verður formlega "þitt distro" með nafni.

---

## Spurningar sem Claude Code á að spyrja notanda áður en byrjað er á Fasa 1

### Vélbúnaður (þarf að athuga á NUC-inum)
1. **RAM:** Hversu mikið RAM er í NUC-inum? (lágmark 8 GB ráðlagt fyrir Plasma 6, helst 16 GB)
2. **Disk:** M.2 NVMe, SATA SSD, eða bæði? Stærð?
3. **CPU útgáfa:** i3-8109U / i5-8259U / i7-8559U?
4. **BIOS útgáfa:** Hvaða útgáfa er virk? Þarf uppfærslu?

### Net og uppsetning
5. **Tenging við uppsetningu:** Ethernet eða Wi-Fi? (ethernet sterklega ráðlagt)
6. **Hostname:** Hvaða nafn á kerfið?
7. **Notandanafn:** Hvaða notandanafn?
8. **Tímabelti:** Staðfesta `Atlantic/Reykjavik`?
9. **Tungumál:** Aðal locale — `is_IS.UTF-8` eða `en_US.UTF-8`?
10. **Lyklaborðsuppstilling:** Íslenskt? (`is-latin1` í console, `is` í Plasma)

### Sniðsval áður en byrjað er
11. **Btrfs eða ext4?** (Ráðlegging: Btrfs fyrir snapshots)
12. **Swap:** Swapfile á Btrfs eða engin swap?
13. **Full disk encryption (LUKS)?** (bætir öryggi en flækir uppsetningu)
14. **Secure Boot:** Virkjað? (venjulega sleppt í fyrstu)

### Ákvarðanir fyrir Fasa 2
15. **KDE pakkasett:** `plasma-meta` (heildarpakki) eða `plasma` (minimal)?
16. **KDE applications:** `kde-applications-meta` (allt) eða velja stykki sjálfir?
17. **Display manager:** SDDM (sjálfgefið) eða Plasma Login Manager (nýrra)?
18. **VS Code útgáfa:** `visual-studio-code-bin` (AUR, MS) eða `code` (OSS)?
19. **AUR helper:** `yay` eða `paru`?
20. **Shell:** `bash` eða `zsh`?
21. **Terminal emulator:** Konsole (sjálfgefið) eða `kitty`/`alacritty`?

### Spurningar fyrir Fasa 3 — UI hönnun (svarað í Claude Design)
22. Plasma panel staðsetning og innihald
23. Application launcher style
24. Activities + workspaces skipulag
25. Litapaletta og þema
26. Möppuskipulag undir `~/`
27. Keyboard shortcuts og workflows
28. Hvaða dotfiles strúktúr og hvaða pall (GitHub/GitLab/Codeberg)
29. Backup pakki (`borg` / `restic`) og ytri diskur / cloud

---

## Verkflæði: hver gerir hvað

### Claude Code (terminal — gerir Fasa 0–2 og Fasa 4)
- Spyr notanda spurninganna að ofan áður en farið er í Fasa 1
- Keyrir Arch installation skref fyrir skref
- Framkvæmir kerfisuppsetningu á NUC-inum
- Setur upp pakka, stillir systemd services, KDE Plasma, dev tools
- Innleiðir UI hönnun frá Claude Design (Fasi 3.2)
- Hjálpar að debug villuboð

### Claude Design (claude.ai/design — gerir UI hluta Fasa 3)
- Mockaupar Plasma desktop layout
- Hannar litapalettu og þema
- Teiknar möppuskipulag
- Gerir hönnunarkerfi fyrir kerfið
- Pakkar handoff bundle fyrir Claude Code

### Skipting milli þeirra
1. Notandi byrjar í **Claude Code** fyrir Fasa 0–2 (uppsetning)
2. Þegar Plasma er uppi og virkt → Notandi opnar **Claude Design** fyrir Fasa 3 hönnun
3. Þegar hönnun tilbúin → Notandi gefur Claude Code handoff bundle → innleiðing
4. Fasi 4 (eigin ISO) er aftur Claude Code verkefni

---

## Athugasemdir til Claude Code

1. **Spyrja notanda spurninganna að ofan áður en farið er í Fasa 1.** Ekki giska á gildi.
2. **Lesa Arch Wiki greinarnar áður en skipanir eru gefnar.** Arch breytist hratt — opinber Wiki er sannleikurinn, ekki þetta skjal.
3. **Aldrei keyra `dd`, `mkfs`, `parted`, eða `cfdisk` skipanir án staðfestingar frá notanda.** Þessar skipanir geta þurrkað út diskinn.
4. **Vinna í áföngum.** Eftir hvern fasa, staðfesta að allt virki áður en haldið er áfram.
5. **Skjala ákvarðanir.** Þegar notandi velur eitthvað (t.d. zsh fram yfir bash), skrifa það niður svo Fasi 3 byggist á réttum forsendum.
6. **Notandi er byrjandi á Arch en hefur notað Ubuntu.** Útskýra hluti, ekki bara framkvæma þá. Þetta er lærdómsverkefni jafn mikið og uppsetningarverkefni.
7. **Engin tímamörk.** Notandi sagði „engin tímamörk" — taka tíma, gera þetta rétt.
8. **Snapshots fyrir uppfærslur.** Eftir Snapper er uppsett, taka snapshot fyrir hverja stóra breytingu.
9. **Engar pakkasamsetningar úr mörgum heimildum á sama tíma.** Ekki blanda Flatpak + AUR + pacman fyrir sama hlut. Veldu einn fyrir hvern pakka.
10. **KDE-specific:** Plasma stillingar eru í `~/.config/` — fjölmargar skrár. Þegar Fasi 3 handoff bundle kemur frá Claude Design, fara breytingar þar inn. Aldrei skrifa beint yfir Plasma config skrá án backup.

---

## Þekktar áhættur og athugasemdir

- **Plasma 6 hefur engar LTS útgáfur.** Nýjar útgáfur koma á 4 mánaða fresti. Snapper snapshots eru því enn mikilvægari.
- **Wayland í KDE er stöðugt en er enn að þroskast.** Sum proprietary forrit (sérstaklega skjáupptaka) geta haft vandamál. Fallback er X11 session — en hverfur í Plasma 6.8.
- **NUC8BEH er 8 ára vélbúnaður (2018).** Intel hefur hætt opinberum BIOS uppfærslum, en ASUS hefur tekið yfir línuna og gefur enn út uppfærslur fyrir suma NUC. Athuga ASUS support síðu.
- **VS Code úr AUR uppfærist með `yay`, ekki `pacman`.** Notandi þarf að keyra `yay -Syu` reglulega til viðbótar `pacman -Syu`.
- **Claude Design er research preview** og krefst Pro/Max/Team/Enterprise áskriftar. Ef notandi hefur ekki aðgang þarf að nota varaleið (t.d. handvirkar Plasma stillingar í gegnum System Settings GUI).

---

## Staðfestar ákvarðanir notanda (maí 2026)

Þessi kafli skráir svör notanda við spurningunum að ofan og þær ákvarðanir sem Claude Code tók fyrir hönd notanda þegar beðinn var um að ráða.

### Net og uppsetning

| # | Atriði | Ákvörðun |
|---|--------|----------|
| 5 | Tenging við uppsetningu | **Wi-Fi** (notum `iwctl` í installer) |
| 6 | Hostname | **AddiOS** |
| 7 | Notandanafn | **arnar** |
| 8 | Tímabelti | **Atlantic/Reykjavik** |
| 9 | Aðal locale | **en_US.UTF-8** (með `is_IS.UTF-8` einnig virku) |
| 10 | Lyklaborð | **Íslenskt** (`is-latin1` í console, `is` í Plasma) |

### Disk og öryggi (Claude Code valdi)

| # | Atriði | Ákvörðun | Rökstuðningur |
|---|--------|----------|---------------|
| 11 | Filesystem | **Btrfs** | Snapshots → "rúlla til baka" virkni er ómetanleg fyrir rolling release |
| 12 | Swap | **Swapfile á Btrfs** | Sveigjanlegri en partition; getur stækkað/minnkað seinna |
| 13 | LUKS encryption | **Sleppt í fyrstu** | Bætir flækjustigi við boot fyrir byrjanda. Hægt að bæta við seinna með nýrri uppsetningu ef vill |
| 14 | Secure Boot | **Disabled fyrst, virkjað seinna** | Notandi sagði að virkja á eftir; sett upp `sbctl` í Fasa 2 til undirbúnings |

### Desktop og forrit

| # | Atriði | Ákvörðun | Rökstuðningur |
|---|--------|----------|---------------|
| 15 | KDE pakkasett | **`plasma-meta`** | Full Plasma desktop |
| 16 | KDE applications | **`kde-applications-meta`** | Notandi vill „allt bara, einfaldara fyrir byrjendur" |
| 17 | Display manager | **Plasma Login Manager** | Notandi valdi nýrra alternatífið |
| 18 | VS Code útgáfa | **`visual-studio-code-bin`** (úr AUR) | Full Microsoft útgáfa, MS Remote-SSH virkar, telemetri má slökkva á |
| 19 | AUR helper | **`yay`** | Vinsælasta, vel viðhaldið, einfalt API |
| 20 | Shell | **bash** | Notandi „more familiar" |
| 21 | Terminal emulator | **`kitty`** + tóladekúr fyrir bash (sjá að neðan) | GPU-accelerated, ligature support, fallegt og hratt |

### Bash með litkóðun (skv. ósk notanda)

Til að bash skipanir séu litkóðaðar á svipaðan hátt og í zsh:

| Tól | Hlutverk | Heimild |
|-----|----------|---------|
| **`ble.sh`** | Bash Line Editor — bætir syntax highlighting og autosuggestions við bash | AUR: `blesh-git` |
| **`starship`** | Cross-shell prompt með git status, þemum, hraður | pacman: `starship` |
| **`eza`** | Litríkur `ls` afleysing | pacman: `eza` |
| **`bat`** | `cat` með syntax highlighting og line numbers | pacman: `bat` |
| **`fzf`** | Fuzzy finder (Ctrl-R history search, file search) | pacman: `fzf` |
| **`ripgrep`** (`rg`) | Fljótari `grep` með litum | pacman: `ripgrep` |
| **`zoxide`** | Snjall `cd` afleysing sem man möppur sem þú heimsækir oft | pacman: `zoxide` |

Sett upp í Fasa 2 og stillt í Fasa 3.

### Spurningar 22–29 (UI hönnun)

Frestað þar til notandi opnar Claude Design fyrir Fasa 3.

---

## Næstu skref

1. **Fasi 0** — sjá [`fasi-0-undirbuningur.md`](./fasi-0-undirbuningur.md) fyrir skref-fyrir-skref leiðbeiningar
2. **Áður en Fasi 1 hefst** — staðfesta RAM stærð, M.2 NVMe stærð, og hvort 2.5" SATA bay sé notuð
3. **Fasi 1** — Claude Code aðstoðar í terminal þegar notandi er kominn að root prompt á Arch live USB
