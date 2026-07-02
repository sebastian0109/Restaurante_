# 🍽️ RestaurOS — Sistema de Gestión de Restaurante

Sistema web de gestión integral para restaurantes, con **7 roles distintos**, control de pedidos, mesas, stock, turnos, reservas y un centro de notificaciones en tiempo real. Construido en **JavaScript vanilla (ES Modules)**, sin frameworks ni build step.

> Proyecto académico — actualmente funciona 100% en el navegador con una API mock en memoria (`api/mockData.js`), pensada para migrar más adelante a un backend real (Node/Express + MySQL) sin cambiar la capa de vistas.

---

## ✨ Características principales

- **7 roles con vistas y permisos propios**: Administrador, Dueño, Jefe de Meseros, Jefe de Cocina, Mesero, Cocinero, Portero/Recepcionista.
- **Gestión de pedidos** con máquina de estados completa: `pendiente → en_preparación → listo → retirado → entregado → cuenta_solicitada → pagado / cancelado`.
- **Plano interactivo de mesas** (SVG) con su propia máquina de estados: `disponible → reservada/ocupada → con_pedido → pendiente_pago → liberable → disponible`.
- **Control de stock e ingredientes**, con solicitudes de reposición y aprobación por rol.
- **Turnos y horarios** por empleado, con cálculo de tiempo restante / horas extra en vivo.
- **Solicitudes de personal** (contratación) con flujo de aprobación.
- **Centro de notificaciones** persistente (localStorage), con:
  - Notificaciones disparadas por acciones del usuario (pedido creado, mesa liberada, solicitud aprobada, etc.)
  - Notificaciones automáticas por tiempo (`js/notifScheduler.js`): pedidos retrasados, mesas ocupadas demasiado tiempo, reservas próximas, turnos por finalizar, horas extra, stock crítico, resumen diario/semanal.
- **Dashboard con gráficos** (Chart.js) según el rol.
- **Diseño responsive**, sidebar colapsable en mobile, tema propio sobre Bootstrap 5.

---

## 🧱 Stack técnico

| Capa | Tecnología |
|---|---|
| Markup / Estilos | HTML5, CSS3, [Bootstrap 5.3](https://getbootstrap.com/), [Bootstrap Icons](https://icons.getbootstrap.com/) |
| Lógica | JavaScript ES Modules (sin frameworks) |
| Gráficos | [Chart.js 4](https://www.chartjs.org/) |
| Tipografía | Sora (títulos) + DM Sans (texto), vía Google Fonts |
| Persistencia | `sessionStorage` (sesión/auth), `localStorage` (notificaciones) |
| Datos | API mock en memoria (`api/mockData.js`), sin backend real todavía |

No requiere `npm install` ni build step — es HTML/JS/CSS servido directamente.

---

## 📁 Estructura del proyecto

```
RestaurOS/
├── index.html                  # Login
├── dashboard.html              # Shell principal (sidebar + topbar + contenido dinámico)
├── css/
│   └── styles.css              # Tema Deep Navy + Warm Amber
├── js/
│   ├── pages.js                # Renderizado de todas las vistas + acciones (Pages, Mesas)
│   ├── businessLogic.js        # Máquinas de estado (pedidos, mesas) y validadores de transición
│   ├── notificationService.js  # Centro de notificaciones (CRUD, categorías, persistencia)
│   └── notifScheduler.js       # Alertas automáticas por tiempo (polling cada 60s)
├── components/
│   ├── sidebar.js               # Menú lateral dinámico según rol
│   └── ui.js                    # Toast, Modal, Loader, tablas, badges, formatters
├── services/
│   ├── auth.js                  # Login, sesión, permisos por rol
│   └── api.js                   # Router de la API mock (pedidos, mesas, stock, turnos, etc.)
└── api/
    └── mockData.js               # Base de datos mock en memoria (MockDB)
```

---

## 🚀 Cómo correrlo localmente

Como el proyecto usa **ES Modules** (`import`/`export`), no se puede abrir `index.html` directamente con `file://` — el navegador bloquea los imports por CORS. Necesitas servirlo con un servidor local simple:

```bash
# Opción 1 — con Python
python3 -m http.server 8080

# Opción 2 — con Node (npx, sin instalar nada global)
npx serve .

# Opción 3 — extensión "Live Server" de VS Code
```

Luego abre `http://localhost:8080` (o el puerto que corresponda) en el navegador.

---

## 👤 Roles y credenciales de demo

La pantalla de login incluye botones de acceso rápido para cada rol:

| Rol | Email | Contraseña |
|---|---|---|
| Administrador | `admin@resto.com` | `admin123` |
| Dueño | `dueno@resto.com` | `dueno123` |
| Jefe de Meseros | `jefe.meseros@resto.com` | `jm123` |
| Jefe de Cocina | `jefe.cocina@resto.com` | `jc123` |
| Mesero | `mesero1@resto.com` | `m123` |
| Cocinero | `cocinero1@resto.com` | `c123` |
| Portero / Recepcionista | `portero1@resto.com` | `p123` |

Cada rol ve un sidebar y un dashboard distintos, y solo puede ejecutar las acciones que le corresponden según `businessLogic.js`.

---

## 🔄 Máquinas de estado

### Pedidos
```
pendiente → en_preparacion → listo → retirado → entregado → cuenta_solicitada → pagado
                                                                              ↘ cancelado
```

### Mesas
```
disponible → reservada → ocupada → con_pedido → pendiente_pago → liberable → disponible
          ↘ ocupada (sin reserva) ↗
```

Cada transición valida **desde qué estado es posible**, **hacia qué estado va** y **qué roles la pueden ejecutar** — todo centralizado en `js/businessLogic.js` (`TRANSICIONES_PEDIDO`, `TRANSICIONES_MESA`).

---

## 🔔 Sistema de notificaciones

- Notificaciones **reactivas**: se disparan al ejecutar una acción (crear pedido, liberar mesa, aprobar solicitud, etc.).
- Notificaciones **proactivas**: `NotifScheduler` revisa cada 60s condiciones de negocio basadas en tiempo (pedido retrasado, turno por terminar, stock crítico, etc.) y evita duplicados con un set de "ya notificado" en memoria de sesión.
- Persisten en `localStorage` y se sincronizan entre pestañas del mismo navegador vía el evento `storage`.

---

## ⚠️ Notas y limitaciones actuales

- **No hay backend real**: todos los datos viven en `api/mockData.js` y se pierden al recargar la página (excepto notificaciones, que usan `localStorage`, y sesión, que usa `sessionStorage`).
- El código está preparado para migrar a un backend Node/Express + MySQL sin tocar la capa de vistas — cada función de `services/api.js` corresponde 1:1 a un endpoint futuro (`GET /api/pedidos`, `PUT /api/mesas/:id/accion`, etc.).
- Algunas fechas en los datos de prueba están fijas (por ejemplo, pedidos de ejemplo con fecha `2024-01-15`); si notas cálculos de tiempo transcurrido que no tienen sentido, probablemente sea por esto.

---

## 🗺️ Roadmap sugerido

- [ ] Backend real (Node/Express + MySQL) reemplazando `mockData.js`
- [ ] Autenticación con JWT real en vez de token simulado en `sessionStorage`
- [ ] Tests automatizados para las máquinas de estado de pedidos y mesas
- [ ] WebSockets para notificaciones en tiempo real entre dispositivos (no solo pestañas)

---

## 📄 Licencia

Proyecto académico — uso educativo.
