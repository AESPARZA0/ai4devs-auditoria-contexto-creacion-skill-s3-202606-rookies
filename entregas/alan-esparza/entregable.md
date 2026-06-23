# Entregable · Sesión 3 — Copilotos IA

- **Nombre / usuario:** Alan Esparza
- **Fecha de entrega:** 22 de junio de 2026
- **Repo auditado en la Parte A:** App Flutter de producción, ~192 archivos Dart, ~104K líneas, con documentación interna extensa en `Docs/`

---

## 1. Hallazgos de la auditoría (Parte A)

> 3-5 cosas que el agente **no pudo inferir** del código y que tendrías que decirle explícitamente.
> Redáctalas para que otra persona las entienda sin contexto adicional. **Sin código propietario ni secretos.**

1. **Generación de localización:** El agente detectó los ARBs y los archivos `app_localizations_*.dart` generados, pero no pudo saber cómo los regenera el equipo. No quedó claro si se corre `flutter gen-l10n` manualmente, si hay un watcher, si los archivos generados se commitean o solo se producen en CI, ni quién actualiza los ARBs cuando se añade texto nuevo. Sin esa información, añadir una clave nueva podría romper el flujo de trabajo sin que nadie lo note a tiempo.

2. **Build y firma para release:** Aunque el proyecto tiene configuración nativa Android/iOS, no hay scripts de build, Makefile, fastlane ni workflows de CI documentados. El agente no pudo inferir el comando exacto de build para producción (flags, targets, signing), dónde viven los keystores y certificados, ni si hay pasos manuales antes de subir a las stores.

3. **Entornos de backend:** El agente no pudo saber si existe staging o QA, cómo se prueban features sin afectar producción, ni si hay algún override local fuera del repo. Cualquier tarea del tipo "conecta esto al ambiente de pruebas" quedaría bloqueada sin contexto explícito.

4. **Obligatoriedad de actualizar `Docs/Technical/`:** La documentación interna indica que Docs es la fuente primaria y el código la validación. El agente no pudo determinar si actualizar el `.md` técnico al modificar un archivo es requisito del equipo o una práctica opcional y asíncrona.

5. **Registro de pantallas nuevas en dos catálogos de analytics:** Al crear una pantalla hay que registrar su nombre en dos mapas distintos (Firebase Analytics y AppMetrics). Son casi idénticos pero no están sincronizados. Si solo registro la ruta en `main.dart`, la pantalla funciona pero queda mal etiquetada en ambos sistemas, sin error visible ni test que lo detecte. Tampoco quedó claro cuál mapa es la fuente de verdad ni si los nombres deben coincidir exactamente entre ambos.

---

## 2. SKILL.md de la skill creada (Parte B)

> Archivo en el repo: [`entregas/alan-esparza/.claude/skills/add-l10n/SKILL.md`](.claude/skills/add-l10n/SKILL.md)

````markdown
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
````

---

## 3. Diario de decisiones

*Skill creada:* `add-l10n` — automatiza el alta de claves de localización en los 5 ARBs del proyecto, aplica sustituciones de términos de dominio y ejecuta `flutter gen-l10n`. La elegí porque hoy hacemos las localizaciones a mano, nos quita mucho tiempo y es un flujo repetitivo que un agente puede ejecutar bien.

*Decisiones de diseño tomadas:*
- **Solo pedir `es_MX` y `en_US` como textos base:** Los locales `es` y `en` son fallbacks; no tiene sentido pedirlos por separado porque se derivan de las variantes regionales principales.
- **Sustituciones en `translation_dictionary.json`:** En la empresa manejamos términos en español que no se traducen literalmente al inglés (y variantes MX vs US). Ese catálogo crece con el tiempo, así que lo externalicé en un JSON en lugar de hardcodearlo en la skill.
- **Confirmación antes de escribir:** No quise que fuera totalmente automático. Las traducciones tienen matices específicos del negocio y prefiero revisar las 5 variantes derivadas antes de que el agente toque los ARBs.

*Qué me resultó fácil:*
- Redactar las instrucciones del flujo completo, porque es algo que hago a diario y el conocimiento está fresco.
- Revisar el resultado de la skill: ya sé exactamente qué espero (5 claves coherentes, sustituciones aplicadas, `gen-l10n` ejecutado).

*Qué me resultó ambiguo o difícil de decidir:*
- Redactar el `description` del frontmatter para que la skill se active automáticamente cuando haga falta, sin que se dispare en tareas que no son de localización.

*Tiempo real invertido:*
- Lectura previa: ~20 min
- Diseño del flujo: ~30 min
- Escritura del `SKILL.md`: ~40 min
- **Total: ~1 h 30 min**

*Qué probarías si tuvieras más tiempo:*
- Ver si la skill es capaz de detectar inconsistencias en las traducciones que le proporcione (por ejemplo, términos de dominio mal aplicados o variantes que no coinciden entre locales).

*¿Usaste IA para crear la skill?* (qué partes generaste con IA y qué partes decidiste tú)
- Usé Claude para plasmar la skill en el archivo `SKILL.md`. El funcionamiento completo — flujo, reglas de derivación, diccionario, confirmaciones y comando de generación — lo decidí yo; Claude solo ayudó a estructurarlo por escrito.

### Resultado de la prueba (Paso 8)

- ¿Se activó cuando lo esperabas? **Sí.** Se activó en el contexto correcto al pedir agregar una localización.
- ¿El resultado fue el que querías? **Sí.** Las 5 traducciones generadas fueron correctas, con las sustituciones de dominio aplicadas como esperaba.
- Si no, ¿qué crees que falló? N/A — el primer intento funcionó como esperaba: insertó las claves en los ARBs y ejecutó `flutter gen-l10n` automáticamente.
