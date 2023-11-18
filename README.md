# VSCodeThemes

> This repo is for educational purposes.

## Structure

Each folder corresponds to a VSCode theme. Inside each folder there is:

- `screenshot.png`: A screenshot corresponding to the visual look
- `link.txt`: A file containg the VSCode Marketplace link
- `colors.csv`: A CSV containg the colors of the flattened image array

## Generation

It has been generated using the following python script:

```python
import os, shutil, numpy as np, sys, cv2, re
from PIL import Image

# Obtener el vector de la imagen
def image_to_vector(image_path):
    # Abrir la imagen
    image = Image.open(image_path)

    # Arreglar imagenes que no corresponden al tamaño común reescalándolas
    if (image.size != (818, 463)):
        img = cv2.imread(image_path)
        res_img = cv2.resize(img, dsize=(818, 463), interpolation=cv2.INTER_CUBIC)
        cv2.imwrite(image_path, res_img)
        image = Image.open(image_path)

    # Convertir la imagen a un array NumPy
    image_array = np.array(image)

    # Coger los canales RGB (rojo, verde y azul)
    red_channel = image_array[:, :, 0].ravel()
    green_channel = image_array[:, :, 1].ravel()
    blue_channel = image_array[:, :, 2].ravel()

    # Concatenar los canales en un solo vector
    result_vector = np.concatenate((red_channel, green_channel, blue_channel))

    return result_vector

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
        print("link not found for " + str(filename))
        return "missing"

# Almacenar en un fichero el vector de una imagen
def save_to_csv(vector, csv_filename):
    np.savetxt(csv_filename, vector, delimiter=",", fmt="%d")

# Compruebe si la carpeta VSCodeThemes ya existe
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
                result_vector = image_to_vector(os.path.join(folder,filename))
                save_to_csv(result_vector, os.path.join(folder,"colors.csv"))
            
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