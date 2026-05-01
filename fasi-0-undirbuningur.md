# Fasi 0 — Undirbúningur

**Markmið:** Komast að root prompt á Arch live USB á NUC-inum, með allt tilbúið til að byrja Fasa 1.

**Hvar er þetta gert:** Mestmegnis á annarri tölvu (eða þessari ef þú ert á Linux). Aðeins skref 2 og 7 eru á NUC-inum.

---

## 0. Hvað þarf að vera við höndina

- [ ] USB drive, **≥ 4 GB** (verður þurrkaður út)
- [ ] Önnur tölva eða sími til að lesa Arch Wiki á meðan á uppsetningu stendur
- [ ] Wi-Fi SSID + lykilorð skrifað niður
- [ ] Mús, lyklaborð og skjár tengt við NUC (HDMI eða TB3→DP)
- [ ] Aflgjafi tengdur við NUC (90W brick)
- [ ] **Þolinmæði** — engin tímamörk, betra að gera þetta rétt

---

## 1. Backup á NUC-inum (ef einhver gögn eru á honum)

Ef einhver gögn eru á NUC-inum sem þú vilt halda → afritaðu á ytri disk eða ský **núna**. Allt á disknum verður þurrkað út í Fasa 1.

Ef NUC-inn er tómur eða þú ert ekki að missa neitt → slepptu þessu skrefi.

---

## 2. Athuga BIOS og vélbúnað á NUC

Þetta gerir þú **á NUC-inum sjálfum**.

