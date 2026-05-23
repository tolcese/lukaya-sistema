# LUKAYA! — Sistema de Control de Operaciones

Sistema web para la gestión y seguimiento de solicitudes de créditos prendarios vehiculares. Permite registrar, editar y monitorear operaciones de financiación con cálculo automático de comisiones y ganancia.

---

## Tecnologías

- **React 18** (UMD, sin build — archivo HTML único autocontenido)
- **Babel Standalone** (transpilación JSX en el browser)
- **Supabase** (base de datos y API REST)
- Sin dependencias de Node.js ni proceso de compilación

---

## Cómo usar

Abrir `index.html` directamente en el navegador. No requiere servidor local ni instalación.

### Credenciales de acceso

| Usuario | Contraseña   |
|---------|-------------|
| admin   | lukaya2026  |

> La sesión se guarda en `sessionStorage` (se cierra al cerrar la pestaña).

---

## Funcionalidades

### Listado de operaciones
- Tabla con todas las solicitudes ordenadas por fecha de creación (más recientes primero).
- Búsqueda por cliente, DNI, dominio, vehículo o concesionaria.
- Filtro por **estado** y por **banco**.
- Barra de resumen con totales: operaciones, liquidadas, importe total y ganancia total LUKAYA!.
- Colores diferenciados por estado (verde = liquidado, rojo suave = baja).

### Nueva operación / Edición
Formulario dividido en secciones:

| Sección | Campos principales |
|---|---|
| Datos Generales | Fecha solicitud, estado, banco, concesionaria, provincia, localidad |
| Datos del Cliente | DNI, nombre, teléfono, mail |
| Datos del Vehículo | Dominio, descripción del vehículo |
| Datos del Crédito | Fecha venc. 1° cuota, tipo de cuota, cantidad, tasa, importe, gasto gestoría |
| Documentación | Checkboxes de documentación requerida |
| Liquidación | Fecha, pago extras, comisión operativo, comisión gestora interior, retribución banco |
| Pago | Estado de pago y fecha; estado de pago comisión gestora y fecha |
| Observaciones | Texto libre |

### Vista detalle
Modal de solo lectura con todos los datos de la operación y los cálculos calculados automáticamente.

### Eliminar
Confirmación antes de borrar. La operación se elimina permanentemente de Supabase.

---

## Cálculos automáticos

Todos se calculan en tiempo real en el formulario y en la tabla, a partir del **Importe Solicitado**:

| Campo | Fórmula |
|---|---|
| 1,2% Imp. Crédito | `importe × 0.012` |
| Comisión LUKAYA! (2%) | `importe × 0.02` |
| A Liquidar Agencia | `importe − 1,2% − 2% − gasto gestoría` |
| **Ganancia LUKAYA!** | `retribución banco + comisión LUKAYA! − comisión gestora − comisión operativo − pago extras` |

---

## Estados posibles

| Estado | Color |
|---|---|
| SOLICITADO | Azul |
| CARGADA DOCUMENTACION | Naranja |
| EN PROCESO LIQ. | Amarillo |
| LIQUIDADO | Verde |
| BAJA | Rojo |

---

## Bancos disponibles

COLUMBIA · SANTANDER · GALICIA · NACIÓN · PROVINCIA · SUPERVIELLE · OTRO

---

## Base de datos (Supabase)

- **Proyecto:** `zohniclbfhvqmurtrhka.supabase.co`
- **Tabla:** `lukaya_solicitudes`

### Estructura de la tabla

| Columna | Tipo | Descripción |
|---|---|---|
| id | uuid / serial | PK, generado automáticamente |
| created_at | timestamp | Generado automáticamente por Supabase |
| fecha_solicitud | date | |
| estado | text | Ver estados posibles |
| banco | text | |
| concesionaria | text | |
| provincia | text | |
| localidad | text | |
| dni_cliente | text | |
| nombre_cliente | text | |
| telefono | text | |
| mail | text | |
| dominio_vehiculo | text | |
| vehiculo | text | |
| fecha_venc_primera_cuota | date | |
| tipo_cuota | text | `FIJA` o `VARIABLE` |
| cant_cuotas | integer | |
| importe_solicitado | numeric | |
| gasto_gestoria | numeric | |
| tasa | numeric | Default: `3.2` |
| pago_extras | numeric | |
| comision_operativo | numeric | |
| comision_gestora_interior | numeric | |
| retribucion_banco | numeric | |
| documento_01_08 | boolean | |
| titulo_unidad | boolean | |
| verif_policial | boolean | |
| informe_dominio | boolean | |
| constancia_entrega | boolean | |
| fecha_liquidacion | date | |
| pagado | boolean | |
| fecha_pagado | date | |
| pago_com_gest | boolean | |
| fecha_pago_com_gest | date | |
| observaciones | text | |

---

## Estructura del código

```
index.html
│
├── <style>          → CSS embebido (diseño completo)
│
└── <script babel>
    ├── Constantes   → SB_URL, SB_KEY, ESTADOS, BANCOS, PROVS, EMPTY
    ├── Utilidades   → pn(), toDisp(), toISO(), calcular(), fmt(), fmtDate(), badgeCls()
    ├── Componentes base
    │   ├── FField      → Campo de formulario (input / select)
    │   ├── CFField     → Campo calculado (readonly)
    │   └── ChkField    → Checkbox estilizado
    ├── Login           → Pantalla de autenticación
    ├── FormModal       → Modal alta / edición
    ├── DetailModal     → Modal vista detalle
    ├── Sistema         → Vista principal (tabla + filtros + totales)
    └── App             → Raíz (manejo de sesión)
```

---

## Notas

- Las credenciales de acceso están hardcodeadas en el HTML. Para un entorno productivo se recomienda migrar la autenticación a Supabase Auth.
- La API key de Supabase incluida es de tipo `publishable` (solo lectura/escritura sobre las tablas con RLS configurado).
- Las fechas se muestran en formato `DD/MM/AAAA` y se guardan en Supabase en formato ISO `YYYY-MM-DD`.
