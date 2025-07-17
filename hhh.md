¡Entendido! Pido disculpas por la redundancia y la estructura confusa en la versión anterior. Tienes toda la razón, para un examen, la claridad y la precisión son fundamentales.

Voy a rehacer el archivo `technical-breakdown.md` desde cero, eliminando todas las repeticiones, mejorando la narrativa, la estructura visual y añadiendo ejemplos de código y rutas explícitas. El objetivo es que sea una "biblia" impecable: **qué hace cada función, dónde está, cómo lo hace y por qué.**

Presta especial atención a la sección sobre la **redundancia de módulos**, ya que es un punto crítico que identificaste y que es importante entender para el examen.

---

`<CodeProject id="spa-gestion-cursos">````markdown file="technical-breakdown.md"
...
```

---

## 2.  Estado Global de la Aplicación (`estadoAplicacion`)

**Ubicación:** `main.js`

**Propósito:** Es el objeto central que mantiene el estado actual de la aplicación. Todas las partes de la aplicación acceden y/o modifican este objeto para reflejar el estado del usuario y la navegación.

**Estructura:**

```javascript
const estadoAplicacion = {
usuarioActual: null, // Objeto con la información del usuario logueado (id, name, email, role, etc.)
rutaActual: "/", // String que representa la ruta actual de la SPA (ej: "/", "/users", "/login")
estaAutenticado: false, // Booleano que indica si hay un usuario autenticado
};

```plaintext

**¿Qué hace y cómo lo hace?**

*   **`usuarioActual`**: Se actualiza al iniciar sesión (`autenticacionService.iniciarSesion`) y se limpia al cerrar sesión (`autenticacionService.cerrarSesion`). Es crucial para determinar los permisos y personalizar la interfaz (ej. mensaje de bienvenida).
*   **`rutaActual`**: Se actualiza cada vez que el usuario navega a una nueva ruta (`navegacionService.navegarA`). Es utilizada por `navegacionService.renderizarContenidoSegunRuta` para decidir qué "página" renderizar.
*   **`estaAutenticado`**: Se establece a `true` al iniciar sesión y a `false` al cerrar sesión. Es fundamental para proteger rutas y mostrar/ocultar elementos de la interfaz (header, sidebar, etc.).

---

## 3. 🌐 Servicios Centrales

Estos servicios son la base de la interacción de la aplicación con el "backend" (json-server) y el almacenamiento del navegador.

### 3.1. `apiService`

**Ubicación:** `services/api-service.js`

**Propósito:** Centralizar todas las peticiones HTTP (CRUD) a la API. Encapsula la lógica de `fetch`, el manejo de errores, la visualización del indicador de carga y la muestra de alertas al usuario.

**¿Qué hace y cómo lo hace?**

Este servicio es el motor de comunicación con `json-server`. Todas las operaciones de datos pasan por aquí.

#### `apiService.realizarPeticionHttp(endpoint, opciones = {})`

*   **Qué hace:** Es la función genérica que ejecuta cualquier petición `fetch` a la API.
*   **Cómo lo hace:**
    1.  **Muestra el spinner de carga:** Llama a `this.mostrarIndicadorCarga(true)`.
    2.  **Realiza la petición `fetch`:** Construye la URL completa (`${this.baseUrl}${endpoint}`) y pasa las `opciones` (método, headers, body).
    3.  **Manejo de errores (`!respuesta.ok`):** Si la respuesta HTTP no es exitosa (ej. 404, 500), lanza un `Error` con el estado HTTP.
    4.  **Parseo de respuesta:** Convierte la respuesta a JSON (`await respuesta.json()`).
    5.  **`try...catch...finally`:**
        *   `try`: Contiene la lógica de la petición.
        *   `catch(error)`: Captura cualquier error (de red, HTTP, JSON), lo loguea en consola y muestra una alerta de tipo "error" al usuario (`this.mostrarAlerta`). Luego, re-lanza el error para que la función que llamó a `realizarPeticionHttp` pueda manejarlo si es necesario.
        *   `finally`: **Siempre** se ejecuta, asegurando que el spinner de carga se oculte (`this.mostrarIndicadorCarga(false)`), independientemente de si la petición fue exitosa o falló.

#### `apiService.obtenerDatos(endpoint)`

*   **Qué hace:** Realiza una petición `GET` para leer datos de un endpoint específico.
*   **Cómo lo hace:** Llama a `this.realizarPeticionHttp(endpoint)`.
*   **Ejemplo de uso:** `apiService.obtenerDatos("/users")` para obtener todos los usuarios.

#### `apiService.crearDatos(endpoint, datosNuevos)`

*   **Qué hace:** Realiza una petición `POST` para crear un nuevo recurso.
*   **Cómo lo hace:** Llama a `this.realizarPeticionHttp(endpoint, { method: "POST", headers: { "Content-Type": "application/json" }, body: JSON.stringify(datosNuevos) })`.
*   **Ejemplo de uso:** `apiService.crearDatos("/users", { name: "Nuevo", email: "new@example.com" })`.

#### `apiService.actualizarDatos(endpoint, datosActualizados)`

*   **Qué hace:** Realiza una petición `PUT` para actualizar un recurso existente.
*   **Cómo lo hace:** Llama a `this.realizarPeticionHttp(endpoint, { method: "PUT", headers: { "Content-Type": "application/json" }, body: JSON.stringify(datosActualizados) })`.
*   **Ejemplo de uso:** `apiService.actualizarDatos("/users/1", { name: "Admin Actualizado" })`.

#### `apiService.eliminarDatos(endpoint)`

*   **Qué hace:** Realiza una petición `DELETE` para eliminar un recurso.
*   **Cómo lo hace:** Llama a `this.realizarPeticionHttp(endpoint, { method: "DELETE" })`.
*   **Ejemplo de uso:** `apiService.eliminarDatos("/users/1")`.

#### `apiService.mostrarIndicadorCarga(mostrar)`

