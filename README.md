# Idioma

🇪🇸 **Español** | [🇬🇧 English](README-en.md)

# SAI - Automatizador de Muxeo MKV desde ASS

Esta herramienta automatiza el proceso de crear archivos MKV utilizando `mkvmerge`. Lee un archivo de subtítulos `.ass` especialmente formateado que contiene no solo los subtítulos, sino también metadatos sobre los archivos de video, audio, fuentes, capítulos y el nombre de salida deseado.

## Prerrequisitos

1.  **`mkvmerge`**: Debes tener `mkvmerge` (parte de MKVToolNix) instalado y accesible en tu PATH, o colocar `mkvmerge.exe` (o `mkvmerge` en Linux/macOS) en el mismo directorio que el ejecutable de `SAI`.

## Uso

Ejecuta la herramienta desde la línea de comandos proporcionando los archivos o rangos de episodios a procesar. La herramienta buscará los archivos `.ass` correspondientes en el directorio actual o en las rutas especificadas.

**Windows:**

Usa `sai.exe` (o `.\sai.exe` si no está en el PATH).

1.  **Especificar archivos `.ass` individuales:**
    ```cmd
    :: Procesar un solo archivo por ruta absoluta
    sai mux C:\Ruta\Completa\a\tu\subtitulo_ep01.ass

    :: Procesar múltiples archivos relativos
    sai mux subs\ep01.ass subs\ep03.ass
    ```

2.  **Especificar un rango de episodios:**
    Busca archivos en el directorio actual que coincidan con el patrón (ej. `*_01.ass` a `*_05.ass`).
    ```cmd
    sai mux 01-05
    ```

3.  **Especificar un número de episodio único:**
    Busca el archivo correspondiente en el directorio actual (ej. `*_04.ass`).
    ```cmd
    sai mux 04
    ```

4.  **Combinación:**
    ```cmd
    sai mux 01-03 C:\ruta\a\otro\subtitulo_extra.ass
    ```

---

**Linux / macOS:**

Usa `sai` (o `./sai` si no está en el PATH).

1.  **Especificar archivos `.ass` individuales:**
    ```bash
    # Procesar un solo archivo por ruta absoluta
    ./sai mux /ruta/completa/a/tu/subtitulo_ep01.ass

    # Procesar múltiples archivos relativos
    ./sai mux subs/ep01.ass subs/ep03.ass
    ```

2.  **Especificar un rango de episodios:**
    Busca archivos en el directorio actual que coincidan con el patrón (ej. `*_01.ass` a `*_05.ass`).
    ```bash
    ./sai mux 01-05
    ```

3.  **Especificar un número de episodio único:**
    Busca el archivo correspondiente en el directorio actual (ej. `*_04.ass`).
    ```bash
    ./sai mux 04
    ```

4.  **Combinación:**
    ```bash
    ./sai mux 01-03 /ruta/a/otro/subtitulo_extra.ass
    ```

---

**Comportamiento General:**

La herramienta procesará cada archivo `.ass` identificado, generando un archivo MKV correspondiente según las instrucciones `SAI` encontradas dentro de cada `.ass`. Los archivos MKV de salida se crearán generalmente en el directorio donde se ejecuta la herramienta, a menos que la directiva `{:outputName}` especifique una ruta diferente dentro del archivo `.ass`.



## Formato del Archivo `.ass` de Entrada

El archivo `.ass` de entrada debe contener líneas de **`Comment:`** (Comentario) con el campo **`Name`** (Actor) establecido en **`SAI`**. El campo *`Text`* de estos comentarios especiales se utiliza para dar instrucciones al `sai-muxer`.

Estas líneas **NO** se incluirán en el archivo de subtítulos final del MKV (excepto las de `{:chapter ...}` que se usan para generar metadatos de capítulo), pero son esenciales para que la herramienta funcione.

Aquí está el desglose de las directivas `SAI` esperadas:

---

### 1. Definición de Archivo de Video Principal

Especifica el archivo de video fuente.

*   **Formato:**
    ```ass
    Comment: 0,0:00:00.00,0:00:00.00,Default,SAI,0,0,0,,{:video <lang>}{<track_name>}<ruta_al_video>
    ```
*   **Parámetros:**
    *   `<lang>`: Código de idioma ISO 639-2 (ej. `jpn`, `eng`, `und`). Se asignará a la pista de video en el MKV.
    *   `{<track_name>}` (Opcional): Nombre descriptivo para la pista de video (ej. `{RAW}`). Si se omite, no se asigna nombre.
    *   `<ruta_al_video>`: Ruta al archivo de video. Puede ser absoluta o relativa al directorio donde se encuentra el archivo `.ass`.
