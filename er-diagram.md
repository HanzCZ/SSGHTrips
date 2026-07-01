# ER Diagram — Datový model

```mermaid
erDiagram

    USER {
        int id PK
        string first_name
        string last_name
        string email
        date birth_date
        boolean is_adult
        enum system_role "teacher | management | student | parent | admin"
    }

    CLASS {
        int id PK
        string name
        int grade_year
        int homeroom_teacher_id FK
    }

    STUDENT_PROFILE {
        int id PK
        int user_id FK
        int class_id FK
    }

    PARENT_STUDENT {
        int parent_user_id FK
        int student_profile_id FK
    }

    TRIP {
        int id PK
        string name
        text description
        enum type "class | school"
        enum status "draft | pending_approval | approved | rejected | returned | published | archived"
        int capacity
        decimal student_budget
        decimal school_budget
        date start_date
        date end_date
        date registration_deadline
        int created_by FK
        int leader_user_id FK
        timestamp created_at
        timestamp updated_at
    }

    TRIP_CLASS {
        int trip_id FK
        int class_id FK
    }

    TRIP_ELIGIBLE_GRADE {
        int trip_id FK
        int grade_year
    }

    TRIP_STAFF {
        int id PK
        int trip_id FK
        int user_id FK
        string role_label
    }

    APPROVAL_RECORD {
        int id PK
        int trip_id FK
        int approver_user_id FK
        enum action "approved | rejected | returned_for_revision"
        text note
        timestamp created_at
    }

    TRIP_REGISTRATION {
        int id PK
        int trip_id FK
        int student_profile_id FK
        int registered_by_user_id FK
        enum status "pending | accepted | rejected"
        text rejection_note
        timestamp registered_at
        timestamp reviewed_at
    }

    NOTIFICATION_SUBSCRIPTION {
        int id PK
        int trip_id FK
        int user_id FK
        timestamp subscribed_at
    }

    %% Relationships

    USER ||--o{ STUDENT_PROFILE : "má profil žáka"
    USER ||--o{ PARENT_STUDENT : "je rodič"
    STUDENT_PROFILE ||--o{ PARENT_STUDENT : "má rodiče"
    CLASS ||--o{ STUDENT_PROFILE : "obsahuje žáky"
    USER ||--o{ CLASS : "je třídní učitel"

    USER ||--o{ TRIP : "vytváří (owner)"
    USER ||--o{ TRIP : "vede (leader)"
    TRIP ||--o{ TRIP_CLASS : "platí pro třídy"
    CLASS ||--o{ TRIP_CLASS : ""
    TRIP ||--o{ TRIP_ELIGIBLE_GRADE : "otevřen pro ročníky"
    TRIP ||--o{ TRIP_STAFF : "má zaměstnance"
    USER ||--o{ TRIP_STAFF : "účastní se jako zaměstnanec"

    TRIP ||--o{ APPROVAL_RECORD : "má záznamy schvalování"
    USER ||--o{ APPROVAL_RECORD : "schvaluje"

    TRIP ||--o{ TRIP_REGISTRATION : "má přihlášky"
    STUDENT_PROFILE ||--o{ TRIP_REGISTRATION : "je přihlášen"
    USER ||--o{ TRIP_REGISTRATION : "přihlašuje (žák nebo rodič)"

    TRIP ||--o{ NOTIFICATION_SUBSCRIPTION : "má odběratele notifikací"
    USER ||--o{ NOTIFICATION_SUBSCRIPTION : "odebírá notifikace"
```

## Klíčové poznámky k datovému modelu

### USER vs. role
- `system_role` určuje co uživatel v systému může dělat globálně
- Jeden uživatel může mít roli `teacher` a zároveň mít dítě jako žáka (rodič) — to by se řešilo přes `PARENT_STUDENT` napojením i bez role `parent` nebo přes multi-role systém (rozhodnutí pro tým)

### TRIP — typ a podmínky přístupu
- `type = class` → přihlášení jen skrz `TRIP_CLASS` (konkrétní třídy)
- `type = school` → přihlášení skrz `TRIP_ELIGIBLE_GRADE` (ročníky), nebo prázdné = všichni

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
