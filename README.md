### Escuela Colombiana de Ingeniería
### Arquitecturas de Software - ARSW
### Federico Barrios Meneses

## Escalamiento en Azure con Maquinas Virtuales, Sacale Sets y Service Plans

### Parte 0 - Entendiendo el escenario de calidad

Adjunto a este laboratorio usted podrá encontrar una aplicación totalmente desarrollada que tiene como objetivo calcular el enésimo valor de la secuencia de Fibonnaci.

**Escalabilidad**
Cuando un conjunto de usuarios consulta un enésimo número (superior a 1000000) de la secuencia de Fibonacci de forma concurrente y el sistema se encuentra bajo condiciones normales de operación, todas las peticiones deben ser respondidas y el consumo de CPU del sistema no puede superar el 70%.

### Parte 1 - Escalabilidad vertical

1. Información de la máquina virtual:

![](images/vminfo.png)

2. Conexión a la máquina por ssh:

![](images/vmconnection.png)

3. Node instalado:

![](images/nodeinstalled.png)

4. Instalación de la aplicación `FibonacciApp` en la VM:

![](images/installingfibonacciapp.png) 

5. Instalando *forever* y usándolo para ejecutar la aplicación:

![](images/installingfibonacciapp2.png)

6. Creación de la regla de puerto de entrada (*Inbound port rule*):

![](images/inboundportrule.png)

7. Probando el endpoint: http://20.96.126.69:3000/fibonacci/6

![](images/fibonaccitest1.png)

8. Tiempos de respuesta para el endpoint usando los siguintes valores:
    * 1000000
	![](images/1000000test.png)
    * 1010000
	![](images/1010000test.png)
    * 1020000
	![](images/1020000test.png)
    * 1030000
	![](images/1030000test.png)
    * 1040000
	![](images/1040000test.png)
    * 1050000
	![](images/1050000test.png)
    * 1060000
	![](images/1060000test.png)
    * 1070000
	![](images/1070000test.png)
    * 1080000
	![](images/1080000test.png)
    * 1090000
	![](images/1090000test.png)

El tiempo promedio de respuesta es de 24,091 segundos

9. Consumo de CPU luego de las pruebas:

![](images/useofcpu.png)

10. Simulación de carga concurrente a nuestro sistema:
    ```
    newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 & newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10
    ```
	
	![](images/newmanexecution.png)
	
	Resultados:
	
	![](images/newmanresults1.png)

11. La cantidad de CPU consumida con peticiones concurrentes:

![](images/useofcpuconcurrent.png)

12. Realizando una estrategia de Escalamiento Vertical, se cambió el tamaño de la máquina de B1ls a B2ms:

![](images/newsize.png)

13. Una vez el cambio se vea reflejado, se repitieron los pasos 8, 9 y 10.

	**Prueba I**: El promedio bajó de 24,091 a 20,627 segundos.

	* 1000000
	![](images/1000000.png)
    * 1010000
	![](images/1010000.png)
    * 1020000
	![](images/1020000.png)
    * 1030000
	![](images/1030000.png)
    * 1040000
	![](images/1040000.png)
    * 1050000
	![](images/1050000.png)
    * 1060000
	![](images/1060000.png)
    * 1070000
	![](images/1070000.png)
    * 1080000
	![](images/1080000.png)
    * 1090000
	![](images/1090000.png)
	* Uso de CPU:
	
	![](images/useofcpu2.png)
	
	**Prueba II**: Carga concurrente. Siguen habiendo errores en algunas peticiones, pero el tiempo en que se atienden de disminuyó.
	
	![](images/newmanresults2-1.png)
	
	* Uso de CPU:
	
	![](images/useofcpuconcurrent2.png)
	
13. Evaluación del escenario de calidad asociado al requerimiento no funcional de escalabilidad:

Implementando esta estrategia de Escalamiento Vertical, se logró una disminución significativa, de al rededor 4 segundos, en los tiempos en que se atienden las peticiones no concurrentes, el tiempo
pasó de 24,091 a 20,627 segundos. Mientras que realizando peticiones concurrentes no se logró del todo un mejoramiento en el requisito no funcional de escalabilidad, aunque el tiempo en que se atendieron bajó
de 4m 19s a 3m 28s y el consumo de CPU es notablemente más bajo en ambos casos, como se ve en las imágenes, sigue habiendo errores en estas peticiones.

**Preguntas**

1. ¿Cuántos y cuáles recursos crea Azure junto con la VM?

Además de la máquina virtual, se crean los siguientes 6 recursos:

![](images/vmresources.png)