*   **Ejemplo:**
    ```ass
    Comment: 0,0:00:00.00,0:00:00.00,Default,SAI,0,0,0,,{:video jpn}{AMZN}E:\Torrents\Video.mkv
    Comment: 0,0:00:00.00,0:00:00.00,Default,SAI,0,0,0,,{:video und}../Video/Episodio_01.mp4
    ```
*   **Obligatorio:** Sí, se necesita al menos un archivo de video.

---

### 2. Definición de Archivo de Audio Principal

Especifica el archivo de audio principal. Si el audio está dentro del archivo de video, puedes especificar el mismo archivo que en `{:video}` o usar un archivo de audio separado.

*   **Formato:**
    ```ass
    Comment: 0,0:00:00.00,0:00:00.00,Default,SAI,0,0,0,,{:audio <lang>}{<track_name>}<ruta_al_audio>
    ```
*   **Parámetros:**
    *   `<lang>`: Código de idioma ISO 639-2 (ej. `jpn`, `esl`, `eng`). Se asignará a la pista de audio.
    *   `{<track_name>}` (Opcional): Nombre descriptivo para la pista de audio (ej. `{Japonés}`).
    *   `<ruta_al_audio>`: Ruta al archivo de audio. Puede ser absoluta o relativa al directorio del `.ass`. Si es la misma que la del video, el script intentará seleccionar la pista de audio de ese archivo.
*   **Ejemplo:**
    ```ass
    Comment: 0,0:00:00.00,0:00:00.00,Default,SAI,0,0,0,,{:audio jpn}{Japonés}E:\Torrents\Video.mkv
    Comment: 0,0:00:00.00,0:00:00.00,Default,SAI,0,0,0,,{:audio eng}Audio/Track_ENG.aac
    ```
*   **Obligatorio:** Sí, se necesita al menos una fuente de audio (puede ser el mismo archivo que el video).

---

### 3. Definición de Idioma y Nombre del Subtítulo Principal

Establece el idioma y nombre para la pista de subtítulos que se está procesando (el propio archivo `.ass`).

*   **Formato:**
    ```ass
    Comment: 0,0:00:00.00,0:00:00.00,Default,SAI,0,0,0,,{:subLang}{<track_name>}<lang_code>
    ```
*   **Parámetros:**
    *   `{<track_name>}` (Opcional): Nombre descriptivo para la pista de subtítulos (ej. `{Mi Fansub}`).
    *   `<lang_code>`: Código de idioma ISO 639-2 (ej. `spa`, `esl`, `es-419`, `eng`).
*   **Ejemplo:**
    ```ass
    Comment: 0,0:00:00.00,0:00:00.00,Default,SAI,0,0,0,,{:subLang}{PCNet no Fansub}es-419
    Comment: 0,0:00:00.00,0:00:00.00,Default,SAI,0,0,0,,{:subLang}eng
    ```
*   **Obligatorio:** Sí, para identificar correctamente la pista de subtítulos principal.

---

### 4. Definición de Nombre del Archivo de Salida

Especifica el nombre del archivo MKV final.

*   **Formato:**
    ```ass
    Comment: 0,0:00:00.00,0:00:00.00,Default,SAI,0,0,0,,{:outputName}{[<prefix>] }<nombre_archivo.mkv>
    ```
*   **Parámetros:**
    *   `{[<prefix>] }` (Opcional): Un prefijo opcional entre corchetes (ej. `{[MiGrupo] }`). Se añadirá al principio del nombre.
    *   `<nombre_archivo.mkv>`: El nombre deseado para el archivo MKV resultante.
*   **Ejemplo:**
    ```ass
    Comment: 0,0:00:00.00,0:00:00.00,Default,SAI,0,0,0,,{:outputName}[PCNet] Serie - 01 (WEB 1080p).mkv
    Comment: 0,0:00:00.00,0:00:00.00,Default,SAI,0,0,0,,{:outputName}Episodio_Final.mkv
    ```
*   **Obligatorio:** Sí, para saber cómo nombrar el archivo final.

---

### 5. Definición del Directorio de Fuentes

Especifica la carpeta que contiene los archivos de fuentes (`.ttf`, `.otf`, `.ttc`) necesarios para los subtítulos. La herramienta buscará *dentro* de esta carpeta las fuentes usadas en el archivo `.ass` (tanto en estilos como en tags `\fn`) y las adjuntará al MKV.

