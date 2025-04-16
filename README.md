import string

def cargar_diccionario(nombre_archivo):
    """Carga palabras correctas desde archivo a un diccionario"""
    diccionario = {}
    try:
        archivo = open(nombre_archivo, 'r', encoding='utf-8')
        for linea in archivo:
            palabra = linea.strip().lower()
            if palabra:
                diccionario[palabra] = True
        archivo.close()
        print(f"Diccionario cargado: {len(diccionario)} palabras")
        return diccionario
    except FileNotFoundError:
        print(f"Error: Archivo '{nombre_archivo}' no encontrado")
        return None
    except Exception as e:
        print(f"Error al leer diccionario: {str(e)}")
        return None

def limpiar_palabra(palabra):
    """Limpia palabra eliminando puntuación y normalizando"""
    translator = str.maketrans('', '', string.punctuation)
    return palabra.translate(translator).lower().strip()

def verificar_archivo(archivo_verificar, diccionario):
    """Verifica ortografía y sugiere correcciones"""
    try:
        archivo = open(archivo_verificar, 'r', encoding='utf-8')
        lineas = archivo.readlines()
        archivo.close()
        
        print(f"\n Verificando: {archivo_verificar}")
        print("=" * 50)
        
        errores_totales = 0
        palabras_incorrectas = {}
        
        for num_linea, linea in enumerate(lineas, 1):
            palabras = linea.split()
            errores_linea = []
            
            for palabra in palabras:
                original = palabra
                palabra_limpia = limpiar_palabra(palabra)
                
                if not palabra_limpia or palabra_limpia.isdigit():
                    continue
                    
                if palabra_limpia not in diccionario:
                    errores_linea.append(original)
                    errores_totales += 1
                    
                    # Sugerir corrección
                    sugerencia = sugerir_correccion(palabra_limpia, diccionario)
                    if palabra_limpia not in palabras_incorrectas:
                        palabras_incorrectas[palabra_limpia] = {
                            'ocurrencias': 1,
                            'sugerencia': sugerencia,
                            'lineas': [num_linea]
                        }
                    else:
                        palabras_incorrectas[palabra_limpia]['ocurrencias'] += 1
                        palabras_incorrectas[palabra_limpia]['lineas'].append(num_linea)
            
            if errores_linea:
                print(f"\nLínea {num_linea}:")
                print(linea.rstrip())
                print("Errores:", " ".join(errores_linea))
        
        # Mostrar resumen detallado
        print("\n" + "=" * 50)
        print(" RESUMEN DE ERRORES")
        print(f"Total errores encontrados: {errores_totales}")
        print(f"Palabras únicas incorrectas: {len(palabras_incorrectas)}")
        
        if palabras_incorrectas:
            print("\n Detalle de palabras incorrectas:")
            for palabra, datos in sorted(palabras_incorrectas.items()):
                lineas_str = ", ".join(map(str, datos['lineas']))
                print(f"\nPalabra: '{palabra}'")
                print(f"Ocurrencias: {datos['ocurrencias']}")
                print(f"Líneas: {lineas_str}")
                print(f"Sugerencia: {datos['sugerencia'] or 'No se encontró sugerencia'}")
    
    except FileNotFoundError:
        print(f"Error: No se pudo abrir '{archivo_verificar}'")
    except Exception as e:
        print(f"Error inesperado: {str(e)}")

def sugerir_correccion(palabra_erronea, diccionario):
    """Sugiere correcciones para palabras erróneas"""
    # Lista de posibles correcciones
    sugerencias = []
    
    # Verificar si es error de acentuación
    for palabra_correcta in diccionario:
        if palabra_correcta.replace('á', 'a').replace('é', 'e').replace('í', 'i').replace('ó', 'o').replace('ú', 'u') == palabra_erronea:
            sugerencias.append(palabra_correcta)
    
    # Verificar errores comunes de escritura
    for palabra_correcta in diccionario:
        if len(palabra_correcta) == len(palabra_erronea):
            diferencias = sum(1 for a, b in zip(palabra_correcta, palabra_erronea) if a != b)
            if diferencias == 1:
                sugerencias.append(palabra_correcta)
    
    # Devolver las mejores sugerencias (hasta 3)
    return ", ".join(sorted(set(sugerencias))[:3])

def main():
    """Función principal del corrector"""
    print("\nCORRECTOR ORTOGRÁFICO AVANZADO")
    print("=" * 35)
    
    # Cargar diccionario
    diccionario = cargar_diccionario("palabras.txt")
    if not diccionario:
        return
    
    # Archivos a verificar
    archivos = ["AlanTuring.txt", "ComputaciónEvolutiva.txt", "PatitoFeo.txt"]
    
    for archivo in archivos:
        verificar_archivo(archivo, diccionario)
        print("\n" + "=" * 50 + "\n")

if __name__ == "__main__":
    main()
