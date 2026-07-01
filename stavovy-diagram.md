# Stavové diagramy

## 1. Stavy výletu

```mermaid
stateDiagram-v2
    [*] --> draft : Owner vytvoří výlet

    draft --> pending_approval : Owner odešle ke schválení

    pending_approval --> approved : Vedení schválí
    pending_approval --> rejected : Vedení zamítne\n(povinná poznámka)
    pending_approval --> returned : Vedení vrátí k opravě\n(povinná poznámka)

    returned --> draft : Owner upraví výlet
    returned --> pending_approval : Owner znovu odešle

    rejected --> [*] : Konec procesu

    approved --> published : Owner publikuje výlet\n(žáci vidí, mohou se přihlásit)

    published --> archived : Výlet proběhl / ukončen ownerem

    archived --> [*]

    note right of draft
        Owner může editovat
        všechny atributy
    end note

    note right of pending_approval
        Výlet je uzamčen
        pro editaci
    end note

    note right of published
        Žáci se mohou přihlašovat
        (do registration_deadline)
    end note
```

---

## 2. Stavy přihlášky žáka

```mermaid
stateDiagram-v2
    [*] --> submitted : Žák / rodič odešle přihlášku

    submitted --> accepted : Třídní výlet\n(automaticky)
    submitted --> pending_review : Školní výlet\n(čeká na review ownera)

    pending_review --> accepted : Owner schválí
    pending_review --> rejected : Owner zamítne\n(povinná poznámka)

    accepted --> cancelled : Žák / rodič zruší přihlášku

    cancelled --> [*]
    rejected --> [*]
    accepted --> [*] : Výlet proběhl

    note right of submitted
        registered_by = žák nebo rodič
    end note

    note right of pending_review
        Owner výletu vidí
        seznam čekajících přihlášek
    end note
```

---

## 3. Přehled přechodů a podmínek

### Výlet

| Z → Do | Kdo | Podmínka |
|--------|-----|----------|
| `draft` → `pending_approval` | Owner | Výlet má vyplněna povinná pole |
| `pending_approval` → `approved` | Vedení | — |
| `pending_approval` → `rejected` | Vedení | Povinná poznámka |
| `pending_approval` → `returned` | Vedení | Povinná poznámka |
| `returned` → `draft` | Systém automaticky | — |
| `approved` → `published` | Owner | — |
| `published` → `archived` | Owner / systém | Po datu výletu |

### Přihláška

| Z → Do | Kdo | Podmínka |
|--------|-----|----------|
| — → `submitted` + `accepted` | Žák / rodič | Výlet je třídní, žák splňuje podmínky |
| — → `submitted` + `pending_review` | Žák / rodič | Výlet je školní, žák splňuje podmínky |
| `pending_review` → `accepted` | Owner | — |
| `pending_review` → `rejected` | Owner | Povinná poznámka |
| `accepted` → `cancelled` | Žák / rodič | Před deadlinem (nebo s povolením ownera) |
