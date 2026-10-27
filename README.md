# Útmutató: x86_64 Ubuntu Server emulálása Apple Silicon (M1/M2/M3/M4) Mac gépeken UTM segítségével

> **Tantárgy:** Számítási rendszerek architektúrája (_Computer Systems Architecture_)  
> **Célközönség:** Egyetemi hallgatók (kezdők számára is részletes, lépésről lépésre követhető formátumban)  
> **Környezet:** macOS (Apple Silicon ARM64) $\rightarrow$ Emulált Ubuntu Server (x86_64 / amd64)

---

## 📖 Tartalomjegyzék

1. [Miért van szükség erre az emulációra? (A probléma háttere)](#1-miért-van-szükség-erre-az-emulációra-a-probléma-háttere)
2. [Előfeltételek és letöltések](#2-előfeltételek-és-letöltések)
3. [Virtuális gép létrehozása az UTM-ben (Lépésről lépésre)](#3-virtuális-gép-létrehozása-az-utm-ben-lépésről-lépésre)
4. [Az Ubuntu Server operációs rendszer telepítése lépésről lépésre](#4-az-ubuntu-server-operációs-rendszer-telepítése-lépésről-lépésre)
5. [A virtuális gép áttekintése az UTM-ben](#5-a-virtuális-gép-áttekintése-az-utm-ben)
6. [Hálózat és Porttovábbítás (Port Forwarding) beállítása](#6-hálózat-és-porttovábbítás-port-forwarding-beállítása)
7. [A szerver indítása és az SSH szolgáltatás konfigurálása](#7-a-szerver-indítása-és-az-ssh-szolgáltatás-konfigurálása)
8. [Csatlakozás a Mac termináljából](#8-csatlakozás-a-mac-termináljából)
9. [Kényelmi funkciók és laboratóriumi eszközök](#9-kényelmi-funkciók-és-laboratóriumi-eszközök)
10. [Gyakori kérdések és hibaelhárítás (FAQ)](#10-gyakori-kérdések-és-hibaelhárítás-faq)

---

## 1. Miért van szükség erre az emulációra? (A probléma háttere)

Az újabb Apple számítógépek (MacBook Air, MacBook Pro, Mac mini, Mac Studio, iMac) a modern **Apple Silicon (M1, M2, M3, M4)** processzorcsaládra épülnek, amelyek utasításkészlet-architektúrája **ARM64**.

Ezzel szemben a _Számítási rendszerek architektúrája_ laboratóriumi feladatai az **Intel/AMD x86_64** (64 bites PC) platformra tervezett Assembly nyelvet és eszközláncot követelik meg:

- A tanult regiszterek (`rax`, `rbx`, `eax`, `ebx` stb.) és gépi utasítások x86_64 specifikusak.
- A laborban használt segédkönyvtárak (`libio.a`, `io.inc`), a saját linker (`nlink`), valamint az automatikus kiértékelő program (`actest`) kizárólag **x86_64 Linux ELF** binárisként érhetők el.
- Az Apple Silicon gépeken futó macOS linker és a natív ARM-alapú GCC **nem képes x86_64 gépi kódú objektumfájlokat linkelni vagy futtatni**, mivel az utasításkészlet alapjaiban tér el.

### Mi a megoldás?

Egy teljes **x86_64 (amd64) architektúrájú Ubuntu Server** környezetet hozunk létre a Mac-en az ingyenes és nyílt forráskódú **UTM** virtualizációs/emulációs szoftver segítségével.

> [!NOTE]
> **Miért grafikus felület nélküli (Server) Linuxot használunk?**  
> Az eltérő processzorarchitektúra emulálása (ARM64 $\rightarrow$ x86_64) számításigényes feladat. A grafikus felhasználói felület (GUI) feleslegesen lassítaná a rendszert. Az Ubuntu Server minimális memóriát és processzorerőforrást fogyaszt, a háttérben fut, és SSH-n keresztül közvetlenül a macOS megszokott, kényelmes termináljából (vagy VS Code-ból) dolgozhatunk benne.

---

## 2. Előfeltételek és letöltések

Mielőtt elkezdenéd a telepítést, töltsd le a következő két komponenst:

1. **UTM Virtual Machine szoftver macOS-re:**
   - Honlap: [https://mac.getutm.app](https://mac.getutm.app)
   - Töltsd le az ingyenes `.dmg` telepítőt a _Download_ gombra kattintva, majd húzd be az `Applications` (Alkalmazások) mappába.

2. **Ubuntu Server x86_64 telepítő lemezkép (ISO):**
   - Honlap: [Ubuntu hivatalos letöltési oldal](https://ubuntu.com/download/server)
   - **Kifejezetten fontos:** Győződj meg róla, hogy a letöltött fájl neve tartalmazza az **`amd64`** vagy **`x86_64`** jelölést (például: `ubuntu-24.04-live-server-amd64.iso`), és **nem** az `arm64`-et!

3. **Egy megosztott mappa létrehozása a Mac-en (opcionális, de erősen ajánlott):**
   - Hozz létre a Mac-eden egy mappát (például a Dokumentumok mappában `Ubuntu-server` vagy `AC-Labor` néven), ahol a feladataidat és assembly fájljaidat tárolod.

---

## 3. Virtuális gép létrehozása az UTM-ben (Lépésről lépésre)

Indítsd el a frissen telepített **UTM** alkalmazást a Mac-eden. Kövesd az alábbi lépéseket a mellékelt képernyőképek alapján!

### 3.1. Új virtuális gép indítása

Az UTM nyitóképernyőjén kattints a **`Create a New Virtual Machine`** (Új virtuális gép létrehozása) gombra, vagy a bal felső sarokban található **`+`** ikonra.

![1. lépés: Új virtuális gép létrehozása](./utm/Screenshot%202026-10-26%20at%2017.13.png)

---

### 3.2. Működési mód kiválasztása: Emulate (Emuláció)

A megjelenő ablakban két opció fogad:

- _Virtualize_: Gyorsabb, de kizárólag a Mac natív (ARM) architektúráját képes futtatni.
- **`Emulate`**: Ezt **kell** választanunk! Lassabb, viszont képes más CPU-architektúrákat (esetünkben x86_64-et) emulálni.

Kattints a piros nyíllal jelölt **`Emulate`** lehetőségre!

![2. lépés: Emulate kiválasztása](./utm/Screenshot%202026-10-26%20at%2017.17.png)

---

### 3.3. Operációs rendszer típus: Linux

A felkínált operációs rendszerek listájából válaszd a **`Linux`** opciót.

![3. lépés: Linux kiválasztása](./utm/Screenshot%202026-10-26%20at%2017.18.png)

---

### 3.4. Hardver konfiguráció (Memória és CPU)

Itt állíthatod be, hogy az emulált gép mekkora hardveres erőforrással gazdálkodhasson:

- **Architecture / Machine:** Alapértelmezetten válaszd az x86*64 kompatibilis PC géptípust (\_Intel ICH9 based PC* vagy _pc-q35_).
- **Memory (RAM):** Állíts be legalább **4096 MiB**-ot (4 GB). Ha a Mac-edben 8 GB memória van, a 3072–4096 MiB ideális; ha 16 GB vagy több, maradhat a 4096 MiB.
- **CPU Cores:** Adj meg **4** magot a zökkenőmentes fordítás és háttérmunka érdekében.

Kattints a **`Continue`** (Folytatás) gombra.

![4. lépés: Hardver beállítások](./utm/Screenshot%202026-10-26%20at%2017.19.png)

---

### 3.5. A letöltött Ubuntu telepítő ISO kiválasztása

A lemezkép konfigurációs ablakban:

1. Győződj meg róla, hogy a **`Boot from ISO image`** jelölőnégyzet be van pipálva.
2. Kattints a **`Browse...`** gombra, és tallózd be a letöltések mappából az előzőleg letöltött Ubuntu Server ISO fájlt (pl. `ubuntu-...-live-server-amd64.iso`).
3. Miután a fájl neve megjelent a mezőben, kattints a **`Continue`** gombra.

![5. lépés: ISO lemezkép betallózása](./utm/Screenshot%202026-10-26%20at%2017.23.png)

---

### 3.6. Tárhely (Storage) méretének megadása

Add meg a virtuális merevlemez méretét:

- Ajánlott méret: **`64 GiB`** (vagy legalább 30–40 GiB).
- _Megjegyzés:_ Ez a méret dinamikusan allokált, ami azt jelenti, hogy kezdetben nem foglal el ennyi valódi helyet a Mac SSD-jén, csak annyit, amennyi ténylegesen fel van telepítve.

Kattints a **`Continue`** gombra.

![6. lépés: Merevlemez mérete](./utm/Screenshot%202026-10-26%20at%2017.24.png)

---

### 3.7. Megosztott mappa kijelölése (Shared Directory)

A virtuális gép és a gazdagép (Mac) közötti kényelmes fájlátvitelhez jelölj ki egy megosztott mappát:

1. Kattints a **`Browse...`** gombra.
2. Válaszd ki az előzőleg létrehozott mappát a Mac-eden (pl. `Ubuntu-server`).
3. Kattints a **`Continue`** gombra.

![7. lépés: Megosztott mappa kijelölése](./utm/Screenshot%202026-10-26%20at%2017.25.png)

---

### 3.8. Összegzés és Mentés

Az összefoglaló ablakban:

- Adj nevet a virtuális gépnek a **Name** mezőben (például: **`AC-Linux-Server`**).
- Ellenőrizd a beállításokat:
  - Architecture: **x86_64**
  - RAM: **4 GB**
  - CPU: **4 Cores**
  - Storage: **64 GB**
- Kattints a **`Save`** (Mentés) gombra.

![8. lépés: Összegzés és mentés](./utm/Screenshot%202026-10-26%20at%2017.26.png)

---

## 4. Az Ubuntu Server operációs rendszer telepítése lépésről lépésre

A virtuális gép mentése után az új géped megjelenik az UTM bal oldali oldalsávjában `AC-Linux-Server` néven. Kattints az UTM ablakában található nagy **Lejátszás (▶)** gombra a gép elindításához! A virtuális gép betölti a csatolt Ubuntu Server telepítőlemezt.

![A virtuális gép elindítása az UTM-ben](./utm/Screenshot%202026-10-26%20at%2017.27.png)

> [!TIP]
> **Navigáció a szöveges telepítőfelületen (Subiquity):**
>
> - A menüpontok és gombok között a billentyűzet nyilaival (`↑`, `↓`) tudsz mozogni.
> - Egy jelölőnégyzet bepipálása vagy opció kijelölése a **Szóköz (Space)** lenyomásával történik.
> - A választás megerősítéséhez és továbblépéshez használd az **Enter** billentyűt (vagy lépj a zölddel jelölt `[ Done ]` / `[ Continue ]` gombra és üss Entert).

---

### 4.1. GRUB indítómenü

A gép indulásakor a GNU GRUB bootmenü fogad. Válaszd a legfelső **`Try or Install Ubuntu Server`** opciót, majd üss **Enter**-t.

![GRUB indítómenü](./linux_setup/Timeline%201_01_01_06_57.jpg)

---

### 4.2. Nyelv kiválasztása (Welcome!)

Válaszd ki a telepítő alapértelmezett nyelvét (ajánlott az **`English`** a nemzetközi terminológia és a könnyebb hibakeresés miatt), majd nyomj **Enter**-t.

![Nyelvválasztás](./linux_setup/Timeline%201_01_01_11_20.jpg)

---

### 4.3. Telepítő frissítése (Installer update available)

Amennyiben az Ubuntu jelzi, hogy újabb verzió érhető el a telepítőből, választhatsz az új telepítő letöltése (_Update to the new installer_), vagy a jelenlegi folytatása (_Continue without updating_) között. Üss **Enter**-t a továbblépéshez.

![Telepítő frissítése](./linux_setup/Timeline%201_01_01_17_15.jpg)

---

### 4.4. Billentyűzetkiosztás (Keyboard configuration)

Válaszd ki a fizikai billentyűzeted kiosztását (pl. `English (US)` vagy `Hungarian`). A nyilakkal lépj le a képernyő alján lévő **`[ Done ]`** gombra, majd nyomj **Enter**-t.

![Billentyűzetkiosztás](./linux_setup/Timeline%201_01_01_18_34.jpg)

---

### 4.5. Telepítés típusa (Choose the type of installation)

Válaszd az alapértelmezett **`(X) Ubuntu Server`** opciót. Lépj a **`[ Done ]`** gombra, és nyomj **Enter**-t.

![Telepítés típusa](./linux_setup/Timeline%201_01_01_22_46.png)

---

### 4.6. Hálózati konfiguráció (Network configuration)

A rendszer automatikusan észleli az emulált hálózati kártyát, és DHCP-n keresztül beállítja. Nem szükséges módosítani: lépj a **`[ Done ]`** gombra, és üss **Enter**-t.

![Hálózati konfiguráció](./linux_setup/Timeline%201_01_01_26_52.jpg)

---

### 4.7. Proxy konfiguráció (Proxy configuration)

Hacsak nem használsz külön egyetemi proxy szervert az internethez, hagyd üresen a mezőt. Lépj a **`[ Done ]`** gombra, és nyomj **Enter**-t.

![Proxy konfiguráció](./linux_setup/Timeline%201_01_01_28_51.jpg)

---

### 4.8. Ubuntu archívum tükörszerver (Mirror configuration)

A telepítő felajánlja a legközelebbi hivatalos Ubuntu tükörszervert (pl. `http://archive.ubuntu.com/ubuntu`). Hagyd az alapértelmezett értéken, lépj a **`[ Done ]`** gombra, és üss **Enter**-t.

![Tükörszerver](./linux_setup/Timeline%201_01_01_31_09.png)

---

### 4.9. Tárhely kiosztás (Guided storage configuration)

Hagyd bejelölve az alapértelmezett **`(X) Use an entire disk`** (a teljes virtuális merevlemez használata) lehetőséget. Lépj a **`[ Done ]`** gombra, és nyomj **Enter**-t.

![Tárhely kiosztás](./linux_setup/Timeline%201_01_01_35_11.jpg)

---

### 4.10. Fájlrendszer összegzés (Storage configuration)

A telepítő áttekintést ad a virtuális meghajtó partícióiról (`/boot`, `/boot/efi`, root partíció). Ellenőrzés után lépj a **`[ Done ]`** gombra, és üss **Enter**-t.

![Fájlrendszer összegzés](./linux_setup/Timeline%201_01_01_38_27.jpg)

---

### 4.11. Formázás megerősítése (Confirm destructive action)

Megjelenik egy biztonsági megerősítő párbeszédablak, amely rákérdez, hogy biztosan megkezded-e a partíciók írását. A nyilakkal válaszd a **`[ Continue ]`** (Folytatás) gombot, majd nyomj **Enter**-t.

![Formázás megerősítése](./linux_setup/Timeline%201_01_01_42_08.jpg)

---

### 4.12. Felhasználói profil létrehozása (Profile configuration)

Töltsd ki a felhasználói adataidat:

- **Your name:** A teljes neved (pl. `david`)
- **Your server's name:** A virtuális szerver neve (pl. `ac-server` vagy `david`)
- **Pick a username:** A bejelentkezési felhasználóneved csupa kisbetűvel (pl. `david`)
- **Choose a password / Confirm your password:** Egy tetszőleges, biztonságos jelszó, amit **jól jegyezz meg**! Javasolt a **root** jelszó használata!

Miután mindegyik mezőt kitöltötted, lépj a **`[ Done ]`** gombra, és nyomj **Enter**-t.

![Felhasználói profil](./linux_setup/Timeline%201_01_01_43_57.png)

---

### 4.13. Ubuntu Pro frissítés kihagyása (Upgrade to Ubuntu Pro)

Válaszd a **`(X) Skip for now`** (Kihagyás egyelőre) opciót, majd lépj a **`[ Continue ]`** gombra, és üss **Enter**-t.

![Ubuntu Pro kihagyása](./linux_setup/Timeline%201_01_01_50_22.jpg)

---

### 4.14. SSH kiszolgáló engedélyezése (SSH configuration)

> [!IMPORTANT]
> A Szóköz (Space) lenyomásával feltétlenül jelöld be az **`[X] Install OpenSSH server`** lehetőséget! Ez az összetevő szükséges ahhoz, hogy a Mac Termináljából vagy VS Code-ból távolról be tudj lépni a szerverre.

A bejelölés után lépj a **`[ Done ]`** gombra, és nyomj **Enter**-t.

![SSH konfiguráció](./linux_setup/Timeline%201_01_01_52_08.png)

---

### 4.15. Kiszolgáló snap csomagok (Featured server snaps)

A felkínált népszerű kiszolgálócsomagok közül a laborfeladatokhoz semmire sincs szükségünk. Nyomj **Enter**-t közvetlenül a **`[ Done ]`** gombon a továbblépéshez.

![Kiszolgáló snap csomagok](./linux_setup/Timeline%201_01_01_57_49.jpg)

---

### 4.16. A rendszer telepítési folyamata

A telepítő megkezdi a rendszerfájlok kiírását és a csomagok konfigurálását. Mivel a Mac egy x86_64 gépet emulál, ez a lépés gépsebességtől függően 5–15 percet is igénybe vehet.

![Telepítési folyamat](./linux_setup/Timeline%201_01_02_01_57.jpg)

---

### 4.17. Telepítés befejezése és újraindítás (Reboot Now)

Amikor a képernyő alsó részén megjelenik a **`[ Reboot Now ]`** gomb, nyomj **Enter**-t az újraindításhoz!

![Telepítés kész - Reboot Now](./linux_setup/Timeline%201_01_02_07_29.jpg)

---

### 4.18. A virtuális gép leállítása a hálózati porttovábbításhoz

Az újraindítás után (vagy a lemez leválasztásakor) állítsd le a virtuális gépet, hogy elvégezhessük az UTM hálózati beállításait:

1. Az UTM ablakának bal felső részén kattints a leállítás gombra, vagy a menüben válaszd az újraindítás/leállítás opciót (_Shuts down and stops the VM_).
2. Erősítsd meg a leállítást.

![Virtuális gép leállítása](./linux_setup/Timeline%201_01_02_10_58.jpg)

![Leállítás jóváhagyása](./linux_setup/Timeline%201_01_02_12_30.jpg)

---

## 5. A virtuális gép áttekintése és a telepítőlemez leválasztása

A leállítást követően a virtuális gép `Stopped` (Leállítva) státuszba kerül. Az UTM-ben az `AC-Linux-Server` adatlapjára kattintva láthatod a hardveres összegzést és a csatolt CD/DVD lemezt:

- A képernyő alján a CD/DVD bejegyzésnél kattints a **`Clear`** gombra, hogy a telepítő ISO lemez eltávolításra kerüljön, és a gép a jövőben közvetlenül a telepített virtuális merevlemezről induljon!

![Adatlap és csatolt perifériák](./utm/Screenshot%202026-10-26%20at%2017.36.png)

---

## 6. Hálózat és Porttovábbítás (Port Forwarding) beállítása

> [!IMPORTANT]
> **Miért elengedhetetlen ez a lépés?**  
> A virtuális gép az UTM-en belül egy zárt belső hálózaton működik. Ahhoz, hogy a Mac saját Termináljából SSH protokollal be tudj jelentkezni az emulált Ubuntu gépre, a Mac egy adott portját (pl. `2222`) át kell irányítani a virtuális gép 22-es (SSH) portjára.

### 6.1. A virtuális gép beállításainak megnyitása

Jelöld ki az `AC-Linux-Server` gépet a listában, majd kattints a jobb felső sarokban található **Beállítások ikonra** (csúszkák / fogaskerék ikon), vagy kattints jobb gombbal a gép nevére és válaszd az **`Edit`** (Szerkesztés) menüpontot.

![Beállítások megnyitása](./utm/Screenshot%202026-10-26%20at%2017.371.png)

---

### 6.2. Hálózati mód átállítása: Emulated VLAN

A beállítások bal oldali menüjében keresd meg a **`Network`** (Hálózat) pontot:

1. Kattints a **Network Mode** legördülő menüre.
2. Változtasd meg a beállítást _Shared Network_-ről a piros nyíllal jelölt **`Emulated VLAN`** értékre! (Ez a mód teszi lehetővé a QEMU belső port forward szabályainak kezelését).

![Hálózati mód átállítása Emulated VLAN-ra](./utm/Screenshot%202026-10-26%20at%2017.372.png)

---

### 6.3. Porttovábbítási szabály hozzáadása (Port Forwarding)

A Network beállításokon belül válaszd a **`Port Forwarding`** (Porttovábbítás) lapot:

1. Kattints a **`New...`** (Új) gombra a szabály hozzáadásához.
2. Töltsd ki a paramétereket pontosan a következők szerint:
   - **Protocol:** `TCP`
   - **Guest Address:** hagyd üresen is(`10.0.2.15`)
   - **Guest Port:** **`22`** _(ez az Ubuntu belső SSH portja)_
   - **Host Address:** hagyd üresen is (`0.0.0.0`)
   - **Host Port:** **`2222`** _(ezen a porton keresztül fogunk kapcsolódni a Mac-ről)_
3. Kattints a szabály melletti **`Save`**, majd a beállítóablak jobb alsó sarkában található **`Save`** gombra!

![Porttovábbítási szabály rögzítése](./utm/Screenshot%202026-10-26%20at%2017.381.png)

---

## 7. A szerver indítása és az SSH szolgáltatás konfigurálása

Most, hogy a virtuális gép és a hálózati porttovábbítás készen áll, indítsd el a gépet az UTM-ben található nagy **Lejátszás (▶)** gombbal.

### 7.1. Bejelentkezés és SSH szerver ellenőrzése

Amikor megjelenik a fekete konzolos képernyő a bejelentkezési kéréssel:

```plaintext
ac-server login: <a_te_felhasznaloneved>
Password: <a_te_jelszavad>
```

_(Gépelés közben a jelszavad biztonsági okokból nem látszik – ez normális! Gépeld be, majd nyomj Entert)._

Miután sikeresen beléptél, futtasd az alábbi parancsokat egymás után:

#### 1. Csomaglista frissítése

```bash
sudo apt update
```

_(Ha kéri a jelszót, írd be ismét)._

#### 2. OpenSSH szerver telepítése (ha a telepítéskor kimaradt volna)

```bash
sudo apt install openssh-server -y
```

#### 3. Az SSH szolgáltatás állapotának ellenőrzése

```bash
sudo systemctl status ssh
```

Ha zöld fénnyel a **`active (running)`** feliratot látod, az SSH szerver sikeresen fut!  
A státusz nézetből a **`q`** billentyű lenyomásával tudsz kilépni.

---

### 7.2. Opcionális: Hibaelhárítás, ha az SSH szerver nem indulna el automatikusan

Amennyiben az SSH szerver státusza nem `active (running)`, futtasd az alábbi karbantartó parancssort a kulcsok és jogosultságok helyreállításához:

```bash
# SSH automatikus indulásának engedélyezése
sudo systemctl enable --now ssh

# Konfiguráció szintaktikai ellenőrzése
sudo sshd -t

# Hiányzó host kulcsok legenerálása
sudo ssh-keygen -A

# Futtatási könyvtár és jogosultságok létrehozása
sudo mkdir -p /run/sshd
sudo chmod 0755 /run/sshd

# Szolgáltatás újraindítása és újraellenőrzése
sudo systemctl restart ssh
sudo systemctl status ssh
```

---

## 8. Csatlakozás a Mac termináljából

Innentől kezdve **nem szükséges az UTM fekete képernyős kis ablakában gépelned**! A virtuális gép csendben maradhat a háttérben.

### 8.1. A macOS Terminál megnyitása

1. Nyomd meg a billentyűzeten a **`Cmd + Space`** kombinációt a Spotlight kereső megnyitásához.
2. Írd be: `Terminal` (vagy `Terminál`), majd üss `Enter`-t.

---

### 8.2. SSH csatlakozási parancs futtatása

Írd be a következő parancsot, helyettesítve a `<felhasznalonev>` részt azzal a felhasználónévvel, amit az Ubuntu telepítésekor megadtál:

```bash
ssh <felhasznalonev>@localhost -p 2222
```

_Példa:_

```bash
ssh david@localhost -p 2222
```

#### Mit csinál ez a parancs?

- `ssh`: A biztonságos távoli bejelentkező program.
- `<felhasznalonev>@localhost`: Megmondja, hogy a helyi gépeden (`localhost`) található virtuális rendszerre akarsz bejelentkezni a saját fiókoddal.
- `-p 2222`: Megadja a portszámot. Emlékezz: az UTM-ben a 2222-es portot irányítottuk át a virtuális gép 22-es SSH portjára!

---

### 8.3. Első csatlakozás: Kulcs elfogadása

A legelső kapcsolódáskor a terminál a következőhöz hasonló biztonsági figyelmeztetést jelenít meg:

```plaintext
The authenticity of host '[localhost]:2222 ([127.0.0.1]:2222)' can't be established.
ED25519 key fingerprint is SHA256:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx.
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

1. Írd be: **`yes`** és nyomj `Enter`-t!
2. Add meg a jelszavadat, majd nyomj `Enter`-t!

Gratulálunk! Ekkor a terminál promptja megváltozik (pl. `david@ac-server:~$`), ami azt jelenti, hogy sikeresen bent vagy a 64 bites x86_64 Ubuntu környezetedben!

---

## 9. Kényelmi funkciók és laboratóriumi eszközök

### 9.1. Gyorsabb csatlakozás beállítása (SSH Config)

Hogy ne kelljen minden alkalommal begépelned a portszámot és címet, hozz létre egy egyszerű hivatkozást a Mac-eden:

A Mac termináljában (nem a virtuális gépben!) futtasd:

```bash
nano ~/.ssh/config
```

Illeszd be a következő sorokat (a `User` értékét írd át a sajátodra):

```ssh-config
Host ac-server
    HostName localhost
    Port 2222
    User david
```

Mentsd el (`Ctrl + O`, majd `Enter`, kilépés: `Ctrl + X`).

Ezután a jövőben elegendő csupán ennyit beírnod a Mac termináljába:

```bash
ssh ac-server
```

---

### 9.2. VS Code használata SSH-n keresztül (Kényelmes grafikus szerkesztés!)

Nem szükséges a terminálban fapados szöveges szerkesztőket (pl. `nano` vagy `vim`) használnod: a **Visual Studio Code** segítségével a Mac-en megszokott, kényelmes grafikus kódszerkesztőben dolgozhatsz úgy, hogy a fájlok valójában az emulált Ubuntu virtuális szerveren élnek, és az assembly fordítás is ott zajlik a háttérben.

Kövesd az alábbi lépéseket a beállításhoz a mellékelt képernyőképek alapján:

#### 9.2.1. A Remote - SSH bővítmény telepítése

1. Nyisd meg a VS Code-ot a Mac-eden.
2. Kattints a bal oldali sávban található **Extensions** (Bővítmények) ikonra (vagy nyomd le a `Cmd + Shift + X` billentyűkombinációt).
3. A keresőmezőbe írd be: `remote-ssh`.
4. Keresd meg a hivatalos, Microsoft által fejlesztett **`Remote - SSH`** bővítményt, és kattints az **`Install`** (Telepítés) gombra.

![Remote - SSH bővítmény telepítése](./vscode/Screenshot%202026-10-27%20at%2014.47.00.png)

---

#### 9.2.2. Kapcsolódás távoli géphez (Connect to Host...)

A bővítmény feltelepülése után:

1. Kattints a VS Code ablakának legalsó kék sávjában, a bal sarokban megjelenő kis távoli kapcsolat ikonra (**`><`**), vagy nyomd le az **`F1`** (esetleg `Cmd + Shift + P`) billentyűt a parancspaletta megnyitásához.
2. A felül legördülő menüből válaszd a piros kerettel jelölt **`Connect to Host...`** lehetőséget.

![Kapcsolódás a gazdagéphez](./vscode/Screenshot%202026-10-27%20at%2014.47.16.png)

---

#### 9.2.3. Új SSH kapcsolat felvétele (+ Add New SSH Host...)

A listából válaszd a piros nyíllal jelölt **`+ Add New SSH Host...`** opciót.

![Új SSH gazdagép hozzáadása](./vscode/Screenshot%202026-10-27%20at%2014.47.17.png)

---

#### 9.2.4. Kapcsolódási parancs megadása

A felugró beviteli mezőbe gépeld be pontosan az alábbi parancsot a saját felhasználóneveddel:

```bash
ssh <felhasznalonev>@localhost -p 2222
```

_(Például: `ssh david@localhost -p 2222`)_  
Ezután nyomj **Enter**-t a megerősítéshez.

![SSH parancs megadása](./vscode/Screenshot%202026-10-27%20at%2014.47.54.png)

---

#### 9.2.5. Konfigurációs fájl kiválasztása

A VS Code megkérdezi, hogy hová mentse el ezt a kapcsolatot. Válaszd a leg első, saját felhasználói konfigurációs fájlodat:  
**`/etc/ssh/ssh_config`**

![SSH config kiválasztása](./vscode/Screenshot%202026-10-27%20at%2014.48.05.png)

---

#### 9.2.6. Csatlakozás a szerverhez és munkakezdés

1. A jobb alsó sarokban megjelenő értesítésben kattints a **`Connect`** gombra, vagy nyisd meg újra a távoli menüt (`><` gomb $\rightarrow$ `Connect to Host...`), és válaszd a listában megjelenő **`localhost`** elemet!
2. Ha a VS Code rákérdez a távoli operációs rendszer típusára, válaszd a **`Linux`** opciót.
3. Írd be a telepítéskor megadott jelszavadat, majd üss **Enter**-t.
4. A VS Code bal alsó sarkában ekkor megjelenik a kék/zöld **`SSH: localhost`** felirat, jelezve, hogy sikeresen és közvetlenül az emulált Ubuntu gépen dolgozol!

![Csatlakozás a szerverhez](./vscode/Screenshot%202026-10-27%20at%2014.48.png)

> [!TIP]
> **Hogyan dolgozz ezután a VS Code-ban?**
>
> - **Mappa megnyitása:** A bal oldali _Explorer_ fülön kattints az **`Open Folder`** gombra, és válaszd ki a saját mappádat (pl. `/home/<felhasznalo>/` vagy a labor projektmappáját).
> - **Beépített terminál:** A felső menüből válaszd a `Terminal -> New Terminal` opciót (vagy nyomd le a `Ctrl + ~` kombinációt). Ez közvetlenül a virtuális Ubuntu szerver terminálját nyitja meg a VS Code ablakának alján! Itt azonnal futtathatod az `nasm`, `gcc`, `actest` parancsokat, miközben fent kényelmesen szerkeszted az assembly forráskódot.

---

### 9.3. Alapvető assembly fejlesztői eszközök telepítése az Ubuntun

Miután bejelentkeztél az Ubuntu Serverre, telepítsd fel az alapvető laboratóriumi eszközöket:

```bash
sudo apt update
sudo apt install nasm gcc libc6-dev binutils
```

Architektúra ellenőrzése az Ubuntu termináljában:

```bash
uname -m
```

Kimenet: **`x86_64`**  
_(Ez igazolja, hogy a környezet valóban 64 bites Intel/AMD utasításkészlettel fut, kompatibilisen a tantárgyi követelményekkel)._

---

## 10. Gyakori kérdések és hibaelhárítás (FAQ)

### K: Telepíthetek grafikus felületet (Desktop / GUI) az Ubuntu Serverre?

Igen, amennyiben mindenképpen grafikus asztali környezetet szeretnél a parancssor helyett, a következő parancs futtatásával telepítheted a hivatalos Ubuntu asztali felületet:

```bash
sudo apt install ubuntu-desktop -y
```

> [!WARNING]
> **Figyelem!** Apple Silicon Mac gépeken az x86*64 emuláció CPU- és számításigényes feladat. A grafikus felület (GNOME / Ubuntu Desktop) betöltése jelentősen megnöveli a memóriahasználatot és érezhetően lassabbá teszi a virtuális gépet.  
> A \_Számítási rendszerek architektúrája* laborfeladataihoz (NASM, GCC, nlink, actest) nincs szükség grafikus felületre: a közvetlen terminálos SSH-kapcsolat vagy a [VS Code Remote - SSH](#92-vs-code-használata-ssh-n-keresztül-kényelmes-grafikus-szerkesztés) kiegészítő nagyságrendekkel gyorsabb, kényelmesebb és korszerűbb fejlesztői élményt biztosít!

### K: Hogyan tudom biztonságosan leállítani a szervert, ha végeztem?

Mindig a következő paranccsal állítsd le a Linuxot, mielőtt bezárod az UTM-et:

```bash
sudo poweroff
```

Így elkerülhető a virtuális fájlrendszer sérülése.

### K: Miért kapok `Connection refused` hibát a Mac termináljában?

- Ellenőrizd az UTM-ben, hogy a virtuális gép valóban el van-e indítva (zöld futásjelző).
- Ellenőrizd a 6. fejezetben leírt Port Forwarding beállítást (Host Port: 2222, Guest Port: 22).
- Győződj meg róla, hogy a hálózati mód **Emulated VLAN**-ra van állítva.

### K: Nagyon lassan indul vagy fut a virtuális gép, ez normális?

Igen, az emuláció (ARM processzor futtat x86 kódokat) számításigényes. Ezért használunk grafikus felület nélküli Ubuntu Servert, amely betöltődés után a memóriában nagyon kevés erőforrást igényel, így az assembly fordítás és futtatás gyors marad.

---

> Készült a **Babeș–Bolyai Tudományegyetem** _Számítási rendszerek architektúrája_ kurzusához.  
> Kérdések vagy problémák esetén fordulj a laborvezetőhöz!
