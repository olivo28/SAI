# Language

[🇪🇸 Español](README.md) | 🇬🇧 **English**


# SAI - MKV Muxing Automator from ASS

This tool automates the process of creating MKV files using `mkvmerge`. It reads a specially formatted `.ass` subtitle file that contains not only the subtitles but also metadata about the video, audio, fonts, chapters, and the desired output filename.

## Prerequisites

1.  **`mkvmerge`**: You must have `mkvmerge` (part of MKVToolNix) installed and accessible in your PATH, or place `mkvmerge.exe` (or `mkvmerge` on Linux/macOS) in the same directory as the `SAI` executable.

## Usage

Run the tool from the command line, providing the files or episode ranges to process. The tool will look for the corresponding `.ass` files in the current directory or the specified paths.

**Windows:**

Use `sai.exe` (or `.\sai.exe` if not in PATH).

1.  **Specify individual `.ass` files:**
    ```cmd
    :: Process a single file by absolute path
    sai mux C:\Full\Path\to\your\subtitle_ep01.ass

    :: Process multiple relative files
    sai mux subs\ep01.ass subs\ep03.ass
    ```

2.  **Specify an episode range:**
    Searches the current directory for files matching the pattern (e.g., `*_01.ass` to `*_05.ass`).
    ```cmd
    sai mux 01-05
    ```

3.  **Specify a single episode number:**
    Searches the current directory for the corresponding file (e.g., `*_04.ass`).
    ```cmd
    sai mux 04
    ```

4.  **Combination:**
    ```cmd
    sai mux 01-03 C:\path\to\other\extra_subtitle.ass
    ```

---

**Linux / macOS:**

Use `sai` (or `./sai` if not in PATH).

1.  **Specify individual `.ass` files:**
    ```bash
    # Process a single file by absolute path
    ./sai mux /full/path/to/your/subtitle_ep01.ass

    # Process multiple relative files
    ./sai mux subs/ep01.ass subs/ep03.ass
    ```

2.  **Specify an episode range:**
    Searches the current directory for files matching the pattern (e.g., `*_01.ass` to `*_05.ass`).
    ```bash
    ./sai mux 01-05
    ```

3.  **Specify a single episode number:**
    Searches the current directory for the corresponding file (e.g., `*_04.ass`).
    ```bash
    ./sai mux 04
    ```

4.  **Combination:**
    ```bash
    ./sai mux 01-03 /path/to/other/extra_subtitle.ass
    ```

---

**General Behavior:**

The tool will process each identified `.ass` file, generating a corresponding MKV file according to the `SAI` instructions found within each `.ass`. The output MKV files will generally be created in the directory where the tool is run, unless the `{:outputName}` directive specifies a different path within the `.ass` file.

## Input `.ass` File Format

The input `.ass` file must contain **`Comment:`** lines with the **`Name`** (Actor) field set to **`SAI`**. The *`Text`* field of these special comments is used to give instructions to the `sai-muxer`.

These lines will **NOT** be included in the final MKV subtitle track (except for `{:chapter ...}` lines, which are used to generate chapter metadata), but they are essential for the tool to work.

Here is the breakdown of the expected `SAI` directives:

---

### 1. Main Video File Definition

Specifies the source video file.

*   **Format:**
    ```ass
    Comment: 0,0:00:00.00,0:00:00.00,Default,SAI,0,0,0,,{:video <lang>}{<track_name>}<path_to_video>
    ```
*   **Parameters:**
    *   `<lang>`: ISO 639-2 language code (e.g., `jpn`, `eng`, `und`). Will be assigned to the video track in the MKV.
    *   `{<track_name>}` (Optional): Descriptive name for the video track (e.g., `{RAW}`). If omitted, no name is assigned.
    *   `<path_to_video>`: Path to the video file. Can be absolute or relative to the directory where the `.ass` file is located.
*   **Example:**
    ```ass
    Comment: 0,0:00:00.00,0:00:00.00,Default,SAI,0,0,0,,{:video jpn}{AMZN}E:\Torrents\Video.mkv
    Comment: 0,0:00:00.00,0:00:00.00,Default,SAI,0,0,0,,{:video und}../Video/Episode_01.mp4
    ```
*   **Required:** Yes, at least one video file is needed.

---

### 2. Main Audio File Definition

Specifies the main audio file. If the audio is within the video file, you can specify the same file as in `{:video}` or use a separate audio file.

*   **Format:**
    ```ass
    Comment: 0,0:00:00.00,0:00:00.00,Default,SAI,0,0,0,,{:audio <lang>}{<track_name>}<path_to_audio>
    ```
