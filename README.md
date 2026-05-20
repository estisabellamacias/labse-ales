# **Variabilidad de la Frecuencia Cardíaca (HRV) y balance autonómico**

## *Desarrollo de la practica*

***Parte A***
- *Actividad simpática y parasimpática del SNA:* El sistema nervioso autónomo (SNA) regula las funciones involuntarias para mantener la homeostasis. La rama simpática se activa en situaciones de estrés o actividad ("lucha o huida"), mientras que la rama parasimpática predomina en estados de reposo, digestión y recuperación.
  
- *Efecto en la frecuencia cardíaca:* Existe un control antagónico sobre el corazón. La actividad simpática libera noradrenalina para aumentar la frecuencia cardíaca, mientras que la actividad parasimpática, a través del nervio vago, libera acetilcolina para disminuirla. El equilibrio entre ambas determina el ritmo cardíaco en cada momento.
  
- *Variabilidad de la frecuencia cardíaca (HRV):* La HRV es la medida de las variaciones de tiempo entre latidos cardíacos sucesivos, específicamente los intervalos R-R. Estos intervalos se extraen de una señal electrocardiográfica (ECG) mediante la detección precisa de los picos R del complejo QRS. Una HRV alta suele indicar un sistema autónomo saludable y adaptable.

- *Diagrama de Poincaré:* Es una herramienta de análisis no lineal que grafica cada intervalo RRn frente al siguiente RRn+1. La geometría de la nube de puntos resultante permite evaluar la dispersión de la señal y calcular índices específicos: el CVI (Cardiac Vagal Index) para la actividad parasimpática y el CSI (Cardiac Sympathetic Index) para la actividad simpática.

***Parte B***

***Parte C***


## *Diagramas de flujo*

<p align="center">
<img src="DIAGRAMAS/diagrama toma de datos.png" width="600">
</p>
<p align="center">
<em>Diagrama de flujo de la toma de la señal.</em>
</p>

<p align="center">
<img src="DIAGRAMAS/diagrama graficas.png" width="600">
</p>
<p align="center">
<em>Diagrama de flujo de las graficas.</em>
</p>

## *Codigos*

### *1. Toma de la señal.*

***Configuración:***

* **`fs`**
  Frecuencia de muestreo (1000 Hz). Permite capturar correctamente los picos R del ECG sin perder información.

* **`max_muestras`**
  Define el tamaño del buffer (4 segundos de señal en pantalla).

* **`datos_buffer`**
  Arreglo donde se almacenan las muestras que se visualizan en la gráfica.

* **`grabando`**
  Variable de control que indica si los datos deben guardarse en un archivo.

* **`corriendo`**
  Controla la ejecución del sistema y permite detener el hilo de adquisición de forma segura.

***Diseño de filtros:***

* **`butter()`**
  Diseña un filtro pasa-banda de orden 4 entre 0.5 y 40 Hz.
  Elimina la deriva de la línea base y el ruido muscular.

* **`iirnotch()`**
  Diseña un filtro notch en 60 Hz.
  Elimina la interferencia de la red eléctrica.

* **`zi_band` y `zi_notch`**
  Condiciones iniciales de los filtros.
  Evitan transitorios bruscos al iniciar el procesamiento.

***DAQ:***

* **`nidaqmx.Task()`**
  Crea la tarea de adquisición y permite la comunicación con la DAQ.

* **`add_ai_voltage_chan()`**
  Configura el canal de entrada analógica donde se adquiere la señal.

* **`cfg_samp_clk_timing()`**
  Define la frecuencia de muestreo (`fs`) y el modo continuo de adquisición.

* **`task.read()`**
  Lee bloques de datos desde la DAQ en tiempo real (en este caso, 50 muestras por iteración).

* **`adquirir_datos()`**
  Función principal de adquisición:
  - Lee datos continuamente
  - Aplica filtrado
  - Actualiza el buffer
  - Guarda datos si es necesario

***Procesamiento de datos:***

* **`lfilter()`**
  Aplica los filtros digitales a la señal en tiempo real.
  Se usa en cascada: primero pasa-banda, luego notch.

* **`np.roll()`**
  Implementa un buffer circular desplazando los datos para mantener solo las muestras más recientes.

* **`np.zeros()`**
  Inicializa arreglos en cero para el buffer y las condiciones iniciales de los filtros.

***Visualización:***

* **`actualizar_plot()`**
  Actualiza la gráfica en tiempo real (~33 FPS).

* **`line.set_ydata()`**
  Actualiza los valores de la señal mostrada en la gráfica.

* **`canvas.draw_idle()`**
  Redibuja la gráfica solo cuando el sistema tiene recursos disponibles, optimizando el rendimiento.

