# CatDex

Mobilní webová appka: potkáš na ulici skutečnou kočku, vyfotíš ji, appka jí vystaví
sběratelskou kartu se jménem, vzácností a statistikami. Sbírka roste. Pokémon,
ale s reálnými kočkami z okolí.

Vedlejší projekt dvou lidí se zaměstnáním na plný úvazek. Reálná kapacita je
**6–10 hodin týdně na osobu**. Každé rozhodnutí se poměřuje tímhle: co nezvládneme
udržovat, to nestavíme.

Neděláme konkurenci existujícím appkám na trhu (např. Battly). Jde o osobní
zážitek ze sousedství, ne sociální síť ani startup — proto je v pořádku, když
appka zůstane čistě lokální i tam, kde by "normální" appka měla sdílení nebo účty.

---

## Zásady, které se neobcházejí

Tyhle věci vyplynuly z návrhu a mají důvod. Když nějaká komplikuje zadání,
zeptej se — neobcházej ji potichu.

1. **Fotka jen z fotoaparátu, nikdy z galerie.** `capture="environment"`.
   Hodnota celé hry je v tom, že ta kočka tam skutečně byla.
2. **Karta a fotka jsou v datech dvě oddělené věci.** Karta = jméno, statistiky,
   vzácnost, čas. Fotka = jeden atribut. Musí být možné sdílet kartu bez fotky.
3. **Souřadnice se ukládají přesné, nikdy se nezobrazují.** Nepřesnost (posun
   100–300 m) se dělá až při zobrazení nebo exportu, ne při ukládání. Přesnost
   se zpátky nedokoupí.
4. **EXIF se strhává vždy.** Překreslením přes canvas. Žádná fotka nesmí opustit
   appku s GPS, časem nebo modelem telefonu.
5. **Statistiky se odvozují z pixelů, ne z náhody.** Otisk fotky je semínko
   generátoru → stejná fotka dá vždy stejnou kočku. Nesmí se dát přetáčet
   dokola do lepšího výsledku.
6. **Offline first.** Žádný server, žádný účet, žádná registrace. Všechno
   v `localStorage` a v paměti telefonu.
7. **Každý úlovek má stabilní ID, `verze` schématu a `vznik` (ISO timestamp).**
   Bez toho nejde budoucí synchronizace.

---

## Datový model

```js
{
  verze: 1,
  ulovky: [{
    id: 'k_lx8f2a_q7p3z',      // stabilní, nikdy se nemění
    verze: 1,
    vznik: '2026-08-18T20:14:03.221Z',
    jmeno: 'Mourek',
    pridomek: 'od popelnic',
    srst: 'černá',              // černá|bílá|šedá|zrzavá|mourovatá|strakatá
    denniDoba: 'noc',           // den|setmění|noc
    vzacnost: 'Vzácná',
    skore: 78,                  // 0–99, z něj plyne vzácnost
    statistiky: { 'Chlupatost': 7, ... },   // vždy 1–10
    setkani: ['2026-08-18T20:14:03.221Z'],  // každé další spatření = další záznam
    poloha: { lat, lon, presnost } | null,  // PŘESNÁ, jen lokálně
    foto: 'data:image/jpeg;base64,...'      // max 900 px, kvalita .72
  }]
}
```

Změna schématu = zvýšit `SCHEMA` **a** napsat migraci, která projde existující
úlovky. Nikdy nepředpokládej, že uživatel má čistou sbírku.

---

## Vzácnost

Skóre vzniká z: barvy srsti (černá +16, strakatá +8), denní doby (noc +26,
setmění +12), tmavosti fotky (+9) a semínka z otisku. Prahy:

| Skóre | Třída |
|---|---|
| 0 | Toulavá |
| 34 | Sousedská |
| 56 | Zvláštní |
| 74 | Vzácná |
| 88 | Mýtická |
| 96 | Legendární |

Čísla se budou ladit podle toho, jak často co padá. Neměň je bez důvodu —
ale když je změníš, uveď proč v commitu.

---

## Statistiky

`Chlupatost`, `Majestátnost`, `Drzost`, `Vypasenost`,
`Ochota nechat se pohladit`. Vždy 1–10.

Cíl je humor, ne přesnost. Když se odhad sekne, je to vtipné, ne rozbité.
`Vypasenost` drž v neutrálním tónu — jde o cizí zvířata.

