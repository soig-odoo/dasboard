# Informe de eventos — tablero o-spreadsheet (edu-mcp)

## Qué contiene este repo

| Fichero | Uso |
|---|---|
| `informe_de_eventos.osheet.json` | **El tablero.** Una sola línea, 97.034 caracteres. Subir a Odoo tal cual. |
| `payload_ai_tool_update_records.json` | Solo para escritura vía herramienta `ai_tool_update_records` (no hace falta para importación manual). |

## Verificación de integridad (importante)

La importación anterior falló porque el JSON pegado por chat llegó corrupto a la base de datos
(`sheets` dejó de ser una lista → `TypeError: object is not iterable` en `fixFigureOffset`).
Descarga el fichero desde GitHub (botón *Raw*) en lugar de copiar/pegar.

SHA-256 del fichero correcto:

```
aab7b9fade5ecefdfb17ce13867184ee714dc0bee7ca77bea1e057899ef89b0b
```

Comprobación rápida (Linux/Mac): `sha256sum informe_de_eventos.osheet.json`
(Windows PowerShell): `Get-FileHash informe_de_eventos.osheet.json -Algorithm SHA256`

## Correcciones aplicadas respecto a la versión anterior

1. `version`: `"20.0"` → `"18.4.14"` — las versiones de datos reales de o-spreadsheet no siguen
   la versión comercial de Odoo (un dashboard oficial de Odoo 20 usa `19.3.10`; el check
   `postdates` rechaza versiones futuras). Con `18.4.14`, Odoo 19 lo carga tal cual y Odoo 20
   lo migra automáticamente hasta su versión actual.
2. Filtro global de fecha: eliminado `rangeType` (esa clave se elimina por migración desde
   18.4.14; el periodo va implícito en `this_month`).
3. Gráficos: convertidos al formato legacy (`dataSets` + `dataSetsHaveTitle` + `labelRange`,
   color por dataset con `backgroundColor`, no `fill`) — compatible con Odoo 19 y, vía
   migración 19.3.2, con Odoo 20.
4. Celdas numéricas: emitidas como strings (`"20"`, no `20`) — en el formato compacto actual
   un número tras una fórmula se interpreta como offset squished y rompe la carga.

Validado cargando el fichero con los paquetes reales `@odoo/o-spreadsheet` 19.0.0 y 20.0.6:
5 hojas, 48 pivotes, 2 listas, 3 gráficos, 0 fórmulas `DATE(`, 0 `#` en pivotes.

## Importación

1. **Borra primero el dashboard roto** de ceoecampus (el que da el TypeError).
2. Descarga `informe_de_eventos.osheet.json` en crudo (botón *Raw* → guardar como).
3. Impórtalo en Odoo por la misma vía que usaste antes (campo *Spreadsheet file* del
   dashboard / importación de `.osheet.json`).
4. Nombre: **Informe de eventos** · Menú: **Ventas** · Publicar: sí.

## Pasos manuales en la base de datos destino

1. **Crear las 14 etiquetas `crm.tag` en este orden** (los IDs 1–14 están asumidos en los
   42 dominios de pivote):
   1 Futuras Consejeras · 2 Promociona · 3 Progresa VIII · 4 Proactiva · 5 Progresa FOMENT ·
   6 Progresa TENERIFE · 7 DTS · 8 FS Sectoriales · 9 People r_Evolution · 10 Ecosistema MK ·
   11 Geoestrategia II · 12 Geoestrategia III · 13 Negociación · 14 Liderazgo Público.
   Si los IDs reales difieren, hay que ajustar el `["order_id.tag_ids","=",ID]` de los 42
   pivotes de programa (hoja principal, 3 bloques de año).
2. **Etiquetar los pedidos de venta confirmados** (`state=sale`) con esas etiquetas: el tablero
   solo cuenta pedidos confirmados, no facturación.
3. Publicar al menú **Ventas**: abrir el dashboard → Archivo → *Añadir al panel*.
4. (Opcional) El presupuesto es estático (fila TOTAL 3.349.000 € y objetivos por comercial);
   si se quieren ejercicios pasados, editar esas celdas a mano.
