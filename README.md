# **Documentación modelo SIR**

### **Proyecto OPS/OMS Minsal**
### **Preparado por:** Equipo de modelamiento

### Introducción

Para el periodo de estudio de este proyecto (comienzos del 2020 hasta el fin del 2021), se puede dividir la pandemia COVID-19 en Chile en 3 etapas, 
de acuerdo a la lógica del modelo SIR de 3 unidades fundamentales que se está utilizando:

  a) Inicio de la pandemia a febrero del 2021: primer período donde aún no había vacunas disponibles. En términos del modelo, equivale al **modelo con una unidad fundamental**.
  
  b) marzo 2021 a agosto 2021: período en que se inicia la vacunación masiva en el país con esquemas primarios, en su gran mayoría, de 2 dosis. En este periodo es válido el **modelo con 2 unidades fundamentales**.
  
  c) septiembre 2021 a diciembre 2021: en este periodo se inicia la administración de la primera dosis de refuerzo a la población y además, aún no hay presencia importante de la variante ómicron. 
  En términos del modelo SIR, corresponde al **modelo con 3 unidades fundamentales** que incluye a la población no vacunada, vacunada con esquema primario y vacunada con dosis de refuerzo.

Para llevar a cabo la calibración de los modelos, se utilizó una **metodología secuencial**, partiendo con el modelo de sólo una capa. Luego se extendió para la segunda capa, 
considerando como fijos algunos parámetros calibrados en el periodo anterior. Lo mismo se repitió para la tercera capa. 
En todos los casos, se utilizaron datos provenientes del Ministerio de Salud para entrenar, en particular, **series de tiempo de fallecidos y casos nuevos**. 
Además, para los modelos que ya incorporan vacunación, se utilizaron series de tiempo de vacunación. 

Para los modelos con dos y tres capas, el periodo de estudio se dividió en dos, para capturar mejor los peaks de casos y lograr un mejor ajuste de las curvas.

Esquemáticamente, el proceso de calibración se puede ver en la figura 1.

<p align="center">
  Figura 1: Flujo de calibraciones modelo SIR de 3 capas. (la fuente de este dibujo está en la misma carpeta)
</p>

<p align="center">
  <img src="https://user-images.githubusercontent.com/74310240/194711064-43caaeff-c0a8-4c91-bd22-222e93122d68.png" />
</p>

Donde: 

* Las líneas punteadas en naranjo muestran la conexión entre la salida de la calibración anterior y la entrada a la siguiente, en forma de CI.
* En los modelos 2 y 3, se destacan con diferente color los parámetros y tasas resultantes de la calibración: negro para el primer piso (“N”, no vacunados), 
  azul para el segundo piso (“V”, vacunados con esquema primarios) y rojo para el tercer piso (“Vr”, vacunados con dosis de refuerzo).
* Podemos clasificar los parámetros y tasas en dos grupos: los que entran con una Condición inicial (CI) y luego se calibran y los que se fijan.
* Para el modelo SIR con una unidad se resolvió el bloque 01-12-2020 a 28-02-2021 completo. El subreporte se limitó al rango [0.6,0.7], 
  ya que múltiples calibraciones previas mostraron que los resultados eran más razonables en términos de interpretación cuando el subreporte se encontraba en ese rango.
* Para el modelo SIR con dos unidades se resolvió el bloque 01-03-2021 a 15-08-2021 en dos bloques: (1) del 01-03 al 15-04 y (2) del 16-04 al 15-08. 
  Calibraciones anteriores mostraron que calibrar el bloque completo no generaba buenos ajustes con los datos de fallecidos y casos y además, 
  que no se lograban capturar correctamente los dos peaks ocurridos en ese periodo, uno en abril y otro en junio (ver figura 2).
* Para el modelo SIR con tres unidades se resolvió el bloque 16-08-2021 a 31-12-2021 en dos bloques: 
  (1) del 16-08 al 30-09 y (2) del 01-10 al 31-12, al igual que en el caso anterior, para lograr un mejor ajuste de las curvas.
  
  
<p align="center">
  Figura 2: Casos nuevos años 2021.  
</p>