*   **Formato:**
    ```ass
    Comment: 0,0:00:00.00,0:00:00.00,Default,SAI,0,0,0,,{:fonts}<ruta_a_la_carpeta_fonts>
    ```
*   **Parámetros:**
    *   `<ruta_a_la_carpeta_fonts>`: Ruta a la carpeta que contiene los archivos de fuentes. Puede ser absoluta o relativa al directorio del `.ass`.
*   **Ejemplo:**
    ```ass
    Comment: 0,0:00:00.00,0:00:00.00,Default,SAI,0,0,0,,{:fonts}Fonts
    Comment: 0,0:00:00.00,0:00:00.00,Default,SAI,0,0,0,,{:fonts}../Shared/Project_Fonts
    ```
*   **Obligatorio:** No, pero muy recomendable si tus subtítulos usan fuentes personalizadas. Si no se especifica, no se adjuntarán fuentes.

---

### 6. Definición de Capítulos

Define los puntos de capítulo para el MKV. Estos se extraen y se guardan en un archivo XML temporal que `mkvmerge` utiliza.

*   **Formato:**
    ```ass
    Comment: 0,<timestamp_inicio>,<timestamp_fin>,Default,SAI,0,0,0,,{:chapter <lang>}<nombre_capitulo>
    ```
*   **Parámetros:**
    *   `<timestamp_inicio>`: El tiempo exacto (ej. `0:01:20.98`) donde comienza el capítulo. El timestamp de fin no se usa para capítulos.
    *   `<lang>`: Código de idioma ISO 639-2 para el nombre del capítulo (ej. `es`, `en`). Si se omite, se usa `und` (indefinido).
    *   `<nombre_capitulo>`: El texto que aparecerá como nombre del capítulo.
*   **Ejemplo:**
    ```ass
    Comment: 0,0:01:20.98,0:01:20.98,Default,SAI,0,0,0,,{:chapter es}Opening - Tema Principal
    Comment: 0,0:22:00.01,0:22:00.01,Default,SAI,0,0,0,,{:chapter en}Ending Theme
    ```
*   **Obligatorio:** No. Si no hay líneas `{:chapter ...}`, el MKV no tendrá capítulos.
*   **Nota Importante:** Las definiciones de capítulos solo se leen del archivo `.ass` principal, no de los archivos incluidos mediante `{:insert}`.

---

### 7. Inclusión de Otros Archivos `.ass` (Inserts)

Permite incluir el contenido (eventos y estilos) de otro archivo `.ass` dentro del principal. Útil para KFX, openings/endings separados, etc.

*   **Formato:**
    ```ass
    Comment: 0,<timestamp_inicio>,<timestamp_fin>,Default,SAI,0,0,0,,{:insert}<ruta_al_otro_ass>
    ```
*   **Parámetros:**
    *   `<timestamp_inicio>`: El tiempo en el archivo *principal* donde debe comenzar el contenido del archivo insertado. Los tiempos dentro del archivo insertado se ajustarán automáticamente basándose en la diferencia entre este timestamp y el de la primera línea del archivo insertado.
    *   `<ruta_al_otro_ass>`: Ruta al archivo `.ass` a incluir. Puede ser absoluta o relativa al directorio del archivo `.ass` *que contiene la línea `{:insert}`*.
*   **Ejemplo:**
    ```ass
    Comment: 0,0:01:20.98,0:01:20.98,Default,SAI,0,0,0,,{:insert}OP_ED/Opening_FX.ass
    Comment: 0,0:22:00.01,0:22:00.01,Default,SAI,0,0,0,,{:insert}../Common/Ending_v2.ass
    ```
*   **Obligatorio:** No.
*   **Notas:**
    *   Se pueden anidar inserciones (un archivo insertado puede insertar otro), pero se detectan y previenen las inserciones recursivas (A inserta B, B inserta A).
    *   Las líneas de `Comment` con `Actor=SAI` dentro de los archivos insertados son ignoradas (no se procesan como comandos ni se incluyen en la salida final), *excepto* por las líneas `{:chapter ...}` que sí se copian al `.ass` temporal, pero no generan capítulos.
    *   Se mezclan los estilos (`[V4+ Styles]`) y eventos (`[Events]`) del archivo insertado con los del principal.

---

### 8. Archivos de Audio Adicionales

Permite incluir pistas de audio adicionales (doblajes, comentarios, etc.).

*   **Formato:**
    ```ass
    Comment: 0,0:00:00.00,0:00:00.00,Default,SAI,0,0,0,,{:extraAudio <lang>}{<track_name>}<ruta_al_audio>
    ```
