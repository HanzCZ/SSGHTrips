> ⚠️ **Toto repo je archivované — obsah se přestěhoval.**
> Aktuální verze žije v privátním repu [HanzCZ/ssgh-dokumentace](https://github.com/HanzCZ/ssgh-dokumentace), složka [`skolni-akce/`](https://github.com/HanzCZ/ssgh-dokumentace/tree/main/skolni-akce). Historie commitů byla při sloučení zachována.

# Systém školních výletů — Dokumentace diagramů

Tato složka obsahuje návrh datového modelu a uživatelských toků pro systém správy školních výletů a exkurzí.

## Obsah

| Soubor | Popis |
|--------|-------|
| [er-diagram.md](er-diagram.md) | ER diagram — datový model |
| [stavovy-diagram.md](stavovy-diagram.md) | Stavové diagramy výletu a přihlášky |
| [user-flows.md](user-flows.md) | Uživatelské toky pro jednotlivé role |
| [architektura-roli.md](architektura-roli.md) | Role a oprávnění v systému |

## Role v systému

| Role | Popis |
|------|-------|
| **Učitel / Owner výletu** | Vytváří výlet, spravuje přihlášky žáků (u školního typu) |
| **Vedení školy** | Schvaluje / zamítá / vrací výlety k opravě |
| **Žák (zletilý)** | Přihlašuje se sám |
| **Žák (nezletilý)** | Přihlásit ho může rodič |
| **Rodič** | Přihlašuje nezletilé dítě |

## Typy výletů

- **Třídní výlet** — platí pro konkrétní třídu/třídy, žáci jsou po schválení přijati automaticky
- **Školní výlet** — může být otevřen pro vybrané ročníky, žáci procházejí schvalovacím procesem ownera výletu
