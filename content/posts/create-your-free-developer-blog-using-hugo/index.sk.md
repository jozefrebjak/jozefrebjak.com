---
title: 'Vytvorte si bezplatný blog pre vývojárov pomocou aplikácie Hugo'
resources:
  - name: 'featured-image'
    src: 'featured-image-sk.png'
categories: ['documentation']
tags: ['hugo', 'installation']
date: 2021-03-22T09:31:48+01:00
draft: false
---

## Úloha Huga

Po dlhom hľadaní a študovaní som sa rozhodol pre moj blog použiť Huga. **Hugo** je generátor statických stránok s otvoreným zdrojovým kódom. Generátor statického webu vytvorí webovú stránku iba raz, v okamihu, keď vytvárame alebo upravujeme nový obsah.

Statická webová stránka nám poskytne **100%** kontrolu nad našim obsahom a webovým dizajnom. Pretože Hugo je predovšetkým o statickej webovej stránke, má menej bezpečnostných problémov. Na strane servera nebeží žiaden backend. Nie je spustené ani len PHP. Nič.

Vďaka tomu je statická webová stránka celkom odolná proti narušeniu bezpečnosti.

## Používanie Huga

Ako už bolo spomenuté, **Hugo** má otvorený zdrojový kód a dá sa nainštalovať pomerne ľahko. Ak používate Mac (ako ja), môžete Huga nainštalovať pomocou aplikácie Homebrew:

### Homebrew & Git

Ak nemáte nainštalovaný Homebrew, môžete ho nainštalovať pomocou::

```shell
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"
```

Uistite sa, že je všetko aktuálne a nainštalujte si Git

```shell
brew update
brew install git
```

### Hugo

#### Inštalácia

Nainštalujte Huga pomocou Homebrew

```shell
brew install hugo
```

Po dokončení inštalácie môžete zadať príkaz:

```shell
hugo version
```

Aby bola inštalácia úspešná, musíte vidieť niečo ako:

{{< admonition type=info open=true >}}
hugo v0.81.0+extended darwin/amd64 BuildDate=unknown
{{< /admonition >}}

#### Vytvorenie webovej stránky

{{< admonition type=note open=true >}}
Prosím, vymeňte **example.com** za vašu doménu alebo názvom stránky
{{< /admonition >}}

```shell
hugo new site example.com
```

Úloha by sa mala ukončiť bez mihnutia oka. Príkay vytvorí nový adresár. Prejdite do tohto adresára pomocou príkazu:

```shell
cd example.com
```

#### Štruktúra adresárov

Spustenie generátora `hugo new site` z príkazového riadku vytvorí adresárovú štruktúru s nasledujúcimi prvkami:

```
.
├── archetypes
├── config.toml
├── content
├── data
├── layouts
├── static
└── themes
```

{{< admonition type=note open=true >}}
Ak sa chcete dozvedieť viac o adresárovej štruktúre, môžete sa ju dozvedieť na oficiálnych stránkach {{< link href="https://gohugo.io/getting-started/directory-structure/" content=GoHugo >}}.
{{< /admonition >}}

Pre nás je najdôležitejší súbor `config.toml`, kde musíme špecifikovať všetky naše nastavenia.

#### Používanie témy

Pretože chceme tráviť všetok voľný čas so svojimi rodinami a už pracujeme viac ako osem hodín denne, je použitie témy tou najlepšou voľbou na začiatok. Osobne som strávil niekoľko hodín hľadaním najlepšej témy blogu pre mňa. Napokon som sa rozhodol pre {{< link href="https://github.com/sunt-programator/CodeIT/" content=CodeIT >}} tému.

Najskôr som skúsil použiť tému {{< link href="https://github.com/dillonzq/LoveIt" content=LoveIT >}}, ale táto téma sa už neudržuje a zistil som, že **CodeIT** áno. Existuje veľa tém, ktoré sú takmer rovnaké, či už s malými alebo väčšimi zmenami, ale táto sa mi páči najviac. Ako som teda tému použil pre moje potreby?

Pretože som chcel mať svoj kód hostovaný na Githube a vy určite chcete tiež, musíte najskôr inicializovať úložisko git.

```shell
git init
```

Z úložiska **CodeIT** urobte submodul nášho adresára::

```shell
git submodule add https://github.com/sunt-programator/CodeIT.git themes/CodeIT
```

