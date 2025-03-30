+++
date = '2025-03-30T06:18:47Z'
draft = false
title = 'Komentáře přes Giscus na blogu'
slug = 'giscus-komentare-na-hugo-blogu'
kategorie = ['Návod']
keywords = ['hugo','GitHub','giscus']
ShowToc = true
TocOpen = true
comments = true
+++

Nasadil jsem systém komentářů [Giscus](https://giscus.app/cs) – lehké, elegantní a plně integrované řešení pro statické stránky postavené na [GitHub Discussions](https://docs.github.com/en/discussions). Díky němu mohou návštěvníci článků zanechat komentář či reakci pomocí svého GitHub účtu.

## Proč jsem si vybral právě Giscus?

Přemýšlel jsem, jak přidat na svůj [Hugo](https://gohugo.io) blog možnost komentářů:

- Hugo generuje **statické stránky**, takže jsem nechtěl nasazovat žádný backend ani databázi.
- Mám repozitář pro blog hostovaný na GitHubu.
- Chtěl jsem komentáře **bez starostí** – tedy bez spamu, falešných účtů a bez nutnosti spravovat další službu.
- Věřím, že čtenáři tohoto blogu mají GitHub účet, nebo jim nebude dělat problém si jej vytvořit.

Giscus mi perfektně zapadl do ekosystému – a navíc umožňuje propojit blogové články s diskusemi přímo na GitHubu.

## Jak Giscus funguje?

Komentáře jsou ukládány jako [GitHub Discussions](https://docs.github.com/en/discussions) v určené kategorii zvoleného repozitáře. Giscus využívá [OAuth](https://docs.github.com/en/developers/apps/identifying-and-authorizing-users-for-github-apps), takže návštěvník se jednorázově přihlásí přes GitHub a poté může komentovat.

Než začnete, je nutné zajistit následující:

1. **Repozitář je veřejný** – jinak komentáře nebudou viditelné.
2. **Aplikace [giscus](https://github.com/apps/giscus) je nainstalovaná** na tomto repozitáři.
3. **Funkce Discussions je povolená** ve [vašem GitHub repozitáři](https://docs.github.com/en/github/administering-a-repository/managing-repository-settings/enabling-or-disabling-github-discussions-for-a-repository).
4. **Vytvořená kategorie diskusí** – ideálně typu *Announcements*, aby komentáře mohla vytvářet pouze aplikace Giscus.

Já si vytvořil kategorii `Articles`, kterou jsem použil v nastavení.

## Implementace do Hugo blogu

### 1. úprava `config.yaml`

Do části `params` jsem přidal následující konfiguraci:

```yaml
params:
  giscus:
    repo: "ZiMi-lab/blog"
    repoID: "R_kgDOOLR8Mw"
    category: "Articles"
    categoryID: "DIC_abcxyz123"
    mapping: "title"
    strict: "0"
    reactionsEnabled: "1"
    emitMetadata: "0"
    inputPosition: "bottom"
    theme: "preferred_color_scheme"
    lang: "cs"
    loading: "lazy"
```

> ⚠️ Pokud máš chybu jako `can't evaluate field repo in type []interface {}`, zkontroluj, zda `giscus` není definovaný jako seznam (např. `- repo: ...`) – musí to být objekt.

### 2. partial šablona `layouts/partials/comments.html`

Vytvořil jsem partial, který načítá JavaScript Giscus a dynamicky vkládá parametry:

```html
{{ if .Site.Params.giscus }}
<script src="https://giscus.app/client.js"
  data-repo="{{ .Site.Params.giscus.repo }}"
  data-repo-id="{{ .Site.Params.giscus.repoID }}"
  data-category="{{ .Site.Params.giscus.category }}"
  data-category-id="{{ .Site.Params.giscus.categoryID }}"
  data-mapping="{{ default "pathname" .Site.Params.giscus.mapping }}"
  data-strict="{{ default "0" .Site.Params.giscus.strict }}"
  data-reactions-enabled="{{ default "1" .Site.Params.giscus.reactionsEnabled }}"
  data-emit-metadata="{{ default "0" .Site.Params.giscus.emitMetadata }}"
  data-input-position="{{ default "bottom" .Site.Params.giscus.inputPosition }}"
  data-theme="{{ default "light" .Site.Params.giscus.theme }}"
  data-lang="{{ default "en" .Site.Params.giscus.lang }}"
  data-loading="{{ default "lazy" .Site.Params.giscus.loading }}"
  crossorigin="anonymous" async>
</script>
{{ end }}
```

### 3. Aktivace komentářů v článku

Stačí v hlavičce článku (front matter) nastavit:

```toml
comments = true
```

nebo v YAML zápisu:

```yaml
comments: true
```

## Užitečné odkazy a inspirace

Pokud si chceš Giscus nastavit na svém Hugo blogu, doporučuji tyto zdroje:

- [Using Giscus with Hugo · Daniël du Toit](https://danieldutoit.net/posts/2024/hugo-using-giscus-2024-09-09/)
- [Adding Giscus to your Hugo Site · Hayden's Blog](https://blog.mrhaydendp.com/posts/adding-giscus-to-hugo-site/)
- [Introducing giscus | laymonage](https://laymonage.com/posts/giscus)
- [How to Create This Blog · Dominik Britz](https://dominikbritz.com/posts/how-to-create-this-blog/)
- [Giscus komentáře na Hugo · Justin Bird](https://justinjbird.com/blog/2023/adding-comments-to-a-hugo-site-using-giscus/)
