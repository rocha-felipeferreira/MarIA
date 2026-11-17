---
name: my_status
description: Consulta cómo va Faena.smmx u otro mapa SimpleMind (.smmx): hoy, un día concreto, la semana, un nodo o proyecto. Informa por bloques (Trabajo / Personal) con barras de progreso, pendientes desplegados, deuda y propuestas de arrastre. Solo lectura.
---

# my_status

## Objetivo

Leer un mapa SimpleMind (`.smmx`) y responder cómo va el día, la semana, un nodo o el mapa completo, agrupando el resultado en dos bloques —**Trabajo** y **Personal**— con prioridades y consejos concretos.

## Reglas fijas

- Responde en el idioma del prompt; breve y concreto.
- **Solo lectura**: nunca escribas, muevas, marques ni renombres nodos del `.smmx`.
- **Carga = pendiente.** Solo las unidades pendientes cuentan como carga, prioridad o riesgo. Un nodo ✅ nunca se cita como carga.
- No inventes estados ni fechas. Si no puedes leer el mapa o no tiene checkboxes, dilo.
- El mapa no lleva fechas: `Lunes`–`Domingo` son **siempre** los de la semana real en curso. Usa el día real, sin avisar de desfase.
- Cierra siempre con 1–3 líneas de qué falta y qué priorizar.
- Informa por **bloques**, no por tareas. El detalle se despliega solo para lo pendiente.

## 1. Localizar el mapa

Raíz: `~/Library/CloudStorage/Dropbox/Mindmaps` (override: `MARIA_MAPS_ROOT`).

- **Ruta explícita** (`/`, `~`, `./`, `../`, `*.smmx`): úsala tal cual.
- **Referencia por nombre**: busca los `.smmx` bajo la raíz y compara el nombre sin extensión.
- **Referencia por título interno**: busca el `<title text="...">` dentro del mapa.
- **Sin argumento**: `Faena.smmx`.
- Si hay varios candidatos, muéstralos y pregunta. No elijas por tu cuenta.

El mapa por defecto está en `Compromisos/Faena.smmx`.

## 2. Leer sin modificar

El `.smmx` es un ZIP y el mapa está en `document/mindmap.xml`. Léelo sin extraerlo ni copiarlo dentro de Dropbox y sin modificar el original.

Dentro, cada `topic` es un nodo plano con `id`, `parent`, `text` y atributos de estado; los hijos se localizan por `parent`. Reconstruye el árbol desde la raíz (`parent="-1"`) y aplica las reglas de la sección 3.

Los totales X/Y no vienen calculados: cuéntalos tú aplicando la sección 3. Los contenedores `roll-up` no aportan estado propio; su estado se deduce de sus hijos.

## 3. Interpretar estados

- ✅ hecho (`checked="true"` o `progress="100"`).
- 🟡 parcial o en curso (`progress` 1–99, o rama con parte hecha).
- ❌ pendiente (checkbox sin marcar).
- • contenedor (`roll-up-progress` o sin checkbox); **no cuenta** directamente: su estado se recalcula desde las hojas.
- Los padres con `checkbox-mode="roll-up-progress"` se recalculan desde las hojas; nunca uses su `checked`.
- `\N` en el texto → ` / `.
- Un 🟡 **no suma** al porcentaje ni a los hechos: se cuenta aparte en `Parciales`.
- `cloudmapref`: solo si hace falta y solo un nivel, sin ciclos. No lo sigas por defecto.

**Barras**: 10 celdas, `█` llenas y `░` vacías, con `round(hechos/total*10)`. Total 0 → `—` (no dividas). Se muestran solo en los dos bloques, nunca por día ni por tarea.

### 3.1 Los dos bloques

Todo nodo se clasifica en uno de dos bloques:

