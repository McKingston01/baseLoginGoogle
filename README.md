## Guía Completa: Configuración de Signing para Android con CI/CD

1.2 Generar el keystore

bashkeytool -genkey -v -keystore release-key.keystore -alias release -keyalg RSA -keysize 2048 -validity 10000

Paso 2: Obtener los SHA Fingerprints

2.1 SHA del keystore Debug (desarrollo)

bashkeytool -list -v -alias androiddebugkey -keystore ~/.android/debug.keystore -storepass android

2.2 SHA del keystore Release (producción)

bashkeytool -list -v -alias release -keystore release-key.keystore

Paso 3: Configurar Firebase
3.1 Ir a Firebase Console

Ve a Firebase Console
Selecciona tu proyecto
Ve a ⚙️ Project Settings
Pestaña "Your apps"
Selecciona tu app Android

3.2 Agregar los SHA-1 y 256 fingerprints

Scroll down hasta "SHA certificate fingerprints"
Click "Add fingerprint"
Agrega los 4 SHA que obtuviste (SHA1 y SHA256 de debug y release)
Click "Save"

3.3 Descargar google-services.json actualizado

Click "Download google-services.json"
Reemplaza el archivo en app/google-services.json

Paso 4: Configurar build.gradle (Module: app)
4.1 Backup del archivo actual
bashcp app/build.gradle app/build.gradle.backup
4.2 Modificar app/build.gradle
kotlinplugins {
id 'com.android.application'
id 'org.jetbrains.kotlin.android'
id 'com.google.gms.google-services'
}

android {
namespace = "com.example.tu app"
compileSdk = 36

    // ===== CONFIGURACIÓN DE SIGNING =====
    signingConfigs {
        getByName("debug") {
        }

        create("release") {
        
            if (System.getenv("KEYSTORE_PASSWORD") != null) {
                storeFile = file("../release-key.keystore")
                storePassword = System.getenv("KEYSTORE_PASSWORD")
                keyAlias = "release"
                keyPassword = System.getenv("KEY_PASSWORD")
            }
        }
    }

    defaultConfig {
        applicationId = "com.example.baselogingoogle"
        minSdk = 24
        targetSdk = 36
        versionCode = 1
        versionName = "1.0"

        testInstrumentationRunner = "androidx.test.runner.AndroidJUnitRunner"
    }

    buildTypes {
        getByName("debug") {
            isDebuggable = true
            applicationIdSuffix = ".debug"
            resValue("string", "app_name", "BaseLogin Debug")
        }

        getByName("release") {
            isMinifyEnabled = true
            isShrinkResources = true
            proguardFiles(
                getDefaultProguardFile("proguard-android-optimize.txt"),
                "proguard-rules.pro"
            )
            resValue("string", "app_name", "BaseLogin")

            // Solo usa signing release si las variables están disponibles
            if (System.getenv("KEYSTORE_PASSWORD") != null) {
                signingConfig = signingConfigs.getByName("release")
            }
        }
    }


Paso 5: Crear archivo gradle.properties (Opcional para desarrollo local)
5.1 Crear/modificar gradle.properties en la raíz del proyecto
properties# Para desarrollo local (NO subir a Git)
# KEYSTORE_PASSWORD=tu_password_seguro
# KEY_PASSWORD=tu_key_password

# Configuraciones de optimización
org.gradle.jvmargs=-Xmx2048m -Dfile.encoding=UTF-8
org.gradle.parallel=true
org.gradle.configureondemand=true
android.useAndroidX=true
kotlin.code.style=official
Paso 6: Configurar .gitignore
6.1 Agregar al .gitignore
gitignore# Keystore files
*.keystore
*.jks

# Local gradle properties (con passwords)
local.properties
gradle.properties.local

# Otros archivos sensibles
google-services.json.backup
## 📁 Estructura inicial del proyecto Android con Hilt + Compose

Este documento describe cómo crear la estructura base de directorios para una app Android modular usando **Jetpack Compose** y **Hilt** como sistema de inyección de dependencias.

> **Entorno:** Windows 10 usando Git Bash  
> **Ruta inicial:** 



## 🛠️ Comando para crear carpetas (ejecutar en Git Bash)

ubicarse dentro del directorio principal del proyecto

mkdir -p presentation/{login,register,home} domain/{model,usecase} data/{repository,source} di


## ⚙️ Configuración de Gradle (Optimización para builds Android)

Se realizaron las siguientes mejoras en el archivo `gradle.properties`:

- Aumento de memoria para el daemon de Gradle (`-Xmx4096m`) para mejorar el rendimiento general en compilaciones.
- Configuración del daemon de Kotlin (`-Xmx2048m`) para evitar errores de conexión y mejorar la estabilidad del compilador.
- Activación de compilación paralela (`org.gradle.parallel=true`) para aprovechar múltiples núcleos de CPU.
- Habilitación del caché de compilación (`org.gradle.caching=true`) para acelerar builds repetitivos (ej. depuración vía USB).
- Se mantuvieron configuraciones importantes como el uso de AndroidX y estilo oficial de Kotlin.

> Estas optimizaciones están diseñadas para entornos de desarrollo Android con al menos 10 GB de RAM.

## compilar en modo debug

./gradlew assembleDebug

una vez que compile para instalar  

adb install -r app/build/outputs/apk/debug/app-debug.apk

exportar claves de la firma sha 

export KEYSTORE_PASSWORD="aquí_va_tu_contraseña_real_del_keystore"
export KEY_PASSWORD="aquí_va_tu_contraseña_real_de_la_key"

verificar
echo "KEYSTORE_PASSWORD configurado: $([ -n "$KEYSTORE_PASSWORD" ] && echo 'SÍ' || echo 'NO')"
echo "KEY_PASSWORD configurado: $([ -n "$KEY_PASSWORD" ] && echo 'SÍ' || echo 'NO')"

# Verificar variables de entorno
echo "KEYSTORE_PASSWORD: $([ -n "$KEYSTORE_PASSWORD" ] && echo 'CONFIGURADO' || echo 'NO CONFIGURADO')"
echo "KEY_PASSWORD: $([ -n "$KEY_PASSWORD" ] && echo 'CONFIGURADO' || echo 'NO CONFIGURADO')"

# Verificar que el keystore existe
ls -la release-key.keystore

# Ver el reporte de signing (esto es clave)
./gradlew signingReport

configurar action con secretos 
ver clave  base64 -w 0 release-key.keystore
configurar en action
5.2 Configurar GitHub Secrets

Ve a tu repositorio en GitHub
Settings → Secrets and variables → Actions
Click en New repository secret
Crea estos 3 secrets:

NameValueKEYSTORE_FILEEl contenido base64 que acabas de copiarKEYSTORE_PASSWORDTu contraseña del keystoreKEY_PASSWORDTu contraseña de la key

y ademas condigurar como base 64 el json de google
base64 -w 0 app/google-services.json
