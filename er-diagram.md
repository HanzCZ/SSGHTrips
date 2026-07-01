# ER Diagram — Datový model

```mermaid
erDiagram

    USER {
        int id
        string first_name
        string last_name
        string email
        date birth_date
        boolean is_adult
        string system_role
    }

    CLASS {
        int id
        string name
        int grade_year
        int homeroom_teacher_id
    }

    STUDENT_PROFILE {
        int id
        int user_id
        int class_id
    }

    PARENT_STUDENT {
        int parent_user_id
        int student_profile_id
    }

    TRIP {
        int id
        string name
        string description
        string type
        string status
        int capacity
        decimal student_budget
        decimal school_budget
        date start_date
        date end_date
        date registration_deadline
        int created_by
        int leader_user_id
        timestamp created_at
        timestamp updated_at
    }

    TRIP_CLASS {
        int trip_id
        int class_id
    }

    TRIP_ELIGIBLE_GRADE {
        int trip_id
        int grade_year
    }

    TRIP_STAFF {
        int id
        int trip_id
        int user_id
        string role_label
    }

    APPROVAL_RECORD {
        int id
        int trip_id
        int approver_user_id
        string action
        string note
        timestamp created_at
    }

    TRIP_REGISTRATION {
        int id
        int trip_id
        int student_profile_id
        int registered_by_user_id
        string status
        string rejection_note
        timestamp registered_at
        timestamp reviewed_at
    }

    NOTIFICATION_SUBSCRIPTION {
        int id
        int trip_id
        int user_id
        timestamp subscribed_at
    }

    %% Relationships

    USER ||--o{ STUDENT_PROFILE : "has student profile"
    USER ||--o{ PARENT_STUDENT : "is parent"
    STUDENT_PROFILE ||--o{ PARENT_STUDENT : "has parent"
    CLASS ||--o{ STUDENT_PROFILE : "contains students"
    USER ||--o{ CLASS : "is homeroom teacher"

    USER ||--o{ TRIP : "creates (owner)"
    USER ||--o{ TRIP : "leads (leader)"
    TRIP ||--o{ TRIP_CLASS : "assigned to class"
    CLASS ||--o{ TRIP_CLASS : "included in trip"
    TRIP ||--o{ TRIP_ELIGIBLE_GRADE : "open to grade"
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