*   **Qué hace:** Muestra u oculta el spinner de carga global.
*   **Cómo lo hace:** Añade o remueve la clase `hidden` del elemento con `id="loadingSpinner"`.

#### `apiService.mostrarAlerta(mensaje, tipo = "success")`

*   **Qué hace:** Muestra un mensaje de alerta temporal al usuario (éxito, error, advertencia).
*   **Cómo lo hace:**
    1.  Crea un nuevo elemento `div` con clases CSS dinámicas (`alert alert-${tipo}`).
    2.  Lo inserta al principio del `mainContent`.
    3.  Lo elimina automáticamente después de 3 segundos usando `setTimeout`.

**Diagrama de Flujo de `apiService.realizarPeticionHttp`:**

```mermaid title="Flujo de apiService.realizarPeticionHttp" type="diagram"
graph TD
    A[Llamada a realizarPeticionHttp] --> B{mostrarIndicadorCarga(true)};
    B --> C[Realizar fetch a API_BASE_URL + endpoint];
    C -- Respuesta HTTP --> D{Respuesta.ok?};
    D -- No --> E[Lanzar Error HTTP];
    D -- Sí --> F[Parsear respuesta a JSON];
    F --> G[Retornar datos];
    E --> H[Capturar Error (catch)];
    H --> I[Mostrar Alerta de Error];
    I --> J[Re-lanzar Error];
    G --> K[Finalmente (finally)];
    J --> K;
    K --> L{mostrarIndicadorCarga(false)};
    L --> M[Fin de la Petición];
```

---

### 3.2. `almacenamientoService`

**Ubicación:** `services/storage-service.js`

**Propósito:** Proporcionar una interfaz unificada y segura para interactuar con `localStorage` y `sessionStorage`, manejando la serialización/deserialización de JSON y los posibles errores.

**¿Qué hace y cómo lo hace?**

Este servicio es crucial para la persistencia de datos en el navegador.

#### `almacenamientoService.guardarEnLocal(clave, valor)`

- **Qué hace:** Guarda un valor en `localStorage`.
- **Cómo lo hace:** Convierte el `valor` a una cadena JSON (`JSON.stringify`) y lo almacena usando `localStorage.setItem(clave, ...)` dentro de un `try...catch` para manejar errores de almacenamiento.
- **Importante para el examen:** Los datos en `localStorage` **persisten** incluso después de cerrar el navegador.


#### `almacenamientoService.obtenerDeLocal(clave)`

- **Qué hace:** Recupera un valor de `localStorage`.
- **Cómo lo hace:** Obtiene la cadena del `localStorage` y la convierte de nuevo a un objeto JavaScript (`JSON.parse`) dentro de un `try...catch`.


#### `almacenamientoService.eliminarDeLocal(clave)`

- **Qué hace:** Elimina un elemento específico de `localStorage`.
- **Cómo lo hace:** Usa `localStorage.removeItem(clave)` dentro de un `try...catch`.


#### `almacenamientoService.guardarEnSesion(clave, valor)`

- **Qué hace:** Guarda un valor en `sessionStorage`.
- **Cómo lo hace:** Similar a `guardarEnLocal`, pero usa `sessionStorage.setItem`.
- **Importante para el examen:** Los datos en `sessionStorage` **se borran** cuando se cierra la pestaña o ventana del navegador.


#### `almacenamientoService.obtenerDeSesion(clave)`

- **Qué hace:** Recupera un valor de `sessionStorage`.
- **Cómo lo hace:** Similar a `obtenerDeLocal`, pero usa `sessionStorage.getItem`.


#### `almacenamientoService.eliminarDeSesion(clave)`

- **Qué hace:** Elimina un elemento específico de `sessionStorage`.
- **Cómo lo hace:** Usa `sessionStorage.removeItem(clave)`.


#### `almacenamientoService.limpiarTodoElAlmacenamiento()`

- **Qué hace:** Elimina todos los datos de `localStorage` y `sessionStorage`.
- **Cómo lo hace:** Llama a `localStorage.clear()` y `sessionStorage.clear()`.


---

### 3.3. `autenticacionService`

**Ubicación:** `services/auth-service.js`

**Propósito:** Gestionar todo el ciclo de vida de la autenticación del usuario: inicio de sesión, registro, cierre de sesión y verificación de la sesión activa.

**¿Qué hace y cómo lo hace?**

Este servicio interactúa con `apiService` para obtener datos de usuarios y con `almacenamientoService` para persistir el estado de la sesión.

#### `autenticacionService.iniciarSesion(correoElectronico, contrasena)`

- **Qué hace:** Autentica a un usuario con sus credenciales.
- **Cómo lo hace:**

1. Obtiene **todos** los usuarios de la API (`apiService.obtenerDatos("/users")`).
2. Busca un usuario que coincida con el `correoElectronico` y `contrasena` proporcionados.
3. Si encuentra un usuario:

1. Guarda el objeto `usuarioEncontrado` en `localStorage` (`almacenamientoService.guardarEnLocal("usuarioActual", usuarioEncontrado)`).
2. Guarda un indicador booleano `true` en `sessionStorage` (`almacenamientoService.guardarEnSesion("estaAutenticado", true)`).
3. Actualiza el estado global de la aplicación (`estadoAplicacion.usuarioActual`, `estadoAplicacion.estaAutenticado`).
4. Muestra una alerta de éxito (`apiService.mostrarAlerta`).



4. Si no encuentra el usuario, lanza un `Error` ("Credenciales inválidas"), que es capturado y mostrado como alerta.





#### `autenticacionService.registrarUsuario(datosNuevoUsuario)`

- **Qué hace:** Registra un nuevo usuario en el sistema.
- **Cómo lo hace:**

1. Obtiene todos los usuarios existentes para verificar si el email ya está registrado.
2. Si el email ya existe, lanza un `Error`.
3. Prepara el objeto `nuevoUsuario` asignándole el `role: "visitor"` por defecto y la `dateOfAdmission` actual.
4. Envía el `nuevoUsuario` a la API (`apiService.crearDatos("/users", nuevoUsuario)`).
5. Muestra una alerta de éxito.





