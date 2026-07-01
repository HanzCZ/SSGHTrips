# User Flow Diagramy

## 1. Učitel — Vytvoření a odeslání výletu ke schválení

```mermaid
flowchart TD
    A([Start: Učitel přihlášen]) --> B[Klikne na 'Nový výlet']
    B --> C{Typ výletu?}

    C -->|Třídní| D[Vybere třídu/třídy]
    C -->|Školní| E[Volitelně vybere ročníky]

    D --> F[Vyplní základní údaje\nnázev, popis, datum,\nkapacitu, budget]
    E --> F

    F --> G[Přidá vedoucího výletu\na další zaměstnance + role]
    G --> H{Je vše vyplněno?}

    H -->|Ne| I[Systém označí chybějící pole]
    I --> F

    H -->|Ano| J[Uloží jako Koncept / draft]
    J --> K{Odeslat ke schválení?}

    K -->|Ano| L[Výlet přejde do pending_approval\nOwner dostane potvrzení]
    K -->|Ne| M[Zůstane v draftu\nOwner může dál editovat]

    L --> N([Konec: čeká na vedení])
    M --> N2([Konec: uloženo v draftu])
```

---

## 2. Vedení školy — Schvalování výletu

```mermaid
flowchart TD
    A([Start: Vedení obdrží notifikaci\nnebo přejde do sekce Ke schválení]) --> B[Zobrazí detail výletu]
    B --> C[Zkontroluje všechny údaje\nnázev, typ, třídy/ročníky,\nkapacita, budget, zaměstnanci]

    C --> D{Rozhodnutí}

    D -->|Schválit| E[Klikne Schválit]
    D -->|Zamítnout| F[Musí napsat poznámku\nProč je výlet zamítnut]
    D -->|Vrátit k opravě| G[Musí napsat poznámku\nCo je třeba opravit]

    E --> H[Výlet → approved\nOwner dostane notifikaci]
    F --> I[Výlet → rejected\nOwner dostane notifikaci s důvodem]
    G --> J[Výlet → returned\nOwner dostane notifikaci s důvodem]

    H --> N([Konec])
    I --> N
    J --> N
```

---

## 3. Učitel — Reakce na vrácení k opravě

```mermaid
flowchart TD
    A([Start: Owner dostane notifikaci o vrácení]) --> B[Zobrazí výlet\na poznámku od vedení]
    B --> C[Upraví potřebné údaje]
    C --> D{Odeslat znovu?}
    D -->|Ano| E[Výlet → pending_approval\nZnovu čeká na vedení]
    D -->|Ne| F[Zůstane v draftu]
    E --> G([Konec])
    F --> G
```

---

## 4. Učitel — Publikování schváleného výletu

```mermaid
flowchart TD
    A([Start: Owner dostane notifikaci o schválení]) --> B[Přejde na detail výletu]
    B --> C[Zkontroluje finální stav]
    C --> D[Klikne Publikovat]
    D --> E[Výlet → published]
    E --> F[Žáci kteří splňují podmínky\nvidí výlet na stránce Výlety/Exkurze]
    F --> G{Žák si zaregistruje\nzájem o notifikaci?}
    G -->|Ano| H[Dostane push/email\naž se otevře přihlašování]
    G -->|Ne| I[Výlet vidí, ale bez notifikace]
    H --> J([Konec])
    I --> J
```

---

## 5. Žák / Rodič — Přihlášení na výlet

```mermaid
flowchart TD
    A([Start: Žák / rodič přejde\nna stránku Výlety]) --> B[Vidí výlety\nkteré splňuje podmínky]
    B --> C[Vybere výlet]
    C --> D[Zobrazí detail výletu\nnázev, datum, cena, popis]

    D --> E{Kdo se přihlašuje?}

    E -->|Zletilý žák| F[Přihlásí se sám]
    E -->|Nezletilý žák| G{Je přihlášen rodič?}

    G -->|Ano| H[Rodič přihlásí dítě]
    G -->|Ne| I[Systém zobrazí upozornění:\nNezletilého může přihlásit\npouze rodič]

    F --> J{Typ výletu?}
    H --> J

    J -->|Třídní| K[Přihláška automaticky\npřijata → accepted]
    J -->|Školní| L[Přihláška čeká\nna schválení ownera → pending]

    K --> M[Žák / rodič dostane\npotvrzení přijetí]
    L --> N[Žák / rodič dostane\npotvrzení odeslání přihlášky]

    M --> Z([Konec])
    N --> Z
    I --> Z
```

---

## 6. Owner výletu — Schvalování přihlášek (pouze školní výlet)

```mermaid
flowchart TD
    A([Start: Owner přejde na\nSprávce přihlášek výletu]) --> B[Vidí seznam přihlášek\nve stavu pending]
    B --> C[Vybere přihlášku]
    C --> D[Zobrazí info o žákovi\ntřída, kontakt, případně rodič]

    D --> E{Rozhodnutí}

    E -->|Přijmout| F[Přihláška → accepted\nŽák / rodič dostane notifikaci]
    E -->|Odmítnout| G[Musí napsat důvod odmítnutí\nPřihláška → rejected\nŽák / rodič dostane notifikaci s důvodem]

    F --> H{Další přihlášky?}
    G --> H

    H -->|Ano| B
    H -->|Ne| I([Konec])
```

---

## 7. Celkový přehled — Orchestrace procesu

```mermaid
sequenceDiagram
    actor O as Owner (učitel)
    actor V as Vedení
    actor Z as Žák / Rodič
    participant S as Systém

    O->>S: Vytvoří výlet (draft)
    O->>S: Odešle ke schválení
    S->>V: Notifikace: nový výlet ke schválení

    alt Schválí
        V->>S: Schválí výlet
        S->>O: Notifikace: výlet schválen
        O->>S: Publikuje výlet
        S->>Z: Výlet zobrazen na stránce
    else Vrátí k opravě
        V->>S: Vrátí s poznámkou
        S->>O: Notifikace: výlet vrácen + poznámka
        O->>S: Upraví a znovu odešle
    else Zamítne
        V->>S: Zamítne s poznámkou
        S->>O: Notifikace: výlet zamítnut + důvod
    end

    alt Třídní výlet
        Z->>S: Přihlásí se (žák/rodič)
        S->>Z: Automaticky přijat ✓
    else Školní výlet
        Z->>S: Přihlásí se (žák/rodič)
        S->>O: Notifikace: nová přihláška čeká
        alt Owner schválí
            O->>S: Schválí přihlášku
            S->>Z: Notifikace: přijat ✓
        else Owner odmítne
            O->>S: Odmítne s důvodem
            S->>Z: Notifikace: odmítnut + důvod
        end
    end
```
