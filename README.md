# Propuesta de Arquitectura: Grupos de Tarjetas para PWA (V2)

## Resumen
Arquitectura actualizada para la gestión de tarjetas en PWA, incorporando **reglas de negocio estrictas** y un **flujo de seguridad por autorización de dispositivos**.

## Reglas de Negocio (Actualizadas)
1.  **Unicidad PWA**: Una PWA (Dispositivo) solo puede estar visualizando **un solo Grupo de Tarjetas** a la vez.
2.  **Unicidad Usuario**: Un Usuario (Tarjetahabiente) solo puede ser dueño de **un solo Grupo de Tarjetas**.
3.  **Seguridad**: Para migrar un grupo a un nuevo dispositivo (PWA), el dispositivo anterior (Master) debe autorizar la operación.

---

## Diagramas de Arquitectura

### 1. Vista General

```mermaid
erDiagram
    CardGroup ||--o{ CardGroupMember : "contiene tarjetas"
    
    PwaGiftCardBackup }|--|| CardGroup : "visualiza (1 máx)"
    
    Tarjetahabiente ||--|| Account_CardGroup_Rel : "se vincula (1:1)"
    CardGroup ||--|| Account_CardGroup_Rel : "pertenece a (1:1)"

    PwaGiftCardBackup ||--o{ DeviceAuthorizationRequest : "gestiona acceso"
```

### 2. Tablas Principales

#### `CardGroup`
El contenedor central.
*   `id`: PK
*   `uuid`: Identificador único público.
*   `recovery_email_hash`: Hash para recuperar por correo.
*   `recovery_phone_hash`: Hash para recuperar por teléfono.

#### `PwaGiftCardBackup` (La PWA)
Ahora contiene referencia directa al grupo activo.
*   `id`: PK
*   `device_id`: Huella del dispositivo.
*   `card_group_id`: FK hacia `CardGroup`. (Regla: 1 PWA -> 1 Grupo).
*   `is_master_device`: Booleano, indica si este dispositivo puede autorizar a otros.

#### `Account_CardGroup_Rel` (Vinculación Usuario)
Tabla 1:1.
*   `th_id`: FK Usuario.
*   `card_group_id`: FK Grupo.

#### `DeviceAuthorizationRequest` (NUEVO: Seguridad)
Maneja el flujo de "Pedir permiso al celular viejo".
*   `id`: PK
*   `card_group_id`: El grupo que se quiere **Unificar** o acceder.
*   `requesting_pwa_id`: El nuevo dispositivo.
*   `authorizing_pwa_id`: El dispositivo anterior (que debe aprobar).
*   `status`: 'PENDING', 'APPROVED', 'DENIED'.
*   `token`: Token para validar la acción.
*   `expires_at`: Ventana de tiempo para autorizar (ej. 15 min).

---

## Flujos Críticos

### A. Agregar Tarjeta en Dispositivo Nuevo (Lógica)
1.  **Tarjeta Nueva (Sin Grupo)**:
    *   Si el Dispositivo NO tiene grupo -> Se crea un NUEVO Grupo para el Dispositivo y se agrega la tarjeta. (Sin permiso).
    *   Si el Dispositivo YA tiene grupo -> Se agrega la tarjeta a su Grupo actual. (Sin permiso).
    
2.  **Tarjeta con Grupo Existente**:
    *   Si el Grupo de la tarjeta == Grupo del Dispositivo -> Se agrega/actualiza vista. (Sin permiso).
    *   Si el Grupo de la tarjeta != Grupo del Dispositivo:
        *   **Conflicto**: El dispositivo quiere reclamar un grupo que es de otro.
        *   **Solicitud**: Se activa el flujo de `DeviceAuthorizationRequest` hacia el dueño actual del grupo.


### B. Usuario Registrado y PWA
1.  Usuario hace login.
2.  Sistema busca su Grupo único en `Account_CardGroup_Rel`.
3.  La PWA actual se actualiza para apuntar a ese Grupo.

### C. Recuperación (PWA Borrada / Datos Perdidos)
Si el usuario borra los datos, pierde su "identidad de dispositivo". Para recuperar el grupo:
1.  **Detección**: Usuario ingresa a una liga de tarjeta del grupo o intenta "recuperar" manualmente.
2.  **Opción Contacto**: Si el `CardGroup` tiene `recovery_email` o `recovery_phone`:
    *   Sistema envía OTP (Código) al medio de contacto.
    *   Si usuario ingresa el código correcto -> El Nuevo Dispositivo toma posesión del Grupo.
3.  **Opción Cuenta**: Si el usuario hace Login, aplica el Flujo B.

