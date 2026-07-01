# ER Diagram — Datový model

```mermaid
erDiagram

    USER ||--o{ STUDENT_PROFILE : "has student profile"
    USER ||--o{ PARENT_STUDENT : "is parent"
    STUDENT_PROFILE ||--o{ PARENT_STUDENT : "has parent"
    CLASS ||--o{ STUDENT_PROFILE : "contains students"
    USER ||--o{ CLASS : "is homeroom teacher"

    USER ||--o{ TRIP : "creates"
    USER ||--o{ TRIP : "owns"
    USER ||--o{ TRIP : "leads"
    TRIP ||--o{ TRIP_CLASS : "assigned to class"
    CLASS ||--o{ TRIP_CLASS : "included in trip"
    TRIP ||--o{ TRIP_ELIGIBLE_GRADE : "open to grade"
    TRIP ||--o{ TRIP_ELIGIBLE_FIELD_OF_STUDY : "open to field of study"
    TRIP ||--o{ TRIP_ELIGIBLE_STUDY_FORM : "open to study form"
    TRIP ||--o{ TRIP_REGISTRATION_CONDITION : "has registration conditions"
    TRIP ||--o{ TRIP_STAFF : "has staff"
    USER ||--o{ TRIP_STAFF : "participates as staff"

    TRIP ||--o{ APPROVAL_RECORD : "has approval records"
    USER ||--o{ APPROVAL_RECORD : "approves"

    TRIP ||--o{ TRIP_REGISTRATION : "has registrations"
    STUDENT_PROFILE ||--o{ TRIP_REGISTRATION : "is registered"
    USER ||--o{ TRIP_REGISTRATION : "registers"

    TRIP ||--o{ NOTIFICATION_SUBSCRIPTION : "has subscribers"
    USER ||--o{ NOTIFICATION_SUBSCRIPTION : "subscribes"
```

## Atributy entit

### USER
| Atribut | Typ | Popis |
|---------|-----|-------|
| id | int PK | |
| first_name | string | |
| last_name | string | |
| email | string | |
| birth_date | date | Pro výpočet zletilosti |
| is_adult | boolean | Vypočítáno z birth_date |
| system_role | enum | `teacher`, `management`, `student`, `parent`, `admin` |

### CLASS
| Atribut | Typ | Popis |
|---------|-----|-------|
| id | int PK | |
| name | string | Název třídy (např. 3.A) |
| grade_year | int | Ročník (1–9) |
| homeroom_teacher_id | int FK → USER | Třídní učitel |

### STUDENT_PROFILE
| Atribut | Typ | Popis |
|---------|-----|-------|
| id | int PK | |
| user_id | int FK → USER | |
| class_id | int FK → CLASS | |

### PARENT_STUDENT
| Atribut | Typ | Popis |
|---------|-----|-------|
| parent_user_id | int FK → USER | |
| student_profile_id | int FK → STUDENT_PROFILE | |

### TRIP
| Atribut | Typ | Popis |
|---------|-----|-------|
| id | int PK | |
| name | string | |
| description | text | Popis výletu |
| boarding_description | text | Popis k nástupu na výlet |
| departure_description | text | Popis ukončení výletu |
| type | enum | `class`, `school` |
| status | enum | `draft`, `pending_approval`, `approved`, `rejected`, `returned`, `published`, `archived` |
| capacity | int | Max počet žáků |
| student_budget | decimal | Co zaplatí žák |
| school_budget | decimal | Příspěvek školy |
| payment_cash | boolean | `true` = hotově na místě, `false` = bankovní převod |
| start_datetime | datetime | Začátek výletu |
| end_datetime | datetime | Konec výletu |
| registration_start_datetime | datetime | Začátek přihlašování — `NULL` = hned po publikaci |
| registration_deadline | datetime | Konec přihlašování |
| created_by | int FK → USER | Kdo záznam vytvořil (audit trail, nemění se) |
| owner_user_id | int FK → USER | Aktuální owner výletu (lze změnit, výchozí = created_by) |
| leader_user_id | int FK → USER | Vedoucí výletu |
| created_at | timestamp | |
| updated_at | timestamp | |

### TRIP_CLASS
| Atribut | Typ | Popis |
|---------|-----|-------|
| trip_id | int FK → TRIP | |
| class_id | int FK → CLASS | Pouze pro `type = class` |

### TRIP_ELIGIBLE_GRADE
| Atribut | Typ | Popis |
|---------|-----|-------|
| trip_id | int FK → TRIP | |
| grade_year | int | Povolený ročník — pouze pro `type = school` |

### TRIP_ELIGIBLE_FIELD_OF_STUDY
| Atribut | Typ | Popis |
|---------|-----|-------|
| trip_id | int FK → TRIP | |
| field_of_study | string | Obor studia — pouze pro `type = school` |

### TRIP_ELIGIBLE_STUDY_FORM
| Atribut | Typ | Popis |
|---------|-----|-------|
| trip_id | int FK → TRIP | |
| study_form | string | Forma studia (např. `denní`, `dálkové`) — pouze pro `type = school` |