#### `autenticacionService.cerrarSesion()`

- **Qué hace:** Cierra la sesión del usuario actual.
- **Cómo lo hace:**

1. Elimina `usuarioActual` de `localStorage` (`almacenamientoService.eliminarDeLocal`).
2. Elimina `estaAutenticado` de `sessionStorage` (`almacenamientoService.eliminarDeSesion`).
3. Limpia el estado global de la aplicación.
4. Redirige al usuario a la página de login (`navegacionService.navegarA("/login")`).
5. Muestra una alerta de confirmación.





#### `autenticacionService.verificarAutenticacionExistente()`

- **Qué hace:** Verifica si hay una sesión activa al cargar la aplicación.
- **Cómo lo hace:**

1. Intenta recuperar `usuarioActual` de `localStorage` y `estaAutenticado` de `sessionStorage`.
2. Si ambos existen, restaura el estado global de la aplicación (`estadoAplicacion.usuarioActual`, `estadoAplicacion.estaAutenticado`) y retorna `true`.
3. Si no, retorna `false`.





**Diagrama de Flujo de Gestión de Sesiones:**

```mermaid
Failed to render diagram
```

---

## 4. ️ Servicios Específicos por Entidad (CRUD)

Estos servicios proporcionan una interfaz de alto nivel para las operaciones CRUD de cada entidad (usuarios, cursos, inscripciones), utilizando internamente el `apiService`.

### 4.1. `usuarioService`

**Ubicación:** `services/user-service.js`

**Propósito:** Realizar operaciones CRUD específicas para la entidad `users`.

**¿Qué hace y cómo lo hace?**

Cada función de este servicio llama a la función correspondiente en `apiService`, pasándole el endpoint `/users` o `/users/{id}`.

- **`usuarioService.obtenerTodosLosUsuarios()`**: Llama a `apiService.obtenerDatos("/users")`.
- **`usuarioService.crearNuevoUsuario(datosUsuario)`**: Llama a `apiService.crearDatos("/users", datosUsuario)` (añade `dateOfAdmission` antes de enviar).
- **`usuarioService.actualizarUsuario(idUsuario, datosUsuario)`**: Llama a `apiService.actualizarDatos(`/users/${idUsuario}`, datosUsuario)`.
- **`usuarioService.eliminarUsuario(idUsuario)`**: Llama a `apiService.eliminarDatos(`/users/${idUsuario}`)`.
- **`usuarioService.obtenerUsuarioPorId(idUsuario)`**: Obtiene todos los usuarios y los filtra por ID.


---

### 4.2. `cursoService`

**Ubicación:** `services/course-service.js`

**Propósito:** Realizar operaciones CRUD específicas para la entidad `courses`.

**¿Qué hace y cómo lo hace?**

Similar a `usuarioService`, pero para cursos.

- **`cursoService.obtenerTodosLosCursos()`**: Llama a `apiService.obtenerDatos("/courses")`.
- **`cursoService.crearNuevoCurso(datosCurso)`**: Llama a `apiService.crearDatos("/courses", datosCurso)`.
- **`cursoService.actualizarCurso(idCurso, datosCurso)`**: Llama a `apiService.actualizarDatos(`/courses/${idCurso}`, datosCurso)`.
- **`cursoService.eliminarCurso(idCurso)`**: Llama a `apiService.eliminarDatos(`/courses/${idCurso}`)`.
- **`cursoService.obtenerCursoPorId(idCurso)`**: Obtiene todos los cursos y los filtra por ID.


---

### 4.3. `inscripcionService`

**Ubicación:** `services/enrollment-service.js`

**Propósito:** Realizar operaciones CRUD específicas para la entidad `enrollments`.

**¿Qué hace y cómo lo hace?**

- **`inscripcionService.obtenerTodasLasInscripciones()`**: Llama a `apiService.obtenerDatos("/enrollments")`.
- **`inscripcionService.inscribirUsuarioEnCurso(idUsuario, idCurso)`**: Prepara los datos de inscripción (userId, courseId, enrollmentDate, status) y llama a `apiService.crearDatos("/enrollments", datosInscripcion)`.
- **`inscripcionService.cancelarInscripcion(idInscripcion)`**: Llama a `apiService.eliminarDatos(`/enrollments/${idInscripcion}`)`.
- **`inscripcionService.obtenerInscripcionesDeUsuario(idUsuario)`**: Obtiene todas las inscripciones y las filtra por `userId`.
- **`inscripcionService.verificarInscripcionUsuario(idUsuario, idCurso)`**: Utiliza `obtenerInscripcionesDeUsuario` y `some()` para verificar si un usuario ya está inscrito en un curso específico.


---

## 5. ️ Utilidades y Gestores de Interfaz

Estos objetos manejan la lógica de la interfaz de usuario, la navegación y la validación.

### 5.1. `validacionUtils`

**Ubicación:** `utils/validation-utils.js`

**Propósito:** Proporcionar funciones reutilizables para validar datos de formularios y otros inputs.

**¿Qué hace y cómo lo hace?**

- **`validacionUtils.validarFormatoCorreoElectronico(correoElectronico)`**:

- **Qué hace:** Verifica si una cadena tiene el formato de correo electrónico válido.
- **Cómo lo hace:** Utiliza una expresión regular (`/^[^\s@]+@[^\s@]+\.[^\s@]+$/`).



- **`validacionUtils.validarCampoNoVacio(valor)`**:

- **Qué hace:** Comprueba si un valor no es nulo, indefinido o una cadena vacía (después de recortar espacios).



- **`validacionUtils.validarCamposRequeridos(datosFormulario, listaCamposRequeridos)`**:

