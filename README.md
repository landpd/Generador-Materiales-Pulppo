### ⚙️ Generador Automático de Assets Inmobiliarios

**1. Descripción general del sistema**

Se trata de un pipeline de automatización desarrollado en Python (ejecutado en Google Colab) que genera piezas gráficas (imágenes estáticas) a partir de datos tabulares (CSV) para promocionar propiedades inmobiliarias. El sistema separa estrictamente la capa de lógica de datos (Python) de la capa de presentación (HTML/CSS inyectado dinámicamente) y utiliza un navegador Chromium en modo *headless* para renderizar los diseños.

**2. Stack tecnológico y librerías core**
*   **Entorno:** Google Colab (con Google Drive montado como sistema de archivos principal).
*   **Ingesta y Procesamiento:** `pandas` (lectura de CSV), `re` (extracción Regex de URLs y limpieza de strings), `requests` y `base64` (codificación de imágenes remotas y locales).
*   **Renderizado:** `html2image` (captura de pantalla del DOM renderizado por Google Chrome).
*   **Sistema de Archivos:** `os`, `glob`, `shutil` (manejo de rutas, creación recursiva de directorios y movimiento de archivos).

**3. Arquitectura de módulos**

El sistema se divide en dos archivos principales:
*   **`mis_plantillas.py` (Capa de Presentación):** Contiene funciones de Python que reciben un diccionario (`datos_propiedad`) y retornan un *f-string* con código HTML5 y CSS3 embebido. Los diseños utilizan posicionamiento absoluto (`position: absolute`) para maquetar los elementos de forma precisa.
*   **Celda B / Script Principal (Capa de Control):** Maneja el flujo de ejecución, extracción, limpieza de datos, y el motor iterativo de renderizado.

**4. Flujo lógico de ejecución (Pipeline)**
1.  **Lectura de datos:** busca el CSV más reciente bajo el patrón `propiedades_1_5_10_con_fotos_*.csv`.
2.  **Iteración de registros:** procesa cada fila del DataFrame (omitiendo valores nulos).
3.  **Procesamiento de assets (imágenes y logos):**
    *   Extrae hasta 6 URLs de fotografías de una cadena mediante expresiones regulares.
    *   Descarga las imágenes web y las codifica en Base64.
    *   Busca localmente un logotipo correspondiente al nombre de la empresa (`[Company Name]_imagotipo_colab_negro.png`). Detecta su MIME type y lo codifica en Base64.
4.  **Limpieza y formateo (reglas de negocio):**
    *   *Dirección:* extrae solo el string anterior a la primera coma para la calle.
    *   *Operación:* formatea a *Capitalize* (ej. 'Venta', 'Renta') y construye el string concatenado (ej. "Casa en condominio en Venta").
    *   *Moneda:* formatea el precio con separadores de miles y adjunta la moneda (ej. "$1,500,000.00 mxn").
    *   *Pluralización:* implementa una función condicional iterando tuplas `(Singular, Plural)` para los atributos (Ej: Si valor == 1 -> "HABITACIÓN", else -> "HABITACIONES"). Retorna etiquetas HTML `<div>`.
5.  **Empaquetado (Data Payload):** consolida todos los assets en un único diccionario estandarizado llamado `datos_propiedad`.
6.  **Router de renderizado:** itera sobre un diccionario de configuración (`PLANTILLAS_A_GENERAR`). Si la plantilla está marcada con `"activo": True`, invoca su función HTML pasándole el payload.
7.  **Salida (Output):** `html2image` captura el HTML con las dimensiones especificadas (`width`, `height`). El archivo se nombra dinámicamente y se mueve a una ruta generada automáticamente: `Diseños_Generados / Nombre Compañía / InternalId / [internal_id]_[empresa]_[sufijo].jpg`.

**5. Estructura del payload (`datos_propiedad`)**

Cualquier nueva plantilla desarrollada debe consumir exclusivamente las siguientes llaves de este diccionario:
*   `img1` a `img6`: Strings (Data URI en Base64 de las fotos de la propiedad).
*   `logo`: String (Data URI en Base64 del logotipo PNG). *Nota CSS: El logo viene en negro; las plantillas aplican `filter: brightness(0) invert(1)` en CSS para volverlo blanco.*
*   `tipo_operacion`: String (Ej: "Departamento en Venta").
*   `precio`: String (Ej: "$2,000,000.00 mxn").
*   `colonia_estado`: String (Ej: "Polanco, Ciudad de México").
*   `calle`: String (Ej: "Av. Presidente Masaryk").
*   `atributos_html`: String con HTML preformateado (Ej: `<div>3 HABITACIONES</div><div>2 BAÑOS</div><div>100 m² TOTALES</div>`).

**6. Configuración de plantillas (Extensibilidad)**

Las plantillas se gestionan mediante el diccionario `PLANTILLAS_A_GENERAR`. Para agregar un diseño nuevo (como un carrusel panorámico), el LLM debe definir:
1. Una nueva función HTML en `mis_plantillas.py`.
2. Una nueva entrada en el diccionario con la estructura: `{"activo": True, "funcion": mis_plantillas.nueva_funcion, "width": X, "height": Y, "sufijo_archivo": "nombre"}`.