### TRIP_REGISTRATION_CONDITION
| Atribut | Typ | Popis |
|---------|-----|-------|
| id | int PK | |
| trip_id | int FK → TRIP | |
| condition_type | enum | `max_absences`, `max_subject_absences`, `max_failing_grades` |
| threshold | int | Maximální povolená hodnota |
| subject_id | int FK → SUBJECT (nullable) | Vyplněno pouze pro `max_subject_absences` |

### TRIP_STAFF
| Atribut | Typ | Popis |
|---------|-----|-------|
| id | int PK | |
| trip_id | int FK → TRIP | |
| user_id | int FK → USER | |
| role_label | string | Např. "Zdravotník", "Průvodce" |

### APPROVAL_RECORD
| Atribut | Typ | Popis |
|---------|-----|-------|
| id | int PK | |
| trip_id | int FK → TRIP | |
| approver_user_id | int FK → USER | |
| action | enum | `approved`, `rejected`, `returned_for_revision` |
| note | text | Povinné při rejected a returned |
| created_at | timestamp | |

### TRIP_REGISTRATION
| Atribut | Typ | Popis |
|---------|-----|-------|
| id | int PK | |
| trip_id | int FK → TRIP | |
| student_profile_id | int FK → STUDENT_PROFILE | |
| registered_by_user_id | int FK → USER | Žák nebo rodič |
| status | enum | `pending`, `accepted`, `rejected` |
| rejection_note | text | Povinné při rejected |
| registered_at | timestamp | |
| reviewed_at | timestamp | |

### NOTIFICATION_SUBSCRIPTION
| Atribut | Typ | Popis |
|---------|-----|-------|
| id | int PK | |
| trip_id | int FK → TRIP | |
| user_id | int FK → USER | |
| subscribed_at | timestamp | |

---

## Klíčové poznámky k datovému modelu

### USER vs. role
- `system_role` určuje co uživatel v systému může dělat globálně
- Jeden uživatel může mít roli `teacher` a zároveň mít dítě jako žáka (rodič) — to by se řešilo přes `PARENT_STUDENT` napojením i bez role `parent` nebo přes multi-role systém (rozhodnutí pro tým)

### TRIP — Owner vs. Created by
- `created_by` se nastaví při vytvoření a **nikdy se nemění** — slouží jako audit trail
- `owner_user_id` se výchozně nastaví na `created_by`, ale lze ho změnit (např. výlet vytvořen za kolegu)
- Oprávnění (editace, publikace, schvalování přihlášek) se váží na `owner_user_id`

### TRIP — typ a podmínky přístupu

**Podmínky viditelnosti** — určují, kdo výlet vůbec uvidí:

| Podmínka | Platí pro typ | Tabulka |
|----------|:-------------:|---------|
| Konkrétní třídy | `class` | `TRIP_CLASS` |
| Ročníky | `school` | `TRIP_ELIGIBLE_GRADE` |
| Obor studia | `school` | `TRIP_ELIGIBLE_FIELD_OF_STUDY` |
| Forma studia | `school` | `TRIP_ELIGIBLE_STUDY_FORM` |

Prázdná junction tabulka = podmínka se neuplatňuje (otevřeno všem v dané kategorii).

**Podmínky přihlášení** (`TRIP_REGISTRATION_CONDITION`) — platí pro oba typy, neovlivňují viditelnost:

| condition_type | Popis | threshold |
|----------------|-------|-----------|
| `max_absences` | Max. celkové absence | počet hodin/dní |
| `max_subject_absences` | Max. absence na konkrétním předmětu | počet hodin, vyžaduje `subject_id` |
| `max_failing_grades` | Max. počet pětek v posledním čtvrtletním hodnocení | počet předmětů |

Pokud žák podmínky přihlášení nesplňuje:
- nemůže se přihlásit sám (systém to zablokuje)
- owner ho může přihlásit ručně (override)

### TRIP — platba
- `payment_cash = true` → žák platí hotově na místě
- `payment_cash = false` → žák platí bankovním převodem (budoucí napojení na účetnictví)

### TRIP — okno přihlašování
- `registration_start_datetime = NULL` → přihlašování začíná okamžitě po publikaci (owner to nemusí vyplňovat)
- `registration_start_datetime` nastavené → přihlašování se otevře až v daný čas (výlet je mezitím viditelný, ale přihlásit se nelze)
- `registration_deadline` — konec přihlašování, po deadline systém přihlášky odmítá

### Přihlášení (`TRIP_REGISTRATION`)
- `registered_by_user_id` = buď sám žák (zletilý), nebo rodič
- `status = pending` pouze u školního výletu — owner musí schválit
- `status = accepted` automaticky u třídního výletu při přihlášení

### Schvalování výletu (`APPROVAL_RECORD`)
- Každá akce vedení se loguje — umožňuje historii a audit trail
- Poznámka (`note`) je povinná při `rejected` a `returned_for_revision`

### Rozpočet
- `student_budget` = co zaplatí žák
- `school_budget` = příspěvek školy na žáka (nebo celkem — upřesnit s týmem)

### Budoucí napojení na účetnictví
- `TRIP_REGISTRATION` bude vstupním bodem pro fakturaci
- Doporučujeme přidat `payment_status` na `TRIP_REGISTRATION` až se bude řešit
