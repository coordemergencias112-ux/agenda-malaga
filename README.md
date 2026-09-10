# Agenda viaria de Málaga — visor público

Calendario **público y de solo lectura** de cortes de carretera, pruebas
deportivas y eventos con afección al tráfico en la provincia de Málaga.

- **Este repositorio** = solo el visor. No contiene ningún enlace ni código del
  Portal 112 (app única). Desde aquí no se puede llegar a la zona de alta de datos.
- **El alta de eventos** se hace desde el módulo `agenda/` del Portal 112, detrás
  de su inicio de sesión.
- **Backend compartido**: Firebase Realtime Database del proyecto `portal-112-b1754`,
  ruta `agenda/eventos`. Este visor solo lee esa ruta; escribir requiere sesión.

## Puesta en marcha (una sola vez)

### 1. Crear el repositorio en GitHub

```bash
cd agenda-malaga
git init -b main
git add .
git commit -m "Visor público de la agenda viaria de Málaga"
gh repo create coordemergencias112-ux/agenda-malaga --public --source=. --push
```

(Si no usas `gh`: crea el repo vacío `agenda-malaga` en la organización
`coordemergencias112-ux` desde la web y luego
`git remote add origin https://github.com/coordemergencias112-ux/agenda-malaga.git && git push -u origin main`.)

### 2. Activar GitHub Pages

En el repo → **Settings → Pages → Build and deployment → Source: Deploy from a
branch → `main` / `(root)`**. La URL pública queda en:

```
https://coordemergencias112-ux.github.io/agenda-malaga/
```

Ese es el enlace que se comparte.

### 3. Abrir la lectura pública en Firebase

En la consola de Firebase → **Realtime Database → Rules**, añade el bloque
`agenda` **dentro** del objeto `rules` que ya tienes (sin tocar `turnos`,
`guardia`, `panelControl`, etc.):

```json
{
  "rules": {

    "agenda": {
      "eventos": {
        ".read": true,
        ".write": "auth != null"
      }
    }

    // ... resto de reglas existentes ...
  }
}
```

- `.read: true` → cualquiera puede leer los eventos (es el objetivo).
- `.write: "auth != null"` → solo se pueden crear/editar/borrar eventos con
  sesión iniciada, es decir, desde el módulo del Portal.

La `firebaseConfig` que aparece en `index.html` **no es un secreto**: está
pensada para ir en código público. La seguridad real la ponen estas reglas.

## Estructura de un evento (`agenda/eventos/<id>`)

```jsonc
{
  "titulo": "La Vuelta 2026 — Etapa 8, paso por Málaga",
  "categoria": "prueba_deportiva",      // ver lista en index.html
  "estado": "previsto",                 // previsto | activo | finalizado | cancelado
  "inicio": "2026-09-12T13:00",
  "fin": "2026-09-12T17:30",
  "descripcion": "Texto para el público…",
  "enlace": "https://…",                // opcional
  "municipios": ["Antequera", "Málaga"],// se rellena solo con los tramos
  "recorrido": [[36.9,-4.5],[36.8,-4.5]],// polilínea opcional del trazado
  "tramos": [
    {
      "via": "A-357",
      "municipio": "Málaga",
      "corte": "13:30",
      "reapertura": "15:00",
      "nota": "Desvío por MA-20",
      "lat": 36.71, "lng": -4.47
    }
  ],
  "actualizado": "2026-09-10T10:00:00.000Z",
  "autor": "coord.emergencias.112@gmail.com"
}
```

## Local

Es HTML estático; ábrelo con cualquier servidor local (necesita `http://`, no
`file://`, por el módulo de Firebase):

```bash
python -m http.server 8080
# http://localhost:8080/
```
