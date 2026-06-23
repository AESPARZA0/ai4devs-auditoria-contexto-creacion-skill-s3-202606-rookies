---
name: add-l10n
description: Agrega una nueva clave de localización a los 5 archivos ARB del proyecto y ejecuta flutter gen-l10n. Aplica automáticamente las sustituciones de términos de dominio por locale usando el diccionario de traducciones.
---

# Agregar localización (add-l10n)

## Flujo

1. **Solicitar al usuario:**
   - `key` — nombre en formato `prefijo_camelCase` (ej. `shop_addToCart`)
   - Texto en `es_MX` (base español México)
   - Texto en `en_US` (base inglés USA)
   - Confirmación antes de escribir

2. **Leer el diccionario** `lib/l10n/translation_dictionary.json`

3. **Derivar las 5 traducciones:**
   - `es` → igual a `es_MX`
   - `es_MX` → texto base provisto
   - `es_US` → aplicar sustituciones del diccionario sobre `es_MX`
   - `en` → igual a `en_US`
   - `en_US` → texto base provisto, aplicar sustituciones del diccionario

4. **Mostrar al usuario las 5 traducciones** y pedir confirmación o correcciones

5. **Insertar la clave** en los 5 ARBs

6. **Ejecutar** `flutter gen-l10n`

---

## Archivos ARB

| Archivo | Locale | Derivación |
|---|---|---|
| `lib/l10n/app_es.arb` | `es` | = `es_MX` |
| `lib/l10n/app_es_MX.arb` | `es_MX` | Texto base español |
| `lib/l10n/app_es_US.arb` | `es_US` | `es_MX` + sustituciones del diccionario |
| `lib/l10n/app_en.arb` | `en` | = `en_US` |
| `lib/l10n/app_en_US.arb` | `en_US` | Texto base inglés |

---

## Diccionario de traducciones

Ubicación: `lib/l10n/translation_dictionary.json`

Estructura — cada locale define qué términos sustituir en su texto:

```json
{
  "es_US": { "término_en_esMX": "reemplazo_para_esUS" },
  "en_US": { "término_en_esMX": "reemplazo_para_enUS" }
}
```

Leer el archivo y aplicar **todas** las sustituciones definidas al generar `es_US` y `en_US`.
Las sustituciones son sensibles a mayúsculas — aplicar exactamente como están definidas.

---

## Convención de claves

- Formato: `prefijo_camelCase`
- Prefijos por feature: `login_`, `general_`, `shop_`, `backoffice_`, `chat_`, `notifications_`, `goalProgram_`, `pathway_`, `expressStart_`, `scheduledMessages_`, `downloads_`
- Insertar junto a las demás claves del mismo prefijo si existen en el archivo

---

## Inserción en ARBs

- Leer el archivo ARB completo
- Localizar el bloque del mismo prefijo y añadir al final del grupo
- Si no hay claves del mismo prefijo, insertar antes del `}` de cierre
- Añadir coma al final de la entrada anterior si no la tiene
- Indentación: 4 espacios
- No dejar coma trailing en la última entrada del archivo

Formato de entrada:
```json
    "miClave": "Texto del valor"
```

---

## Comando de generación

Tras escribir los 5 archivos, ejecutar:

```
flutter gen-l10n
```

Esto regenera `lib/l10n/app_localizations.dart`, `app_localizations_en.dart` y `app_localizations_es.dart`.

---

## Notas

- Si el usuario corrige alguna traducción en el paso de confirmación, actualizar antes de escribir
- Si la clave ya existe en algún ARB, reportarlo y no sobreescribir
- El usuario puede omitir `en_US` si pide que lo traduzcas; en ese caso proponer una traducción y pedir confirmación