- **Trabajo**: ramas cuyo nombre sea o contenga `Work`, `MarIA`, `Tech` (`Tech Excellence`, `Tech in Action`), `DigiNews`, `Slack`, `Doc` u `Onboarding`. **Lista cerrada**: la palabra "trabajo" no clasifica por sí sola (`día de trabajo - donar 75` es Personal).
- **Personal**: todo lo demás (Gym, Yoga, Comida, Súper, Empadronamiento, Terapia, Arrels, Mani…). Sin tercera categoría.

El nombre de la rama manda sobre sus hijos: `Work → MarIA → …` sigue siendo Trabajo.

**Nodo sin clasificar**: si un nodo **pendiente** no encaja en la lista de Trabajo y no es evidentemente Personal, **pregunta** antes de dar el informe (una pregunta corta; ofrece añadirlo a la lista). Si el nodo ya está hecho, no interrumpas: cuéntalo como Personal y anótalo en una línea al final (`Sin clasificar: 1`).

### 3.2 Cómo se cuenta

- **Trabajo**: cuenta las **hojas** del subárbol.
- **Personal**: cuenta la **actividad principal**, una unidad por actividad. Sus subnodos **no suman**: `Empadronamiento` con 7 documentos es 1 unidad; `Súper` con 13 ítems es 1 unidad.
- El estado de una actividad de Personal con hijos se deduce de **su subárbol** (todo hecho → ✅, parte hecha → 🟡, nada → ❌), aunque su propio checkbox diga otra cosa.
- **Contenedores agrupadores**: los nodos que solo agrupan (roll-up sin checkbox propio, o bloques como `Huelga 11N`) se **atraviesan**: no cuentan como unidad y no cortan la cuenta. `Miércoles → Huelga 11N → día de trabajo - donar 75 → Arrels / karma yoga` aporta 2 unidades (Arrels, karma yoga), no 4 ni 1.
- Un padre con checkbox propio y con hijos es contenedor de su rama: cuenta según el bloque al que pertenezca.

## 4. Elegir el contexto

| El prompt... | Ámbito |
|---|---|
| no da argumentos | Mapa `Faena`: hoy + resumen semanal |
| dice "hoy" | Día real, con efecto en la semana |
| dice un día ("viernes", "sábado") | Ese nodo, con efecto en la semana |
| dice "semana" | Semana laboral, por bloques |
| dice "todo" | Todo el ámbito activo, por bloques y prioridad |
| dice "backlog", "Brasil" | Ese nodo, separado de la semana |
| dice un nodo o proyecto ("MarIA", "Yoga") | Ese subárbol |
| da ruta + alcance | Ese mapa y ese alcance |

**Ámbito de la semana**, por orden de precedencia:

1. **Lo explícito manda**: si pides `sábado`, `domingo` o `finde`, esos días entran en el cómputo.
2. **Si el día real es fin de semana**, el ámbito incluye el finde hasta hoy: Lunes–Sábado o Lunes–Domingo.
3. **En cualquier otro caso** el ámbito es **Lunes–Viernes**: sábado y domingo no cuentan, no se muestran, no son deuda ni destino de arrastre.

La cabecera declara siempre el rango activo (`· Semana Lunes–Viernes`).

Si el día o nodo no existe, muestra las opciones y pregunta. Si el prompt es ambiguo ("cómo voy"), usa hoy y ofrece la semana.

**"Hoy"** = día real de la semana, no el primer día del mapa. El mapa no tiene fechas: los nodos de día son siempre la semana real en curso. Un título con fecha (ej. `Huelga 11N`) es solo la etiqueta del evento, nunca un indicio de otra semana: no lo menciones salvo que se pregunte.

## 5. Evaluar

- **Ritmo semanal**: días cerrados vs. días transcurridos del ámbito activo (5, 6 o 7 días).
- **Esfuerzo por bloque**: qué bloque concentra el pendiente y cuántas unidades son, no cuántas hojas.
- **Deuda**: los días anteriores que quedaron abiertos se mencionan **solo dentro de `Atención`**, con su desglose por bloque. No hay línea propia de deuda.
- **Sobrecarga**: avisa si un día supera ~8 unidades. No es una regla dura: explica el motivo.
- **Contenedores vacíos**: menciónalos (ej. un día sin tareas) porque suelen ser un hueco aprovechable.