2. ¿Brevemente describa para qué sirve cada recurso?

**Red virtual:** se encarga de la comunicación entre recursos de azure, cumple el mismo propósito que una red tradicional pero ofrece beneficios de Azure como escalabilidad y disponibilidad.

**Dirección IP pública:** esta dirección es asignada a un recurso hasta que este se elimine y permite que los recursos de azure se comuniquen con internet y otros servicios públicos de azure.

**Grupo de seguridad de red:** Permite definir reglas de entrada y salida que filtren el tráfico de red entrante o saliente de varios tipos de recursos de Azure.

**Interfaz de red:** permite que una máquina virtual Azure se comunique con internet y con recursos locales.

**Clave ssh:** es una clave que solo se puede obtener al momento de crear la máquina o se puede usar una ya existente, sirve para permitir la conexión ssh desde equipos externos.

**Disco:** es el almacenamiento de la máquina virtual de Azure.

3. ¿Al cerrar la conexión ssh con la VM, por qué se cae la aplicación que ejecutamos con el comando `npm FibonacciApp.js`? ¿Por qué debemos crear un *Inbound port rule* antes de acceder al servicio?

Al estar conectados a la máquina virtual mediante SSH, los procesos que ejecutemos serán, por llamarlos de alguna manera, "procesos de instancia" relacionados a esa conexión, por lo cual finalizarán con la conexión misma.

Se debe crear una regla de entrada en el puerto 3000 porque en caso de no hacerlo, es imposible acceder al equipo desde una máquina diferente, esta regla de puerto de entrada peremite esa comunicación mediante TCP,UDP o ICMP desde las 
direcciones IP que se indiquen.

4. Adjunte tabla de tiempos e interprete por qué la función tarda tando tiempo.

Tamaño B1ls:

![](images/table1.png)

Tamaño B2ms:

![](images/table2.png)

La función tarda mucho tiempo ya que se puede ver que hace siempre el proceso iterativo del cálculo de Fibonacci y no hace memorización.

5. Adjunte imágen del consumo de CPU de la VM e interprete por qué la función consume esa cantidad de CPU.

Cada petición consume muchos recursos de la cpu debido a que el programa calcula el número de manera iterativa y no evita el recálculo de las operaciones.

![](images/useofcpu.png)

6. Adjunte la imagen del resumen de la ejecución de Postman. Interprete:

- El tiempo promedio de ejecución para cada petición fue de 26.7s y se recibió un total de 1020.52 KB de información.
- Fallaron 5 peticiones concurrentes en la conexión debido a que el servidor no soporta concurrencia.

![](images/newmanresults2-1.png)

7. ¿Cuál es la diferencia entre los tamaños `B2ms` y `B1ls` (no solo busque especificaciones de infraestructura)?

El tamaño B1ls tiene menos capacidad, es mucho más económico y solo está disponible para linux a diferencia de el tamaño B2ms. Sin embargo, ambas máquinas son de uso general, y proporcionan un uso equilibrado de la CPU, son utilizadas para entornos de desarrollo y pruebas, por lo general el tráfico soportado por estas es bajo/medio.

**B1ls:**
- 1 VCPU o núcleos
- 0.5 GB de memoria RAM
- 2 Discos de datos
- Costo por mes: $4.56 USD

**B2ms:**
- 2 VCPU o núcleos
- 8 GB de memoria RAM
- 4 Discos de datos
- Costo por mes: $72.85 USD

8. ¿Aumentar el tamaño de la VM es una buena solución en este escenario?, ¿Qué pasa con la FibonacciApp cuando cambiamos el tamaño de la VM?

Aumentar el tamaño de la máquina puede significar un consumo menor de recursos, pero no implica una mejora significativa en los tiempos de respuesta de las peticiones ni en la capacidad de respuesta concurrente del sistema. Al cambiar el tamaño de la VM el programa de Fibonacci sigue funcionando de igual manera, solo que el consumo que este genera sobre los recursos de la VM es menor.

9. ¿Qué pasa con la infraestructura cuando cambia el tamaño de la VM? ¿Qué efectos negativos implica?

Cuando se cambia el tamaño de la VM la infraestructura no estará disponible por cierto tiempo, ya que al cambiar el tamaño se debe reiniciar y puede tardar unos minutos. Con lo cual la disponibilidad del sistema se verá afectada y las peticiones realizadas durante ese tiempo no se atenderán.

10. ¿Hubo mejora en el consumo de CPU o en los tiempos de respuesta? Si/No ¿Por qué?

