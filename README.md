# Propuesta de Arquitectura: Grupos de Tarjetas para PWA

## Resumen
Esta propuesta describe una nueva arquitectura para la gestión de tarjetas de regalo en la PWA, desacoplando la identidad del dispositivo (PWA) de la identidad del usuario (Cuenta Social). Se introduce el concepto de **Grupo de Tarjetas** (`CardGroup`) como entidad central.

## Problema Actual
Actualmente, existe una relación directa o confusa entre los respaldos de PWA y los usuarios, lo que complica escenarios como:
*   Usuarios anónimos (sin cuenta) que quieren guardar tarjetas.
*   Usuarios que cambian de dispositivo y quieren recuperar sus tarjetas fácilmente.
*   Usuarios con múltiples dispositivos.

## Solución Propuesta: "Card Groups"
Crear una entidad intermedia `CardGroup` que agrupa las tarjetas.
*   **La PWA (Dispositivo)** se conecta a un Grupo.
*   **El Usuario (Cuenta)** reclama la propiedad de un Grupo.
*   Las tarjetas pertenecen al Grupo, no al usuario ni al dispositivo directamente.

---

## Diagramas de Arquitectura

### 1. Vista General
Muestra las entidades principales y sus relaciones de alto nivel.

```mermaid
erDiagram
    CardGroup ||--o{ ValeRegaloComprado : "contiene"
    
    PwaGiftCardBackup ||--o{ Pwa_CardGroup_Rel : "gestiona"
    CardGroup ||--o{ Pwa_CardGroup_Rel : "se visualiza en"
    
    Tarjetahabiente ||--o{ Account_CardGroup_Rel : "posee"
    CardGroup ||--o{ Account_CardGroup_Rel : "propiedad de"
```

### 2. Detalle: Conexión PWA (Dispositivo)
Cómo un dispositivo físico (móvil/desktop) accede a las tarjetas. Puede haber múltiples dispositivos viendo el mismo grupo.

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

### 3. Detalle: Conexión Usuario (Cuenta)
Cómo un usuario registrado toma posesión de un grupo. Esto habilita beneficios, recuperación en nuevos dispositivos y seguridad.

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

### 4. Detalle: Estructura del Grupo
La relación entre el grupo y las tarjetas individuales (`ValeRegaloComprado`).

```mermaid
erDiagram
    CardGroup {
        bigint id PK
        string nombre "Nombre opcional"
        string uuid "ID único global"
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

---

## Beneficios Principales
1.  **Flexibilidad**: Permite uso anónimo inmediato (creando un Grupo sin dueño) y registro posterior (asignando dueño al Grupo).
2.  **Multidispositivo**: Un usuario puede escanear un QR en otro dispositivo y "sincronizar" el grupo fácilmente.
3.  **Simplicidad**: La lógica de negocio (vencimientos, saldos) se mantiene en `ValeRegaloComprado`, pero la agrupación se gestiona limpiamente en `CardGroup`.