- **Qué hace:** Itera sobre una lista de nombres de campos y verifica si cada uno está presente y no vacío en el objeto `datosFormulario`.
- **Cómo lo hace:** Usa `validarCampoNoVacio` para cada campo y acumula los errores en un array.



- **`validacionUtils.validarFormularioCompleto(datosFormulario, camposRequeridos)`**:

- **Qué hace:** Realiza una validación integral de un formulario, combinando validaciones de campos requeridos, formato de email, longitud de contraseña, etc.
- **Cómo lo hace:** Llama a `validarCamposRequeridos`, `validarFormatoCorreoElectronico`, y añade validaciones específicas para `password`, `phone` y `enrollNumber`. Retorna un array con todos los mensajes de error.



- **`validacionUtils.limpiarDatosEntrada(datosOriginales)`**:

- **Qué hace:** Elimina espacios en blanco al inicio y al final de todas las propiedades de tipo `string` en un objeto.
- **Cómo lo hace:** Itera sobre las propiedades del objeto y aplica `trim()` a las cadenas.





---

### 5.2. `navegacionService`

**Ubicación:** `utils/navigation-service.js`

**Propósito:** Implementar la navegación de la Single Page Application (SPA) sin recargar la página, gestionando el historial del navegador y la renderización condicional del contenido.

**¿Qué hace y cómo lo hace?**

Este servicio es el corazón de la experiencia SPA.

#### `navegacionService.navegarA(rutaDestino)`

- **Qué hace:** Cambia la URL en el navegador y actualiza el contenido de la aplicación sin una recarga completa de la página.
- **Cómo lo hace:**

1. Usa `history.pushState(null, null, rutaDestino)` para modificar la URL en la barra de direcciones del navegador sin provocar una recarga.
2. Actualiza `estadoAplicacion.rutaActual` con la nueva ruta.
3. Llama a `this.renderizarContenidoSegunRuta()` para que la interfaz se actualice.





#### `navegacionService.renderizarContenidoSegunRuta()`

- **Qué hace:** Decide qué contenido HTML mostrar en el `mainContent` basándose en la `rutaActual` y el estado de autenticación/rol del usuario.
- **Cómo lo hace:**

1. **Verificación de autenticación:** Si la ruta no es pública (`/login`, `/register`) y el usuario no está autenticado, redirige a `/login`.
2. **Actualización de UI:** Llama a `this.actualizarElementosNavegacion()` para mostrar/ocultar el header y sidebar, y actualizar el mensaje de bienvenida.
3. **Renderizado condicional (`switch`):** Utiliza una estructura `switch` para llamar a la función de renderizado de página correspondiente en `renderizadorPaginas` (ej. `renderizadorPaginas.renderizarPaginaLogin()`, `renderizadorPaginas.renderizarDashboard()`).
4. **Control de acceso por rol:** Para rutas como `/users` (solo admin) o `/enrollments` (solo visitante), verifica el rol del `usuarioActual` antes de renderizar. Si el rol no es el adecuado, redirige a `/`.





#### `navegacionService.esRutaPublica(ruta)`

- **Qué hace:** Determina si una ruta no requiere autenticación.
- **Cómo lo hace:** Comprueba si la `ruta` está en un array predefinido de rutas públicas (`["/login", "/register"]`).


#### `navegacionService.esUsuarioAdministrador()`/ `navegacionService.esUsuarioVisitante()`

- **Qué hace:** Verifican el rol del `usuarioActual` en `estadoAplicacion`.
- **Cómo lo hace:** Acceden a `estadoAplicacion.usuarioActual.role`.


#### `navegacionService.actualizarElementosNavegacion()`

- **Qué hace:** Muestra u oculta el header y sidebar, y actualiza el mensaje de bienvenida según el estado de autenticación y el rol.
- **Cómo lo hace:** Añade/remueve la clase `hidden` a los elementos del DOM (`header`, `sidebar`, `mainContent`) y actualiza el `textContent` de `userWelcome`. También llama a `actualizarEnlaceActivoEnNavegacion`.


#### `navegacionService.actualizarEnlaceActivoEnNavegacion()`

- **Qué hace:** Resalta el enlace de navegación activo en el sidebar.
- **Cómo lo hace:** Itera sobre todos los enlaces con la clase `nav-menu a`, remueve la clase `active` de todos y la añade al enlace cuyo `href` coincide con `estadoAplicacion.rutaActual`.


**Diagrama de Flujo de Navegación SPA:**

```mermaid
Failed to render diagram
```

---

### 5.3. `renderizadorPaginas`

**Ubicación:** `main.js`

**Propósito:** Generar y renderizar el contenido HTML dinámico de cada "página" de la SPA dentro del elemento `mainContent`.

**¿Qué hace y cómo lo hace?**

Cada función de este objeto se encarga de construir una cadena HTML y asignarla al `innerHTML` del `mainContent`. También se encarga de llamar a las funciones para cargar datos específicos de la página.

#### `renderizadorPaginas.renderizarPaginaLogin()`

- **Qué hace:** Muestra el formulario de inicio de sesión.
- **Cómo lo hace:** Inyecta el HTML del formulario en `mainContent`. **Importante:** Después de inyectar el HTML, añade el `event listener` al formulario de login (`formularioInicioSesion`) para que `manejadorEventos.procesarInicioSesion` lo gestione.


#### `renderizadorPaginas.renderizarPaginaRegistro()`

- **Qué hace:** Muestra el formulario de registro de nuevos usuarios.
- **Cómo lo hace:** Inyecta el HTML del formulario en `mainContent`. Similar al login, añade el `event listener` al formulario de registro (`formularioRegistroUsuario`) para `manejadorEventos.procesarRegistroUsuario`.


#### `renderizadorPaginas.renderizarDashboard()`

- **Qué hace:** Muestra el panel de control principal, con estadísticas y acciones rápidas según el rol del usuario.
- **Cómo lo hace:**

1. Inyecta la estructura base del dashboard en `mainContent`.
2. Llama a `this.cargarDatosEstadisticasDashboard()` para poblar las estadísticas y los cursos del usuario (si es visitante) de forma asíncrona.





