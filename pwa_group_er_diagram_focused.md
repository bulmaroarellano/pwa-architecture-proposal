# Diagramas ER Enfocados: Arquitectura "Grupo de Tarjetas" (V2)

Diagramas actualizados según reglas de negocio flexibles:
1.  **1 PWA = N Grupos**.
2.  **1 Usuario = N Grupos**.
3.  **Flujo de Autorización entre Dispositivos**.

## 1. Vista General (Simplificada)

```mermaid
erDiagram
    CardGroup ||--o{ CardGroupMember : "contiene tarjetas"
    
    PwaGiftCardBackup }|--|{ Pwa_CardGroup_Rel : "visualiza (N)"
    CardGroup ||--|{ Pwa_CardGroup_Rel : "se ve en (N)"
    
    Tarjetahabiente ||--|{ Account_CardGroup_Rel : "posee (N)"
    CardGroup ||--|| Account_CardGroup_Rel : "pertenece a (1 Dueño)"

    PwaGiftCardBackup ||--o{ DeviceAuthorizationRequest : "solicita/autoriza"
```

---

## 2. Detalle: PWA y Autorización
Muestra que una PWA puede tener múltiples grupos.

```mermaid
classDiagram
    class PwaGiftCardBackup {
        +bigint id
        +string device_id
        +string fingerprint
    }

    class Pwa_CardGroup_Rel {
        +bigint id
        +bigint pwa_id
        +bigint card_group_id
        +boolean is_active "Si este es el grupo visible actual"
    }

    class DeviceAuthorizationRequest {
        +bigint id
        +bigint card_group_id
        +bigint requesting_pwa_id "Nuevo Dispositivo"
        +bigint authorizing_pwa_id "Dispositivo Anterior"
        +enum status
    }

    class CardGroup {
        +bigint id
        +string nombre
    }

    PwaGiftCardBackup "1" --> "*" Pwa_CardGroup_Rel : tiene
    Pwa_CardGroup_Rel "*" --> "1" CardGroup : apunta a
    PwaGiftCardBackup "1" --> "*" DeviceAuthorizationRequest : crea solicitud
```

---

## 3. Detalle: Usuario y Grupo (Relación 1 a N)
Un usuario puede ser dueño de múltiples grupos.

```mermaid
classDiagram
    class Tarjetahabiente {
        +bigint id
        +string email
    }

    class Account_CardGroup_Rel {
        +bigint id
        +bigint th_id FK
        +bigint card_group_id FK
        +datetime ligado_at
    }

    class CardGroup {
        +bigint id
        +string uuid
    }

    Tarjetahabiente "1" --> "*" Account_CardGroup_Rel : tiene
    Account_CardGroup_Rel "*" --> "*" CardGroup : vinculado
```

---

## 4. Detalle: Miembros del Grupo
Relación entre las tarjetas y el grupo. Se usa una tabla intermedia `CardGroupMember` para flexibilidad.

```mermaid
erDiagram
    CardGroup {
        bigint id PK
        string nombre
        string recovery_email_hash "Para recuperar si se borra PWA"
        string recovery_phone_hash "Para recuperar si se borra PWA"

    }

    CardGroupMember {
        bigint id PK
        bigint card_group_id FK
        bigint vale_regalo_id FK
        datetime added_at
    }

    ValeRegaloComprado {
        bigint id PK
        string folio
        decimal monto
    }

    CardGroup ||--o{ CardGroupMember : "tiene"
    ValeRegaloComprado ||--o{ CardGroupMember : "es parte de"
```