Si hubo mejora en el uso de CPU ya que el consumo con peticiones concurrentes pasó del 18% al 10% al hacer el cambio de tamaño, esto se debe a que se disponen de más recursos para hacer los cálculos. Por otra parte, los tiempos de respues disminuyeron muy poco, 4 segundos en promedio.

11. Aumente la cantidad de ejecuciones paralelas del comando de postman a `4`. ¿El comportamiento del sistema es porcentualmente mejor?

### Parte 2 - Escalabilidad horizontal

#### Crear el Balanceador de Carga

Antes de continuar puede eliminar el grupo de recursos anterior para evitar gastos adicionales y realizar la actividad en un grupo de recursos totalmente limpio.

1. El Balanceador de Carga es un recurso fundamental para habilitar la escalabilidad horizontal de nuestro sistema, por eso en este paso cree un balanceador de carga dentro de Azure tal cual como se muestra en la imágen adjunta.

![](images/part2/part2-lb-create.png)

2. A continuación cree un *Backend Pool*, guiese con la siguiente imágen.

![](images/part2/part2-lb-bp-create.png)

3. A continuación cree un *Health Probe*, guiese con la siguiente imágen.

![](images/part2/part2-lb-hp-create.png)

4. A continuación cree un *Load Balancing Rule*, guiese con la siguiente imágen.

![](images/part2/part2-lb-lbr-create.png)

5. Cree una *Virtual Network* dentro del grupo de recursos, guiese con la siguiente imágen.

![](images/part2/part2-vn-create.png)

#### Crear las maquinas virtuales (Nodos)

Ahora vamos a crear 3 VMs (VM1, VM2 y VM3) con direcciones IP públicas standar en 3 diferentes zonas de disponibilidad. Después las agregaremos al balanceador de carga.

1. En la configuración básica de la VM guíese por la siguiente imágen. Es importante que se fije en la "Avaiability Zone", donde la VM1 será 1, la VM2 será 2 y la VM3 será 3.

![](images/part2/part2-vm-create1.png)

2. En la configuración de networking, verifique que se ha seleccionado la *Virtual Network*  y la *Subnet* creadas anteriormente. Adicionalmente asigne una IP pública y no olvide habilitar la redundancia de zona.

![](images/part2/part2-vm-create2.png)

3. Para el Network Security Group seleccione "avanzado" y realice la siguiente configuración. No olvide crear un *Inbound Rule*, en el cual habilite el tráfico por el puerto 3000. Cuando cree la VM2 y la VM3, no necesita volver a crear el *Network Security Group*, sino que puede seleccionar el anteriormente creado.

![](images/part2/part2-vm-create3.png)

4. Ahora asignaremos esta VM a nuestro balanceador de carga, para ello siga la configuración de la siguiente imágen.

![](images/part2/part2-vm-create4.png)

5. Finalmente debemos instalar la aplicación de Fibonacci en la VM. para ello puede ejecutar el conjunto de los siguientes comandos, cambiando el nombre de la VM por el correcto

```
git clone https://github.com/daprieto1/ARSW_LOAD-BALANCING_AZURE.git

curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.34.0/install.sh | bash
source /home/vm1/.bashrc
nvm install node

cd ARSW_LOAD-BALANCING_AZURE/FibonacciApp
npm install

npm install forever -g
forever start FibonacciApp.js
```

Realice este proceso para las 3 VMs, por ahora lo haremos a mano una por una, sin embargo es importante que usted sepa que existen herramientas para aumatizar este proceso, entre ellas encontramos Azure Resource Manager, OsDisk Images, Terraform con Vagrant y Paker, Puppet, Ansible entre otras.

Después de seguir los pasos se crearon los siguientes recursos:

![](images/recursos-loadbalancer.png)

#### Probar el resultado final de nuestra infraestructura

1. Porsupuesto el endpoint de acceso a nuestro sistema será la IP pública del balanceador de carga, primero verifiquemos que los servicios básicos están funcionando, consuma los siguientes recursos:

```
http://52.155.223.248/
http://52.155.223.248/fibonacci/1
```
![](images/parte2punto1a.png)

![](images/parte2punto1b.png)


2. Realice las pruebas de carga con `newman` que se realizaron en la parte 1 y haga un informe comparativo donde contraste: tiempos de respuesta, cantidad de peticiones respondidas con éxito, costos de las 2 infraestrucruras, es decir, la que desarrollamos con balanceo de carga horizontal y la que se hizo con una maquina virtual escalada.