#### `renderizadorPaginas.cargarDatosEstadisticasDashboard()`

- **Qué hace:** Obtiene datos de usuarios, cursos e inscripciones para mostrar estadísticas en el dashboard.
- **Cómo lo hace:**

1. Usa `Promise.all` para obtener datos de `usuarioService`, `cursoService` e `inscripcionService` en paralelo (más eficiente).
2. Genera HTML diferente para el contenedor de estadísticas (`contenedorEstadisticasDashboard`) según si el usuario es `admin` o `visitor`.
3. Si es `visitor`, llama a `this.cargarCursosEspecificosDelUsuario` para mostrar sus inscripciones.





#### `renderizadorPaginas.cargarCursosEspecificosDelUsuario(inscripcionesUsuario, todosLosCursos)`

- **Qué hace:** Muestra los cursos en los que el usuario visitante está inscrito.
- **Cómo lo hace:** Filtra `todosLosCursos` basándose en `inscripcionesUsuario` y genera cards HTML para cada curso.


#### `renderizadorPaginas.renderizarPaginaUsuarios()`

- **Qué hace:** Muestra la tabla de gestión de usuarios (solo para administradores).
- **Cómo lo hace:** Inyecta el HTML de la tabla en `mainContent` y llama a `this.cargarYMostrarListaUsuarios()` para poblar la tabla.


#### `renderizadorPaginas.cargarYMostrarListaUsuarios()`

- **Qué hace:** Obtiene todos los usuarios y los renderiza en la tabla de gestión de usuarios.
- **Cómo lo hace:** Llama a `usuarioService.obtenerTodosLosUsuarios()` y genera filas HTML (`<tr>`) para cada usuario, incluyendo botones de "Editar" y "Eliminar" que llaman a funciones en `gestorCrud`.


#### `renderizadorPaginas.renderizarPaginaCursos()`

- **Qué hace:** Muestra la lista de cursos, con una vista diferente para administradores (tabla de gestión) y visitantes (cards de inscripción).
- **Cómo lo hace:** Inyecta la estructura base y llama a `this.cargarYMostrarCursos()`.


#### `renderizadorPaginas.cargarYMostrarCursos()`

- **Qué hace:** Obtiene todos los cursos e inscripciones y los renderiza según el rol del usuario.
- **Cómo lo hace:**

1. Obtiene cursos e inscripciones usando `Promise.all`.
2. Si es `admin`, genera una tabla con detalles del curso, número de inscritos y botones de "Editar"/"Eliminar".
3. Si es `visitor`, genera cards para cada curso, indicando si ya está inscrito, si el curso está lleno, o un botón para "Inscribirse".





#### `renderizadorPaginas.renderizarPaginaInscripciones()`

- **Qué hace:** Muestra la tabla de inscripciones activas del usuario visitante.
- **Cómo lo hace:** Inyecta la estructura base y llama a `this.cargarYMostrarInscripcionesUsuario()`.


#### `renderizadorPaginas.cargarYMostrarInscripcionesUsuario()`

- **Qué hace:** Obtiene las inscripciones del usuario actual y la información de los cursos asociados, y las muestra en una tabla.
- **Cómo lo hace:**

1. Obtiene inscripciones del usuario (`inscripcionService.obtenerInscripcionesDeUsuario`) y todos los cursos.
2. Si no hay inscripciones, muestra un mensaje.
3. Si hay inscripciones, genera filas de tabla con detalles del curso y un botón para "Cancelar" la inscripción.





#### `renderizadorPaginas.renderizarPagina404()`

- **Qué hace:** Muestra una página de error cuando la ruta no existe.
- **Cómo lo hace:** Inyecta un mensaje de error y botones para volver al inicio o a la página anterior.


---

### 5.4. `manejadorEventos`

**Ubicación:** `main.js`

**Propósito:** Centralizar las funciones que procesan los eventos de envío de formularios (login, registro).

**¿Qué hace y cómo lo hace?**

Estas funciones son llamadas por los `event listeners` adjuntos a los formularios después de que el HTML es renderizado.

#### `manejadorEventos.procesarInicioSesion(evento)`

- **Qué hace:** Maneja el envío del formulario de login.
- **Cómo lo hace:**

1. `evento.preventDefault()`: Evita la recarga de la página.
2. Obtiene `email` y `password` del `FormData`.
3. Realiza una validación básica de campos vacíos.
4. Llama a `autenticacionService.iniciarSesion(correoElectronico, contrasena)`.
5. Si es exitoso, navega al dashboard (`navegacionService.navegarA("/")`).
6. Captura y loguea cualquier error.





#### `manejadorEventos.procesarRegistroUsuario(evento)`

- **Qué hace:** Maneja el envío del formulario de registro.
- **Cómo lo hace:**

1. `evento.preventDefault()`: Evita la recarga de la página.
2. Obtiene todos los datos del nuevo usuario del `FormData`.
3. Llama a `validacionUtils.validarFormularioCompleto()` para una validación exhaustiva. Si hay errores, los muestra.
4. Si la validación es exitosa, llama a `autenticacionService.registrarUsuario(datosNuevoUsuario)`.
5. Si es exitoso, navega a la página de login (`navegacionService.navegarA("/login")`).
6. Captura y loguea cualquier error.





---

### 5.5. `gestorCrud`

**Ubicación:** `main.js`

**Propósito:** Orquestar las operaciones CRUD (Crear, Leer, Actualizar, Eliminar) que se inician desde la interfaz de usuario, interactuando con los servicios de entidad y actualizando la vista.

**¿Qué hace y cómo lo hace?**

Estas funciones son llamadas directamente desde los botones de la interfaz (ej. `onclick="gestorCrud.iniciarEdicionUsuario(id)"`).

#### `gestorCrud.iniciarEdicionUsuario(idUsuario)`

