DISTRIBUIDORA EL MAR APP – README (S4–S7)
=========================================

Resumen
-------
Aplicación Android (Kotlin) que simula el flujo de compra y despacho de productos alimenticios con:
- Autenticación por correo/contraseña (Firebase Authentication).
- Cálculo de distancia (Haversine) y costo de envío según reglas de negocio.
- Fallback de ubicación para dispositivos sin Google Play Services (LocationManager).
- Control de temperatura (cadena de frío) con conversión a radianes (evidencia por Logcat).
- Prototipo no funcional (Pencil) y Canvas (Canva) incluidos en /docs.

Requisitos
----------
- Android Studio (Koala/Iguana o superior) + Gradle integrados.
- JDK 17.
- minSdk = 23 (ajuste requerido por Firebase Authentication).
- Dispositivo/emulador Android (recomendado API 26–34).
- Conexión a Internet (para Firebase).

Clonar / Descargar
------------------
Opción 1 (ZIP):
- Descarga el archivo ZIP del proyecto (S4–S7), descomprime y abre la carpeta raíz en Android Studio.

Opción 2 (Git):
- git clone <URL_DEL_REPO>
- Abrir en Android Studio.

Configuración de Firebase (obligatoria)
---------------------------------------
1) Crear proyecto en https://console.firebase.google.com
2) Agregar una app Android:
   - packageName: usa el de tu proyecto (por ejemplo, "com.example.distribuidoraelmar" o "com.example.distribuidoraelmarapp").
   - (Opcional) SHA-1 si usaras Google Sign-In. Para correo/contraseña NO es obligatorio.
3) Descargar el archivo **google-services.json** y colocarlo en **app/** (raíz del módulo).
4) Habilitar **Authentication > Sign-in method > Email/Password**.
5) Crear **Realtime Database** (modo de prueba durante el desarrollo):
   - Copiar la URL de la base, por ejemplo:
     https://distribuidoraelmarapp-default-rtdb.firebaseio.com/
   - Si tu `google-services.json` no trae la URL, el código usa:
     FirebaseDatabase.getInstance("<URL_DE_TU_DB>")
     (Verifica que coincida con tu proyecto).

Dependencias clave (ya declaradas en build.gradle.kts)
------------------------------------------------------
- Firebase Auth KTX
- Firebase Realtime Database KTX
- Play Services Location
- AndroidX AppCompat/Material/RecyclerView/ConstraintLayout
- ViewBinding habilitado

Estructura principal (src y lógica)
----------------------------------
app/src/main/java/.../

auth/
  - AuthActivity.kt               -> Login/Registro con Firebase (email/clave @gmail.com recomendado)
data/
  - Product.kt                    -> Modelo de producto
  - SampleData.kt                 -> Catálogo de ejemplo
  - CartManager.kt                -> Carrito y subtotal
  - Prefs.kt                      -> (legacy) almacenamiento local (ya reemplazado por Firebase Auth)
  - ShippingCalculator.kt         -> Reglas de costo de envío (0, 150/km, 300/km)
  - GeoUtils.kt                   -> Fórmula de Haversine (km)
  - UserLocation.kt               -> Modelo para guardar ubicación en DB
ui/
  adapters/
    - ProductAdapter.kt           -> Adaptador del listado de productos
  - ProductListActivity.kt        -> Catálogo y navegación al checkout
  - CheckoutActivity.kt           -> Subtotal, distancia, tarifa aplicada y total
  - FreezerActivity.kt            -> Temperatura, alerta y conversión a radianes (System.out -> Logcat)
util/
  - MainActivity.kt (si existe)    -> Menú/launcher alternativo

app/src/main/res/layout/
  - activity_auth.xml
  - activity_product_list.xml
  - item_product.xml
  - activity_checkout.xml
  - activity_freezer.xml

AndroidManifest.xml                -> Activities, permisos y exported flags
build.gradle.kts (módulo app)      -> Plugins, compileSdk/targetSdk, minSdk=23, dependencias Firebase/Location
settings.gradle / build.gradle     -> Configuración de proyecto

Reglas de negocio del despacho
------------------------------
- Compra >= $50.000 y distancia <= 20 km  -> Envío $0.
- Compra entre $25.000 y $49.999          -> $150 por km.
- Compra < $25.000                        -> $300 por km.
La lógica está encapsulada en ShippingCalculator.kt y el resultado se muestra en Checkout.

Cálculo de distancia (Haversine)
--------------------------------
- Referencia de destino: Plaza de Armas de Santiago (aprox. -33.4372, -70.6506).
- GeoUtils.haversineKm(latUsuario, lonUsuario, refLat, refLon) retorna km (double).
- En dispositivos con GMS se usa FusedLocationProvider; sin GMS, LocationManager.

Compilación y ejecución
-----------------------
1) Abrir el proyecto en Android Studio.
2) Esperar la sincronización de Gradle.
3) Verificar minSdk=23; JDK 17 configurado.
4) Colocar google-services.json en app/.
5) Build > Make Project (o Build APKs).
6) APK debug: app/build/outputs/apk/debug/app-debug.apk
7) Instalar en dispositivo (habilitar "Fuentes desconocidas" si se instala por fuera de Play).
8) Permitir el permiso de **Ubicación** al abrir la app (requerido para el cálculo del envío).

Pruebas rápidas (checklist)
---------------------------
- Login:
  - Registrar un usuario (email válido y clave >= 6).
  - Iniciar sesión y navegar al catálogo.
- Catálogo:
  - Agregar/Quitar productos y revisar subtotal.
- Checkout:
  - Ver que se muestre la **distancia calculada** y la **tarifa aplicada** (0, 150 o 300).
  - Recalcular distancia con el botón correspondiente.
- Freezer:
  - Ingresar temperatura y confirmar.
  - Revisar en Logcat la línea:
    System.out.println("RADIANES => input=<temp>; radianes=<valor>")
  - Si temp > -12 °C, se debe informar alerta (según UI).

Prototipo y Canvas
------------------
- /docs/canvas.png                          -> Modelo Canvas (Canva).
- /docs/prototipo/login.png                 -> Pencil
- /docs/prototipo/catalogo.png              -> Pencil
- /docs/prototipo/checkout.png              -> Pencil
- /docs/prototipo/temperatura.png           -> Pencil

Dispositivos sin Google Play Services
-------------------------------------
- La app detecta ausencia de GMS y usa LocationManager (NETWORK/GPS).
- Asegúrate de activar la Ubicación del dispositivo (GPS/Wi-Fi).
- Mensajes de error claros si no se obtiene posición.

Seguridad / Privacidad
----------------------
- No se incluye google-services.json real en repositorios públicos.
- Las cuentas de prueba se crean en Firebase Authentication.
- Realtime Database: usar reglas por UID en producción.

Problemas comunes y soluciones
------------------------------
- "Error al guardar ubicación": verifica URL exacta de Realtime Database en getInstance().
- "Falla de compilación por Java": usar JDK 17 en Project Structure.
- "No calcula distancia": confirma permiso de Ubicación y que el proveedor esté activo.
- "Login no entra": valida método Email/Password habilitado en Firebase.

Créditos
--------
- Desarrollado como parte de Taller de Aplicaciones Móviles (Semanas 4–7).
- Autor: Luis Mandujano Morales
