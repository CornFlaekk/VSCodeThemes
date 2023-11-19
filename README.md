# VSCodeThemes

> Este repositorio es una adaptación de <https://github.com/gerane/VSCodeThemes> para cumplir nuestras necesidades:
> - Guardar la captura de pantalla de cada tema
> - Almacenar el enlace al marketplace de cada tema
> - Obtener la paleta de los colores más reelevantes de cada tema

## Tabla de Contenidos

- [Proceso de Adaptación](#proceso-de-adaptación)
- [Script de Python Completo](#script-de-python-completo)
- [Script de Python Explicado por Pasos](#script-de-python-explicado-por-pasos)
  - [Librerías Necesarias](#librerías-necesarias)
  - [Clonar Repositorio de GitHub](#clonar-repositorio-de-github)
  - [Recorrer las Carpetas de Temas](#recorrer-las-carpetas-de-temas)
    - [Eliminar `.gitignore` y `.gitattribute`](#eliminar-gitignore-y-gitattribute)
    - [Seleccionar Directorios con Temas](#seleccionar-directorios-con-temas)
    - [Recorrer los Ficheros de las Carpetas](#recorrer-los-ficheros-de-las-carpetas)
      - [Renombrar Imágenes](#renombrar-imágenes)
      - [Borrar Ficheros Innecesarios](#borrar-ficheros-innecesarios)
      - [Guardar Colores de una Imagen](#guardar-colores-de-una-imagen)
      - [Obtener Enlace al Marketplace](#obtener-enlace-al-marketplace)
    - [Eliminar SubCarpetas](#eliminar-subcarpetas)
  - [Eliminar la Carpeta Zenburn](#eliminar-la-carpeta-zenburn)

## Proceso de Adaptación

El proceso de adaptación ha consistido en:

1. Eliminar ficheros innecesarios (`.gitignore`, `.gitattributes`)
2. Renombrar las carpetas a un formato más legible
3. Renombrar las imágenes para obtener un formato común
4. Guardar colores principales de cada imagen
5. Obtener el enlace al Marketplace de cada tema
6. Eliminar ficheros innecesarios
7. Eliminar tema *zenburn* (incompleto)

## Script de Python Completo

```python
import os, shutil, numpy as np, sys, re, colorthief, csv
from PIL import Image


def get_dominant_colors(image_path):
    image = colorthief.ColorThief(image_path)
    dominant_colors = image.get_palette(color_count=5)
    return dominant_colors

# Obtener de un README.md su enlace al marketplace de VSCode
def get_link(filename):
    with open(filename, "r") as f:
        data = f.read()

    # Busca el enlace en el texto
    link_match = re.search(r"https?://marketplace[^\s]+", data)

    # Devuelve el enlace si se encuentra
    if link_match:
        return link_match.group(0).replace(").","")
    else:
        return "missing"

# Almacenar en un fichero el vector de una imagen
def save_to_csv(dominant_colors, filename):
    with open(filename, 'w', newline='') as csvfile:
        writer = csv.writer(csvfile)
        writer.writerow(['R', 'G', 'B'])
        for color in dominant_colors:
            writer.writerow(color)


# Compruebe si la carpeta VSCodeThemes ya existe
# os.chdir("/content/drive/MyDrive/ColorIDEr")
if os.path.exists("VSCodeThemes"):

    # Si la carpeta existe, elimínela
    shutil.rmtree("VSCodeThemes")

# Obtenga la URL del repositorio de GitHub
repo_url = "https://github.com/gerane/VSCodeThemes"

# Clone el repositorio
os.system("git clone " + repo_url)
os.chdir("VSCodeThemes")

folderCount = 0
# Itere sobre las carpetas
for folder in os.listdir():

    #Necesario para eliminar el .gitignore y .gitattribute
    if os.path.isfile(folder):
        filepath = os.path.basename(folder)
        os.remove(filepath)
        continue

    # Eliminar los directorios que no se correspondan a temas
    if not os.path.basename(folder).startswith("gerane.Theme-"):
        shutil.rmtree(folder)
        continue

    # Renombrar los directorios para eliminar del título "gerane.Theme-" y tener títulos más legibles
    else:
        old_name = folder
        new_name = old_name.replace("gerane.Theme-", "")
        os.rename(folder, new_name)
        folder = new_name

    # Itere sobre los archivos dentro de la carpeta
    for file in os.listdir(folder):

        # Obtenga la ruta completa del archivo
        filepath = os.path.join(folder, file)

        # Compruebe si el elemento es un archivo
        if os.path.isfile(filepath):

            # Obtenga el nombre del archivo
            filename = os.path.basename(filepath)

            # Hacer que todas las imágenes tengan la terminación .png en minúsculas
            if filename == "screenshot.PNG":
                os.rename(os.path.join(folder, filename), os.path.join(folder, "screenshot.png"))
                filename = "screenshot.png"

            # Si el nombre del archivo no es "screenshot.PNG", elimínelo
            if filename not in ["screenshot.png", "README.md"]:
                os.remove(filepath)

            # Si el fichero es una imagen, calcular su vector y almacenarlo en un fichero colors.csv
            if filename == "screenshot.png":
                folderCount = folderCount + 1
                sys.stdout.write('\r')
                sys.stdout.write("[%.2f%% : %d/284]" % ((folderCount/284)*100, folderCount))
                dominant_colors = get_dominant_colors(os.path.join(folder,filename))
                save_to_csv(dominant_colors, os.path.join(folder,"colors.csv"))

            # Coger del fichero README.md el enlace, guardarlo en un fichero link.txt y borrar el README
            if filename == "README.md":
                link = get_link(os.path.join(folder,"README.md"))
                with open(os.path.join(folder,"link.txt"), "w") as f:
                    f.write(link)
                os.remove(os.path.join(folder,"README.md"))

       # Si no es un archivo borrar el directorio
        else:
            shutil.rmtree(filepath)

# Zenburn theme is incomplete (no image + no marketplace link)
shutil.rmtree("zenburn")
```

## Script de Python Explicado por Pasos

### Librerías Necesarias

```python
import os, shutil, numpy as np, sys, re, colorthief, csv
from PIL import Image
```

> AVISO: `colorthief` y `pandas` pueden **NO** venir instaladas. En tal caso ejecutar en terminal:
> `pip install colorthief pandas`


El primer paso es importar las librerías necesarias:

- `os` : operaciones relacionadas con el SO (moverse entre directorios, obtener rutas...)
- `shutil` : eliminar directorios que no están vacíos
- `numpy` : operaciones con vectores, matrices y funciones matemáticas
- `sys` : escribir por terminal
- `re` : operaciones con expresiones regulares
- `colorthief` : extraer los colores principales de una imagen
- `csv` : manejas ficheros .csv

### Clonar Repositorio de GitHub

```python

if os.path.exists("VSCodeThemes"):
    shutil.rmtree("VSCodeThemes")

repo_url = "https://github.com/gerane/VSCodeThemes"

os.system("git clone " + repo_url)
os.chdir("VSCodeThemes")
```

Si existe la carpeta *VSCodeThemes* de una ejecución anterior se elimina para empezar de cero. Posteriormente se clona el repositorio <https://github.com/gerane/VSCodeThemes>

### Recorrer las Carpetas de Temas

```python
folderCount = 0
for folder in os.listdir():
```

Iteraremos sobre todas las carpetas que hay en el respositorio, cada una conteniendo un tema. A mayores llevaremos un contador de cuantas hemos recorrido para notificar por terminal del progreso.

#### Eliminar `.gitignore` y `.gitattribute`

```python
    if os.path.isfile(folder):
        filepath = os.path.basename(folder)
        os.remove(filepath)
        continue
```

En el repositorio hay un fichero `.gitignore` y un `.gitattribute`. Como no nos interesan cuando los encuentre los elimina. Cualquier otro fichero que esté se eliminará, porque los temas están en directorios.

#### Seleccionar Directorios con Temas

```python
    if not os.path.basename(folder).startswith("gerane.Theme-"):
        shutil.rmtree(folder)
        continue

    else:
        old_name = folder
        new_name = old_name.replace("gerane.Theme-", "")
        os.rename(folder, new_name)
        folder = new_name
```

Todos los directorios que contienen temas comienzan por *"gerane.Theme-"*, en caso de encontrar uno que no corresponda se elimina.

Si el directorio se corresponde se renombra eliminando la parte *"gerane.Theme-"* para hacer su nombre más legible.

> Ejemplo : `gerane.Theme-Dracula/` se convierte en `Dracula/`

#### Recorrer los Ficheros de las Carpetas

```python
    for file in os.listdir(folder):
        filepath = os.path.join(folder, file)
```

Ahora que se modificaron los directorios los recorreremos buscando los ficheros que nos interesan. A mayores tendremos una variable `filepath` con la ruta al fichero actual.

##### Renombrar Imágenes

```python
            if filename == "screenshot.PNG":
                os.rename(os.path.join(folder, filename), os.path.join(folder, "screenshot.png"))
                filename = "screenshot.png"
```

Dado que en el repositorio las capturas tienen 2 nombres posibles:

- screenshot.**PNG**
- screenshot.**png**

Por conveniencia las nombraremos todas *screenshot.**png***.

##### Borrar Ficheros Innecesarios

```python
            if filename not in ["screenshot.png", "README.md"]:
                os.remove(filepath)
```

Dado que solo nos interesan las *capturas* y los *READMEs* eliminaremos el resto de ficheros.

##### Guardar Colores de una Imagen

```python
            if filename == "screenshot.png":
                folderCount = folderCount + 1
                sys.stdout.write('\r')
                sys.stdout.write("[%.2f%% : %d/284]" % ((folderCount/284)*100, folderCount))
                dominant_colors = get_dominant_colors(os.path.join(folder,filename))
                save_to_csv(dominant_colors, os.path.join(folder,"colors.csv"))
```

Cada vez que encontramos una imagen actualizamos al usuario el contador de progreso (se analiza una sola imagen por cada tema).

Las líneas de `sys.stdout.write()` se encargan de escribir siempre en la misma línea (sin `\n`) el progreso actual con el siguiente formato:
`[ProgresoActual% : CarpetasRecorridas/TotalCarpetas]`

Para obtener los colores dominantes de cada imagen usamos la función `get_dominant_colors(ruta_imagen)`:

```python
def get_dominant_colors(image_path):
    image = colorthief.ColorThief(image_path)
    dominant_colors = image.get_palette(color_count=5)
    return dominant_colors
```

Cargamos la imagen con la librería *ColorThief*, y con su función `get_palette(color_count=5)` obtenemos los 5 colores más usados en la imagen por orden.

Finalmente guardamos el resultado en un fichero *colors.csv* con la función `save_to_csv(colores, ruta_colors_csv)`

```python
def save_to_csv(dominant_colors, filename):
    with open(filename, 'w', newline='') as csvfile:
        writer = csv.writer(csvfile)
        writer.writerow(['R', 'G', 'B'])
        for color in dominant_colors:
            writer.writerow(color)
```

Abrimos el fichero en modo escritura (opción 'w' en `open()`) y almacenamos el contenido con la siguiente forma:

```csv
R,G,B
4,13,30
59,100,172
172,119,129
23,149,58
88,60,62
```

##### Obtener Enlace al Marketplace

```python
            if filename == "README.md":
                link = get_link(os.path.join(folder,"README.md"))
                with open(os.path.join(folder,"link.txt"), "w") as f:
                    f.write(link)
                os.remove(os.path.join(folder,"README.md"))
```

Cuando encontramos el fichero *README* obtenemos su enlace y lo almacenamos en un fichero *link.txt*. Posteriormente borramos el fichero *README*.

Para obtener el enlace usamos la función `get_link(ruta_readme)`:

```python
# Obtener de un README.md su enlace al marketplace de VSCode
def get_link(filename):
    with open(filename, "r") as f:
        data = f.read()

    # Busca el enlace en el texto
    link_match = re.search(r"https?://marketplace[^\s]+", data)

    # Devuelve el enlace si se encuentra
    if link_match:
        return link_match.group(0).replace(").","")
    else:
        return "missing"
```

Esta función abre el fichero y busca una cadena que encaje con la forma `https://marketplace...`. 

Como el fichero contiene el enlace con una línea como la del ejemplo a continuación eliminamos los caracteres ")." con la función `.replace(").","")`

> Ejemplo : \[Visual Studio Marketplace](https://marketplace.visualstudio.com/items/gerane.Theme-DarkCF).

#### Eliminar SubCarpetas

```python
        if os.path.isfile(filepath):
            ...

        else:
            shutil.rmtree(filepath)
```

Dentro del directorio de cada tema hay carpetas que no nos interesan, por tanto cuando detecta que no son ficheros las eliminamos.

### Eliminar la Carpeta Zenburn

```python
shutil.rmtree("zenburn")
```

Finalmente eliminamos el directorio *zenburn* porque este está incompleto (le falta *captura del tema* y un *README* con el enlace al Marketplace).
