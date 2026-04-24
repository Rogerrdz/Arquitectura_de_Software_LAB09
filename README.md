# Escalamiento en Azure con Maquinas Virtuales (FibonacciApp)

Este proyecto documenta una practica de escalabilidad vertical y horizontal en Azure usando una aplicacion Node.js que calcula la secuencia de Fibonacci. Se incluye el paso a paso de despliegue en VM(s), pruebas de carga con Newman y analisis de comportamiento de CPU, disponibilidad y tiempos de respuesta.

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

#### Parte 2 - Escalabilidad horizontal con Load Balancer (paso a paso)

Antes de continuar, puede eliminar el grupo de recursos anterior para evitar gastos adicionales y realizar la actividad en un grupo de recursos limpio.

##### Limitaciones encontradas antes de crear las VMs

- El tamano requerido en el laboratorio (`B1ls`) no estaba disponible en zona 1 para esta suscripcion/regionalidad; solo se permitia en zonas 2 y 3.
- La cuota de IPs publicas estandar disponible en la suscripcion fue de 3, por lo que solo pude asignar IP publica al balanceador, VM1 y VM2.
- Por la restriccion anterior, VM3 se aprovisiono para trabajar dentro de la red privada y atender trafico por backend pool del balanceador.

![Limitacion de tamano por zona](images/Desarrollo/Limitacion_1_B1ls_ningun_tamaño_disponible_para_zona_1.png)
![Limitacion de IPs publicas](images/Desarrollo/Limitacion_2_No_se_puede_crear_VM3_suscripcion_estudiante_solo_deja_tene_3_ips_publicas_Load_B,VM1,VM2.png)

##### Crear el Balanceador de Carga

El Balanceador de Carga es un recurso clave para habilitar la escalabilidad horizontal del sistema.

1. Cree el Load Balancer en Azure.

![Creacion Load Balancer](images/Desarrollo/Creacion_balanceador_de_carga.png)

2. Cree el Backend Pool.

![Creacion Backend Pool](images/Desarrollo/Grupo_de_backend_creado.png)

3. Cree el Health Probe.

![Creacion Health Probe](images/Desarrollo/Creacion_Sondeo_de_Estado%20.png)

4. Cree la Load Balancing Rule.

![Creacion Load Balancing Rule](images/Desarrollo/Regla_de_equilibrio_de_carga_agregada.png)

5. Cree la Virtual Network en el mismo grupo de recursos.

![Creacion Virtual Network](images/Desarrollo/Creacion_red_virtual.png)

##### Crear las maquinas virtuales (Nodos)

Ahora se crean 2 VMs (VM1 y VM2) y luego se agregan al balanceador de carga.

1. Configuracion basica de la VM (zona de disponibilidad por nodo).

![Configuracion basica VM](images/Desarrollo/Creacion_VM1.png)

2. Configuracion de networking: use la Virtual Network y Subnet creadas previamente. Asigne IP publica segun la disponibilidad de cuota y habilite redundancia de zona cuando aplique.

![Configuracion networking VM](images/Desarrollo/Creacion_VM2.png)

3. Para el Network Security Group seleccione modo avanzado y cree una regla de entrada (Inbound) para habilitar trafico por el puerto 3000.

![Creacion NSG](images/Desarrollo/Grupo_de_seguridad_de_red_VM1.png)

4. Asigne la VM al balanceador de carga.

![Asociacion VM con LB](images/Desarrollo/VM1_asignada_a_balanceador_de_carga.png)

5. Finalice creacion de la VM y repeti el proceso para VM2.

![IP del balanceador y nodos](images/Desarrollo/ip_del_Load_balancer_en_las_dos_maquinas_para_pruebas_con_newman.png)

##### Instalacion de la aplicacion en cada VM

Ejecute en cada VM (ajustando usuario/ruta segun el nombre real de la maquina):

