// ╔══════════════════════════════════════════════════════════╗
// ║          CLIMA DEL REINO — app.js                        ║
// ║   API utilizada: OpenWeatherMap                          ║
// ║   Aprenderemos: fetch, async/await, manejo de errores    ║
// ╚══════════════════════════════════════════════════════════╝

// ══════════════════════════════════════════════════════════
//  🔑 API KEY — ¡MUY IMPORTANTE!
//
//  Una "API Key" es como una contraseña especial que le
//  demostramos a OpenWeather para probar que somos nosotros.
//
//  PASOS PARA OBTENER TU API KEY GRATIS:
//  1. Ve a https://openweathermap.org/
//  2. Haz clic en "Sign Up" y crea una cuenta
//  3. Ve a "My API Keys" en tu perfil
//  4. Copia tu clave y pégala aquí abajo
//  5. ¡Puede tardar hasta 2 horas en activarse!
// ══════════════════════════════════════════════════════════
const API_KEY = 'TU_API_KEY_AQUI'; // 👈 Reemplaza esto con tu clave

// ══════════════════════════════════════════════════════════
//  🌐 URL BASE
//
//  Esta es la dirección del servidor de OpenWeather.
//  Es como la "dirección" de la dungeon a la que vamos.
//  Siempre empieza igual, solo cambia la ciudad al final.
// ══════════════════════════════════════════════════════════
const URL_BASE = 'https://api.openweathermap.org/data/2.5/weather';

// ══════════════════════════════════════════════════════════
//  📌 REFERENCIAS AL DOM
//
//  Aquí "agarramos" los elementos HTML con su id.
//  Es como decirle a JavaScript: "ese botón es ESTE botón".
// ══════════════════════════════════════════════════════════
const inputCiudad    = document.getElementById('inputCiudad');
const btnBuscar      = document.getElementById('btnBuscar');
const estadoCarga    = document.getElementById('estadoCarga');
const panelError     = document.getElementById('panelError');
const textoError     = document.getElementById('textoError');
const tarjetaClima   = document.getElementById('tarjetaClima');

// Elementos dentro de la tarjeta de resultados
const ciudadNombre   = document.getElementById('ciudadNombre');
const ciudadPais     = document.getElementById('ciudadPais');
const tempNumero     = document.getElementById('tempNumero');
const tempDescripcion= document.getElementById('tempDescripcion');
const tempIcono      = document.getElementById('tempIcono');
const humedad        = document.getElementById('humedad');
const viento         = document.getElementById('viento');
const sensacion      = document.getElementById('sensacion');
const visibilidad    = document.getElementById('visibilidad');
const tempMax        = document.getElementById('tempMax');
const tempMin        = document.getElementById('tempMin');
const horaActualizacion = document.getElementById('horaActualizacion');

// ══════════════════════════════════════════════════════════
//  ⌨️ EVENTO: Enter en el input
//
//  Escuchamos si el usuario presiona "Enter" dentro del
//  campo de texto. Si lo hace, es lo mismo que hacer clic
//  en el botón de buscar.
// ══════════════════════════════════════════════════════════
inputCiudad.addEventListener('keydown', function(evento) {
  // "evento.key" nos dice qué tecla se presionó
  if (evento.key === 'Enter') {
    buscarClima(); // llamamos la función de búsqueda
  }
});

// ══════════════════════════════════════════════════════════
//  🔍 FUNCIÓN: buscarClima()
//
//  Esta función se llama cuando el usuario hace clic en
//  "EXPLORAR". Toma la ciudad del input y llama a la API.
// ══════════════════════════════════════════════════════════
function buscarClima() {
  // 1. Leer lo que escribió el usuario y quitar espacios extra
  const ciudad = inputCiudad.value.trim();

  // 2. Validar que no esté vacío
  //    Si el usuario no escribió nada, le avisamos y paramos
  if (!ciudad) {
    mostrarError('¡Debes escribir el nombre de una ciudad, cazador!');
    return; // "return" detiene la función aquí mismo
  }

  // 3. Llamar a la función que consulta la API
  consultarAPI(ciudad);
}