* **`root.after()`**
  Programa la actualización periódica de la gráfica sin bloquear la interfaz.

  
***Interfaz gráfica:***

* **`tk.Tk()`**
  Crea la ventana principal de la aplicación.

* **`Button`**
  Permite iniciar y detener la grabación de datos.

* **`Label`**
  Muestra el estado actual del sistema (visualizando o grabando).

* **`toggle_guardado()`**
  Cambia el estado de grabación y actualiza la interfaz.

***Guardado de datos:***

* **`np.savetxt()`**
  Guarda los datos filtrados en un archivo de texto en tiempo real.

* **`grabando`**
  Controla cuándo se escriben los datos en el archivo.

***Control del sistema:***

* **`threading.Thread()`**
  Ejecuta la adquisición de datos en un hilo separado para evitar que la interfaz se congele.

* **`on_closing()`**
  Maneja el cierre seguro del programa:
  - Detiene el hilo de adquisición
  - Cierra la interfaz correctamente

* **`root.mainloop()`**
  Inicia el ciclo principal de la interfaz gráfica.
  

### *2. Codigo para las graficas*

***Carga de datos:***

* **`np.loadtxt()`**
  Carga la señal ECG desde un archivo de texto (`datos_ECG_filtrado.txt`).

* **`Fs`**
  Frecuencia de muestreo (1000 Hz), usada para convertir muestras a tiempo y calcular intervalos.

***Filtrado de la señal:***

* **`butter()`**
  Diseña un filtro pasa-banda (0.5 – 40 Hz) de orden 2.
  Elimina ruido de baja y alta frecuencia.

* **`lfilter()`**
  Aplica el filtro a la señal ECG cargada.
  Genera la señal filtrada final (`ecg`).

* **`b, a`**
  Coeficientes del filtro IIR que definen su comportamiento.

***Segmentación:***

* **`seg1` y `seg2`**
  Dividen la señal en dos intervalos de tiempo:
  - Segmento 1: primeros 120 segundos
  - Segmento 2: siguientes 120 segundos

Esto permite comparar cambios fisiológicos (por ejemplo, fatiga o estrés).


***Detección de picos R:***

* **`find_peaks()`**
  Detecta los picos R del ECG usando:

  - **`distance`**
    Distancia mínima entre picos (0.4 s) para evitar falsas detecciones.

  - **`height`**
    Umbral dinámico basado en la media + desviación estándar de la señal.

* **`picos`**
  Contiene los índices donde ocurren los latidos.

***Procesamiento de intervalos RR:***

* **`np.diff()`**
  Calcula la diferencia entre picos consecutivos.

* **`rr`**
  Intervalos RR en milisegundos:
  Representan el tiempo entre latidos cardíacos.

***Visualización ECG:***

* **`np.arange()`**
  Genera el eje de tiempo para graficar.

* **`plt.plot()`**
  Grafica la señal ECG en un intervalo específico (60–70 s).

* **`picos_vis`**
  Filtra los picos R que están dentro del rango visualizado.

* **Gráfica**
  - Señal en negro
  - Picos R en rojo
  Permite validar visualmente la detección.

***Análisis HRV (Variabilidad del Ritmo Cardíaco):***

* **`rr_n` y `rr_n1`**
  Pares consecutivos de intervalos RR para análisis dinámico.

***Parámetros de Poincaré:***

* **`SD1`**
  Mide la variabilidad a corto plazo (cambios latido a latido).

* **`SD2`**
  Mide la variabilidad a largo plazo.

* **`CSI`**
  Índice de estrés cardíaco (relación SD2/SD1).

* **`CVI`**
  Índice logarítmico de variabilidad.

***Gráfica de Poincaré:***

* **`plt.scatter()`**
  Grafica \( RR_n \) vs \( RR_{n+1} \).

* **Línea diagonal**
  Representa igualdad entre intervalos consecutivos.

* **Interpretación**
  - Dispersión amplia → mayor variabilidad
  - Dispersión reducida → posible fatiga o estrés

***Función principal de análisis:***

* **`analizar_segmento()`**
  Ejecuta todo el procesamiento para cada segmento:
  - Detecta picos R
  - Calcula intervalos RR
  - Genera gráficas
  - Calcula métricas HRV

* **`return`**
  Devuelve un diccionario con los resultados del análisis.

***Ejecución del análisis:***

* **`res1` y `res2`**
  Almacenan los resultados de cada segmento.

***Presentación de resultados:***

* **`print()`**
  Muestra una tabla comparativa con:

  - Media RR
  - SDNN
  - SD1
  - SD2
  - CSI
  - CVI

* **Formato tabular**
  Facilita la comparación entre segmentos.