```
git clone https://github.com/daprieto1/ARSW_LOAD-BALANCING_AZURE.git

curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.34.0/install.sh | bash
source /home/<usuario_vm>/.bashrc
nvm install node

cd ARSW_LOAD-BALANCING_AZURE/FibonacciApp
npm install

npm install forever -g
forever start FibonacciApp.js
```

- Evidencia 

![Instalacion de FibonacciApp en VM1 y VM2 - parte 1](images/Desarrollo/Instalacion_fibonacciApp_VM1_y_VM2_part1.png)
![Instalacion de FibonacciApp en VM1 y VM2 - parte 2](images/Desarrollo/Instalacion_fibonacciApp_VM1_y_VM2_part2.png)
Este proceso se realizo manualmente VM por VM. Para automatizar este aprovisionamiento se pueden usar herramientas como Azure Resource Manager, imagenes de disco, Terraform, Vagrant, Packer, Puppet o Ansible.

##### Probar el resultado final de la infraestructura

El endpoint de acceso del sistema es la IP publica del Load Balancer:

```
http://52.155.223.248/
http://52.155.223.248/fibonacci/1
```

![Prueba endpoint raiz en balanceador](images/Desarrollo/prueba_balanceador_de_carga_ip.png)
![Prueba endpoint fibonacci en balanceador](images/Desarrollo/prueba_balanceador_de_carga_ip_fibonacci.png)

Luego ejecute pruebas de carga con Newman (como en Parte 1) y compare resultados de rendimiento y costo entre:

- Infraestructura con escalabilidad vertical (1 VM de mayor tamano).
- Infraestructura con escalabilidad horizontal (varias VMs detras de Load Balancer).

##### Informe comparativo (Parte 1 vs Parte 2)

| Criterio | Parte 1 (Vertical) | Parte 2 (Horizontal con LB) |
|---|---|---|
| Latencia bajo concurrencia | Alta variabilidad y picos altos | Menor variabilidad por distribucion de carga |
| Tasa de exito | Afectada por saturacion en nodo unico | Mayor tasa de exito al repartir peticiones |
| Punto unico de falla | Si | Se reduce significativamente |
| Escalado | Cambiar SKU (downtime posible) | Agregar/quitar nodos progresivamente |
| Costo | Menor complejidad, pero limitado en crecimiento | Mayor costo base (LB + mas VMs), mejor relacion costo/rendimiento bajo carga |

##### Informe de newman 1 (Punto 2)

Observaciones principales del comportamiento con balanceo horizontal:

1. Se incrementa la cantidad de peticiones exitosas cuando la carga se reparte entre varios nodos.
2. El uso de CPU por VM se distribuye de forma mas uniforme frente al escenario de nodo unico.
3. La probabilidad de timeout/errores por saturacion disminuye al aumentar el numero de instancias.
4. Al pasar de 2 a 4 ejecuciones paralelas de Newman, el sistema responde mejor porque existe mayor capacidad de procesamiento concurrente en backend pool.

![Newman paralelo VM1 y VM2 - parte 1](images/Desarrollo/prueba_newman_en_paralelo_VM1_y_VM2_part1.png)
![Newman paralelo VM1 y VM2 - parte 2](images/Desarrollo/prueba_newman_en_paralelo_VM1_y_VM2_part2.png)

## Parte 2 - Respuestas a Preguntas

### 1. Tipos de balanceadores en Azure, SKU e IP publica

- Tipos de balanceadores:
	- Public Load Balancer: expone servicios hacia Internet.
	- Internal Load Balancer: balancea trafico privado dentro de una VNet.
- SKU:
	- Basic: menor capacidad, menos caracteristicas, menor resiliencia.
	- Standard: mayor capacidad, mejor seguridad/escala, recomendado para produccion.
- Por que necesita IP publica:
	- Para recibir trafico entrante desde clientes externos y enrutarlo a las VMs del backend.

### 2. Proposito del Backend Pool

El Backend Pool define el conjunto de instancias (NICs/VMs) que recibiran trafico distribuido por el balanceador.

### 3. Proposito del Health Probe