## 6. Priorizar

Ordena por urgencia e impacto:

1. Deuda de días pasados.
2. Hoy: primero lo fijo (cita, papeleo, clase), después el trabajo flexible.
3. Resto de la semana, por cercanía.
4. Backlog solo si se pide.

Dentro de cada bloque: fechas explícitas → tareas que cierran una rama → bajo esfuerzo → orden del mapa.

Da **máximo 3 prioridades**, cada una con su motivo en una línea. Si todo cabe, dilo en vez de inventar urgencias.

## 7. Arrastre entre días (solo propuesta)

Si preguntan si arrastrar algo a otro día, o si hoy está sobrecargado:

1. Calcula el pendiente de hoy y la carga del día destino.
2. Clasifica: **fija** (cita, papeleo, gimnasio, terapia) vs. **flexible** (trabajo, estudio, compra).
3. Propón qué mover y qué no, con motivo.
4. Avisa si el destino queda sobrecargado.
5. **Nunca muevas nada.** Ofrece el plan para que el usuario lo haga en SimpleMind.

Los destinos son solo días del ámbito activo: en viernes no propongas sábado ni domingo; si el destino es el lunes, dilo como `cruza de semana`. No arrastres tareas fijas. No vacíes un día para llenar otro.

## 8. Formato de salida

Orden fijo, sin tablas ni bloques de código:

1. Cabecera: `📊 <Mapa> · <Ámbito> · Semana <rango>`.
2. Barras por bloque de la semana: `Trabajo ████████░░ 89%`.
3. Línea de días del ámbito: `✅ Lunes 2/2 · 🟡 Jueves 14/15 · ❌ Viernes 0/6`.
4. Bloques de hoy: barra + pendientes desplegados, una línea por actividad principal.
5. Cierre: `Pendientes: N · Parciales: M · Hechos: K`.
6. `Atención`, `Prioridades`, `Consejo`.

Reglas de detalle:

- Lo hecho nunca se despliega tarea por tarea: `✅ 16` y punto.
- Un subnodo de Personal solo se muestra si explica el pendiente: `❌ Empadronamiento (faltan 2 docs)`. Si no, la actividad sola.
- Las barras van solo en los dos bloques, en la semana y en hoy.
- Las barras (semana y hoy) solo llevan bloque, barra y %: sin X/Y y sin emojis de estado. El ❌/🟡 se marca únicamente en la tarea desplegada y en la línea de días.
- Por defecto, hoy + semana. Si se pide un día concreto o un nodo, quita la parte de hoy.

Variantes:

- **semana**: bloques con los pendientes de todo el ámbito, agrupados por bloque y etiquetados con su día.
- **nodo**: progreso, barra del bloque al que pertenece y qué falta.
- **arrastre**: bloques `→ a <día>`, `→ de <día> a hoy` y `sin mover`, con motivos.

Cierra con `Atención`, `Prioridades` y `Consejo` (1–3 líneas).

## 9. Casos límite

- Archivo no descargado, corrupto o sin `document/mindmap.xml`: informa y pide otra ruta. No improvises.
- Mapa sin checkboxes: dilo y muestra solo la estructura.
- Varios mapas o nodos coinciden: candidatos y pregunta.
- Nodo enorme: pliega ramas y muestra totales; despliega solo lo pendiente.
- Nodo sin clasificar: aplica 3.1 (preguntar si está pendiente).
- Bloque o día sin unidades: `—`, sin barra y sin dividir.
- Enlaces `cloudmapref`: un nivel, sin ciclos, solo si aporta.

## 10. Prohibiciones

- No escribir en el `.smmx` ni extraerlo en Dropbox.
- No marcar, mover, renombrar ni borrar nodos.
- No deducir hechos que no estén marcados.
- No presentar una propuesta de arrastre como si ya estuviera aplicada.
- No inventar un tercer bloque ni reclasificar a mano fuera de 3.1.