// ══════════════════════════════════════════════════════════
//  🌐 FUNCIÓN PRINCIPAL: consultarAPI()
//
//  Aquí está la MAGIA de las APIs.
//  "async" significa que esta función puede esperar
//  respuestas de servidores sin bloquear todo lo demás.
// ══════════════════════════════════════════════════════════
async function consultarAPI(ciudad) {

  // ── PASO 1: Mostrar estado de carga ───────────────────
  //  Ocultamos resultados anteriores y mostramos "cargando"
  mostrarEstado('cargando');
  btnBuscar.disabled = true; // desactivar el botón mientras carga

  // ── PASO 2: Construir la URL completa ─────────────────
  //
  //  La URL final se ve así:
  //  https://api.openweathermap.org/data/2.5/weather
  //    ?q=Santiago        ← ciudad que buscamos
  //    &appid=TU_CLAVE    ← nuestra llave de acceso
  //    &units=metric      ← temperatura en °C (no °F)
  //    &lang=es           ← descripción en español
  //
  //  Usamos "encodeURIComponent" para que los espacios y
  //  caracteres especiales no rompan la URL.
  const url = `${URL_BASE}?q=${encodeURIComponent(ciudad)}&appid=${API_KEY}&units=metric&lang=es`;

  // ── PASO 3: Hacer la petición con fetch ───────────────
  //
  //  "try/catch" es como un escudo:
  //  - "try" intenta ejecutar el código
  //  - "catch" atrapa cualquier error que ocurra
  try {

    // fetch() hace la petición al servidor.
    // "await" espera a que el servidor responda antes de continuar.
    const respuesta = await fetch(url);

    // ── PASO 4: Verificar si la respuesta fue exitosa ──
    //
    //  respuesta.ok = true  → todo bien (código 200)
    //  respuesta.ok = false → algo salió mal (código 404, 401, etc.)
    if (!respuesta.ok) {
      // Manejo de errores específicos por código HTTP
      if (respuesta.status === 404) {
        throw new Error('Ciudad no encontrada. ¡Esa dungeon no existe en el mapa!');
      } else if (respuesta.status === 401) {
        throw new Error('API Key inválida. Revisa tu clave en app.js');
      } else {
        throw new Error(`Error del servidor: código ${respuesta.status}`);
      }
    }

    // ── PASO 5: Convertir la respuesta a JSON ──────────
    //
    //  La API nos devuelve texto en formato JSON.
    //  .json() lo convierte a un objeto de JavaScript
    //  para que podamos usarlo fácilmente.
    const datos = await respuesta.json();

    // ── PASO 6: Mostrar los datos en pantalla ──────────
    mostrarResultados(datos);

  } catch (error) {
    // Si algo falló en cualquier paso, llegamos aquí
    mostrarError(error.message);

  } finally {
    // "finally" SIEMPRE se ejecuta, haya error o no
    // Reactivamos el botón sin importar el resultado
    btnBuscar.disabled = false;
  }
}

// ══════════════════════════════════════════════════════════
//  📊 FUNCIÓN: mostrarResultados()
//
//  Recibe el objeto JSON de la API y actualiza la pantalla.
//
//  ASÍ SE VE LA RESPUESTA JSON DE OPENWEATHER:
//  {
//    name: "Santiago",
//    sys: { country: "CL" },
//    main: {
//      temp: 22.5,
//      feels_like: 21.0,
//      humidity: 60,
//      temp_max: 25,
//      temp_min: 18
//    },
//    weather: [{ description: "cielo despejado", icon: "01d" }],
//    wind: { speed: 3.5 },
//    visibility: 10000
//  }
// ══════════════════════════════════════════════════════════
function mostrarResultados(datos) {

  // ── Nombre de la ciudad y país ─────────────────────
  ciudadNombre.textContent = datos.name.toUpperCase();
  ciudadPais.textContent   = `◈ ${datos.sys.country} ◈`;

  // ── Temperatura principal ──────────────────────────
  //  Math.round() redondea el número (ej: 22.7 → 23)
  tempNumero.textContent   = `${Math.round(datos.main.temp)}°C`;

  // ── Descripción del clima ──────────────────────────
  //  La descripción viene en minúsculas, la ponemos bonita
  const desc = datos.weather[0].description;
  tempDescripcion.textContent = desc.charAt(0).toUpperCase() + desc.slice(1);

  // ── Emoji según el clima ───────────────────────────
  //  El código "icon" de la API nos dice qué tipo de clima es
  tempIcono.textContent = obtenerEmoji(datos.weather[0].icon);

  // ── Estadísticas secundarias ───────────────────────
  humedad.textContent   = `${datos.main.humidity}%`;
  viento.textContent    = `${Math.round(datos.wind.speed * 3.6)} km/h`;
  sensacion.textContent = `${Math.round(datos.main.feels_like)}°C`;

  // La visibilidad viene en metros, la convertimos a km
  const visKm = (datos.visibility / 1000).toFixed(1);
  visibilidad.textContent = `${visKm} km`;

  tempMax.textContent = `${Math.round(datos.main.temp_max)}°C`;
  tempMin.textContent = `${Math.round(datos.main.temp_min)}°C`;

  // ── Hora de actualización ──────────────────────────
  const ahora = new Date();
  horaActualizacion.textContent = `ÚLTIMA ACTUALIZACIÓN: ${ahora.toLocaleTimeString('es-CL')}`;

  // ── Mostrar la tarjeta ─────────────────────────────
  mostrarEstado('resultado');
}

