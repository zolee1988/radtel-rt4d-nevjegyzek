## Inspiráció és köszönet

Ez a script közvetlenül a **W1AEX** által publikált kiváló útmutató alapján készült:

> **https://www.w1aex.com/dmrid/dmrid.html**

W1AEX kitalálta, hogyan alakítsa át a [RadioID.net](https://radioid.net)-ről származó user.csv-t, majd lépésről-lépésre dokumentálta a teljes folyamatot az RT-4D közösség számára.  
Ez a script mindössze automatizálja azt, amit W1AEX leírt, kicsit átalakítva a magyarországi DMR felhasználókra szűkítve.

Köszönjük, W1AEX, hogy elvégezted a nehéz munkát, és megosztottad a közösséggel.

---

## A probléma

A Radtel RT-4D egy jól használható DMR kézirádió, de a kijelzője és a címjegyzéke szigorú korlátokkal rendelkezik:

- **3 soros kijelző**, soronként legfeljebb **18 karakterrel**
- A címjegyzék maximális mérete **12 289 kB**

A RadioID.net nyers kontaktadatbázisa túl nagy és túl rendezetlen ahhoz, hogy közvetlenül betölthető legyen a rádióba. A mezők értékei hosszúak és nincsenek rövidítve, a felesleges oszlopok csak foglalják a helyet, és a név két külön mezőben szerepel, amit az RT-4D nem tud hatékonyan kihasználni.

Ennek a scriptnek a célja, hogy az adatokat összezsugorítsa és átalakítsa, hogy azok beleférjenek a rádió korlátaiba, miközben továbbra is tisztán és áttekinthetően jelennek meg azon a három kis soron. Kifejezetten a magyarországi DMR felhasználókat listázza, és mivel kis ország vagyunk, a régiók helyett a település nevét írja ki.

---

## Az adatforrás

A [RadioID.net](https://radioid.net) a világ rádióamatőreinek közösségi fenntartású, hiteles nyilvántartása, amely a **DMR ID-ket** (Digital Mobile Radio azonosítókat) tartalmazza.

### Mi az a DMR ID?

A DMR egy széles körben használt digitális üzemmód a rádióamatőrök körében.  
Minden olyan engedéllyel rendelkező rádióamatőr, aki DMR hálózatokat (például BrandMeister, DMR-MARC vagy TGIF) szeretne használni, egyedi numerikus azonosítót kap, ez a **DMR ID** vagy **Radio ID**.  
Ezeket az azonosítókat a RadioID.net adja ki, miután ellenőrizte a hivatalos rádióamatőr engedélyadatbázisok alapján.

### A `user.csv` adatbázis

A RadioID.net a teljes felhasználói adatbázist közzéteszi itt:

```
https://radioid.net/static/user.csv
```

Ez a fájl rendszeresen frissül, és minden regisztrált DMR-felhasználóról egy sort tartalmaz a következő mezőkkel:

| Oszlop | Leírás |
|---|---|
| `RADIO_ID` | A felhasználó egyedi numerikus DMR azonosítója |
| `CALLSIGN` | A rádióamatőr hívójele |
| `FIRST_NAME` | Keresztnév |
| `LAST_NAME` | Vezetéknév |
| `CITY` | Település |
| `STATE` | Állam, megye vagy régió |
| `COUNTRY` | Ország |

Az adatbázis **több százezer rádióamatőrt tartalmaz több mint 150 országból**, így ez az egyik legátfogóbb, nyilvánosan elérhető rádióamatőr-adatforrás.
Mivel közösségi alapú és hivatalos engedélyekhez kötött, jól tükrözi az aktuálisan aktív DMR-felhasználókat.

---

## Mit csinál a script?

A mi módosított scriptünk a következő átalakításokat végzi el (a W1AEX által leírt módszer alapján), **kifejezetten a magyar rekordokra** és a **CITY mező megtartására** optimalizálva:

1. **Letölti** a `user.csv` fájlt
2. **Ellenőrzi**, hogy az oszlopok szerkezete megegyezik‑e a RadioID.net által használt formátummal; ha változás történt, a script hibával leáll
3. **Kiszűri** az összes rekord közül csak azokat, ahol a `COUNTRY` mező értéke **"Hungary"**
4. **Összefűzi** a `FIRST_NAME` és `LAST_NAME` mezőket egyetlen névmezővé, legfeljebb **18 karakterig**
5. **Kiüríti** a `LAST_NAME` mezőt (az RT‑4D számára nincs rá szükség)
6. **Meghagyja** a `CITY` mezőt, és **14 karakterre rövidíti**
7. **A COUNTRY mezőt** minden magyar rekordnál **HUN** értékre állítja
8. **Hozzáad** egy üres záró oszlopot (az RT‑4D CPS formátuma ezt megköveteli)
9. **Kiírja** az eredményt a `user_rt4d_hungary.csv` fájlba
10. **Ellenőrzi** a kimeneti fájl méretét, és figyelmeztet, ha meghaladja az RT‑4D 12 289 KB‑os címjegyzék‑limitjét

---

## Követelmények

- Python 3.8+
- [`requests`](https://pypi.org/project/requests/)

A szükséges függőség telepítése::

```bash
pip install requests
```

---

## Használat

```bash
# Alap használat — a user_rt4d_hungary.csv fájlt írja az aktuális könyvtárba
python dmrid_hungary_download.py

# Egyedi kimeneti útvonal megadása
python dmrid_hungary_download.py --output /path/to/my_contacts.csv

# A letöltött nyers fájl megtartása ellenőrzéshezn
python dmrid_hungary_download.py --keep-download
```

### Options

| Kapcsoló | Alapértelmezés | Leírás |
|---|---|---|
| `--output` | `user_rt4d_hungary.csv` | A formázott kimeneti fájl elérési útja |
| `--keep-download` | off | A nyers letöltött CSV megtartása a feldolgozás után is |

---

## Importálás az RT-4D CPS-be

1. Nyisd meg az RT-4D CPS szoftvert
2. Navigálj az **Address Book** menüpontra
3. Válaszd az **Import** funkciót, majd tallózd be a generált `user_rt4d_hungary.csv` fájlt
4. **Write**-tal írd bele a rádióba

---

## Előre legenerált adatbázis

Ha nem szeretnéd minden alkalommal lefuttatni a scriptet, az elkészített magyar adatbázis közvetlenül is letölthető innen:

**https://github.com/zolee1988/radtel-rt4d-nevjegyzek/blob/main/user_rt4d_hungary.csv**

Ez a fájl ugyanazzal a módszerrel készült, mint amit a script használ, és teljesen kompatibilis az RT‑4D CPS import funkciójával.

---

## Ha szeretnéd támogatni a projektet és a munkámat, PayPal-on tudsz támogatást küldeni:

[![Donate](https://img.shields.io/badge/PayPal-Donate-blue.svg)](https://www.paypal.me/zolikakiss)