*   **Parámetros:** Igual que `{:audio}`.
*   **Ejemplo:**
    ```ass
    Comment: 0,0:00:00.00,0:00:00.00,Default,SAI,0,0,0,,{:extraAudio esl}{Doblaje Latino}Audio/Track_ESL.ac3
    Comment: 0,0:00:00.00,0:00:00.00,Default,SAI,0,0,0,,{:extraAudio jpn}{Comentarios Staff}Extras/Commentary.ogg
    ```
*   **Obligatorio:** No. Puedes añadir tantos como necesites.

---

### 9. Archivos de Subtítulos Adicionales

Permite incluir otras pistas de subtítulos (otros idiomas, subtítulos forzados, etc.).

*   **Formato:**
    ```ass
    Comment: 0,0:00:00.00,0:00:00.00,Default,SAI,0,0,0,,{:extraSubs <lang>}{<track_name>}<ruta_al_subtitulo>
    ```
*   **Parámetros:**
    *   `<lang>`: Código de idioma ISO 639-2.
    *   `{<track_name>}` (Opcional): Nombre descriptivo.
    *   `<ruta_al_subtitulo>`: Ruta al archivo de subtítulos (puede ser `.ass`, `.srt`, etc., lo que `mkvmerge` soporte). Absoluta o relativa al directorio del `.ass` principal.
*   **Ejemplo:**
    ```ass
    Comment: 0,0:00:00.00,0:00:00.00,Default,SAI,0,0,0,,{:extraSubs eng}{Official Subs}Subs/English.srt
    Comment: 0,0:00:00.00,0:00:00.00,Default,SAI,0,0,0,,{:extraSubs spa}{Forzados}Signs_Forced.ass
    ```
*   **Obligatorio:** No. Puedes añadir tantos como necesites.

---

## Integración con Aegisub (Script Lua Opcional)

Para facilitar la creación de las líneas de configuración `SAI` y ejecutar el multiplexado directamente desde Aegisub, se proporciona un script Lua opcional. Este script añade nuevas opciones al menú "Automatización" de Aegisub.

### Instalación del Script Lua

1.  Descarga el archivo `.lua` (`cif.SAIMuxer.lua`).
2.  Abre Aegisub.
3.  Ve al menú `Archivo` -> `Abrir carpeta de scripts de Aegisub`.
4.  Se abrirá una carpeta en tu explorador de archivos. Entra en la subcarpeta `automation`, y dentro de ella, en la carpeta `autoload`.
5.  Copia o mueve el archivo `.lua` descargado a esta carpeta `automation/autoload`.
6.  Reinicia Aegisub. El script debería cargarse automáticamente.

### Configuración Inicial (¡Importante!)

Antes de poder usar las funciones del script, **debes** indicarle dónde se encuentran los ejecutables de `SAI` y `mkvmerge`.

1.  Abre Aegisub (con el script ya instalado).
2.  Ve al menú `Automatización` -> `SAI Muxer` -> `Configuración`.
3.  Aparecerá una ventana:
    *   **Ruta a sai.exe:** Escribe la ruta completa al ejecutable `sai.exe` (o `sai` en Linux), o usa el botón `SAI...` para buscarlo.
    *   **Ruta a mkvmerge.exe:** Escribe la ruta completa al ejecutable `mkvmerge.exe` (o `mkvmerge`), o usa el botón `MKVMerge...` para buscarlo.
4.  Haz clic en `Guardar`. La configuración se guardará en un archivo llamado `sai_config.conf` dentro de tu carpeta de usuario de Aegisub (`?user`).

### Uso de las Macros en Aegisub

Una vez instalado y configurado, el script añade las siguientes opciones al menú `Automatización` -> `SAI Muxer`:

1.  **`Agregar líneas de pistas`**:
    *   **Propósito:** Abre una interfaz gráfica para generar las líneas de comentario `SAI` básicas (`{:video ...}`, `{:audio ...}`, `{:subLang ...}`, `{:fonts ...}`, `{:outputName ...}`, `{:extraAudio ...}`, `{:extraSubs ...}`, `{:insert ...}`).
    *   **Cómo funciona:** Selecciona las casillas de las pistas que quieres definir, elige idiomas y escribe nombres de pista opcionales. Puedes añadir campos para pistas de audio/subtítulos extra usando los botones `Pista audio +1` / `Pista subs +1`. Al hacer clic en `Agregar`, el script insertará las líneas `Comment:` correspondientes (con `Actor=SAI`) al **principio** de tu archivo de subtítulos.
    *   **Nota:** Este comando *no* te pide las rutas a los archivos; solo genera la estructura de la línea. Necesitarás usar "Agregar rutas..." después, o añadirlas manualmente. Por defecto, para `{:fonts}` añadirá `{:fonts}Fonts`.

