# 67Mates — guía de publicación

App de ejercicios de cálculo mental (Capacitor, web pura envuelta en nativo). Este documento es la guía completa y actualizada para llevarla a TestFlight (iOS, vía Codemagic) y a Google Play (Android, build local).

## Lo que ya está hecho en este repo

- `appId` / `applicationId` definitivo en **iOS y Android**: `com.alexburnar.sixsevenmates`
- Plataformas nativas generadas: `ios/` y `android/`
- Iconos y splash screens generados para ambas plataformas (claro y oscuro) a partir de `app-icon-1024.png`, vía `@capacitor/assets`
- `codemagic.yaml` — build + firma + subida automática a TestFlight para iOS
- Keystore de release de Android ya generado: `android/67mates-release.keystore` (alias `67mates`), con sus credenciales en `android/keystore.properties` — **ninguno de los dos se sube a git** (están en `.gitignore` a propósito)
- Política de privacidad: `docs/privacy.html`, lista para publicarse con GitHub Pages

## ⚠️ Antes de nada: copia de seguridad del keystore

`android/67mates-release.keystore` + `android/keystore.properties` son las credenciales que firman la app de Android. **Si se pierden, no podrás publicar nunca más una actualización de esta app en Google Play** (tendrías que crear una app nueva desde cero). Cópialos ahora mismo a un sitio seguro fuera de este proyecto (gestor de contraseñas, disco cifrado, etc.) — no van a subirse a GitHub.

---

## Parte 1 — Android: build local y subida manual a Google Play

Tal y como pediste, esto se compila aquí mismo, en este proyecto, con el SDK de Android que ya tienes instalado.

### 1.1 Compilar el .aab firmado

```bash
cd mathquiz-ios
npx cap sync android
cd android
./gradlew bundleRelease
```

El archivo firmado queda en:
```
android/app/build/outputs/bundle/release/app-release.aab
```

### 1.2 Crear la app en Google Play Console

1. Ve a [play.google.com/console](https://play.google.com/console) → **Crear app**
2. Nombre: `67Mates`, idioma predeterminado: Español, tipo: App, gratuita
3. Completa el cuestionario de declaraciones (app no dirigida a menores como categoría principal, no es app de noticias, etc. — según corresponda)

### 1.3 Ficha de la tienda (Store listing)

- **Categoría:** Educación
- **Descripción corta / completa:** ejercicios de cálculo mental (suma, resta, multiplicación, división), niveles de dificultad, modo infinito, rachas y estadísticas de progreso
- **Icono 512×512:** puedes usar `app-icon-1024.png` (Play lo reescala)
- **Feature graphic (1024×500):** **falta por crear** — no hay ninguno en el proyecto. Es obligatorio para publicar. Puedes pedírmelo aparte si quieres que te lo genere.
- **Capturas de pantalla:** las 4 de `screenshots-app-store/` sirven también aquí (Play acepta capturas de teléfono en varias proporciones; si te las rechaza por aspect ratio, dímelo y te las reexporto al tamaño de Android)

### 1.4 Política de privacidad (obligatoria)

Necesitas una URL pública. La más rápida con lo que ya tienes:

1. En GitHub, entra en el repo `67mates` → **Settings → Pages**
2. En "Build and deployment" → Source: **Deploy from a branch**, Branch: **main**, carpeta **/docs** → Save
3. En 1-2 minutos la política estará en: `https://alexburnar.github.io/67mates/privacy.html`
4. Pega esa URL en Play Console (Store listing → Política de privacidad) y también en App Store Connect (más abajo)

### 1.5 Data Safety (formulario de seguridad de datos)

Como la app no recopila ni comparte ningún dato (todo es `localStorage` local), en el formulario de Play Console declara: **"No recopilamos datos de los usuarios"** en todas las categorías (ubicación, información personal, actividad en la app, etc.).

### 1.6 Clasificación de contenido

Completa el cuestionario IARC — al ser una app de ejercicios matemáticos sin contenido sensible, debería quedar en la clasificación más baja (PEGI 3 / Todos los públicos).

### 1.7 Subir el build

1. **Producción → Pruebas → Prueba interna** (recomendado empezar aquí, no directo a producción)
2. Crea una versión nueva, sube `app-release.aab`
3. Añade testers (tu email u otros) en la lista de la pista de prueba interna
4. Guarda y envía a revisión (las pistas de prueba interna no requieren revisión de Google, suele estar disponible en minutos)

---

## Parte 2 — iOS: Codemagic (build + TestFlight automáticos)

### 2.1 Subir este proyecto a GitHub ✅ hecho

Repo ya creado y con el primer commit subido: `git@github.com:alexburnar/67mates.git` (rama `main`).

### 2.2 Crear la app en App Store Connect ✅ hecho

App creada: [appstoreconnect.apple.com/apps/6806802721/distribution](https://appstoreconnect.apple.com/apps/6806802721/distribution)
- Nombre: `67Mates` · Idioma principal: Español (España) · Bundle ID: `com.alexburnar.sixsevenmates` (registrado en Certificates, Identifiers & Profiles) · Apple ID numérico: `6806802721`

### 2.3 Conectar el repo en Codemagic

1. Entra en [codemagic.io](https://codemagic.io) con el mismo equipo que usáis para BloomStrong
2. **Add application** → conecta el repo `alexburnar/67mates` (GitHub)
3. Cuando pregunte el tipo de proyecto, elige **"Flutter App" no** — elige el flujo de **configuración YAML** (este repo ya trae `codemagic.yaml` con el workflow `ios-release` definido, Codemagic lo detecta automáticamente)
4. Verifica en **Team settings → Integrations** que la integración `codemagic_asc_api_key` (la misma de BloomStrong) sigue activa — si es la misma cuenta de Apple Developer, no hay que crear nada nuevo

### 2.4 Rellenar el ASC_APP_ID ✅ hecho

`ASC_APP_ID: "6806802721"` ya está puesto en `codemagic.yaml` — los builds incrementarán el número de versión automáticamente.

### 2.5 Lanzar el build

En Codemagic, workflow **"67Mates — iOS"** → **Start new build** (rama `main`). Con esto:
- Instala dependencias, sincroniza Capacitor, instala CocoaPods
- Firma automáticamente (Codemagic crea el certificado/perfil si no existen)
- Compila el `.ipa`
- Lo sube automáticamente a **TestFlight**

Tarda entre 10 y 20 minutos. Recibirás un email en `alexburnar@gmail.com` cuando termine.

### 2.6 Añadir testers en TestFlight

En App Store Connect → tu app → **TestFlight** → añade tu email (o el de quien vaya a probarla) como tester interno. Recibirá una invitación para instalar la app vía la app TestFlight — no hace falta esperar revisión de Apple para pruebas internas.

---

## Checklist para poder mandarlo a testear hoy

- [x] Push del repo a `git@github.com:alexburnar/67mates.git`
- [ ] Activar GitHub Pages (Settings → Pages → main /docs) para la URL de privacidad
- [x] Crear la app en App Store Connect (Apple ID `6806802721`) y copiar su Apple ID numérico
- [ ] Conectar el repo en Codemagic y lanzar el workflow `ios-release`
- [ ] Añadirte como tester interno en TestFlight
- [ ] Compilar el `.aab` de Android aquí (`./gradlew bundleRelease`) y subirlo a Play Console → Prueba interna
- [ ] Guardar `android/67mates-release.keystore` + `android/keystore.properties` en un sitio seguro fuera del proyecto
