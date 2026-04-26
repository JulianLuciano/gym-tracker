# Gym Tracker

App web para seguimiento de rutina de gimnasio. Diseñada para uso personal con foco en rehabilitación de hernia L5-S1, registro de cargas progresivas y temporizador de descanso con lógica de contexto.

---

## Stack

- **Frontend:** HTML + CSS + JS vanilla, sin frameworks
- **Backend:** Node.js + Express, únicamente para servir el estático
- **Storage:** localStorage (browser) — sin base de datos por ahora
- **Deploy:** Railway, CD automático desde `main`

---

## Estructura

```
gym-tracker/
├── public/
│   ├── index.html        # Estructura HTML + lógica JS
│   └── style.css         # Estilos, variables de color, tipografía
├── server.js             # Express sirve /public como estático
├── package.json
├── .gitignore
└── README.md
```

El CSS vive separado del HTML. Las variables de color y tipografía están definidas en `:root` dentro de `style.css`, lo que facilita ajustes visuales sin tocar la lógica. Cuando se integre al Personal Hub, la JS se modulariza.

---

## Rutina

4 días de entrenamiento full body con énfasis rotativo:

| Día | Énfasis | Notas |
|-----|---------|-------|
| Día 1 | Tren inferior + pecho | Sentadilla, press banca, remo, fondos |
| Día 2 | Posterior + hombro | RDL, press militar, dominadas, hip thrust |
| Día 3 | Unilateral + brazo | Búlgara, farmer's carry, curl, extensiones |
| Día 4 | Tren superior puro | Opcional — sin carga espinal |

Cada día arranca con un **bloque de calentamiento fijo de 15 minutos** prescrito por fisioterapeuta para la hernia L5-S1: movilidad de cadera, Cat-Cow, Dead Bug, Hip Abduction con banda, Squat y RDL con carga liviana.

---

## Scheduling

La app sugiere qué día entrenar basándose en el historial de sesiones guardado en localStorage.

Lógica de progresión: D1 → D2 → D3 → (D4 opcional el sábado) → D1. Los días 1, 2 y 3 requieren al menos un día de descanso entre sí. El Día 4 no puede pegarse al Día 1 por solapamiento de pecho. Los viernes están bloqueados por preferencia del usuario.

Cuando arranque la temporada de rugby (julio), el sábado se convierte en día de deporte y la rutina vuelve a 3 días semanales — esto está pendiente como ajuste.

---

## Ejercicios y hernia L5-S1

Cada ejercicio tiene asignado un nivel de carga espinal:

- **⭐⭐ crítico** — ejercicios terapéuticos clave o de alta carga axial: RDL, sentadilla, hip thrust, farmer's carry. Tienen nota técnica específica visible en la card.
- **⭐ moderado** — ejercicios con carga espinal presente pero controlable: remo, búlgara, dominadas, plancha.
- Sin badge — sin carga espinal relevante.

Los ejercicios marcados como críticos son también los más terapéuticos cuando se ejecutan bien — el badge no significa "evitar" sino "prestar atención".

---

## Temporizador

Un solo timer global en la parte superior. El usuario siempre le da play manualmente — nunca arranca solo.

**Pastillas de tiempo:** 45s / 60s / 90s / 120s + campo numérico libre. Todo en una sola línea, sin wrap.

**Lógica automática de contexto** (se activa mientras no haya intervención manual):

1. Al elegir un día → el timer se setea al tiempo de descanso del primer ejercicio pendiente
2. Entre series → ese mismo tiempo por defecto
3. Al marcar un ejercicio como completado → se setea a 90s (descanso entre ejercicios). Cuando ese timer termina → se setea automáticamente al tiempo del siguiente ejercicio pendiente
4. El label del timer siempre muestra el nombre del próximo ejercicio sin completar

**Modo manual:** si el usuario toca cualquier pastilla o el campo libre, la lógica automática se desactiva y el timer queda en ese valor indefinidamente — hasta que completa otro ejercicio, momento en que la lógica se retoma (90s → tiempo del siguiente).

**Al terminar:** beep triple (Web Audio API) + vibración si el dispositivo lo soporta.

**Animación del círculo:** al setear un tiempo el arco se llena instantáneamente. Al darle play empieza a drenar linealmente hasta cero.

---

## Cards de ejercicio

Cada ejercicio muestra:
- Nombre + badge de hernia + tiempo de descanso recomendado + botón de completar
- Series y repeticiones
- Nota técnica (si aplica)
- Campo de carga en kg con la última carga registrada como referencia

Al marcar un ejercicio como completado, la card se **colapsa** mostrando solo la primera línea. Se puede re-expandir tocando cualquier parte del header. El estado de colapso persiste en localStorage durante la sesión.

---

## Registro de cargas

Campo numérico por ejercicio (step 2.5kg). Al ingresar un valor se guarda en localStorage con fecha. Se mantienen las últimas 20 entradas por ejercicio. La última carga registrada aparece como referencia en la card.

La progresión histórica completa (gráfico por ejercicio) está pendiente en el roadmap.

---

## Persistencia

Todo el estado se guarda en `localStorage` bajo la key `gym_v4`:

```json
{
  "sessions": [{ "day": 1, "date": "Sun Apr 27 2025" }],
  "loads": {
    "d1e1": [{ "kg": 80, "date": "27/04" }]
  },
  "session": {
    "day": 1,
    "completed": { "d1e1": true },
    "collapsed": { "d1e1": true }
  }
}
```

El schema está pensado para migrar limpiamente a Supabase cuando se integre al Personal Hub — `sessions` y `loads` se convierten en tablas directamente.

---

## Roadmap

- [ ] Migrar storage a Supabase — sync cross-device y persistencia real
- [ ] Integrar al Personal Hub (Node/Express + Railway existente)
- [ ] Historial de progresión de carga por ejercicio con gráfico
- [ ] Ajuste de scheduling para temporada de rugby (julio) — sábado pasa a deporte, vuelve a 3 días semanales