2.  **`Agregar líneas de capítulos`**:
    *   **Propósito:** Abre una interfaz para generar las líneas `{:chapter ...}`.
    *   **Cómo funciona:** Similar al anterior, puedes añadir más campos de capítulo con el botón `Capítulo +1`. Selecciona el idioma y escribe el nombre de cada capítulo. Al hacer clic en `Agregar`, insertará las líneas `Comment:` al **principio** del archivo.
    *   **Nota Importante:** Las líneas de capítulo se insertarán con `Start Time` y `End Time` en `0:00:00.00`. **Deberás seleccionar manualmente cada línea de capítulo generada y ajustar su `Start Time` en Aegisub** al tiempo correcto donde debe comenzar ese capítulo.

3.  **`Agregar rutas en líneas de pistas...`**:
    *   **Propósito:** Ayuda a rellenar las rutas de archivo/carpeta en las líneas `SAI` que *ya existen* en tu script.
    *   **Cómo funciona:** Ejecuta esta macro. El script buscará líneas `Comment:` con `Actor=SAI` que contengan etiquetas como `{:video ...}`, `{:audio ...}`, `{:fonts ...}`, `{:insert ...}`, etc. Para cada línea encontrada, abrirá un diálogo de selección de archivo (o carpeta para `{:fonts}`) apropiado para esa etiqueta. Selecciona el archivo/carpeta correspondiente. El script modificará la línea existente, añadiendo la ruta seleccionada al final.

4.  **`Multiplexar`**:
    *   **Propósito:** Ejecuta el comando `sai mux` utilizando el archivo de subtítulos *actualmente abierto y guardado* en Aegisub.
    *   **¡Prerrequisitos Cruciales!**
        *   La configuración de rutas (paso inicial) debe estar hecha.
        *   El archivo `.ass` actual **debe estar guardado**.
        *   Todas las líneas `SAI` necesarias (video, audio, subLang, outputName, etc.) deben estar presentes en el archivo `.ass`.
        *   Todas las rutas en esas líneas `SAI` deben ser correctas y apuntar a archivos/carpetas existentes.
    *   **Cómo funciona:** Lee la configuración, construye el comando `sai mux "tu_archivo.ass"` y lo ejecuta.
    *   **Salida y Errores:** El progreso y cualquier mensaje de error de `mkvmerge` o `sai` se mostrarán en la ventana de **Registro de Depuración** de Aegisub (puedes abrirla con `Ctrl+Shift+L` o desde el menú `Ver`). ¡Revisa esta ventana si algo falla!

5.  **`Configuración`**:
    *   Abre la ventana de configuración descrita en el paso inicial para cambiar las rutas de `sai` o `mkvmerge`.

### Flujo de Trabajo Típico con el Script Lua

1.  **Configurar:** (Solo la primera vez) Usa `SAI Muxer -> Configuración` para indicar las rutas de `sai` y `mkvmerge`.
2.  **Abrir/Crear Subtítulo:** Trabaja en tu archivo `.ass` en Aegisub.
3.  **Agregar Líneas Base:** Usa `SAI Muxer -> Agregar líneas de pistas` para generar la estructura inicial de configuración `SAI` al principio del archivo.
4.  **Agregar Capítulos (Opcional):** Usa `SAI Muxer -> Agregar líneas de capítulos`.
5.  **Ajustar Tiempos de Capítulos:** **Manualmente**, selecciona cada línea `{:chapter ...}` y ajusta su `Start Time` en Aegisub.
6.  **Guardar el Archivo:** Guarda tu archivo `.ass`.
7.  **Agregar Rutas:** Usa `SAI Muxer -> Agregar rutas en líneas de pistas...` para seleccionar los archivos/carpetas correspondientes a cada línea `SAI` generada.
8.  **Revisar y Guardar:** Asegúrate de que todas las líneas `SAI` y sus rutas son correctas. Guarda el archivo `.ass` **nuevamente**.
9.  **Multiplexar:** Usa `SAI Muxer -> Multiplexar`.
10. **Verificar Log:** Revisa la ventana de Registro de Depuración de Aegisub (`Ctrl+Shift+L`) para ver si el proceso fue exitoso o si hubo errores.
