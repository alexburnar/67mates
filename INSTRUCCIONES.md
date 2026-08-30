> ⚠️ **Desactualizado:** este documento describe el flujo 100% manual original (compilar a mano en Xcode). El flujo actual usa Codemagic para compilar y subir a TestFlight automáticamente — ver [README.md](README.md). Se deja este documento como referencia por si algún día hace falta compilar a mano.

# Cómo convertir esto en una app de iOS y subirla a la App Store

Este proyecto usa **Capacitor**, que envuelve tu web (HTML/CSS/JS) en una app iOS nativa real. Todo esto se hace en un **Mac con Xcode instalado**. Pásale esta carpeta entera a tu amigo con la cuenta de Apple Developer.

## Requisitos previos (los tiene que tener tu amigo)

- Mac con **Xcode** instalado (gratis, desde la Mac App Store)
- **Node.js** instalado (desde https://nodejs.org, versión 18 o superior)
- **CocoaPods** instalado (`sudo gem install cocoapods` en Terminal)
- Cuenta de **Apple Developer** (99$/año) — ya la tenéis
- **Xcode Command Line Tools**: `xcode-select --install`

## Pasos

### 1. Instalar dependencias

Abre Terminal, entra en esta carpeta y ejecuta:

```bash
cd mathquiz-ios
npm install
```

### 2. Añadir la plataforma iOS

```bash
npx cap add ios
npx cap sync ios
```

Esto genera una carpeta `ios/` con el proyecto Xcode nativo.

### 3. Cambiar el identificador de la app (IMPORTANTE)

Antes de compilar, edita `capacitor.config.json` y cambia:

```json
"appId": "com.alexburnar.sixsevenmates"   // ya configurado así, no hace falta cambiarlo
```

por un identificador único vuestro, por ejemplo `com.nombredetuamigo.calculadora`. Luego vuelve a ejecutar `npx cap sync ios`.

### 4. Abrir en Xcode

```bash
npx cap open ios
```

Esto abre Xcode automáticamente.

### 5. Configurar firma (Signing)

En Xcode:
1. Selecciona el proyecto "App" en el panel izquierdo
2. Ve a la pestaña **Signing & Capabilities**
3. En **Team**, selecciona la cuenta de Apple Developer de tu amigo
4. Xcode generará el perfil de aprovisionamiento automáticamente
5. Cambia el **Bundle Identifier** para que coincida con el `appId` que pusiste en el paso 3

### 6. Añadir el icono de la app (ya incluido en este zip)

Ya tienes todo listo en la carpeta `AppIcon.appiconset/` (incluye todos los tamaños necesarios + `Contents.json`) y también el icono suelto en `app-icon-1024.png` por si tu amigo prefiere arrastrarlo a mano.

**Opción A — copiar la carpeta completa (recomendado, más rápido):**
1. En Finder, entra en la carpeta que generó `npx cap add ios`: `ios/App/App/Assets.xcassets/`
2. Borra la carpeta `AppIcon.appiconset` que hay ahí dentro
3. Copia la carpeta `AppIcon.appiconset` que viene en este zip y pégala en su lugar
4. Vuelve a Xcode (o ábrelo si no lo tenías abierto) y comprueba en `Assets.xcassets → AppIcon` que todos los tamaños se ven bien

**Opción B — a mano:**
1. En Xcode, ve a `App/App/Assets.xcassets/AppIcon`
2. Arrastra `app-icon-1024.png` al hueco grande (1024pt / App Store). En versiones recientes de Xcode con un único hueco, esto ya es suficiente y Xcode genera el resto automáticamente.

### 7. Probar en un dispositivo o simulador

Selecciona un simulador arriba (ej. "iPhone 15") y pulsa el botón ▶️ Play para probar que todo funciona.

### 8. Crear la ficha en App Store Connect

Tu amigo debe ir a https://appstoreconnect.apple.com y:
1. Crear una nueva app
2. Usar el mismo Bundle ID configurado en Xcode
3. Rellenar nombre, descripción, categoría, capturas de pantalla, política de privacidad (URL obligatoria, puede ser una página simple diciendo que la app no recopila datos), y clasificación de edad

**Capturas de pantalla — ya incluidas en `screenshots-app-store/`:**

Son capturas reales de la app (no maquetas), generadas al tamaño exacto que exige Apple para el iPhone 6.9" (1320×2868 px), que es el único tamaño obligatorio en 2026 — App Store Connect escala automáticamente para el resto de iPhones.

- `1_menu.png` — pantalla de selección de operaciones y nivel
- `2_quiz.png` — pantalla de pregunta con teclado numérico
- `3_resultados.png` — resumen de resultados con revisión de respuestas
- `4_stats.png` — estadísticas y progreso

Al subir la ficha, en la sección "Capturas de pantalla del iPhone" arrastra estos 4 archivos en el hueco de "6.9 pulgadas". No hace falta subir más tamaños, Apple las escala solo.

### 9. Archivar y subir

En Xcode:
1. Selecciona **Any iOS Device (arm64)** como destino (no un simulador)
2. Menú **Product → Archive**
3. Cuando termine, se abre el **Organizer** → pulsa **Distribute App**
4. Elige **App Store Connect** → **Upload**
5. Sigue el asistente (usa las opciones automáticas de firma)

### 10. Enviar a revisión

Vuelve a App Store Connect, selecciona el build que acabas de subir en la ficha de la app, y pulsa **Enviar para revisión**. Apple tarda normalmente entre 24-48 horas en revisarla.

## Notas importantes

- **Datos guardados**: la app usa `localStorage` del navegador para guardar el historial y las estadísticas. Con Capacitor esto sigue funcionando igual, los datos se guardan localmente en el dispositivo.
- **Sin conexión a internet**: la app no necesita internet para funcionar, así que no hará falta pedir permisos de red.
- **Política de privacidad**: aunque la app no recopila ni envía ningún dato (todo es local), Apple exige igualmente una URL de política de privacidad. Puede ser una página muy simple que diga que no se recopilan datos.
- Si Apple rechaza la primera versión por motivos menores de metadatos (nombre, descripción, capturas), es muy normal — se corrige y se vuelve a enviar sin problema.
