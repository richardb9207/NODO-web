# NODO en vivo

NODO es un agente animado en [Rive](https://rive.app). Tiene 14 emociones, 15 variantes de color, un halo de estado, mirada que sigue al cursor y cinco estilos visuales: clásico, pixel art, sticker, doodle y low poly.

- **Demo:** https://richardb9207.github.io/nodo-demo/ (o abre `index.html`)
- **Archivo de animación:** `NODO.riv`

## Cómo usar NODO en tu app

### 1. Qué hay dentro de `NODO.riv`

Cada estilo es un artboard. Todos comparten la misma máquina de estados y los mismos datos, así que el código para controlarlos es el mismo.

| Estilo    | Artboard       |
|-----------|----------------|
| Clásico   | `NODO_Agent`   |
| Pixel art | `NODO_Pixel`   |
| Sticker   | `NODO_Sticker` |
| Doodle    | `NODO_Doodle`  |
| Low poly  | `NODO_Poly`    |

- **Máquina de estados:** `NODO`
- **Datos (view model):** se enlazan solos con `autoBind: true`

| Propiedad        | Tipo   | Valores |
|------------------|--------|---------|
| `face → emotion` | enum   | `normal`, `happy`, `content`, `surprised`, `thinking`, `reading`, `doubt`, `worried`, `excited`, `listening`, `speaking`, `sleeping`, `error`, `celebrating` |
| `status`         | enum   | `idle`, `active`, `success`, `warning`, `risk` |
| `variant`        | enum   | `white`, `blue`, `green`, `purple`, `orange`, `pink`, `gray`, `yellow`, `teal`, `red`, `indigo`, `lime`, `brown`, `coral`, `sky` |
| `lookX`, `lookY` | number | de `-1` a `1` (`-1` izquierda / arriba, `0` centro, `1` derecha / abajo) |

`emotion` vive dentro del view model anidado `face`. Las demás propiedades están en el view model principal.

La mirada ya sigue al cursor por sí sola. Usa `lookX` y `lookY` solo cuando quieras dirigirla desde la app, por ejemplo hacia un panel.

### 2. Cargar NODO en la web

Usa el runtime `webgl2`. El runtime `canvas` no dibuja los difuminados del estilo clásico.

```html
<canvas id="nodo" width="800" height="800" style="width: 400px; height: 400px"></canvas>

<script src="https://cdn.jsdelivr.net/npm/@rive-app/webgl2@2.43.1/rive.js"></script>
<script>
  const r = new rive.Rive({
    src: "NODO.riv",
    canvas: document.getElementById("nodo"),
    artboard: "NODO_Agent",
    stateMachine: "NODO",
    autoplay: true,
    autoBind: true,
    layout: new rive.Layout({ fit: rive.Fit.Contain }),
    onLoad: () => {
      r.resizeDrawingSurfaceToCanvas();
      const vm = r.viewModelInstance;
      vm.viewModel("face").enum("emotion").value = "happy";
      vm.enum("status").value = "active";
      vm.enum("variant").value = "orange";
    },
  });
</script>
```

Sirve la página desde un servidor (por ejemplo GitHub Pages o `python3 -m http.server`). Si la abres con doble clic, el navegador bloquea la carga de `NODO.riv`.

### 3. Cambiar emoción, halo, variante y mirada

Cuando NODO ya cargó (dentro de `onLoad` o después), cambia los valores así:

```js
const vm = r.viewModelInstance;

vm.viewModel("face").enum("emotion").value = "thinking"; // emoción
vm.enum("status").value = "warning";                    // color del halo
vm.enum("variant").value = "teal";                       // color del cuerpo
vm.number("lookX").value = 1;                            // mirar a la derecha
vm.number("lookY").value = 0;
```

Las transiciones entre emociones son automáticas; no hace falta animar nada desde el código.

### 4. Cambiar de estilo

El estilo es el artboard, y un artboard no se puede cambiar en caliente. Para cambiar de estilo, crea una instancia nueva y vuelve a aplicar el estado actual:

```js
let r = null;
const current = { emotion: "normal", status: "idle", variant: "white" };

function loadStyle(artboard) {
  if (r) r.cleanup();
  r = new rive.Rive({
    src: "NODO.riv",
    canvas: document.getElementById("nodo"),
    artboard,
    stateMachine: "NODO",
    autoplay: true,
    autoBind: true,
    layout: new rive.Layout({ fit: rive.Fit.Contain }),
    onLoad: () => {
      r.resizeDrawingSurfaceToCanvas();
      const vm = r.viewModelInstance;
      vm.viewModel("face").enum("emotion").value = current.emotion;
      vm.enum("status").value = current.status;
      vm.enum("variant").value = current.variant;
    },
  });
}

loadStyle("NODO_Sticker");
```

Guarda en `current` cada cambio que hagas, para que el nuevo estilo arranque igual que el anterior.

### 5. Conectar eventos de la app

Una forma simple de traducir lo que pasa en el producto a una reacción de NODO:

```js
const EVENTS = {
  analysis_started:    { emotion: "thinking",    status: "active"  },
  document_loaded:     { emotion: "reading",     status: "active"  },
  missing_information: { emotion: "doubt",       status: "warning" },
  risk_detected:       { emotion: "worried",     status: "risk"    },
  design_validated:    { emotion: "celebrating", status: "success" },
  user_idle:           { emotion: "normal",      status: "idle"    },
};

function onAppEvent(name) {
  const e = EVENTS[name];
  if (!e) return;
  const vm = r.viewModelInstance;
  vm.viewModel("face").enum("emotion").value = e.emotion;
  vm.enum("status").value = e.status;
}
```

### 6. Otras plataformas

Los runtimes de Rive para iOS, Android, Flutter y React usan los mismos nombres: artboard, máquina de estados `NODO` y las propiedades `emotion` (dentro de `face`), `status`, `variant`, `lookX` y `lookY`. Consulta la [documentación de runtimes de Rive](https://rive.app/docs/runtimes/getting-started) para la sintaxis de cada uno.

## Editar NODO en Rive

- Cada estilo tiene dos artboards: el personaje (`NODO_Estilo`) y su cara (`NODO_Face_Estilo`).
- Al exportar, Rive solo incluye los artboards marcados como **Component** y el que esté activo. Todos los de NODO ya están marcados. Si creas uno nuevo, márcalo también o no aparecerá en `NODO.riv`.