---

## Co teď NEstavět

Sbírka je zatím **soukromá**, takže tohle je zbytečná práce:

- moderace, nahlašování obsahu, veřejná mapa, žebříčky
- detekce fotky z obrazovky, ochrana proti podvádění
- rozpoznávání, že je na fotce vůbec kočka (rohlík dostane kartu, budiž)
- identifikace konkrétního jedince — výzkumný problém, nelez do toho
- účty, přihlášení, backend
- reklamy, platby

Až se bude sdílet, řeší se to všechno znovu a od začátku.

---

## Roadmapa

**Fáze 1 (běží):** foto → karta → sbírka, lokálně. Záloha do JSON.
**Fáze 2:** vlastní mapa sbírky (jen moje kočky, jen pro mě). Série
("14 dní v řadě"), odznaky, roční přehled. PWA manifest + ikona.
**Fáze 3:** nejisté, možná se vůbec nestaví. Původní nápad: sdílení jedné
karty jako obrázek — bez fotky, bez polohy. Ale appka může zůstat čistě
lokální i tady — vyhne se to starostem (moderace, srovnávání s appkami
jako Battly) a odpovídá to tomu, že jde o osobní věc, ne sociální síť.
Rozhodne se blíž k realizaci, neřešit teď.
**Fáze 4:** teprve tady se řeší server, účty a monetizace. Nový směr
(zatím jen myšlenka pro budoucnost, nestavět teď): appka jde nativně na
Android/iOS (App Store/Play Store), protože jen tam jde paywall vynutit —
na statické webovce v `localStorage` se trial i platba dají trikem
obejít donekonečna. Model: 7denní trial, pak hard paywall, cca 5 USD/EUR
přes App Store/Play Store IAP (ten řeší ověření platby za nás). Nahrazuje
dřívější představu jednorázové platby 199 Kč na webu — přesná cena a
mechanismus (jednorázově vs. předplatné) se doladí blíž k realizaci.

Nepředbíhej fáze. Když narazíš na něco, co by fáze 2 potřebovala mít
připravené v datech už teď, řekni to — to je jediná legitimní výjimka.

---

## Technika a konvence

- **Vanilla HTML/CSS/JS, jeden soubor `index.html`.** Žádný build, žádný
  bundler, žádný framework. Nasazuje se to na GitHub Pages jako statický
  soubor. Rámec přijde, až bude co rámovat.
- **Kód i UI česky.** Názvy proměnných a funkcí česky bez diakritiky
  (`vyrobKartu`, `hledejZnamou`, `pamet`). Drž se toho, co už v souboru je.
- **Žádné závislosti přes npm.** Fonty z Google Fonts přes `<link>`.
- **Cílová zařízení:** iOS Safari a Chrome na Androidu, mobilní šířky.
  Desktop je vedlejší. Testuje se prstem na telefonu, ne v okně prohlížeče.
- **Fotoaparát a GPS vyžadují HTTPS.** GitHub Pages ho má, `file://` ne.
- **`prefers-reduced-motion` se respektuje**, focus stavy musí být vidět.
- Nikdy nepřidávej analytiku, tracking ani externí skripty bez zeptání.

## Vizuální směr

Noční ulice pod sodíkovou lampou. Modrošedý asfalt (`--asfalt #161B26`),
jantarové světlo (`--sodik #F2A93B`), mentolová jako druhý akcent
(`--mata #7FD1C1`). Typografie: Bricolage Grotesque (displej),
Newsreader (text), JetBrains Mono (data a statistiky).

Barvy ber z CSS proměnných, nové nezaváděj bez důvodu. Karta úlovku je
to jediné místo, kde se smí utrácet efektem — všechno ostatní zůstává tiché.

---

## Jak s tebou chceme pracovat

- **Malé kroky.** Jedna změna, jeden commit, jde to otevřít na telefonu a
  vyzkoušet. Žádné velké přepisy bez dohody.
- **Než začneš psát, řekni plán**, pokud jde o víc než ~30 řádků.
- **Neopravuj po cestě věci, o které nikdo neprosil.** Když si všimneš
  problému, napiš ho na konec odpovědi.
- **Když je zadání nejasné, zeptej se.** Lepší jedna otázka než hodina
  přepisování.
- Commity česky, v rozkazovacím způsobu: `přidej odznak za sérii dnů`.
