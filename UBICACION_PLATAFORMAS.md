# Ubicación (GPS) — configuración por plataforma

El código Dart ya está listo (`lib/services/ubicacion/`), pero **el GPS no
funciona hasta declarar los permisos en cada plataforma**. Esos archivos viven
en las carpetas `android/`, `ios/` y `web/` de tu proyecto local.

Aplicá lo que corresponda a las plataformas que uses y volvé a compilar
(`flutter clean && flutter pub get` si algo no toma efecto).

---

## Android

Archivo: `android/app/src/main/AndroidManifest.xml`

Agregá estas líneas **dentro de `<manifest>` y antes de `<application>`**:

```xml
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
```

- `ACCESS_FINE_LOCATION` es la que da precisión de GPS, la que necesitás para
  ubicar un poste.
- `ACCESS_COARSE_LOCATION` acompaña a la anterior: desde Android 12 el usuario
  puede conceder solo ubicación aproximada, y sin declararla la app falla en ese
  caso en vez de degradarse.

**No** agregues `ACCESS_BACKGROUND_LOCATION`: yoDibujo solo usa la ubicación con
la app abierta, y pedir permiso de segundo plano complica la revisión en Play
Store sin darte nada.

### Versión mínima de Android

`geolocator` requiere `minSdkVersion 21` o mayor. Revisá
`android/app/build.gradle`:

```gradle
defaultConfig {
    minSdkVersion 21   // o superior
}
```

---

## iOS

Archivo: `ios/Runner/Info.plist`

Agregá dentro del `<dict>` principal:

```xml
<key>NSLocationWhenInUseUsageDescription</key>
<string>yoDibujo usa tu ubicación para centrar el mapa y colocar postes en el punto donde estás.</string>
```

El texto **lo lee el usuario** en el diálogo de permiso, y Apple rechaza apps
cuya justificación sea vaga. Explicá para qué se usa, no que "la app necesita
ubicación".

Si más adelante usás el seguimiento continuo con la app en segundo plano,
agregarías también `NSLocationAlwaysAndWhenInUseUsageDescription`. Hoy no hace
falta.

---

## Web

No hay archivo que tocar, pero:

- El navegador **solo entrega ubicación sobre HTTPS** (o `localhost`). En un
  host estático como Netlify eso ya se cumple.
- El permiso lo pide el navegador, no el sistema. Si el usuario lo bloquea, se
  resuelve desde el candado de la barra de direcciones — por eso el mensaje de
  "permiso bloqueado" manda a los ajustes.

---

## macOS (si compilás escritorio)

Archivos: `macos/Runner/DebugProfile.entitlements` y `Release.entitlements`.

```xml
<key>com.apple.security.personal-information.location</key>
<true/>
```

Y en `macos/Runner/Info.plist`, la misma clave
`NSLocationWhenInUseUsageDescription` que en iOS.

---

## Cómo probar que quedó bien

1. **Permiso concedido:** tocás el botón de ubicación y el mapa se centra con el
   punto azul.
2. **Permiso denegado una vez:** aparece el aviso con la opción de reintentar.
3. **Permiso denegado para siempre:** el aviso ofrece "Abrir ajustes" (no
   "Reintentar", porque volver a pedirlo no haría nada).
4. **GPS apagado:** el aviso ofrece abrir los ajustes de ubicación del sistema.
5. **Bajo techo:** si la precisión pasa de 20 m, el punto se dibuja naranja en
   vez de azul y avisa la precisión, para que no coloques un poste a ciegas.

El caso 3 es el que más se rompe en pruebas: en Android, una vez que denegás dos
veces, el sistema no vuelve a mostrar el diálogo. Para repetir la prueba hay que
borrar los datos de la app o reinstalarla.