- **Qué hace:** Abre el modal para editar un usuario existente.
- **Cómo lo hace:** Llama a `gestorModales.mostrarModalCrearUsuario(idUsuario)`.


#### `gestorCrud.confirmarEliminacionUsuario(idUsuario)`

- **Qué hace:** Pide confirmación antes de eliminar un usuario.
- **Cómo lo hace:**

1. Muestra un `confirm()` nativo del navegador.
2. Si el usuario confirma, llama a `usuarioService.eliminarUsuario(idUsuario)`.
3. Muestra una alerta de éxito y recarga la lista de usuarios (`renderizadorPaginas.cargarYMostrarListaUsuarios()`).





#### `gestorCrud.iniciarEdicionCurso(idCurso)`

- **Qué hace:** Abre el modal para editar un curso existente.
- **Cómo lo hace:** Llama a `gestorModales.mostrarModalCrearCurso(idCurso)`.


#### `gestorCrud.confirmarEliminacionCurso(idCurso)`

- **Qué hace:** Pide confirmación antes de eliminar un curso y sus inscripciones relacionadas.
- **Cómo lo hace:**

1. Muestra un `confirm()`.
2. Si confirma:

1. Obtiene todas las inscripciones relacionadas con el curso.
2. Itera y llama a `inscripcionService.cancelarInscripcion()` para cada una.
3. Finalmente, llama a `cursoService.eliminarCurso(idCurso)`.
4. Muestra una alerta de éxito y recarga la lista de cursos.








#### `gestorCrud.procesarInscripcionEnCurso(idCurso)`

- **Qué hace:** Maneja la lógica para inscribir a un usuario en un curso.
- **Cómo lo hace:**

1. Verifica si el usuario ya está inscrito en el curso (`inscripcionService.verificarInscripcionUsuario`).
2. Verifica la capacidad del curso (`cursoService.obtenerCursoPorId` y `inscripcionService.obtenerTodasLasInscripciones`).
3. Si las validaciones pasan, llama a `inscripcionService.inscribirUsuarioEnCurso(estadoAplicacion.usuarioActual.id, idCurso)`.
4. Muestra una alerta de éxito y recarga la vista de cursos.





#### `gestorCrud.confirmarCancelacionInscripcion(idInscripcion, nombreCurso)`

- **Qué hace:** Pide confirmación antes de cancelar una inscripción.
- **Cómo lo hace:**

1. Muestra un `confirm()`.
2. Si confirma, llama a `inscripcionService.cancelarInscripcion(idInscripcion)`.
3. Muestra una alerta de éxito y recarga la lista de inscripciones del usuario.





**Diagrama de Flujo de Creación/Edición de Usuario (Admin):**

```mermaid
Failed to render diagram
```

---

### 5.6. `gestorModales`

**Ubicación:** `main.js`

**Propósito:** Gestionar la visualización, ocultación y el contenido de los modales (ventanas emergentes) utilizados para formularios de creación y edición.

**¿Qué hace y cómo lo hace?**

Este objeto centraliza la lógica de los modales.

#### `gestorModales.mostrarModalCrearUsuario(idUsuario = null)`

- **Qué hace:** Genera el HTML del formulario para crear o editar un usuario y lo muestra en el modal.
- **Cómo lo hace:**

1. Determina si es una operación de edición (`esEdicion`) basándose en `idUsuario`.
2. Construye la cadena HTML del formulario de usuario, incluyendo campos para nombre, email, contraseña, rol, teléfono y matrícula. La contraseña es opcional en edición.
3. Llama a `this.mostrarModal(contenidoModal)` para inyectar el HTML y hacer visible el modal.
4. Si es edición, llama a `this.cargarDatosUsuarioEnModal(idUsuario)` para precargar los datos del usuario.
5. Añade un `event listener` al formulario del modal para que `this.procesarEnvioFormularioUsuario` lo maneje.





#### `gestorModales.cargarDatosUsuarioEnModal(idUsuario)`

- **Qué hace:** Carga los datos de un usuario existente en el formulario del modal para su edición.
- **Cómo lo hace:**

1. Llama a `usuarioService.obtenerUsuarioPorId(idUsuario)`.
2. Si encuentra el usuario, rellena los campos del formulario del modal con los datos obtenidos.





#### `gestorModales.procesarEnvioFormularioUsuario(evento, idUsuario = null)`

- **Qué hace:** Maneja el envío del formulario de usuario dentro del modal (tanto para creación como para edición).
- **Cómo lo hace:**

1. `evento.preventDefault()`: Evita la recarga.
2. Obtiene los datos del formulario.
3. Realiza validaciones usando `validacionUtils.validarFormularioCompleto()`. Si hay errores, los muestra.
4. Si es edición (`idUsuario` no es `null`), llama a `usuarioService.actualizarUsuario()`.
5. Si es creación, llama a `usuarioService.crearNuevoUsuario()`.
6. Muestra una alerta de éxito, oculta el modal (`this.ocultarModal()`) y recarga la lista de usuarios (`renderizadorPaginas.cargarYMostrarListaUsuarios()`).





#### `gestorModales.mostrarModalCrearCurso(idCurso = null)`

- **Qué hace:** Genera el HTML del formulario para crear o editar un curso y lo muestra en el modal.
- **Cómo lo hace:** Similar a `mostrarModalCrearUsuario`, pero con campos específicos para cursos (título, descripción, instructor, fecha de inicio, duración, capacidad).


#### `gestorModales.cargarDatosCursoEnModal(idCurso)`

- **Qué hace:** Carga los datos de un curso existente en el formulario del modal para su edición.
- **Cómo lo hace:** Llama a `cursoService.obtenerCursoPorId(idCurso)` y rellena los campos del formulario.


#### `gestorModales.procesarEnvioFormularioCurso(evento, idCurso = null)`