El Health Probe verifica periodicamente el estado de cada nodo (por ejemplo, puerto o endpoint). Si una instancia falla, deja de recibir trafico hasta recuperarse.

### 4. Proposito de la Load Balancing Rule y persistencia de sesion

- La Load Balancing Rule define como se mapea trafico de frontend (IP/puerto) hacia backend (pool/puerto/protocolo/probe).
- Tipos de persistencia de sesion en Azure Load Balancer:
	- None (5-tuple hash): distribucion sin afinidad fija de cliente.
	- Client IP: afinidad por IP de origen.
	- Client IP and Protocol: afinidad por IP de origen y protocolo.
- Importancia:
	- Puede ayudar a apps con estado en memoria por nodo.
	- Si se usa excesivamente, puede desbalancear carga y afectar escalabilidad horizontal.

### 5. Virtual Network, Subnet, address space y address range

- Virtual Network (VNet): red privada logica en Azure para aislar y conectar recursos.
- Subnet: segmento de la VNet para organizar y aplicar politicas/ruteo.
- Address space: bloque CIDR total de la VNet (ejemplo: `10.0.0.0/16`).
- Address range de subnet: sub-bloques dentro del address space (ejemplo: `10.0.1.0/24`).

Sirven para planear direccionamiento IP, separar capas y evitar conflictos de red.

### 6. Availability Zone e IP zone-redundant

- Availability Zone: datacenters fisicamente separados dentro de una misma region.
- Se seleccionan 3 zonas para aumentar tolerancia a fallos y disponibilidad.
- IP zone-redundant: recurso de IP publica que se mantiene disponible frente a falla de una zona, no atado a una sola.

### 7. Proposito del Network Security Group

El NSG controla trafico entrante/saliente por reglas (IP, puerto, protocolo, prioridad). En este laboratorio se habilito el puerto `3000` para permitir el acceso a la app.

## Diagrama de despliegue 

```mermaid
flowchart LR
		U[Cliente] --> PIPLB[IP publica del Load Balancer]
		PIPLB --> LB[Azure Load Balancer Standard]
		LB --> BP[Backend Pool]
		BP --> VM1[VM1 - Zona 1]
		BP --> VM2[VM2 - Zona 2]
		BP --> VM3[VM3 - Zona 3]

		subgraph VNET[Virtual Network]
			VM1
			VM2
			VM3
		end

		NSG[Network Security Group\nInbound 3000] -. protege .-> VNET
		HP[Health Probe] --> LB
		LBR[Load Balancing Rule 80/3000] --> LB
```

### Pruebas end-to-end

Las pruebas E2E se realizan consumiendo el endpoint de Fibonacci y validando respuesta y tiempo.

```
http://<IP_PUBLICA_VM>:3000/fibonacci/1000000
```

Adicionalmente, para carga concurrente:

```
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10
```

## Despliegue

Notas adicionales para desplegar el sistema en un entorno real.

- El despliegue se realiza sobre varias VM Linux en Azure con un Balanceador de carga que las antecede.
- Se debe abrir el puerto 3000 en el NSG para exponer el servicio en cada una de estas.

## Construido Con

* [Node.js](https://nodejs.org/) - Entorno de ejecucion principal
* [Express](https://expressjs.com/) - Framework web
* [big-integer](https://www.npmjs.com/package/big-integer) - Manejo de enteros grandes
* [Newman](https://www.npmjs.com/package/newman) - Ejecucion de colecciones Postman por linea de comandos
* [Microsoft Azure](https://azure.microsoft.com/) - Infraestructura de VM y monitoreo

## Autores

* **Diego Andres Trivino** - *Elaboracion Laboratorio Inicial*
* **Rogerrdz** - *Desarrollo del Laboratorio*

## Licencia

Este proyecto esta licenciado bajo ISC (ver `FibonacciApp/package.json`).

## Agradecimientos

* Escuela Colombiana de Ingenieria
* Arquitecturas de Software - ARSW
* Documentacion de Azure y herramientas de monitoreo
