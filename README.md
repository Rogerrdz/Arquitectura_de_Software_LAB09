# Escalamiento en Azure con Maquinas Virtuales (FibonacciApp)

Este proyecto documenta una practica de escalabilidad vertical en Azure usando una aplicacion Node.js que calcula la secuencia de Fibonacci. Se incluye el paso a paso de despliegue en una VM, pruebas de carga con Newman y analisis de comportamiento de CPU y tiempos de respuesta.

## Repositorio

- URL del repositorio: https://github.com/Rogerrdz/Arquitectura_de_Software_LAB09.git
- Nombre del repositorio: Arquitectura_de_Software_LAB09
- Carpeta raiz del laboratorio: LAB_09
- Carpeta de la aplicacion: FibonacciApp

## Primeros Pasos

Estas instrucciones le permitiran tener una copia del proyecto en ejecucion en su maquina local para desarrollo y pruebas. Revise la seccion de despliegue para conocer como publicarlo en una VM de Azure.

### Prerrequisitos

Elementos que necesita instalar para ejecutar el software:

- Cuenta de Azure con permisos para crear VMs
- Cliente SSH
- Git
- Node.js v18+ y npm v9+
- Newman para pruebas concurrentes

Evidencia de versiones y preparacion del entorno:

![Versiones de Node y npm](images/Desarrollo/Version_npm_node.png)

```
node --version
npm --version
```

### Instalacion

A continuacion presento la serie de pasos que realice para dejar el entorno de desarrollo en funcionamiento.

#### Parte 1 - Escalabilidad vertical (paso a paso)

1. Cree la VM en Azure (grupo SCALABILITY_LAB, VM VERTICAL-SCALABILITY, Ubuntu, tamano B1ms/B1ls segun disponibilidad).

![Creacion de VM](images/Desarrollo/Creacion_maquina_virtual.png)
![Propiedades de VM](images/Desarrollo/Propiedades_de_la_maquina_virtual.png)

2. Me conecte por SSH a la VM:

```
ssh scalability_lab@<IP_PUBLICA_VM>
```

3. Instale Node.js y npm en la VM.

![Instalacion de Node en VM](images/Desarrollo/Descarga_node_en_ña_maquina_virtual.png)

4. Clone el repositorio e instale las dependencias:

```
git clone https://github.com/Rogerrdz/Arquitectura_de_Software_LAB09.git
cd Arquitectura_de_Software_LAB09/FibonacciApp
npm install
```

![Repositorio clonado e instalacion npm](images/Desarrollo/Repositorio_clonado_Fibonacci_npm.png)

5. Ejecute la aplicacion:

```
node FibonacciApp.js
```

![App escuchando en puerto 3000](images/Desarrollo/FibonacciApp.js_corriendo_puerto_3000.png)

6. Configure la regla de entrada para habilitar el puerto 3000 en networking.

![Configuracion regla de entrada](images/Desarrollo/Configuracion_regla_de_entrada.png)
![Configuracion regla de entrada 2](images/Desarrollo/Configuracion_regla_de_entrada2.png)

7. Valide el endpoint:

```
http://<IP_PUBLICA_VM>:3000/fibonacci/6
```

![Prueba de endpoint](images/Desarrollo/Prueba_de_Endpoint.png)

8. Medi los tiempos del endpoint para valores altos de n (1000000 a 1090000) usando Network en el navegador.

![Pruebas de velocidad](images/Desarrollo/pruebas_de_velocidad.png)

9. Revise las metricas de CPU en Azure Monitor.

![Consumo CPU](images/Desarrollo/consumo_de_CPU_Maquina_Virtual_Despues_de_las_pruebas_solo_metrica_CPU.png)
![Metricas de VM](images/Desarrollo/Metricas_de_consumo_de_Maquina_Virtual_Despues_de_las_pruebas.png)

10. Instale Newman y ejecute pruebas concurrentes.

![Instalacion de Newman](images/Desarrollo/Instalacion_newman.png)

```
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10
```

![Ejecucion Newman parte 1](images/Desarrollo/Ejecucion_de_pruebas_Newman_parte1.png)
![Ejecucion Newman parte 2](images/Desarrollo/Ejecucion_de_pruebas_Newman_parte2.png)
![Ejecucion Newman parte 3](images/Desarrollo/Ejecucion_de_pruebas_Newman_parte3.png)
![Ejecucion Newman parte 4](images/Desarrollo/Ejecucion_de_pruebas_Newman_parte4.png)