*   **Parameters:**
    *   `<lang>`: ISO 639-2 language code (e.g., `jpn`, `esl`, `eng`). Will be assigned to the audio track.
    *   `{<track_name>}` (Optional): Descriptive name for the audio track (e.g., `{Japanese}`).
    *   `<path_to_audio>`: Path to the audio file. Can be absolute or relative to the `.ass` directory. If it's the same as the video file, the script will attempt to select the audio track from that file.
*   **Example:**
    ```ass
    Comment: 0,0:00:00.00,0:00:00.00,Default,SAI,0,0,0,,{:audio jpn}{Japanese}E:\Torrents\Video.mkv
    Comment: 0,0:00:00.00,0:00:00.00,Default,SAI,0,0,0,,{:audio eng}Audio/Track_ENG.aac
    ```
*   **Required:** Yes, at least one audio source is needed (can be the same file as the video).

---

### 3. Main Subtitle Language and Name Definition

Sets the language and name for the subtitle track being processed (the `.ass` file itself).

*   **Format:**
    ```ass
    Comment: 0,0:00:00.00,0:00:00.00,Default,SAI,0,0,0,,{:subLang}{<track_name>}<lang_code>
    ```
*   **Parameters:**
    *   `{<track_name>}` (Optional): Descriptive name for the subtitle track (e.g., `{My Fansub}`).
    *   `<lang_code>`: ISO 639-2 language code (e.g., `spa`, `esl`, `es-419`, `eng`).
*   **Example:**
    ```ass
    Comment: 0,0:00:00.00,0:00:00.00,Default,SAI,0,0,0,,{:subLang}{PCNet no Fansub}es-419
    Comment: 0,0:00:00.00,0:00:00.00,Default,SAI,0,0,0,,{:subLang}eng
    ```
*   **Required:** Yes, to correctly identify the main subtitle track.

---

### 4. Output Filename Definition

Specifies the name of the final MKV file.

*   **Format:**
    ```ass
    Comment: 0,0:00:00.00,0:00:00.00,Default,SAI,0,0,0,,{:outputName}{[<prefix>] }<filename.mkv>
    ```
*   **Parameters:**
    *   `{[<prefix>] }` (Optional): An optional prefix in brackets (e.g., `{[MyGroup] }`). It will be added to the beginning of the name.
    *   `<filename.mkv>`: The desired name for the resulting MKV file.
*   **Example:**
    ```ass
    Comment: 0,0:00:00.00,0:00:00.00,Default,SAI,0,0,0,,{:outputName}[PCNet] Series - 01 (WEB 1080p).mkv
    Comment: 0,0:00:00.00,0:00:00.00,Default,SAI,0,0,0,,{:outputName}Episode_Final.mkv
    ```
*   **Required:** Yes, to know how to name the final file.

---

### 5. Fonts Directory Definition

Specifies the folder containing the font files (`.ttf`, `.otf`, `.ttc`) required for the subtitles. The tool will search *inside* this folder for the fonts used in the `.ass` file (in both styles and `\fn` tags) and attach them to the MKV.

*   **Format:**
    ```ass
    Comment: 0,0:00:00.00,0:00:00.00,Default,SAI,0,0,0,,{:fonts}<path_to_fonts_folder>
    ```
*   **Parameters:**
    *   `<path_to_fonts_folder>`: Path to the folder containing the font files. Can be absolute or relative to the `.ass` directory.
*   **Example:**
    ```ass
    Comment: 0,0:00:00.00,0:00:00.00,Default,SAI,0,0,0,,{:fonts}Fonts
    Comment: 0,0:00:00.00,0:00:00.00,Default,SAI,0,0,0,,{:fonts}../Shared/Project_Fonts
    ```
*   **Required:** No, but highly recommended if your subtitles use custom fonts. If not specified, no fonts will be attached.

---

### 6. Chapter Definition

Defines chapter points for the MKV. These are extracted and saved to a temporary XML file that `mkvmerge` uses.

*   **Format:**
    ```ass
    Comment: 0,<start_timestamp>,<end_timestamp>,Default,SAI,0,0,0,,{:chapter <lang>}<chapter_name>
    ```
*   **Parameters:**
    *   `<start_timestamp>`: The exact time (e.g., `0:01:20.98`) where the chapter begins. The end timestamp is not used for chapters.
    *   `<lang>`: ISO 639-2 language code for the chapter name (e.g., `es`, `en`). If omitted, `und` (undefined) is used.
    *   `<chapter_name>`: The text that will appear as the chapter name.
*   **Example:**
    ```ass
    Comment: 0,0:01:20.98,0:01:20.98,Default,SAI,0,0,0,,{:chapter es}Opening - Main Theme
    Comment: 0,0:22:00.01,0:22:00.01,Default,SAI,0,0,0,,{:chapter en}Ending Theme
    ```
*   **Required:** No. If there are no `{:chapter ...}` lines, the MKV will not have chapters.
*   **Important Note:** Chapter definitions are only read from the main `.ass` file, not from files included via `{:insert}`.

