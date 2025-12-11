# 🕵️ The Silent Whisper: Desafío de Canal Lateral (AES-128)

> **Categoría:** Criptografía / Side-Channel Analysis (SCA)  
> **Dificultad:** Intermedia  
> **Objetivo:** Recuperar la Clave Secreta AES  

## 📜 Escenario

Nuestros agentes han interceptado un dispositivo seguro mientras realizaba operaciones de cifrado AES-128. Aunque el algoritmo AES es matemáticamente seguro, el hardware donde se ejecuta **tiene una fuga de información**: su consumo de energía varía sutilmente dependiendo de los datos que procesa.

Hemos logrado capturar:
1.  **1.000 Trazas de Potencia:** Mediciones del consumo eléctrico del chip durante el cifrado.
2.  **1.000 Plaintexts:** Los mensajes de texto original (16 bytes) que entraron al chip.

Tu misión es utilizar técnicas de **Análisis de Canal Lateral (SCA)**, como *Correlation Power Analysis (CPA)* o *Deep Learning*, para encontrar la correlación entre los textos y el consumo, y así extraer la **Clave Secreta de 16 bytes**.

---

## 📂 Contenido del Repositorio

* **`aes_challenge_public.npz`**: El archivo principal con el dataset completo (optimizado para ocupar poco espacio). Contiene las 1.000 trazas y sus textos asociados.
* **`trace_combined_view.png`**: Una imagen de referencia para que veas cómo luce la señal eléctrica y el ruido.

### Especificaciones del Dataset (.npz)
El archivo contiene dos arrays alineados (el índice 0 de uno corresponde al 0 del otro):
1.  **`plaintext`**: Array de `(1000, 16)`. Contiene los bytes de entrada (enteros 0-255).
2.  **`traces`**: Array de `(1000, 3500)`. Contiene los puntos flotantes del consumo de energía.

---

## 🛠️ Cómo obtener los datos (Elige tu método)

### Opción A: Usar Python (Recomendado para el ataque)
Si vas a programar tu ataque en Python (scikit-learn, tensorflow, scripts propios), esta es la forma más rápida y directa de cargar los pares.

```python
import numpy as np

def cargar_datos(archivo='aes_challenge_public.npz'):
    try:
        data = np.load(archivo)
        # Empaquetamos en pares: (Texto, Traza)
        dataset = list(zip(data['plaintext'], data['traces']))
        print(f"✅ Cargados {len(dataset)} pares de datos.")
        return dataset
    except FileNotFoundError:
        print("❌ Error: No se encuentra el archivo .npz")
        return []

# Ejemplo de uso:
datos = cargar_datos()
texto_0, traza_0 = datos[0] # Primer par
