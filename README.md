# Lited-Iso Builder 🛠️

Este repositorio automatiza la creación de imágenes ISO optimizadas y reducidas de **Windows 11** utilizando GitHub Actions y scripts en PowerShell (basados en la filosofía de Tiny11). 

El objetivo es permitir la generación desatendida ("Headless") de sistemas Windows 11 sumamente ligeros, ideales para máquinas virtuales, hardware antiguo o entornos de Integración Continua (CI/CD) donde los recursos son limitados.

---

## 🚀 Cómo usar (GitHub Actions)

La construcción de la ISO está completamente automatizada mediante GitHub Actions. No necesitas ejecutar nada en tu PC local.

1. Ve a la pestaña **Actions** en este repositorio.
2. Selecciona el workflow **"Build Lite Windows ISO"**.
3. Haz clic en **"Run workflow"**.
4. Se desplegará un menú con varias opciones personalizables.

### Opciones de Compilación

| Parámetro | Descripción |
| :--- | :--- |
| **Release Tag** | (Opcional) Permite especificar una versión exacta de UUP Dump (ej. `22631.3593.PRO.X64.ES`). Si lo dejas en `latest`, buscará automáticamente la última versión de Windows 11 disponible. |
| **Build Type** | Define el nivel de agresividad de la limpieza. Ver la sección de [Modos: Lite vs Core](#%EF%B8%8F-modos-lite-vs-core) abajo. |
| **Image Index** | Edición de Windows a modificar (1=Home, 4=Education, **6=Pro**, 7=Pro N). |
| **Architecture** | Arquitectura objetivo (x64 o arm64). |
| **Language** | Idioma de la ISO (ej. ES, EN). |

### Flags Modulares (Avanzados)

Podés tildar estas casillas para aplicar modificaciones extremas a la imagen. 
> [!WARNING]
> **Tené mucho cuidado:** Algunas de estas opciones rompen funciones de Windows de forma permanente.

- `remove_winre`: Elimina el Entorno de Recuperación de Windows (WinRE) para ahorrar espacio.
- `remove_defender`: **[PELIGROSO]** Elimina completamente los paquetes y bloquea los servicios de Windows Defender.
- `disable_windows_update`: **[PELIGROSO]** Bloquea los servicios y las políticas de red de Windows Update, impidiendo cualquier actualización futura.
- `export_esd`: Exporta la imagen en formato `.esd` en lugar de `.wim`, lo que reduce drásticamente el tamaño final de la ISO a cambio de mayor tiempo de compilación.

---

## ⚖️ Modos: Lite vs Core

El workflow te permite elegir entre dos filosofías principales a través del menú **Build Type**:

### 🟢 Modo Lite (Seguro)
El modo Lite es un Windows 11 limpio, libre de bloatware (Edge, OneDrive, Telemetría, apps patrocinadas) y sin requisitos absurdos de hardware (TPM / Secure Boot anulados), pero **mantiene la integridad estructural de Windows**.
- Es seguro para el uso diario.
- Permite instalar nuevas características de Windows (WSL, Hyper-V).
- Permite reparaciones del sistema (`sfc /scannow`).
- *Nota: Podés usar el modo Lite y aún así tildar la desactivación de Defender/Updates si lo deseás.*

### 🔴 Modo Core (Destructivo / Extremo)
El modo Core hace todo lo que hace el Lite, pero adicionalmente aplica un **"Extreme WinSxS Trim"**. Esto elimina casi toda la carpeta `C:\Windows\WinSxS` (el almacén de componentes de Windows), dejando solo lo mínimo para que el sistema arranque.
- Produce la ISO de menor tamaño posible.
- **Rompe irremediablemente el sistema de "Servicing"**: Es imposible actualizar Windows o instalar nuevas características.
- Ideal para servidores headless desechables, VMs de un solo uso o hardware hiper-limitado.

---

## 📜 Detalles Técnicos

El workflow ejecuta el script `scripts/tiny11maker-headless.ps1` inyectando los parámetros seleccionados en la interfaz web. El script se encarga de:

1. Extraer la ISO original.
2. Montar el registro offline (`SYSTEM`, `SOFTWARE`, `NTUSER`, etc.).
3. Eliminar paquetes vía DISM (Appx, Edge, OneDrive, etc.).
4. Aplicar inyecciones al registro (bypass TPM, deshabilitar telemetría, bloqueos de Defender/Update).
5. (Si se elige Core) Tomar posesión de `WinSxS` y purgar subdirectorios no vitales.
6. Desmontar y reempaquetar la ISO.