<p align="center">
  <img src="https://user-images.githubusercontent.com/74310240/194711300-a7ab24cb-f7a4-4ca1-89d6-dc60a5877f67.png" />
</p>

### Algunos resultados
 
Estas son las calibraciones finales que elegimos por mostrar el mejor ajuste con las curvas. 
La totalidad de las salidas está en la carpeta- llamada “calibraciones finales” que se encuentra en la documentación.
 
1) Modelo SIR con una unidad
 
<p align="center">
  <img src="https://user-images.githubusercontent.com/74310240/194712375-c1d55364-c3e7-46c8-96a4-a8abe81b5a40.png" />
</p>

2) Modelo SIR con dos unidades
 
<p align="center">
  <img src="https://user-images.githubusercontent.com/74310240/194712506-8af56584-2189-4e34-9138-c2f6e99d324f.png" />
</p> 

3) Modelo SIR con tres unidades

<p align="center">
  <img src="https://user-images.githubusercontent.com/74310240/194712623-ad213518-4011-4750-972e-65c225f84cb7.png" />
</p> 

### Códigos

Para cada uno de los tres modelos presentados en la sección anterior, se tienen 2 archivos:

  1) Archivo .py, donde se cargan las librerías a utilizar, se leen las condiciones iniciales, se establece el periodo para los datos de entrenamiento 
  y validación, se fijan tasas y parámetros, se llama al módulo que carga y resuelve el modelo, se generan las salidas y los gráficos.
  2) Archivo .stan, donde se escribe el modelo que lee el módulo stan. En este archivo se define la EDO a resolver, los parámetros a calibrar, 
  los fijos y las CI.
  
Además, si se quiere ejecutar en el clúster, se agrega un archivo adicional en bash, que se puede encontrar con el nombre “script…”. 
Este archivo tiene el formato estándar que utiliza el gestor de colas slurm que opera en el clúster, donde se solicitan los recursos para 
la ejecución (cores y memoria), se cargan los módulos de software requeridos (en este caso, python) y se escribe la línea de comando para ejecutar. 
En este caso la línea de comandos recibe también los parámetros de entrada, el número de job (número que asigna el gestor de colas del clúster para 
hacer seguimiento del trabajo) y el archivo que recibe como input.

La estructura de estos archivos es la misma para los tres modelos. En el caso del archivo python, encontramos lo siguiente:

```
#Importación de librerías
   from….import…

#Lectura de parámetros de entrada
   param = float(sys.arg[...])

#Compilación del modelo stan
   sm = pystan.StanModel(file=pathStan+stanmod)

#Definición de periodo de entrenamiento/validación/predicción
   t0=... tf=... tval=... tpred=...

#Carga del archivo de datos de entrenamiento
      data = pd.read_csv(...)

#Creación de array de entrenamiento/validación/predicción
     T,T_val,T_pred = np.arange(...)

#Definición de valores para parámetros fijos
   mu=..., ut=..., q_a,q_s=...

#Preparación de datos para stan
    stan_data = {...}

#Instrucciones para correr el modelo. 
#iter: cantidad de iteraciones, ojalá no menos que 200
#warmup: usar iter/2
#chains: usar la capacidad del computador (4-8 en PC, 10 en clúster)
#max_treedepth: generalmente 14, pero 11 también da buenos resultados en mucho #menos tiempo
   fit = sm.sampling(data=stan_data, iter=200, warmup=100, chains=10,
                  'max_treedepth': 14})

#Reporte de salida
   pystan.check_hmc_diagnostics(fit, verbose=True)
   ….
   print(summary)

#Ploteo y guardado de salidas para todos los compartimentos
   def plot_ode:
   ….
   params = [....]
   ….

#Ploteo y guardado de salidas vs. datos de entrenamiento
   plt.title('Fallecidos')
   ….
   plt.title(“Casos totales”')
   ….
   plt.savefig(....)
   plt.show()
```

#### COMPLETAR

La estructura del archivo stan es la siguiente: 
```
functions{
   // blablabla, una breve descripción de la sección
}

data{

}

transformed data{

}

parameters{

}

transformed parameters{

}

model{

}

generated quantities{

}

```