---

### 7. Inclusion of Other `.ass` Files (Inserts)

Allows including the content (events and styles) of another `.ass` file within the main one. Useful for KFX, separate openings/endings, etc.

*   **Format:**
    ```ass
    Comment: 0,<start_timestamp>,<end_timestamp>,Default,SAI,0,0,0,,{:insert}<path_to_other_ass>
    ```
*   **Parameters:**
    *   `<start_timestamp>`: The time in the *main* file where the content of the inserted file should begin. Timestamps within the inserted file will be automatically adjusted based on the difference between this timestamp and the first line's timestamp in the inserted file.
    *   `<path_to_other_ass>`: Path to the `.ass` file to include. Can be absolute or relative to the directory of the `.ass` file *containing the `{:insert}` line*.
*   **Example:**
    ```ass
    Comment: 0,0:01:20.98,0:01:20.98,Default,SAI,0,0,0,,{:insert}OP_ED/Opening_FX.ass
    Comment: 0,0:22:00.01,0:22:00.01,Default,SAI,0,0,0,,{:insert}../Common/Ending_v2.ass
    ```
*   **Required:** No.
*   **Notes:**
    *   Inserts can be nested (an inserted file can insert another), but recursive inserts (A inserts B, B inserts A) are detected and prevented.
    *   `Comment` lines with `Actor=SAI` within inserted files are ignored (not processed as commands nor included in the final output), *except* for `{:chapter ...}` lines, which are copied to the temporary `.ass` but do not generate chapters.
    *   Styles (`[V4+ Styles]`) and events (`[Events]`) from the inserted file are merged with those of the main file.

---

### 8. Additional Audio Files

Allows including additional audio tracks (dubs, commentaries, etc.).

*   **Format:**
    ```ass
    Comment: 0,0:00:00.00,0:00:00.00,Default,SAI,0,0,0,,{:extraAudio <lang>}{<track_name>}<path_to_audio>
    ```
*   **Parameters:** Same as `{:audio}`.
*   **Example:**
    ```ass
    Comment: 0,0:00:00.00,0:00:00.00,Default,SAI,0,0,0,,{:extraAudio esl}{Latin Dub}Audio/Track_ESL.ac3
    Comment: 0,0:00:00.00,0:00:00.00,Default,SAI,0,0,0,,{:extraAudio jpn}{Staff Commentary}Extras/Commentary.ogg
    ```
*   **Required:** No. You can add as many as you need.

---

### 9. Additional Subtitle Files

Allows including other subtitle tracks (other languages, forced subtitles, etc.).

*   **Format:**
    ```ass
    Comment: 0,0:00:00.00,0:00:00.00,Default,SAI,0,0,0,,{:extraSubs <lang>}{<track_name>}<path_to_subtitle>
    ```
*   **Parameters:**
    *   `<lang>`: ISO 639-2 language code.
    *   `{<track_name>}` (Optional): Descriptive name.
    *   `<path_to_subtitle>`: Path to the subtitle file (can be `.ass`, `.srt`, etc., whatever `mkvmerge` supports). Absolute or relative to the main `.ass` directory.
*   **Example:**
    ```ass
    Comment: 0,0:00:00.00,0:00:00.00,Default,SAI,0,0,0,,{:extraSubs eng}{Official Subs}Subs/English.srt
    Comment: 0,0:00:00.00,0:00:00.00,Default,SAI,0,0,0,,{:extraSubs spa}{Forced}Signs_Forced.ass
    ```
*   **Required:** No. You can add as many as you need.

---

## Aegisub Integration (Optional Lua Script)

To facilitate the creation of `SAI` configuration lines and run the muxing process directly from Aegisub, an optional Lua script is provided. This script adds new options to Aegisub's "Automation" menu.

### Lua Script Installation

1.  Download the `.lua` file (`cif.SAIMuxer.lua`).
2.  Open Aegisub.
3.  Go to the `File` menu -> `Open Aegisub Scripts folder`.
4.  A folder will open in your file explorer. Enter the `automation` subfolder, and within that, the `autoload` folder.
5.  Copy or move the downloaded `.lua` file into this `automation/autoload` folder.
6.  Restart Aegisub. The script should load automatically.

### Initial Setup (Important!)

Before you can use the script's functions, you **must** tell it where the `SAI` and `mkvmerge` executables are located.

1.  Open Aegisub (with the script already installed).
2.  Go to the `Automation` menu -> `SAI Muxer` -> `Configuración` (Configuration).
3.  A window will appear:
    *   **Ruta a sai.exe:** Enter the full path to the `sai.exe` (or `sai` on Linux) executable, or use the `SAI...` button to browse for it.
    *   **Ruta a mkvmerge.exe:** Enter the full path to the `mkvmerge.exe` (or `mkvmerge`) executable, or use the `MKVMerge...` button to browse for it.
