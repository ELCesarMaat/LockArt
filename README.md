# LockArt · Carátula en bloqueo

Módulo LSPosed que vuelve a mostrar la **carátula de la música a pantalla completa en la pantalla de bloqueo**, como en Android 10 y en las ROMs personalizadas. Está hecho para el *Scene Container* de Android 17, donde los mods antiguos ya no funcionan.

<p align="center">
  <img src="docs/bloqueo.png" alt="Pantalla de bloqueo con la carátula de la canción de fondo" width="320">
</p>

## Qué hace

- Toma la carátula de lo que esté sonando (Spotify, YouTube Music o cualquier app con sesión multimedia) y la pone de fondo en el bloqueo, debajo del reloj, las notificaciones y el reproductor.
- La carátula sigue fotograma a fotograma la animación real del bloqueo: aparece al encender la pantalla, se oculta en Always-On Display y se desvanece al desbloquear.
- Al cambiar de canción hace un fundido cruzado entre carátulas. Si la app pasa un instante por «pausa» al cambiar de pista, espera un poco antes de quitarla para que no parpadee.
- Se puede **desenfocar** y oscurecer con un **velo**, para que el reloj y las notificaciones se sigan leyendo.
- Trae una app de ajustes con vista previa en vivo de la canción actual y un indicador de si el módulo está cargado en SystemUI.

## Cómo funciona

![Capas de la pantalla de bloqueo](docs/capas.svg)

Engancha dos puntos de SystemUI:

| Gancho | Para qué |
|---|---|
| `WindowRootView.onAttachedToWindow` | Mete la vista de la carátula en el índice 0 de la ventana del bloqueo: por encima del fondo de pantalla y por debajo de todo lo demás. Sin Scene Container (Android 14–16) usa `NotificationShadeWindowView`. |
| `KeyguardTransitionRepositoryImpl.emitTransition` | Recibe cada paso de transición del bloqueo (de dónde a dónde y en qué porcentaje) para que la carátula siga la animación del sistema. |

Dentro de SystemUI sigue las sesiones multimedia con `MediaSessionManager` y lee la carátula de los metadatos de la sesión (imagen o URI). El desenfoque se calcula una sola vez por carátula, sobre una copia reducida, así que no consume GPU en cada fotograma. Los ajustes se mandan de la app a SystemUI con una transmisión protegida por un permiso de firma: ninguna otra app puede cambiarlos.

## Requisitos

- Android 12 o superior. **Probado en un Pixel 9 Pro XL con Android 17** (`CP3A.261005.005`). En otras versiones o ROMs puede que los ganchos no coincidan.
- Root (KernelSU o Magisk) y [LSPosed](https://github.com/LSPosed/LSPosed).

## Instalarlo

1. Descarga `LockArt-x.y.apk` de la página de [Releases](../../releases) e instálalo:
   ```bash
   adb install LockArt-2.1.apk
   ```
2. En LSPosed, activa **Carátula en bloqueo** con el alcance **Interfaz del sistema** (`com.android.systemui`); viene preseleccionado.
3. Abre la app **Carátula en bloqueo** y pulsa **Reiniciar SystemUI** (la app pide root para eso). También sirve reiniciar el teléfono.
4. Pon música y bloquea la pantalla.

## Ajustes

| Ajuste | Qué hace | Por defecto |
|---|---|---|
| Mostrar carátula | Activa o desactiva la carátula sin desactivar el módulo. | Activado |
| Mantener en pausa | Deja la carátula aunque pauses la música (si no, se quita al pausar). | Desactivado |
| Desenfoque | De nítida a 25. | Nítida |
| Velo | Oscurece la carátula de 0 a 80 % para que se lea el texto. | 25 % |
| Velo automático | Usa un velo blanco cuando tu fondo de bloqueo pide texto oscuro. | Activado |
| Reiniciar SystemUI | Hace falta después de activar o actualizar el módulo. | — |

### Ejemplo: carátula de fondo discreta

Desenfoque en 15 y velo en 40 %: la carátula queda como un fondo de color de la canción y el reloj se lee perfecto. Para verla nítida y protagonista, desenfoque en «Nítida» y velo en 10–20 %.

## Si algo no funciona

La tarjeta de estado de la app indica qué pasa:

| Estado | Significado | Qué hacer |
|---|---|---|
| **Activo en SystemUI** | Todo bien; muestra cuántas sesiones multimedia ve. | Pon música y bloquea. |
| **SystemUI no responde** | El módulo no está cargado. | Actívalo en LSPosed con alcance «Interfaz del sistema» y reinicia SystemUI. |
| **Cargado, pero sin vista** | El módulo cargó pero no encontró la pantalla de bloqueo. | Reinicia SystemUI. Si sigue igual, tu versión de Android no es compatible. |
| Aviso de root al reiniciar | La app no tiene root. | Concédelo en KernelSU/Magisk o reinicia el teléfono. |

Para ver el log del módulo:

```bash
adb logcat -s LockArt
```

## Código fuente y modificaciones

Este repositorio publica **solo el APK compilado**. Si quieres modificarlo, adaptarlo a otra versión de Android o a otra ROM, **contáctame** abriendo un [issue](../../issues) o por GitHub ([@ELCesarMaat](https://github.com/ELCesarMaat)).

Para compilarlo se necesita:

- Android Studio con JDK 17 o superior, Android SDK 37 y el Gradle incluido en el proyecto (AGP 9, Kotlin 2 y Jetpack Compose con Material 3).
- La API de Xposed 82 (`de.robv.android.xposed:api:82`, solo para compilar).
- Una clave de firma propia. La app y el gancho se comunican con un permiso de firma, así que una versión firmada con otra clave no se instala encima de esta: hay que desinstalar la anterior.
- Un teléfono rooteado con LSPosed para probarlo, idealmente con la misma versión de SystemUI.

## Aviso

Modifica SystemUI en tiempo de ejecución. Si SystemUI se cierra en bucle, desactiva el módulo en LSPosed o arranca en el modo seguro de KernelSU/Magisk, que apaga todos los módulos. Úsalo bajo tu responsabilidad. Las carátulas que muestra pertenecen a sus respectivos dueños.
