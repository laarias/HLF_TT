Trabajo de Título HLF

# Modelo de captura de pacientes oncológicos :hospital:  

Este modelo de captura de pacientes oncológicos se realizó en marco de una colaboración entre la UC y HLF para el proyecto de __Optimización de la gestión de listas de espera oncológicas mediante el análisis de datos y mejoras en los procesos de atención__. Es decir, es un trabajo en colaboración interdisiplinaria con el equipo de la UC compuesto por Rodrigo Carrasco, Jocelyn Dunstan, José Peña y Luis Arias. Por parte del Hospital de la Florida están presentes Luis Ávila, Verónica Morales y el equipo de Gestoras Oncológicas del Hospital de la Florida, compuesto por Marcela Maira, Marie Anne Pérez, Daisy Tello y Tannia [inserta apellido].

Para esto se entrega el siguiente repositorio, que contiene distintos jupyter notebooks que se explicarán más adelante. [Hacer resumen de los puntos a tratar]


## Consideraciones generales :octocat:

Este es un modelo de captura de pacientes oncológicos. Para esto se requieren diferentes recursos de datos y una estructura de flujo de datos. Las bases de datos ocupadas son la que se encuentran disponible en la reporteria de la Apolo, que escencialmente es reporteria de Thalamus. 

### Problemas encontrados :x:
* La reporteria son archivos csv, por lo que existe pérdida de datos al momento de carga de dataframes en Python.
* En los últimos tres meses, solo se lleva 3 problemas oncológicos levantados y trabajando en una identificación de etapa y estado del paciente. Ya que estos requieren una validación clínica por parte de un médico especialista.
* La búsqueda de pacientes se hace mediante expresiones regulares, por lo que puede entregar información erronea en algunos casos.
* Hasta el momento no se puede obtener los tiempos entre atenciones o tratamiento histórico de pacientes en una etapa más avanzada, ya que el dato en los reportes se sobreescribe.
* Como es un trabajo dinámico, es decir, las enfermeras gestionan y evolucionan pacientes, y al mismo tiempo identificar la etapa y estado de los pacientes, es un trabajo lento.
* El trabajo desarrollado presenta un tiempo de ejecución entre 20 a 30 minutos, dependiendo de la cantidad de problemas oncológicos que se requiera ver.


### Consideraciones para solucionar los problemas mencionados anteriormente :thinking:
* La solución a la pérdida de información es consumir la información directamente de tablas a través del lenguaje SQL y/o programas similares, esto depende del Data Warehouse implementado.
* La búsqueda y etiquetas se puede mejorar usando algún LLM acorde.
* El punto de obtener los tiempos entre atenciones, la información histórica se puede reconstruir a través de reporteria identificandos los hitos que marcan las etapas de un paciente. Pero se cae en el tema de que se puede haber pérdida de información.
* El tiempo de ejecución se puede mejorar paralelizando procesos de carga y filtrado de pacientes, ya sea con una mejor libreria y ocupar un computador o servidor dedicado para esta tarea específica.

## Ejecución :computer:
Existen distintos módulos de ejecución, se van a mencionar en orden ya que unos dependen del anterior.