- **Qué hace:** Maneja el envío del formulario de curso dentro del modal.
- **Cómo lo hace:** Similar a `procesarEnvioFormularioUsuario`, pero llama a `cursoService.actualizarCurso()` o `cursoService.crearNuevoCurso()`, y recarga la lista de cursos. Incluye validación de capacidad (entre 1 y 100).


#### `gestorModales.mostrarModal(contenidoHtml)`

- **Qué hace:** Hace visible el modal y le inyecta el contenido HTML proporcionado.
- **Cómo lo hace:** Remueve la clase `hidden` del elemento `modal` y asigna `contenidoHtml` al `innerHTML` de `modalBody`. También enfoca el primer campo del formulario para mejorar la UX.


#### `gestorModales.ocultarModal()`

- **Qué hace:** Oculta el modal y limpia su contenido.
- **Cómo lo hace:** Añade la clase `hidden` al elemento `modal` y vacía el `innerHTML` de `modalBody`.


---

## 6.  Inicialización de la Aplicación (`DOMContentLoaded`)

**Ubicación:** `main.js` (al final del archivo)

**Propósito:** Es el punto de entrada de la aplicación. Se ejecuta una vez que el DOM (Document Object Model) está completamente cargado, asegurando que todos los elementos HTML estén disponibles antes de que el JavaScript intente interactuar con ellos.

**¿Qué hace y cómo lo hace?**

```javascript
document.addEventListener("DOMContentLoaded", () => {
// 1. Verificar autenticación al cargar la aplicación
autenticacionService.verificarAutenticacionExistente();

// 2. Configurar navegación SPA interceptando clicks en enlaces
document.addEventListener("click", (evento) => {
if (evento.target.matches("[data-link]")) {
evento.preventDefault(); // Evita la recarga de la página
const rutaDestino = evento.target.getAttribute("href");
navegacionService.navegarA(rutaDestino);
}
});

// 3. Manejar navegación del navegador (botones atrás/adelante)
window.addEventListener("popstate", () => {
estadoAplicacion.rutaActual = location.pathname; // Actualiza la ruta actual
navegacionService.renderizarContenidoSegunRuta(); // Vuelve a renderizar
});

// 4. Configurar botón de cerrar sesión
document.getElementById("logoutBtn").addEventListener("click", () => {
autenticacionService.cerrarSesion();
});

// 5. Configurar cierre de modal (botón 'x' y clic fuera)
document.querySelector(".close").addEventListener("click", () => {
gestorModales.ocultarModal();
});
document.getElementById("modal").addEventListener("click", function (evento) {
if (evento.target === this) {
gestorModales.ocultarModal();
}
});

// 6. Configurar atajos de teclado (Escape para cerrar modal, Ctrl+H/U/C para navegar)
document.addEventListener("keydown", (evento) => {
if (evento.key === "Escape" && !document.getElementById("modal").classList.contains("hidden")) {
gestorModales.ocultarModal();
}
if (evento.ctrlKey) {
switch (evento.key) {
case "h": evento.preventDefault(); navegacionService.navegarA("/"); break;
case "u": if (navegacionService.esUsuarioAdministrador()) { evento.preventDefault(); navegacionService.navegarA("/users"); } break;
case "c": evento.preventDefault(); navegacionService.navegarA("/courses"); break;
}
}
});

// 7. Establecer la ruta inicial y renderizar el contenido
estadoAplicacion.rutaActual = location.pathname || "/";
navegacionService.renderizarContenidoSegunRuta();

// Mensaje de inicialización completa en consola
console.log("✅ Aplicación SPA inicializada correctamente...");
});

```plaintext

---

## 7. ⚠️ Redundancia de Código y Estructura de Módulos (¡Punto Crítico para el Examen!)

**Observación Importante:**

En el archivo `main.js` proporcionado, existe una **redundancia significativa** en la definición de los servicios (`apiService`, `almacenamientoService`, `autenticacionService`, etc.) y utilidades (`validacionUtils`, `navegacionService`).

*   **¿Dónde está la redundancia?**
    *   Cada uno de estos servicios/utilidades tiene su propia definición en un archivo `.js` separado dentro de las carpetas `services/` o `utils/`.
    *   Sin embargo, en `main.js`, estas mismas definiciones (o versiones ligeramente modificadas) **se repiten**.

*   **¿Por qué es un problema?**
    1.  **Duplicación de lógica:** Se mantiene el mismo código en múltiples lugares, lo que dificulta la actualización y puede introducir errores si un cambio se hace en un lugar pero no en otro.
    2.  **Confusión:** No está claro cuál es la "verdadera" definición del servicio.
    3.  **Mantenimiento:** Aumenta la complejidad del código y el tiempo necesario para depurar o añadir nuevas funcionalidades.
    4.  **Sobrecarga de memoria:** Aunque en este contexto de SPA pequeña no es crítico, en aplicaciones más grandes podría llevar a una mayor huella de memoria al definir los mismos objetos varias veces.

*   **¿Cómo debería ser idealmente (en un proyecto moderno)?**
    *   En un proyecto moderno de JavaScript (usando Node.js, Webpack, Vite, etc.), se utilizarían **módulos ES6 (`import`/`export`)**.
    *   Cada archivo de servicio (`api-service.js`, `auth-service.js`, etc.) exportaría su objeto (`export const apiService = { ... }`).
    *   El archivo `main.js` simplemente **importaría** estos servicios (`import { apiService } from './services/api-service.js';`).
    *   Esto asegura que cada servicio se defina una sola vez y se reutilice donde sea necesario.

*   **¿Por qué está así en este proyecto (contexto de examen)?**
    *   Es probable que esta estructura se haya mantenido para un contexto de examen específico donde se requiere que **todos los servicios estén "disponibles globalmente"** en el `main.js` sin usar un sistema de módulos moderno (que requeriría un bundler como Webpack).
    *   Al incluir los archivos de servicio con `<script src="...">` en `index.html`, estos servicios se adjuntan al objeto `window` (o al ámbito global), haciéndolos accesibles en `main.js`. La repetición en `main.js` podría ser un artefacto de un proceso de desarrollo o una instrucción específica para el examen.