1. Ræstu NUC. Á Intel splash screen, ýttu **F2** til að fara í BIOS (Visual BIOS).
2. Skráðu hjá þér:
   - **BIOS útgáfa** (efst á skjánum, t.d. `BECFL357.86A.0093...`)
   - **RAM stærð** (Memory tab — skráðu *hvert dimm slot* og hraða)
   - **Storage** (Devices → Drive Configuration — M.2 NVMe model og stærð, hvort 2.5" SATA er notuð og þá hvað)
3. Stilltu þetta:
   - **Boot → Boot Priority** → UEFI Boot ON, Legacy Boot **OFF**
   - **Boot → Secure Boot** → **Disabled** (við virkjum það seinna í Fasa 3+)
   - **Devices → SATA** → AHCI (default, en staðfesta)
   - **Devices → USB** → leyfa USB boot
4. **F10** til að vista og restart.

Þegar Claude Code spyr í Fasi 1, gefðu þér tíma til að segja:
- RAM stærð (t.d. „16 GB, 2× 8 GB DDR4-2400")
- Storage stærð (t.d. „500 GB NVMe, engin SATA")
- BIOS útgáfa

---

## 3. Sækja Arch ISO

Á tölvu sem þú ætlar að nota til að búa til USB-inn:

```bash
mkdir -p ~/Downloads/arch && cd ~/Downloads/arch

curl -O https://archlinux.org/iso/latest/archlinux-x86_64.iso
curl -O https://archlinux.org/iso/latest/archlinux-x86_64.iso.sig
curl -O https://archlinux.org/iso/latest/sha256sums.txt
curl -O https://archlinux.org/iso/latest/b2sums.txt
```

Á Windows: opnaðu vafra og sæktu sömu skrár frá <https://archlinux.org/download/>.

> ISO-inn er um 1.2 GB. Það getur tekið nokkrar mínútur.

---

## 4. Staðfesta ISO (mikilvægt!)

Tvær athuganir — hvor um sig staðfestir mismunandi hluti.

### 4a. SHA256 checksum (athugar að ISO sé ekki skemmd)

```bash
sha256sum -c sha256sums.txt --ignore-missing
```

Á að segja: `archlinux-x86_64.iso: OK`

### 4b. GPG signature (athugar að ISO sé ekki fölsk)

```bash
# Sækja Arch keyring lykla (ef ekki nú þegar)
sudo pacman-key --refresh-keys 2>/dev/null || \
  curl -O https://archlinux.org/iso/latest/archlinux-x86_64.iso.sig

# Staðfesta
gpg --keyserver-options auto-key-retrieve --verify archlinux-x86_64.iso.sig archlinux-x86_64.iso
```

Á að segja: `Good signature from "Pierre Schmitz <pierre@archlinux.de>"` (eða annar Arch developer).

> ⚠️ Ef checksum eða signature klikka — **ekki halda áfram**. Sæktu ISO-inn aftur.

---

## 5. Búa til bootable USB

> ⚠️ **VIÐVÖRUN:** Næsta skipun þurrkar allt út af USB drive-inum. Athugaðu nafnið **vandlega** áður en þú keyrir.

### Á Linux

```bash
# 1. Stinga USB-inum í (ekki í NUC enn — í þessa tölvu)
# 2. Finna nafnið hans (passa! /dev/sda er venjulega innri diskurinn)
lsblk -o NAME,SIZE,MODEL,TRAN
```

Þú átt að sjá USB-inn sem t.d. `/dev/sdb` (eða `/dev/sdc` ef þú ert með marga diska). Stærðin og MODEL hjálpa þér að bera kennsl á hann.

```bash
# 3. Skrifa á USB (skiptu sdX út fyrir rétta nafnið — t.d. sdb)
sudo dd bs=4M if=archlinux-x86_64.iso of=/dev/sdX conv=fsync oflag=direct status=progress
sync
```

Þetta tekur 2–5 mínútur. Þegar `dd` er búið og `sync` skilar, er USB tilbúinn.

### Á Windows

Notaðu **Rufus** (<https://rufus.ie>):

1. Device: USB drive
2. Boot selection: `archlinux-x86_64.iso`
3. Partition scheme: **GPT**
4. Target system: **UEFI (non CSM)**
5. Smelltu **START**, samþykktu **DD mode** ef beðið er um

### Á macOS

```bash
diskutil list
# Finna USB diskinn (t.d. /dev/disk4)
diskutil unmountDisk /dev/diskN
sudo dd bs=4m if=archlinux-x86_64.iso of=/dev/rdiskN status=progress
```

---

## 6. Lokatékk áður en haldið er í Fasa 1

- [ ] Bootable USB tilbúinn
- [ ] Wi-Fi SSID og lykilorð skrifað niður
- [ ] RAM stærð staðfest
- [ ] Storage stærð staðfest
- [ ] BIOS útgáfa skráð
- [ ] BIOS sett í UEFI mode + Secure Boot disabled
- [ ] Annar skjár/sími tilbúinn til að lesa <https://wiki.archlinux.org/title/Installation_guide>
- [ ] Mús, lyklaborð, skjár, aflgjafi tengt við NUC

---

## 7. Boot inn í Arch live USB

Núna gerir þú þetta **á NUC-inum**:

1. Stingdu USB-inum í USB port á NUC.
2. Ræstu NUC.
3. Á Intel splash screen, ýttu **F10** til að velja boot device.
4. Veldu USB-inn — gæti birst sem `UEFI: <USB nafn>` eða svipað.
5. Í Arch boot menu velurðu **„Arch Linux install medium (x86_64, UEFI)"**.
6. Bíddu í 30–60 sekúndur þangað til kemur:
   ```
   root@archiso ~#
   ```

**Þegar þú sérð root prompt — segðu Claude Code „komin á root prompt" og við byrjum Fasa 1.**

---

## Bilanaleit

**„No bootable device" eða boot menu sýnir ekki USB:**
- Athugaðu að USB sé í USB **3.0 (blár) port** að framan eða aftan
- Athugaðu að UEFI Boot sé virkt og Legacy Boot disabled
- Athugaðu að Secure Boot sé **disabled**
- Reyndu annan USB drive — sumir drives eru skrítnir með NUC-um

**Boot menu sýnir USB en hann hangir á svarta skjánum:**
- Bíddu lengur — fyrsti boot getur tekið 1–2 mínútur
- Reyndu „Arch Linux install medium" með `nomodeset` (þrýstu `e` í menu, bættu `nomodeset` aftan í línuna)

**Wi-Fi virkar ekki á live USB:**
- Förum í gegnum það í Fasa 1 — `iwctl` er ekki sjálfvirkt
