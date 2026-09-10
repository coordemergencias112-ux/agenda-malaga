# Agenda de eventos de Málaga — visor público

Calendario **público y de solo lectura** de eventos de la provincia de Málaga:
fiestas populares, música, ferias, romerías, cultura, deporte, actos
institucionales y cortes de carretera / afección al tráfico.

- **Este repositorio** = solo el visor. No contiene ningún enlace ni código del
  Portal 112 (app única). Desde aquí no se puede llegar a la zona de alta de datos.
- **El alta de eventos** se hace desde el módulo `agenda/` del Portal 112, detrás
  de su inicio de sesión.
- **Backend compartido**: Firebase Realtime Database del proyecto `portal-112-b1754`,
  ruta `agenda/eventos`. Este visor solo lee esa ruta; escribir requiere sesión.

## Reglas de Firebase (una sola vez)

En la consola de Firebase → **Realtime Database → Rules**, dentro del objeto
`rules` que ya existe:

```json
"agenda": {
  "eventos": {
    ".read": true,
    ".write": "auth != null"
  }
}
```

`.read: true` → cualquiera lee. `.write: "auth != null"` → solo con sesión.
La `firebaseConfig` de `index.html` no es un secreto: la seguridad la ponen
estas reglas.

## Estructura de un evento (`agenda/eventos/<id>`)

```jsonc
{
  "titulo": "Feria de San Bernabé — Marbella",
  "categoria": "feria",     // musica | festival | feria | romeria | cultural |
                            // tradicion | gastronomico | deportivo |
                            // prueba_deportiva | institucional | mercado |
                            // obras | aviso | otro
  "estado": "previsto",     // previsto | activo | finalizado | cancelado
  "ambito": "municipal",    // municipal | comarcal | provincial
  "inicio": "2026-06-07T12:00",
  "fin": "2026-06-11T04:00",
  "todoElDia": false,
  "municipios": ["Marbella"],
  "lugar": "Recinto ferial y casco antiguo",
  "direccion": "Av. ...",
  "lat": 36.51, "lng": -4.88,           // ubicación principal (opcional)
  "descripcion": "Texto para el público…",
  "programa": [ { "hora": "12:00", "actividad": "Pasacalles" } ],
  "organizador": "Ayuntamiento de Marbella",
  "web": "https://…",
  "telefono": "…",
  "entrada": "gratuito",                // gratuito | entrada | invitacion
  "entradasUrl": "https://…",
  "publico": "Todos los públicos",
  "cartelUrl": "https://….jpg",
  "afeccionTrafico": true,              // si true, se muestran tramos + recorrido
  "dispositivo": "Protección Civil + Cruz Roja…",
  "recorrido": [[36.9,-4.5],[36.8,-4.5]],
  "tramos": [
    { "via": "A-7", "municipio": "Marbella", "corte": "18:00",
      "reapertura": "23:00", "nota": "Desvío por…", "lat": 36.5, "lng": -4.9 }
  ],
  "actualizado": "2026-05-01T10:00:00.000Z",
  "autor": "coord.emergencias.112@gmail.com"
}
```

Todos los campos salvo `titulo` e `inicio` son opcionales; el panel de alta
guarda solo lo que se rellena.

## Local

HTML estático; sírvelo por `http://` (el módulo de Firebase no funciona con `file://`):

```bash
python -m http.server 8080
```
