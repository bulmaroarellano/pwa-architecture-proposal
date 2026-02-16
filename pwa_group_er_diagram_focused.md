# Diagramas ER Enfocados: Arquitectura "Grupo de Tarjetas" (V2)

Diagramas actualizados según reglas de negocio estrictas:
1.  **1 PWA = 1 Grupo**.
2.  **1 Usuario = 1 Grupo**.
3.  **Flujo de Autorización entre Dispositivos**.

## 1. Vista General (Simplificada)

```mermaid
erDiagram
    CardGroup ||--o{ CardGroupMember : "contiene tarjetas"
    
    PwaGiftCardBackup }|--|| CardGroup : "visualiza (1 grupo máx)"
    
    Tarjetahabiente ||--|| Account_CardGroup_Rel : "se vincula (1:1)"
    CardGroup ||--|| Account_CardGroup_Rel : "pertenece a (1:1)"

    PwaGiftCardBackup ||--o{ DeviceAuthorizationRequest : "solicita/autoriza"
```

---

## 2. Detalle: PWA y Autorización
Muestra que una PWA solo puede tener un Grupo activo y el mecanismo de seguridad para cambiar de dispositivo.

```mermaid
classDiagram
    class PwaGiftCardBackup {
        +bigint id
        +string device_id
        +bigint card_group_id FK "Grupo Activo (Solo 1)"
        +string fingerprint
    }

    class DeviceAuthorizationRequest {
        +bigint id
        +bigint card_group_id
        +bigint requesting_pwa_id "Nuevo Dispositivo"
        +bigint authorizing_pwa_id "Dispositivo Anterior"
        +string token
        +enum status "PENDING, APPROVED, DENIED"
        +datetime expires_at
    }

    class CardGroup {
        +bigint id
        +string nombre
    }

    PwaGiftCardBackup "*" --> "1" CardGroup : visualiza
    PwaGiftCardBackup "1" --> "*" DeviceAuthorizationRequest : crea solicitud
```

---

## 3. Detalle: Usuario y Grupo (Relación 1 a 1)
Regla estricta: Un usuario solo puede tener un grupo ligado.

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

    Tarjetahabiente "1" --> "1" Account_CardGroup_Rel : tiene
    CardGroup "1" --> "1" Account_CardGroup_Rel : propiedad de
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
