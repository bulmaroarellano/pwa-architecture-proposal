# Diagramas ER Enfocados: Arquitectura "Grupo de Tarjetas"

Para facilitar la lectura, he dividido el diagrama en partes más pequeñas y específicas.

## 1. Vista General (Simplificada)
Muestra solo las entidades principales y cómo se conectan. Ideal para entender la arquitectura a grandes rasgos.

```mermaid
erDiagram
    CardGroup ||--o{ ValeRegaloComprado : "contiene"
    
    PwaGiftCardBackup ||--o{ Pwa_CardGroup_Rel : "gestiona"
    CardGroup ||--o{ Pwa_CardGroup_Rel : "se ve en"
    
    Tarjetahabiente ||--o{ Account_CardGroup_Rel : "posee"
    CardGroup ||--o{ Account_CardGroup_Rel : "propiedad de"
```

---

## 2. Detalle: Lado PWA (Dispositivo)
Enfocado en cómo el dispositivo (PWA) se conecta al Grupo de Tarjetas.

```mermaid
classDiagram
    class PwaGiftCardBackup {
        +bigint id
        +string device_id
        +datetime last_seen_at
    }

    class Pwa_CardGroup_Rel {
        +bigint id
        +bigint pwa_id
        +bigint card_group_id
        +datetime ligado_at
        +boolean is_active
    }

    class CardGroup {
        +bigint id
        +string nombre
    }

    PwaGiftCardBackup "1" --> "*" Pwa_CardGroup_Rel : tiene
    Pwa_CardGroup_Rel "*" --> "1" CardGroup : apunta a
```

---

## 3. Detalle: Lado Cuenta (Usuario)
Enfocado en cómo el usuario registrado se conecta al Grupo de Tarjetas para reclamar propiedad.

```mermaid
classDiagram
    class Tarjetahabiente {
        +bigint id
        +string email
        +string nombre
    }

    class Account_CardGroup_Rel {
        +bigint id
        +bigint th_id
        +bigint card_group_id
        +datetime ligado_at
        +string rol
    }

    class CardGroup {
        +bigint id
        +string nombre
    }

    Tarjetahabiente "1" --> "*" Account_CardGroup_Rel : tiene
    Account_CardGroup_Rel "*" --> "1" CardGroup : vincula
```

---

## 4. Detalle: Grupo y sus Tarjetas
Muestra la estructura interna del Grupo y las Tarjetas que contiene.

```mermaid
erDiagram
    CardGroup {
        bigint id PK
        string nombre
        string uuid
        datetime created_at
    }

    ValeRegaloComprado {
        bigint id PK
        bigint card_group_id FK
        string folio
        decimal monto
        int estatus
        datetime fecha_vencimiento
    }

    CardGroup ||--o{ ValeRegaloComprado : "contiene (1:N)"
```