// ══════════════════════════════════════════════════════════
//  😊 FUNCIÓN: obtenerEmoji()
//
//  Convierte el código de icono de OpenWeather a un emoji.
//  Los códigos son como: "01d" (sol de día), "10n" (lluvia noche)
// ══════════════════════════════════════════════════════════
function obtenerEmoji(codigoIcono) {
  // Tomamos solo los primeros 2 caracteres del código
  const codigo = codigoIcono.slice(0, 2);

  // Tabla de equivalencias código → emoji
  const emojis = {
    '01': '☀️',   // cielo despejado
    '02': '⛅',   // pocas nubes
    '03': '☁️',   // nubes dispersas
    '04': '☁️',   // nubes rotas
    '09': '🌧️',  // lluvia ligera
    '10': '🌦️',  // lluvia
    '11': '⛈️',   // tormenta
    '13': '❄️',   // nieve
    '50': '🌫️',  // niebla
  };

  // Si no encontramos el código, devolvemos un emoji genérico
  return emojis[codigo] || '🌡️';
}

// ══════════════════════════════════════════════════════════
//  🚀 FUNCIÓN: buscarRapido()
//
//  Cuando el usuario hace clic en un "portal rápido",
//  escribimos la ciudad en el input y buscamos directamente.
// ══════════════════════════════════════════════════════════
function buscarRapido(ciudad) {
  inputCiudad.value = ciudad; // poner la ciudad en el input
  buscarClima();              // buscar inmediatamente
}

// ══════════════════════════════════════════════════════════
//  🎭 FUNCIÓN: mostrarEstado()
//
//  Controla qué panel se muestra en pantalla:
//  - 'cargando' → spinner de carga
//  - 'error'    → mensaje de error
//  - 'resultado'→ tarjeta con los datos del clima
// ══════════════════════════════════════════════════════════
function mostrarEstado(estado) {
  // Ocultamos todos los paneles primero
  estadoCarga.classList.add('hidden');
  panelError.classList.add('hidden');
  tarjetaClima.classList.add('hidden');

  // Mostramos solo el que necesitamos
  if (estado === 'cargando') {
    estadoCarga.classList.remove('hidden');
  } else if (estado === 'resultado') {
    tarjetaClima.classList.remove('hidden');
  }
  // 'error' lo maneja la función mostrarError()
}

// ══════════════════════════════════════════════════════════
//  ⚠️ FUNCIÓN: mostrarError()
//
//  Muestra un mensaje de error al usuario.
// ══════════════════════════════════════════════════════════
function mostrarError(mensaje) {
  // Ocultamos todos los paneles
  mostrarEstado('');

  // Ponemos el mensaje y mostramos el panel de error
  textoError.textContent = mensaje;
  panelError.classList.remove('hidden');
}

// ══════════════════════════════════════════════════════════
//  ✨ FUNCIÓN: crearParticulas()
//
//  Crea partículas flotantes de fondo para el efecto visual.
//  Cada partícula es un div pequeño con posición y velocidad
//  aleatoria.
// ══════════════════════════════════════════════════════════
function crearParticulas() {
  const contenedor = document.getElementById('particulas');
  const cantidad = 25; // número de partículas

  for (let i = 0; i < cantidad; i++) {
    const particula = document.createElement('div');
    particula.className = 'particula';

    // Posición horizontal aleatoria (entre 0% y 100% del ancho)
    particula.style.left = Math.random() * 100 + 'vw';

    // Duración aleatoria entre 8 y 20 segundos
    const duracion = 8 + Math.random() * 12;
    particula.style.animationDuration = duracion + 's';

    // Retraso aleatorio para que no empiecen todas juntas
    particula.style.animationDelay = (Math.random() * 15) + 's';

    // Tamaño aleatorio entre 1px y 3px
    const tamaño = 1 + Math.random() * 2;
    particula.style.width  = tamaño + 'px';
    particula.style.height = tamaño + 'px';

    contenedor.appendChild(particula);
  }
}

// ── Iniciar partículas cuando la página carga ────────────
window.addEventListener('DOMContentLoaded', crearParticulas);