3. Agregue una 4 maquina virtual y realice las pruebas de newman, pero esta vez no lance 2 peticiones en paralelo, sino que incrementelo a 4. Haga un informe donde presente el comportamiento de la CPU de las 4 VM y explique porque la tasa de éxito de las peticiones aumento con este estilo de escalabilidad.

```
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10
```

**Preguntas**

* ¿Cuáles son los tipos de balanceadores de carga en Azure y en qué se diferencian?, ¿Qué es SKU, qué tipos hay y en qué se diferencian?, ¿Por qué el balanceador de carga necesita una IP pública?

**Tipos de balanceadores de carga**
- Balanceador de carga público: Se utilizan para equilibrar la carga proveniente de internet a las máquinas virtuales, la dirección ip pública y el puerto del tráfico entrante son asignados a la dirección privada.
- Balanceador de carga interno (privado): Se utiliza para equilibrar la carga del tráfico dentro de una red privada, solo utilizan ips privadas en la interfaz


**Stock Keeping Unit (SKU):** Las unidades de mantenimiento de existencias son un código único asignado a un servicio en azure existen y representan la posibilidad para comprar existestencias. Existen varios tipos de sku
entre ellos: estándar, componente, ensamblaje, paquete, colección y virtual.

**¿Por qué el balanceador de carga necesita una IP pública?:** El balanceador de carga necesita una IP pública para poder cumplir con su función que es capturar el tráfico y distribuirlo correctamente entre los diferentes nodos de la red.


* ¿Cuál es el propósito del *Backend Pool*?

Backend Pool es un componente del balacenador de carga que define el grupo de recursos que brindarán tráfico para una Load Balancing Rule determinada, es un grupo de máquinas virtuales o instancias que atienden 
las solicitudes entrantes. Para escalar de manera rentable y satisfacer grandes volúmenes de tráfico entrante, generalmente se recomienda agregar más instancias a este grupo.

* ¿Cuál es el propósito del *Health Probe*?

Cuando se configura un nuevo balanceador de carga se crea una Health probe que usará el balanceador para determinar si las instancias dentro del Backend Pool están en buen estado, si una instancia falla un determinado
número de veces entonces esta dejará de dirigir tráfico hacia ella hasta que empiece a pasar las pruebas de estado nuevamente.

* ¿Cuál es el propósito de la *Load Balancing Rule*? ¿Qué tipos de sesión persistente existen, por qué esto es importante y cómo puede afectar la escalabilidad del sistema?.

**Propósito de la Load Balancing Rule:** Una Load Balancing Rule de un Load Balancer se usa para definir la manera de distibuir el tráfico entrante a todas las instancias dentro del Backend Pool.

**Tipos de sesión persistente:** Existen dos tipos de sesiones persistentes que son: None(Hash-based) y Client IP.

* ¿Qué es una *Virtual Network*? ¿Qué es una *Subnet*? ¿Para qué sirven los *address space* y *address range*?

**Virtual network:** Es una tecnología de red que permite extender la red de área local sobre una red pública o no controlada (Internet). permite enviar y recibir datos sobre redes compartidas o públicas comportandose como 
una red privada aprovechando la funcionalidad, seguridad y políticas de gestión de una red privada. se implementan realizando conexiones dedicadas y/o cifrado.

**Subnet:** Es una segmentación de la red virtual (o cualquier red en general), permiten asignar una o varias subredes a la misma, estas subredes cuentan con un rango de direcciones apropiadas para una organización adecuada.

**Address space:** Sirve para especificar un rango de direcciones ip privadas personalizadas. Azure asigna a los recursos de una red virtual una dirección IP privada desde el espacio de direcciones que asigne.

**Address range:** Indica cuantas direcciones se tienen en un address space.

* ¿Qué son las *Availability Zone* y por qué seleccionamos 3 diferentes zonas?. ¿Qué significa que una IP sea *zone-redundant*?

**Availability Zones:** Son Ubicaciones geográficas únicas dentro de una región. Cada zona se compone de varios centros de datos equipados con alimentación, refrigeración y redes independientes.

Seleccionamos 3 zonas de disponibilidad diferentes para poder tener una mejor disponibiliad y tolerancia a fallos dentro del sistema.

**Zone-redundant:** Cuando utilizamos una ip zone-redundant azure separa física y lógicamente el gateway dentro de una region, lo cual permite mejorar la conectividad de la red privada.

* ¿Cuál es el propósito del *Network Security Group*?

Permite filtrar el tráfico hacia y desde los recursos en una red virtual de Azure, un grupo de seguridad permite definir reglas de entrada y salida que permitan o denieguen el tráfico de red entrante o saliente.




