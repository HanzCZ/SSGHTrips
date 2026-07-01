# Role a oprávnění

## Přehled rolí

```mermaid
graph TD
    subgraph Systémové role
        ADMIN[Admin]
        MGMT[Vedení školy\nředitel / zástupce]
        TEACHER[Učitel / Owner výletu]
        STUDENT_A[Žák zletilý]
        STUDENT_M[Žák nezletilý]
        PARENT[Rodič]
    end
```

## Matice oprávnění

| Akce | Admin | Vedení | Učitel (owner) | Učitel (neowner) | Žák zletilý | Žák nezletilý | Rodič |
|------|:-----:|:------:|:--------------:|:----------------:|:-----------:|:-------------:|:-----:|
| Vytvořit výlet | ✓ | ✓ | ✓ | ✓ | — | — | — |
| Editovat svůj výlet (draft/returned) | ✓ | — | ✓ | — | — | — | — |
| Odeslat výlet ke schválení | ✓ | — | ✓ | — | — | — | — |
| Schválit / zamítnout / vrátit výlet | ✓ | ✓ | — | — | — | — | — |
| Publikovat schválený výlet | ✓ | — | ✓ | — | — | — | — |
| Vidět výlet (published, splňuje podmínky) | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| Přihlásit se na výlet | — | — | — | — | ✓ | — | — |
| Přihlásit nezletilé dítě | — | — | — | — | — | — | ✓ |
| Schvalovat přihlášky (školní výlet) | ✓ | — | ✓ | — | — | — | — |
| Zrušit přihlášku | — | — | ✓ | — | ✓ | — | ✓ |
| Zobrazit všechny výlety (admin pohled) | ✓ | ✓ | — | — | — | — | — |

## Podmínky pro zobrazení výletu žákovi

```mermaid
flowchart LR
    A[Výlet je published] --> B{Typ výletu}
    B -->|Třídní| C{Je žák ve\nvybrané třídě?}
    B -->|Školní| D{Jsou vybrány ročníky?}

    C -->|Ano| SHOW[✓ Výlet se zobrazí]
    C -->|Ne| HIDE[✗ Výlet se nezobrazí]

    D -->|Ne — otevřeno všem| SHOW
    D -->|Ano| E{Je žák\nve vybraném ročníku?}
    E -->|Ano| SHOW
    E -->|Ne| HIDE
```

## Podmínky pro přihlášení na výlet

```mermaid
flowchart TD
    A[Žák chce na výlet] --> B{Splňuje podmínky\npro zobrazení?}
    B -->|Ne| STOP[✗ Nemůže se přihlásit]
    B -->|Ano| C{Je otevřeno přihlašování?\nregistration_deadline}
    C -->|Ne| STOP
    C -->|Ano| D{Je kapacita výletu\nještě volná?}
    D -->|Ne| STOP2[✗ Výlet je plný]
    D -->|Ano| E{Je žák zletilý?}
    E -->|Ano| F[Přihlásí se sám]
    E -->|Ne| G[Musí přihlásit rodič]
    F --> OK[✓ Přihláška odeslána]
    G --> OK
```

## Poznámky k implementaci

### Určení zletilosti
- `is_adult` na `USER` se počítá z `birth_date` dynamicky nebo se aktualizuje denním jobbem
- Doporučeno: dynamická kontrola při přihlašování (`birth_date + 18 let ≤ today`)

### Napojení rodič–dítě
- Rodič vidí pouze děti napojené přes `PARENT_STUDENT`
- Přihlásit může pouze děti, které splňují podmínky výletu

### Vedení vs. Admin
- `management` = ředitel / zástupce — schvaluje výlety
- `admin` = systémový správce — má přístup ke všemu pro technickou správu
- Může být jedna role s různými podsady oprávnění (upřesnit s týmem)
