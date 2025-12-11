# 🕵️ The Silent Whisper: Desafío de Canal Lateral (AES-128)

> **Categoría:** Criptografía / Side-Channel Analysis (SCA)    
> **Objetivo:** Recuperar la Clave Secreta AES  

## 📜 Escenario

Nuestros agentes han interceptado un dispositivo seguro mientras realizaba operaciones de cifrado AES-128. Aunque el algoritmo AES es matemáticamente seguro, el hardware donde se ejecuta **tiene una fuga de información**: su consumo de energía varía sutilmente dependiendo de los datos que procesa.

Hemos logrado capturar:
1.  **500 Trazas de Potencia:** Mediciones del consumo eléctrico del chip durante el cifrado.
2.  **500 Plaintexts:** Los mensajes de texto original (16 bytes) que entraron al chip.

Tu misión es utilizar técnicas de **Análisis de Canal Lateral (SCA)**, como *Correlation Power Analysis (CPA)* o *Deep Learning*, para encontrar la correlación entre los textos y el consumo, y así extraer la **Clave Secreta de 16 bytes**.

---

## 📂 Contenido del Repositorio

* **`aes_challenge_public.npz`**: El archivo principal con el dataset completo. Contiene las 1.000 trazas y sus textos asociados.
* **`trace_combined_view.png`**: Una imagen de referencia para que veas cómo luce la señal eléctrica y el ruido.

### Especificaciones del Dataset (.npz)
El archivo contiene dos arrays alineados (el índice 0 de uno corresponde al 0 del otro):
1.  **`plaintext`**: Array de `(500, 16)`. Contiene los bytes de entrada (enteros 0-255).
2.  **`traces`**: Array de `(500, 3500)`. Contiene los puntos flotantes del consumo de energía.

---

## 🛠️ Cómo obtener los datos (Elige tu método)

### Opción A: Usar Python (Recomendado para el ataque)
Si vas a programar tu ataque en Python (scikit-learn, tensorflow, scripts propios), esta es la forma más rápida y directa de cargar los pares.

```python

import numpy as np
import sys

# Configuración
INPUT_FILE = 'aes_challenge_public.npz'
OUTPUT_FILE = 'dataset_completo.py' # Este será el archivo final para los participantes

def generar_archivo_python():
    print(f"📂 Cargando datos de {INPUT_FILE}...")
    
    try:
        data = np.load(INPUT_FILE)
        plaintexts = data['plaintext']
        traces = data['traces']
    except FileNotFoundError:
        print("❌ Error: No se encuentra el archivo .npz")
        return

    total = len(plaintexts)
    print(f"✅ Datos cargados: {total} trazas.")
    print(f"⚠️  Generando '{OUTPUT_FILE}'. Esto creará un archivo de código MUY grande (>50MB).")
    print("⏳ Escribiendo... (Paciencia, puede tardar unos segundos)")

    with open(OUTPUT_FILE, "w") as f:
        # 1. Cabecera e imports
        f.write("import numpy as np\n\n")
        f.write("print('Cargando arrays gigantes en memoria... espere un momento.')\n\n")

        # 2. Escribir Plaintexts (Es rápido)
        f.write("# --- LISTA DE PLAINTEXTS (500 muestras x 16 bytes) ---\n")
        f.write("# Variable: plaintexts\n")
        f.write("plaintexts = np.array([\n")
        
        for i, p in enumerate(plaintexts):
            # Convertimos a lista simple [1, 2, 3...]
            f.write(f"    {list(p)}, # ID: {i}\n")
            
        f.write("], dtype=np.uint8)\n\n")

        # 3. Escribir Trazas (La parte pesada)
        f.write("# --- LISTA DE TRAZAS DE POTENCIA (500 muestras x 3500 puntos) ---\n")
        f.write("# Variable: traces\n")
        f.write("traces = np.array([\n")
        
        for i, t in enumerate(traces):
            # Formateamos los floats para que ocupen menos (4 decimales)
            # Ejemplo: [0.1234, -1.5432, ...]
            valores_str = ", ".join([f"{x:.4f}" for x in t])
            f.write(f"    [{valores_str}], # ID: {i}\n")
            
            # Barra de progreso visual en tu consola
            if i % 50 == 0:
                print(f"   -> Escribiendo traza {i}/{total}...")

        f.write("], dtype=np.float32)\n\n")
        
        f.write("print('✅ ¡Datos cargados correctamente! Variables disponibles: plaintexts, traces')\n")

    print(f"✅ ¡TERMINADO! Se ha creado '{OUTPUT_FILE}'.")
    print("   Ahora puedes subir este archivo o pasárselo a los participantes.")

if __name__ == "__main__":
    generar_archivo_python()
```

### Opción B: Convertir todo a TXT (Para ver los números "en claro")
Si prefieres tener un archivo de texto gigante (`dataset_completo.txt`) con todos los datos para leerlos o importarlos en otro programa (Matlab, Excel, C++...), crea un archivo llamado `exportar_txt.py` con el siguiente código y ejecútalo:

```python
import numpy as np
import sys

# Configuración
INPUT_FILE = 'aes_challenge_public.npz'
OUTPUT_FILE = 'dataset_completo.txt'

def exportar_todo_a_txt():
    print(f"📂 Cargando {INPUT_FILE}...")
    
    try:
        data = np.load(INPUT_FILE)
        plaintexts = data['plaintext']
        traces = data['traces']
    except FileNotFoundError:
        print("❌ Error: No encuentro el archivo .npz")
        return

    total = len(plaintexts)
    print(f"✅ Cargados {total} pares de datos.")
    print(f"💾 Escribiendo en '{OUTPUT_FILE}'... (Esto puede tardar unos segundos)")

    with open(OUTPUT_FILE, "w") as f:
        # Escribimos cabecera
        f.write(f"# DATASET EXPORTADO DEL CHALLENGE AES\n")
        f.write(f"# Total Trazas: {total}\n")
        f.write(f"# Formato: [ID] -> PLAINTEXT (Hex) -> TRAZA (Decimal)\n")
        f.write("-" * 50 + "\n\n")

        # Bucle para exportar CADA par
        for i in range(total):
            text = plaintexts[i]
            trace = traces[i]

            # 1. Convertir Plaintext a Hex string
            text_hex = " ".join([f"{b:02x}" for b in text])

            # 2. Convertir Traza a string (4 decimales para ahorrar espacio)
            trace_str = ", ".join([f"{val:.4f}" for val in trace])

            # 3. Escribir bloque
            f.write(f"[ID: {i}]\n")
            f.write(f"PLAINTEXT: {text_hex}\n")
            f.write(f"TRAZA: {trace_str}\n")
            f.write("\n" + "="*20 + "\n\n")
            
            # Barra de progreso
            if i % 100 == 0:
                print(f"   -> Procesadas {i} de {total} trazas...")

    print(f"✅ ¡TERMINADO! Datos guardados en '{OUTPUT_FILE}'.")

if __name__ == "__main__":
    exportar_todo_a_txt()