4.  Click `Guardar` (Save). The configuration will be saved to a file named `sai_config.conf` inside your Aegisub user folder (`?user`).

### Using the Macros in Aegisub

Once installed and configured, the script adds the following options to the `Automation` -> `SAI Muxer` menu:

1.  **`Agregar líneas de pistas` (Add Track Lines)**:
    *   **Purpose:** Opens a graphical interface to generate the basic `SAI` comment lines (`{:video ...}`, `{:audio ...}`, `{:subLang ...}`, `{:fonts ...}`, `{:outputName ...}`, `{:extraAudio ...}`, `{:extraSubs ...}`, `{:insert ...}`).
    *   **How it works:** Check the boxes for the tracks you want to define, choose languages, and enter optional track names. You can add fields for extra audio/subtitle tracks using the `Pista audio +1` / `Pista subs +1` buttons. Clicking `Agregar` (Add) will insert the corresponding `Comment:` lines (with `Actor=SAI`) at the **beginning** of your subtitle file.
    *   **Note:** This command does *not* ask for file paths; it only generates the line structure. You will need to use "Add Paths..." afterwards or add them manually. By default, for `{:fonts}` it will add `{:fonts}Fonts`.

2.  **`Agregar líneas de capítulos` (Add Chapter Lines)**:
    *   **Purpose:** Opens an interface to generate `{:chapter ...}` lines.
    *   **How it works:** Similar to the previous one, you can add more chapter fields with the `Capítulo +1` button. Select the language and enter the name for each chapter. Clicking `Agregar` (Add) will insert the `Comment:` lines at the **beginning** of the file.
    *   **Important Note:** Chapter lines will be inserted with `Start Time` and `End Time` set to `0:00:00.00`. **You must manually select each generated chapter line and adjust its `Start Time` in Aegisub** to the correct time where that chapter should begin.

3.  **`Agregar rutas en líneas de pistas...` (Add Paths to Track Lines...)**:
    *   **Purpose:** Helps fill in the file/folder paths in the `SAI` lines that *already exist* in your script.
    *   **How it works:** Run this macro. The script will look for `Comment:` lines with `Actor=SAI` containing tags like `{:video ...}`, `{:audio ...}`, `{:fonts ...}`, `{:insert ...}`, etc. For each line found, it will open an appropriate file (or folder for `{:fonts}`) selection dialog for that tag. Select the corresponding file/folder. The script will modify the existing line, adding the selected path at the end.

4.  **`Multiplexar` (Multiplex)**:
    *   **Purpose:** Executes the `sai mux` command using the subtitle file *currently open and saved* in Aegisub.
    *   **Crucial Prerequisites!**
        *   The path configuration (initial setup) must be done.
        *   The current `.ass` file **must be saved**.
        *   All necessary `SAI` lines (video, audio, subLang, outputName, etc.) must be present in the `.ass` file.
        *   All paths in those `SAI` lines must be correct and point to existing files/folders.
    *   **How it works:** Reads the configuration, builds the `sai mux "your_file.ass"` command, and executes it.
    *   **Output and Errors:** Progress and any error messages from `mkvmerge` or `sai` will be displayed in Aegisub's **Debugging Log** window (you can open it with `Ctrl+Shift+L` or from the `View` menu). Check this window if something fails!

5.  **`Configuración` (Configuration)**:
    *   Opens the configuration window described in the initial setup step to change the paths for `sai` or `mkvmerge`.

### Typical Workflow with the Lua Script

1.  **Configure:** (First time only) Use `SAI Muxer -> Configuración` to set the paths for `sai` and `mkvmerge`.
2.  **Open/Create Subtitle:** Work on your `.ass` file in Aegisub.
3.  **Add Base Lines:** Use `SAI Muxer -> Agregar líneas de pistas` to generate the initial `SAI` configuration structure at the beginning of the file.
4.  **Add Chapters (Optional):** Use `SAI Muxer -> Agregar líneas de capítulos`.
5.  **Adjust Chapter Times:** **Manually**, select each `{:chapter ...}` line and adjust its `Start Time` in Aegisub.
6.  **Save the File:** Save your `.ass` file.
7.  **Add Paths:** Use `SAI Muxer -> Agregar rutas en líneas de pistas...` to select the corresponding files/folders for each generated `SAI` line.
8.  **Review and Save:** Ensure all `SAI` lines and their paths are correct. Save the `.ass` file **again**.
9.  **Multiplex:** Use `SAI Muxer -> Multiplexar`.
10. **Check Log:** Review the Aegisub Debugging Log window (`Ctrl+Shift+L`) to see if the process was successful or if there were errors.