**Para el examen, es crucial que puedas:**

1.  **Identificar esta redundancia.**
2.  **Explicar por qué es una mala práctica** en el desarrollo de software.
3.  **Describir cómo se resolvería** en un entorno de desarrollo moderno (usando `import`/`export`).
4.  **Entender que, a pesar de la redundancia, la aplicación funciona** porque las definiciones en `main.js` sobrescriben o coexisten con las cargadas desde los archivos de servicio, y el flujo de llamadas interno sigue siendo el mismo.

---

## 8. 🔑 Conceptos Clave para el Examen

Aquí se resumen los conceptos más importantes que debes dominar para el examen, con énfasis en cómo se aplican en este proyecto:

*   **Single Page Application (SPA):**
    *   **Definición:** La aplicación carga una única página HTML (`index.html`). La navegación entre "páginas" se simula dinámicamente con JavaScript, manipulando el DOM y la URL del navegador.
    *   **Implementación:** `navegacionService` utiliza `history.pushState()` para cambiar la URL sin recargar. Los enlaces con `data-link` son interceptados para activar esta navegación SPA. `renderizadorPaginas` se encarga de inyectar el HTML dinámico.
    *   **Ventajas:** Experiencia de usuario más fluida, menos recargas, mejor rendimiento percibido.

*   **Roles Diferenciados (Admin vs. Visitante):**
    *   **Definición:** Los usuarios tienen roles (`admin` o `visitor`) que determinan sus permisos y las funcionalidades a las que pueden acceder.
    *   **Implementación:** El `role` se almacena en `estadoAplicacion.usuarioActual`. `navegacionService.esUsuarioAdministrador()` y `navegacionService.esUsuarioVisitante()` son las funciones clave para verificar el rol. La interfaz (`renderizadorPaginas`) y las acciones CRUD (`gestorCrud`) se adaptan dinámicamente según el rol (ej. el admin ve "Gestión de Usuarios", el visitante ve "Mis Inscripciones").

*   **`localStorage` vs. `sessionStorage`:**
    *   **Definición:** Mecanismos de almacenamiento de datos en el navegador.
    *   **`localStorage`**: Almacenamiento **persistente**. Los datos permanecen incluso después de cerrar el navegador.
        *   **Uso en el proyecto:** Se usa para `usuarioActual` (`almacenamientoService.guardarEnLocal("usuarioActual", usuario)`), permitiendo que el usuario permanezca logueado entre sesiones del navegador.
    *   **`sessionStorage`**: Almacenamiento **temporal**. Los datos se borran cuando se cierra la pestaña o ventana del navegador.
        *   **Uso en el proyecto:** Se usa para `estaAutenticado` (`almacenamientoService.guardarEnSesion("estaAutenticado", true)`), indicando que la sesión está activa en la pestaña actual.
    *   **Servicio:** `almacenamientoService` encapsula la interacción con ambos.

*   **Manejo de Errores y Carga (Loading Spinner / Alerts):**
    *   **Definición:** Estrategias para informar al usuario sobre el estado de las operaciones y los problemas.
    *   **Implementación:**
        *   Todas las peticiones a la API (`apiService.realizarPeticionHttp`) están envueltas en bloques `try...catch...finally`.
        *   `try`: Contiene la lógica principal de la petición.
        *   `catch(error)`: Captura errores de red o de la API, los loguea en consola y muestra una alerta de tipo "error" al usuario (`apiService.mostrarAlerta("error")`).
        *   `finally`: **Siempre** se ejecuta, asegurando que el spinner de carga (`apiService.mostrarIndicadorCarga(false)`) se oculte, sin importar si la petición fue exitosa o falló.
        *   `apiService.mostrarIndicadorCarga(true/false)` controla la visibilidad del spinner.
        *   `apiService.mostrarAlerta()` crea y muestra mensajes temporales de éxito, error o advertencia.

*   **Delegación de Eventos:**
    *   **Definición:** En lugar de añadir un `event listener` a cada elemento individual, se añade un único `event listener` a un ancestro común (ej. `document`).
    *   **Implementación:** El `event listener` principal en `document` (`document.addEventListener("click", ...)` en `main.js`) intercepta todos los clics. Luego, verifica si el elemento clicado o uno de sus ancestros tiene el atributo `data-link` para manejar la navegación SPA. Esto es más eficiente para elementos generados dinámicamente.

*   **Modularización (a pesar de la redundancia):**
    *   **Definición:** Organización del código en unidades lógicas (módulos) con responsabilidades claras y bien definidas.
    *   **Implementación:** El código está lógicamente dividido en servicios (`api-service`, `auth-service`, `user-service`, etc.) y utilidades (`validation-utils`, `navigation-service`). Cada servicio tiene una responsabilidad única.
    *   **Beneficios:** Facilita el mantenimiento, la comprensión y la reutilización del código.

*   **CRUD (Create, Read, Update, Delete):**
    *   **Definición:** Las cuatro operaciones básicas de persistencia de datos.
    *   **Implementación:**
        *   **Capa HTTP:** `apiService` (`obtenerDatos`, `crearDatos`, `actualizarDatos`, `eliminarDatos`) maneja las peticiones `GET`, `POST`, `PUT`, `DELETE` respectivamente.
        *   **Capa de Entidad:** `usuarioService`, `cursoService`, `inscripcionService` proporcionan métodos específicos para cada entidad (ej. `usuarioService.crearNuevoUsuario()`).
        *   **Capa de UI:** `gestorCrud` orquesta las acciones desde la interfaz de usuario (ej. `gestorCrud.confirmarEliminacionUsuario()`).

---

¡Espero que esta versión revisada y detallada sea la "biblia" perfecta para tu examen! He puesto especial énfasis en la claridad, la estructura y la explicación de cada parte, incluyendo la importante nota sobre la redundancia. ¡Mucha suerte!
```

`</CodeProject>`