11. Aplique escalamiento vertical cambiando el tamano de la VM y repeti las pruebas.

![Cambio de tamano](images/Desarrollo/Cambio_tamaño_de_RAM_para_prueba_con_Newman.png)
![Metricas posteriores con Newman](images/Desarrollo/Metricas_de_consumo_de_Maquina_Virtual_Despues_de_las_pruebas_con_Newman.png)

## Ejecucion de pruebas

En esta seccion se explica como ejecutar las pruebas del sistema.

### Pruebas end-to-end

Las pruebas E2E se realizan consumiendo el endpoint de Fibonacci y validando respuesta y tiempo.

```
http://<IP_PUBLICA_VM>:3000/fibonacci/1000000
```

Adicionalmente, para carga concurrente:

```
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10
```

### Pruebas de estilo de codigo

Actualmente el proyecto no tiene linter/configuracion de estilo ni pruebas unitarias automatizadas.

```
npm test
```

El script actual retorna mensaje de prueba no definida.

## Despliegue

Notas adicionales para desplegar el sistema en un entorno real.

- El despliegue se realiza sobre una VM Linux en Azure.
- Se debe abrir el puerto 3000 en el NSG para exponer el servicio.
- Para disponibilidad en sesiones SSH cerradas, se recomienda usar un process manager (por ejemplo, `forever` o `pm2`).

## Construido Con