* El primero módulo de ejecución es ```todos_los_notebooks.ipynb```, este contiene distintos _jupyter notebooks_ para algunos problemas oncológicos. Se presenta en el siguiente diagrama de flujo de datos.
  ![](https://github.com/laarias/HLF_TT/blob/main/imagenes/diagrama%20notebooks.jpg)
  En este flujo se puede observar los notebooks de los problemas oncológicos y los dataframe que entrega.
* El segundo módulo de ejecución es ```V2_consolidado_todos_cancer.ipynb```, este puede correr independiente a primero.
  ![](https://github.com/laarias/HLF_TT/blob/main/imagenes/consolidar%20bases%20gestoras.jpg)
* El tercer módulo de ejecución corresponde a ```V2_notebook_fallecidos.ipynb```
  ![](https://github.com/laarias/HLF_TT/blob/main/imagenes/fallecidos%20y%20estadisticas.jpg)
* El cuarto módulo de ejecución corresponde a ```V3_notebook_derivaciones.ipynb```
  ![](https://github.com/laarias/HLF_TT/blob/main/imagenes/diagrama%20APS.jpg)


### Contenido módulos de ejecución :books: 

* Módulo <1>: El primer módulo corresponde a los notebooks de los problemas oncológicos
    * Módulo <1<sub>1</sub>> Obtención de la reporteria de forma automática. Estos reportes son:
        * Reportes Ambulatorios históricos
        * Reportes Ambulatorios
        * Reportes IEEH
        * Reportes Solicitudes de derivación (Interconsultas)
        * Cruce GES
    * Módulo <1<sub>2</sub>>
        * Se procesa la data histórica en los reportes ambulatorios, por estado de estamento profesional, estado de cita. Esto se ocupará más adelante para la última atención.
        * Se procesa la data de reportes ambulatorios, ieeh y SIC mediante búsqueda de patrones y codificación en diagnósticos, creando un consolidado de pacientes únicos por problema oncológico
        * Se carga la información del Cruces GES por problema oncológico correspondiente
        * Con esas dos fuentes de información se hace un cruce para la obtención de los pacientes encontrados.
    * Módulo <1<sub>3</sub>>
        * Se carga la base de la gestora oncológica
        * Se cruza con el consolidado previamente hecho para obtener los pacientes que no están presentes en la base de la gestora oncológica
        * Se guardan estadísticas de llegadas de pacientes (pacientes encontrados por fecha)
        * Se escribe un archivo excel en la carpeta del problema oncológico con los pacientes nuevos encontrados
        * Se guardan las últimas atenciones del los pacientes en las especialidades buscadas (no todos presentan última atención)
        * Se guardan las futuras atenciones de los pacientes de las bases.
* Módulo <2>: El segundo módulo corresponde a consolidar las bases de las gestoras oncológicas
    * Módulo <2<sub>1</sub>> 
        * Se cargan las bases de las gestoras oncológicas.
        * Las bases que presentan un problema oncologico se cargan con la primera función para homologar los nombres de el problema en la base de datos.
        * Las bases que presentan más de un problema oncológico se cargan con la segunda función, ya que no es necesario homologar su problema oncológico.
        * Se guarda el consolidado.
* Módulo <3>: El tercer módulo corresponde a agregar pacientes fallecidos y generar estadísticas.
    * Módulo <3<sub>1</sub>> 
        * Se cargan la base de datos de pacientes fallecidos.
        * Se carga el consolidado de pacientes oncolóigicos.
    * Módulo <3<sub>2</sub>> 
        * Se cruzan los fallecidos con el consolidado, esto cambia el estado y etapa del paciente.
        * Se generan estadísticas de pacientes activos y fallecidos.
* Módulo <4>: El cuarto módulo corresponde a agregar pacientes provenientes de las solicitudes de interconsultas 
    * Módulo <4<sub>1</sub>> 
        * Primero se carga la reporteria de SIC y GES.
    * Módulo <4<sub>2</sub>> 
        * Se procesa los datos de SIC, limpiando NRO_IDENTIFICADOR, filtrando fecha y por especialidad.
        * Se recuperan pacientes según el problema oncológico.
        * * Se crea un dataframe para concatenar con el de GES.
    * Módulo <4<sub>3</sub>> 
        * Se procesa los datos de GES, se filtran por problema GES, garantía y tomando los últimos 10 días.
        * Se crea un dataframe para concatenar con el de SIC
    * Módulo <4<sub>4</sub>> 
        * Del concatenado se homologan los nombres GES
        * Se eliminan duplicados
        * Se compara con los pacientes que ya están en la base
    * Módulo <4<sub>5</sub>>
        * Se separan los pacientes GES 
        * Se hace un filtro a especialidades de destino que nos interesan
        * Se concatena de nuevo con los pacientes que están filtrados por la especialidad que nos interesa con los pacientes GES (_Esto se hace ya que si se aplica el filtro con todos los pacientes, después no entrega los pacientes GES_)
        * Se guardan los pacientes nuevos
    * Módulo <4<sub>6</sub>>
        * Se hace un proceso similar al paso Módulo <4<sub>5</sub>> para obtener la citas futuras.
        * Se guarda las citas futuras.
    * Módulo <4<sub>7</sub>>
        * Se carga el reporte Ambulatorio histórico para obtener el estado actual de la solicitud de interconsulta.
        * Se guarda los estados nuevos.
      


## Supuestos y consideraciones adicionales :thinking:
Los supuestos que realicé durante el proyecto son los siguientes:

1. Se ocupan solo personas que tienen RUT, ya que para las citas pasadas, futuras y todo el seguimiento requiere el RUT, no se consideran los que tienen pasaporte o RUT temporales.
2. Existen varios procesos que pueden ser módulos, por lo que este modelo podria ser una clase.
3. El módulo 4 se creó de los últimos por lo que no había pasado al pipeline del módulo 1, esto se podría agregar sin problemas.
4. Si se modulariza, la clase madre podria tener una función para agregar distintos diagnosticos.
5. Existen varios valores/nombres hardcodeados en el código, por ejemplo los nombres del problema oncológico cuando se aplica un filtro, también cuando se cargan los nombres del Cruce GES, la cantidad de dias para atras que se ve tanto en GES como en la reporteria.