Príkaz **git submodule add** vytvorí priečinok **CodeIT** v priečinku `/themes` a vytvorí súbor `.gitmodules`, ktorý obsahuje:

```
[submodule "themes/CodeIT"]
path = themes/CodeIT
url = https://github.com/sunt-programator/CodeIT.git
```

Teraz máme tému načítanú a na jej použitie stačí upraviť súbor `.config.toml`.

Tu je základný súbor `.config.toml` na testovanie našej témy:

```toml
baseURL = "http://example.com/"
defaultContentLanguage = "en"
languageCode = "en"
title = "My New Hugo Site"
theme = "CodeIT"

[params]
  version = "0.2.X"

[menu]
  [[menu.main]]
    identifier = "posts"
    pre = ""
    post = ""
    name = "Posts"
    url = "/posts/"
    title = ""
    weight = 1
  [[menu.main]]
    identifier = "tags"
    pre = ""
    post = ""
    name = "Tags"
    url = "/tags/"
    title = ""
    weight = 2
  [[menu.main]]
    identifier = "categories"
    pre = ""
    post = ""
    name = "Categories"
    url = "/categories/"
    title = ""
    weight = 3

[markup]
  [markup.highlight]
    noClasses = false
```

{{< admonition type=note open=true >}}
Ak si ho chcete upraviť podľa svojich potrieb, tu je odkaz na oficiálnu {{< link href="https://codeit.suntprogramator.dev/theme-documentation-basics/" content=dokumentáciu >}} alebo sa môžete inšpirovať z môjho {{< link href="https://github.com/jozefrebjak/jozefrebjak.com/blob/master/config.toml" content=nastavenia >}}.
{{< /admonition >}}

#### Lokálne spustenie webovej stránky

{{< admonition type=note open=true >}}
Pretože téma na implementáciu niektorých funkcií používa v aplikácii Hugo `.Scratch` dôrazne sa odporúča pridať parameter `--disableFastRender` do príkazu ` hugo serve` pre živý náhľad stránky, ktorú upravujeme.
{{< /admonition >}}

Spustite pomocou nasledujúceho príkazu:

```shell
hugo serve --disableFastRender
```

Ak je všetko v poriadku, uvidíte:

```
Built in 105 ms
Watching for changes in /Users/USER/example.com/{archetypes,content,data,layouts,static,themes}
Watching for config changes in /Users/USER/example.com/config.toml
Environment: "development"
Serving pages from memory
Web Server is available at localhost:1313
Press Ctrl+C to stop
```

Prejdite na adresu {{< link "localhost:1313" >}} a pozrite si svoju bežiacu stránku.

{{< figure src="new-hugo-site.png" >}}

#### Pridanie prvého príspevku

Nový obsah môžete vytvoriť príkazom:

```sh
hugo new posts/blog-post-1.md
```

Príkaz vytvorí súbor `blog-post-1.md` v priečinku `/content/posts`.

Podľa potreby upravte obsah nového súboru. Uložte a zatvorte súbor a Hugo automaticky zistí zmenu novo pridaného blogového príspevku:

Napríklad:

```md
---
title: 'Blog Post 1'
date: 2021-03-22T12:11:52+01:00
draft: false
---

# Test Header

Test text
```

{{< figure src="new-hugo-site-post.png" >}}

Keď máte web presne taký, aký ho chcete, zabite démonový server pomocou kombinácie klávesov [Ctrl] + a vytvorte web pomocou príkazu (spusteného z koreňového adresára):

```sh
hugo
```

Stránka sa veľmi rýchlo vytvorí a vytvorí nový verejný priečinok vo vnútri koreňového adresára dokumentu. Nahrajte tento priečinok na svoj hosting a máte funkčnú stránku.

## Záver

A to je pre začiatok všetko. Takto jednoducho je možné vytvoriť staticku stránku pomocou Huga. Ak sa vám podarilo prejsť celým týmto príspevkom, tak gratulujem! V nasledujúcom blogovom príspevku sa budem venovať tomu, ako získať zadarmo hosting blogu pomocou služby Netlify.

Netlify poskytuje služby nepretržitého nasadenia, globálne CDN, ultrarýchle DNS, SSL jedným kliknutím, rozhranie založené na prehliadači, CLI a mnoho ďalších funkcií pre správu nášej statickej web stránky pomocou Huga.