* [Node.js](https://nodejs.org/) - Entorno de ejecucion principal
* [Express](https://expressjs.com/) - Framework web
* [big-integer](https://www.npmjs.com/package/big-integer) - Manejo de enteros grandes
* [Newman](https://www.npmjs.com/package/newman) - Ejecucion de colecciones Postman por linea de comandos
* [Microsoft Azure](https://azure.microsoft.com/) - Infraestructura de VM y monitoreo

## Contribuciones

Lea [CONTRIBUTING.md](https://gist.github.com/PurpleBooth/b24679402957c63ec426) para conocer el proceso de envio de pull requests.

## Versionado

Se utiliza [SemVer](http://semver.org/) para el versionado.

## Autores

* **Diego Andres Trivino** - *Elaboracion Laboratorio Inicial*
* **Rogerrdz** - *Desarrollo del Laboratorio*

## Licencia

Este proyecto esta licenciado bajo ISC (ver `FibonacciApp/package.json`).

## Agradecimientos

* Escuela Colombiana de Ingenieria
* Arquitecturas de Software - ARSW
* Documentacion de Azure y herramientas de monitoreo

## Parte 1 - Respuestas a Preguntas

### 1. Cuantos y cuales recursos crea Azure junto con la VM?

A lo largo del desarrollo observe 7 recursos que coincidere como mas importantes:

1. Maquina virtual
2. Direccion IP publica
3. Grupo de seguridad de red (NSG)
4. Red virtual (VNet)
5. Interfaz de red (NIC)
6. Disco administrado
7. Conexion SSH

![Recursos del grupo](images/Desarrollo/1_pregunta_recursos_VM_Grupo_de_recursos.png)

### 2. Para que sirve cada recurso?

1. Maquina virtual: ejecuta la FibonacciApp.
2. IP publica: permite acceso externo (SSH/HTTP).
3. NSG: controla trafico entrante y saliente por reglas.
4. VNet: red privada logica donde vive la VM.
5. NIC: conecta la VM con VNet e IPs.
6. Disco: almacenamiento persistente del SO y datos.
7. Clave SSH: autenticacion segura para acceso remoto.

![VM principal](images/Desarrollo/1_pregunta_recursos_VM_principal.png)
![IP publica](images/Desarrollo/1_pregunta_recursos_VM_ip_publica.png)
![NSG](images/Desarrollo/1_pregunta_recursos_VM_Grupo_de_seguridad_de_red.png)
![Interfaz de red](images/Desarrollo/1_pregunta_recursos_VM_interfaz_de_red.png)
![Red virtual](images/Desarrollo/1_pregunta_recursos_VM_Red_virtual.png)
![Disco](images/Desarrollo/1_pregunta_recursos_VM_Disco.png)

### 3. Por que se cae la app al cerrar SSH y por que crear Inbound rule?

- Si se ejecuta `node FibonacciApp.js` en una sesion SSH interactiva, al cerrar sesion termina el proceso hijo de la shell.
- La Inbound rule habilita trafico entrante al puerto 3000; sin esa regla, el NSG bloquea acceso externo al servicio.

### 4. Tabla de tiempos e interpretacion

| n | Tiempo aprox |
|---|---|
| 1000000 | 9.06 s |
| 1010000 | 9.49 s |
| 1020000 | 12.82 s |
| 1030000 | 9.95 s |
| 1040000 | 10.17 s |
| 1050000 | 10.53 s |
| 1060000 | 10.74 s |
| 1070000 | 10.98 s |
| 1080000 | 11.17 s |
| 1090000 | 11.38 s |

Interpretacion: los tiempos son altos y tienden a crecer para n grandes, porque el calculo de Fibonacci para estos valores exige muchas operaciones y el servidor atiende solicitudes pesadas en una sola VM con recursos limitados.

### 5. Consumo de CPU e interpretacion

La CPU presenta picos y promedio reportado alrededor de 5.09% en la metrica mostrada. Aunque no supera 70% en esta evidencia, el sistema si evidencia saturacion funcional bajo concurrencia por errores de conexion.

![CPU metrica](images/Desarrollo/consumo_de_CPU_Maquina_Virtual_Despues_de_las_pruebas_solo_metrica_CPU.png)

### 6. Resumen Newman e interpretacion

Durante las pruebas observe:

- Tiempos de peticion entre 7.8 s y 15.9 s.
- Tiempo promedio aproximado entre 10 s y 11.2 s.
- Fallos `ECONNRESET` en varias iteraciones.
- Ejemplo de resumen: 10 requests ejecutados, 4 a 5 fallidos.

Interpretacion: bajo concurrencia, la VM no sostiene el volumen de peticiones y hay reinicios/cortes de conexion del socket.

![Resumen Newman](images/Desarrollo/Ejecucion_de_pruebas_Newman_parte2.png)
![Detalle de errores Newman](images/Desarrollo/Ejecucion_de_pruebas_Newman_parte3.png)

### 7. Diferencia entre B2ms y B1ls 

B2ms ofrece mayor capacidad para cargas sostenidas y concurrentes; B1ls esta pensado para cargas ligeras o intermitentes. En este caso, la diferencia practica es estabilidad bajo carga: con una VM mas robusta disminuye la probabilidad de errores por saturacion.

### 8. Es buena solucion aumentar tamano de VM?

Es una solucion rapida y simple sin necesidad de rediseñar , pero parcial. Mejora capacidad de un nodo unico, aunque no elimina el cuello de botella de arquitectura monolitica en una sola instancia.

### 9. Que pasa con la infraestructura al cambiar tamano?

- Azure reprovisiona/reinicia la VM para aplicar el nuevo SKU.
- Puede haber indisponibilidad temporal durante el cambio.
- Aumenta el costo mensual.
- Sigue existiendo un solo punto de falla.

### 10. Hubo mejora en CPU o tiempos?

Hubo mejora parcial, pero no una solucion definitiva. En metricas se observan promedios de CPU de referencia cercanos a 5.46% y en otra captura cerca de 1.91%, dependiendo de ventana y carga aplicada evidenciando una mejora tanto en tiempos como en CPU. Aun asi, el comportamiento concurrente sigue mostrando limites operativos.

### 11. Con 4 ejecuciones paralelas de Newman mejora porcentualmente?

Con base en las evidencias de las ejecuciones paralelas, el comportamiento mejora de forma limitada pero no proporcional al aumento de carga. Aunque se observamos peticiones exitosas, tambien aparecen fallos y variabilidad alta en latencias, por lo que el rendimiento porcentual no escala de manera lineal al pasar de 2 a 4 ejecuciones concurrentes.

En escalamiento vertical puro la mejora es parcial: aumenta algo la capacidad, pero se mantiene el cuello de botella de una sola instancia. Para una mejora porcentual sostenida conviene escalar horizontalmente.

![Pregunta 11 - evidencia 1](images/Desarrollo/10_pregunta_4_ejecuciones_parte1.png)
![Pregunta 11 - evidencia 2](images/Desarrollo/10_pregunta_4_ejecuciones_parte2.png)
![Pregunta 11 - evidencia 3](images/Desarrollo/10_pregunta_4_ejecuciones_parte3.png)
![Pregunta 11 - evidencia 4](images/Desarrollo/10_pregunta_4_ejecuciones_parte4.png)
![Pregunta 11 - evidencia 5](images/Desarrollo/10_pregunta_4_ejecuciones_parte5.png)
![Pregunta 11 - evidencia 6](images/Desarrollo/10_pregunta_4_ejecuciones_parte6.png)




