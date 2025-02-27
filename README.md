### Escuela Colombiana de Ingeniería
### Arquitecturas de Software - ARSW

## Escalamiento en Azure con Maquinas Virtuales, Sacale Sets y Service Plans

### Dependencias
* Cree una cuenta gratuita dentro de Azure. Para hacerlo puede guiarse de esta [documentación](https://azure.microsoft.com/es-es/free/students/). Al hacerlo usted contará con $100 USD para gastar durante 12 meses.

### Parte 0 - Entendiendo el escenario de calidad

Adjunto a este laboratorio usted podrá encontrar una aplicación totalmente desarrollada que tiene como objetivo calcular el enésimo valor de la secuencia de Fibonnaci.

**Escalabilidad**
Cuando un conjunto de usuarios consulta un enésimo número (superior a 1000000) de la secuencia de Fibonacci de forma concurrente y el sistema se encuentra bajo condiciones normales de operación, todas las peticiones deben ser respondidas y el consumo de CPU del sistema no puede superar el 70%.

### Parte 1 - Escalabilidad vertical

1. Diríjase a el [Portal de Azure](https://portal.azure.com/) y a continuación cree una maquina virtual con las características básicas descritas en la imágen 1 y que corresponden a las siguientes:
    * Resource Group = SCALABILITY_LAB
    * Virtual machine name = VERTICAL-SCALABILITY
    * Image = Ubuntu Server 
    * Size = Standard B1ls
    * Username = scalability_lab
    * SSH publi key = Su llave ssh publica

![Imágen 1](images/part1/part1-vm-basic-config.png)

2. Para conectarse a la VM use el siguiente comando, donde las `x` las debe remplazar por la IP de su propia VM (Revise la sección "Connect" de la virtual machine creada para tener una guía más detallada).

    `ssh scalability_lab@xxx.xxx.xxx.xxx`

3. Instale node, para ello siga la sección *Installing Node.js and npm using NVM* que encontrará en este [enlace](https://linuxize.com/post/how-to-install-node-js-on-ubuntu-18.04/).
4. Para instalar la aplicación adjunta al Laboratorio, suba la carpeta `FibonacciApp` a un repositorio al cual tenga acceso y ejecute estos comandos dentro de la VM:

    `git clone <your_repo>`

    `cd <your_repo>/FibonacciApp`

    `npm install`

5. Para ejecutar la aplicación puede usar el comando `npm FibinacciApp.js`, sin embargo una vez pierda la conexión ssh la aplicación dejará de funcionar. Para evitar ese compartamiento usaremos *forever*. Ejecute los siguientes comando dentro de la VM.

    ` node FibonacciApp.js`

6. Antes de verificar si el endpoint funciona, en Azure vaya a la sección de *Networking* y cree una *Inbound port rule* tal como se muestra en la imágen. Para verificar que la    aplicación funciona, use un browser y user el endpoint `http://xxx.xxx.xxx.xxx:3000/fibonacci/6`. La respuesta debe ser `The answer is 8`.
![](images/part1/part1-vm-3000InboudRule.png)

   Evidencia:

   ![image](https://github.com/Tianrojas/Lab9-ARSW_LOAD-BALANCING_AZURE/assets/62759668/599689c9-7613-46f8-8e62-e704c8719d44)
   

8. La función que calcula en enésimo número de la secuencia de Fibonacci está muy mal construido y consume bastante CPU para obtener la respuesta. Usando la consola del Browser documente los tiempos de respuesta para dicho endpoint usando los siguintes valores:
    * 1000000
    * 1010000
    * 1020000
    * 1030000
    * 1040000
    * 1050000
    * 1060000
    * 1070000
    * 1080000
    * 1090000
     
    ![image](https://github.com/Tianrojas/Lab9-ARSW_LOAD-BALANCING_AZURE/assets/62759668/5553fdaa-c100-4f25-8cdc-f784de758c75)
 

9. Dírijase ahora a Azure y verifique el consumo de CPU para la VM. (Los resultados pueden tardar 5 minutos en aparecer).
   ![Imágen 2](images/part1/part1-vm-cpu.png)

   Evidencia:
   ![image](https://github.com/Tianrojas/Lab9-ARSW_LOAD-BALANCING_AZURE/assets/62759668/0af37f0c-d7e7-4257-99e5-f9e9f081425a)


9. Ahora usaremos Postman para simular una carga concurrente a nuestro sistema. Siga estos pasos.
    * Instale newman con el comando `npm install newman -g`. Para conocer más de Newman consulte el siguiente [enlace](https://learning.getpostman.com/docs/postman/collection-runs/command-line-integration-with-newman/).
    * Diríjase hasta la ruta `FibonacciApp/postman` en una maquina diferente a la VM.
    * Para el archivo `[ARSW_LOAD-BALANCING_AZURE].postman_environment.json` cambie el valor del parámetro `VM1` para que coincida con la IP de su VM.
    * Ejecute el siguiente comando.

    ```
    newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
    newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10
    ```
    ![image](https://github.com/Tianrojas/Lab09-ARSW_LOAD-BALANCING_AZURE/assets/62759668/596522d2-ce08-45f8-a318-052d8403e031)
    ![image](https://github.com/Tianrojas/Lab09-ARSW_LOAD-BALANCING_AZURE/assets/62759668/e5f871b6-7d5d-4f29-9a29-ba5ad7b756c4)

  
    


10. La cantidad de CPU consumida es bastante grande y un conjunto considerable de peticiones concurrentes pueden hacer fallar nuestro servicio. Para solucionarlo usaremos una estrategia de Escalamiento Vertical. En Azure diríjase a la sección *size* y a continuación seleccione el tamaño `B2ms`.

    ![Imágen 3](images/part1/part1-vm-resize.png)

11. Una vez el cambio se vea reflejado, repita el paso 7, 8 y 9.
    ![image](https://github.com/Tianrojas/Lab09-ARSW_LOAD-BALANCING_AZURE/assets/62759668/ca638b12-1e6a-4289-a0c5-d49ea9ad5cac)
    ![image](https://github.com/Tianrojas/Lab09-ARSW_LOAD-BALANCING_AZURE/assets/62759668/cee8ac4e-1378-4345-92fd-14b2eda38adf)
    ![image](https://github.com/Tianrojas/Lab09-ARSW_LOAD-BALANCING_AZURE/assets/62759668/99f13aab-4b20-462c-af2e-317af997da08)
    ![image](https://github.com/Tianrojas/Lab09-ARSW_LOAD-BALANCING_AZURE/assets/62759668/864b71d5-3845-4d4a-9175-a1308b17b4c4)  

      
  
13. Evalue el escenario de calidad asociado al requerimiento no funcional de escalabilidad y concluya si usando este modelo de escalabilidad logramos cumplirlo. \
    El escenario de calidad delineado se ha validado de manera efectiva mediante la aplicación de escalabilidad vertical. Al incrementar las características de la máquina virtual, se optimiza de manera más eficiente, generando una reducción significativa en los tiempos de respuesta de las solicitudes en comparación con situaciones anteriores. Esta mejora se refleja de manera clara en la representación visual, donde se evidencia la disminución notable en el consumo de CPU de la máquina virtual.


14. Vuelva a dejar la VM en el tamaño inicial para evitar cobros adicionales.

**Preguntas**

1. ¿Cuántos y cuáles recursos crea Azure junto con la VM?
   * Grupo de recursos
   * Red virtual (VNet)
   * Interfaz de red (NIC)
   * Dirección IP pública
   * Discos
   * Grupo de seguridad de red (NSG)
   * Almacenamiento
2. ¿Brevemente describa para qué sirve cada recurso?
   * Grupo de recursos: Un contenedor que almacena recursos relacionados para una aplicación Azure. El grupo de recursos incluye aquellos recursos que se crean con la VM, como las redes virtuales y los discos.
   * Red virtual (VNet): Cada VM se conecta a una red virtual en la que puede comunicarse con otras VM dentro de la misma red.
   * Interfaz de red (NIC): Cada VM tiene una o más NIC para comunicarse con la red virtual.
   * Dirección IP pública: Si se configura, la VM puede tener una dirección IP pública para comunicarse con Internet.
   * Discos: Cada VM tiene al menos un disco para el sistema operativo y puede tener discos adicionales para datos.
   * Grupo de seguridad de red (NSG): Un NSG contiene reglas de seguridad que permiten o deniegan el tráfico de red entrante a la VM.
   * Almacenamiento: Los discos de las VM se almacenan en una cuenta de almacenamiento de Azure 
3. ¿Al cerrar la conexión ssh con la VM, por qué se cae la aplicación que ejecutamos con el comando `npm FibonacciApp.js`? ¿Por qué debemos crear un *Inbound port rule* antes de acceder al servicio? \
   Cuando se ejecuta una aplicación en una sesión SSH y luego se cierra la sesión, la aplicación también se cierra. Esto se debe a que la aplicación se está ejecutando en el contexto de la sesión SSH. Cuando la sesión se cierra, todos los procesos asociados a esa sesión también se cierran. \
Para mantener la aplicación en ejecución después de cerrar la sesión SSH, se puede utilizar herramientas como screen, tmux o nohup. Estas herramientas permiten que los procesos continúen ejecutándose en segundo plano, incluso después de cerrar la sesión SSH1. \
En cuanto a la creación de una regla de puerto entrante (Inbound port rule), es necesaria para permitir el tráfico entrante a la aplicación desde fuera de la red de Azure. Azure, al igual que muchos otros proveedores de servicios en la nube, bloquea todo el tráfico entrante por defecto por razones de seguridad. Por lo tanto, se debe crear una regla de puerto entrante para permitir las conexiones a tu aplicación.

4. Adjunte tabla de tiempos e interprete por qué la función tarda tando tiempo.
   | N             | Standard_B1ls (s)    | Standars_B2ms (s) |
   |---------------|----------------------|-------------------|
   | 1000000       | 20.91 s              | 15.78 s           |
   | 1010000       | 42.24 s              | 31.70 s           |
   | 1020000       | 63.86 s              | 47.63 s           |
   | 1030000       | 13.18 s              | 63.99 s           |
   | 1040000       | 20.97 s              | 80.69 s           |
   | 1050000       | 86.90 s              | 97.68 s           |
   | 1060000       | 20.39 s              | 11.50 s           |
   | 1070000       | 15.60 s              | 13.34 s           |
   | 1080000       | 18.04 s              | 17.12 s           | 
   | 1090000       | 22.91 s              | 15.27 s           |
5. Adjunte imágen del consumo de CPU de la VM e interprete por qué la función consume esa cantidad de CPU.
   ![image](https://github.com/Tianrojas/Lab09-ARSW_LOAD-BALANCING_AZURE/assets/62759668/cee8ac4e-1378-4345-92fd-14b2eda38adf) \
   Cada solicitud ejecutada ejerce una carga significativa en la unidad central de procesamiento (CPU) debido a la ejecución de cálculos redundantes. Esto se debe a la falta de aplicación de métodos o técnicas, como la memorización, que posibiliten el almacenamiento de cálculos previamente realizados. Además, la ausencia de implementación de concurrencia resulta en un mayor consumo de recursos y un tiempo de respuesta prolongado.
6. Adjunte la imagen del resumen de la ejecución de Postman. Interprete:
    * Tiempos de ejecución de cada petición.
    * Si hubo fallos documentelos y explique.
      ![image](https://github.com/Tianrojas/Lab09-ARSW_LOAD-BALANCING_AZURE/assets/62759668/99f13aab-4b20-462c-af2e-317af997da08)
      ![image](https://github.com/Tianrojas/Lab09-ARSW_LOAD-BALANCING_AZURE/assets/62759668/864b71d5-3845-4d4a-9175-a1308b17b4c4) \
      Los tiempos de ejecución aunque tienen gran diferencia a los iniciales, se siguen presentando fallos, se cree que es debido a la estabilidad de la conexión de la maquina virtual. 
7. ¿Cuál es la diferencia entre los tamaños `B2ms` y `B1ls` (no solo busque especificaciones de infraestructura)? \
   Los tamaños B2ms y B1ls pertenecen a la serie B de Azure, que son máquinas virtuales ampliables. Estas máquinas virtuales están diseñadas para cargas de trabajo que no necesitan el rendimiento completo de la CPU de forma continua, como servidores web, pruebas de concepto, bases de datos pequeñas y entornos de desarrollo. \
   La principal diferencia entre B2ms y B1ls radica en sus capacidades y uso previsto:
      * B2ms: Este tamaño de máquina virtual tiene 2 vCPU, 8 GiB de memoria y 16 GiB de almacenamiento temporal (SSD). El rendimiento base de la CPU de la máquina virtual es del 60%, con créditos iniciales de 60 y puede acumular hasta 864 créditos1. Este tamaño es adecuado para cargas de trabajo que requieren más memoria y capacidad de CPU.
      * B1ls: Este tamaño de máquina virtual tiene 1 vCPU, 0.5 GiB de memoria y 4 GiB de almacenamiento temporal (SSD). El rendimiento base de la CPU de la máquina virtual es del 10%, con créditos iniciales de 30 y puede acumular hasta 72 créditos1. Este tamaño es el más pequeño y menos costoso disponible en Azure y es adecuado para cargas de trabajo muy ligeras o para pruebas de concepto1.
8. ¿Aumentar el tamaño de la VM es una buena solución en este escenario?, ¿Qué pasa con la FibonacciApp cuando cambiamos el tamaño de la VM? \
   A pesar de que incrementar las dimensiones de la máquina virtual resulta en una reducción del consumo de recursos (como se evidenció en la utilización de la CPU), aún es necesario reconsiderar la estructura de FibonacciApp para que memorice los valores previamente calculados. Este ajuste busca optimizar aún más los tiempos de respuesta de la aplicación. \
No obstante, es crucial tener en cuenta que solo un servidor está manejando todas las solicitudes web, lo que significa que no se garantiza una disponibilidad constante en caso de una avalancha de peticiones.
9. ¿Qué pasa con la infraestructura cuando cambia el tamaño de la VM? ¿Qué efectos negativos implica?
   La reinicialización de la máquina virtual implica la necesidad de establecer una nueva conexión segura mediante el protocolo SSH. Este reinicio resulta en un periodo en el cual la máquina no está disponible para ofrecer sus servicios web, lo que conlleva a la falta de atención a todas las peticiones que se realicen durante dicho lapso.
10. ¿Hubo mejora en el consumo de CPU o en los tiempos de respuesta? Si/No ¿Por qué?
   Si, dado que la máquina virtual dispone de más recursos para realizar sus cálculos y atender a las peticiones.
11. Aumente la cantidad de ejecuciones paralelas del comando de postman a `4`. ¿El comportamiento del sistema es porcentualmente mejor?
   Utilizando, con el size inicial
   ```
   newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 4
   ```
   Se obtiene: \
   ![image](https://github.com/Tianrojas/Lab09-ARSW_LOAD-BALANCING_AZURE/assets/62759668/ab1328b6-d058-478b-a377-297995b067ce)   
   ![image](https://github.com/Tianrojas/Lab09-ARSW_LOAD-BALANCING_AZURE/assets/62759668/55e30c8c-5f46-401b-b02d-189f19c113c5)
   ![image](https://github.com/Tianrojas/Lab09-ARSW_LOAD-BALANCING_AZURE/assets/62759668/8bd29f41-ac1e-40fc-ac9f-0ad76e17d364)
   ![image](https://github.com/Tianrojas/Lab09-ARSW_LOAD-BALANCING_AZURE/assets/62759668/9315651e-ed11-4620-ae7a-9860378d37cd) \
   No mejora.
   
   

   

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

Se crearon 2VM sin problemas, en la tercera hubieron problemas en los creditos de azure
![image](https://github.com/Tianrojas/Lab09-ARSW_LOAD-BALANCING_AZURE/assets/62759668/69b5cc04-4000-47cd-ae47-d298adb88cbb)
![image](https://github.com/Tianrojas/Lab09-ARSW_LOAD-BALANCING_AZURE/assets/62759668/66145ebc-6784-4339-b554-386672f12f26)

El balanceador
![image](https://github.com/Tianrojas/Lab09-ARSW_LOAD-BALANCING_AZURE/assets/62759668/ad05fb43-bb70-4787-abc1-67adec580c2e)

Y el resto de componentes
![image](https://github.com/Tianrojas/Lab09-ARSW_LOAD-BALANCING_AZURE/assets/62759668/1ddf6ea5-1f2b-4078-a563-57418bb1ebe5)


#### Probar el resultado final de nuestra infraestructura

1. Porsupuesto el endpoint de acceso a nuestro sistema será la IP pública del balanceador de carga, primero verifiquemos que los servicios básicos están funcionando, consuma los siguientes recursos:

```
http://52.155.223.248/
http://52.155.223.248/fibonacci/1
```
![image](https://github.com/Tianrojas/Lab09-ARSW_LOAD-BALANCING_AZURE/assets/62759668/160bb58e-f85c-42fc-8c8f-ec47218f3db9)
![image](https://github.com/Tianrojas/Lab09-ARSW_LOAD-BALANCING_AZURE/assets/62759668/4190cb5a-a61f-43b3-a984-7cc515831d3e)

2. Realice las pruebas de carga con `newman` que se realizaron en la parte 1 y haga un informe comparativo donde contraste: tiempos de respuesta, cantidad de peticiones respondidas con éxito, costos de las 2 infraestrucruras, es decir, la que desarrollamos con balanceo de carga horizontal y la que se hizo con una maquina virtual escalada.

    |Escalamiento|Costos    |Tiempo promedio de respuesta|Peticiones Exitosas|
    |:----------:|:--------:|:--------------------------:|:-----------------:|
    |  Vertical  | $66.48  |             25.3s           |         20        |
    | Horizontal |  $16.69  |             26.6s          |         19        |

3. Agregue una 4 maquina virtual y realice las pruebas de newman, pero esta vez no lance 2 peticiones en paralelo, sino que incrementelo a 4
4. Haga un informe donde presente el comportamiento de la CPU de las 4 VM y explique porque la tasa de éxito de las peticiones aumento con este estilo de escalabilidad.

```
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10
```
   * VM1 \
     ![image](https://github.com/Tianrojas/Lab09-ARSW_LOAD-BALANCING_AZURE/assets/62759668/7e68d64b-add0-4036-9767-54fc0ea405ba)
   * VM2 \
     ![image](https://github.com/Tianrojas/Lab09-ARSW_LOAD-BALANCING_AZURE/assets/62759668/9b2bb35d-abd4-4298-84de-c752f66d08f2)  

**Preguntas**

* ¿Cuáles son los tipos de balanceadores de carga en Azure y en qué se diferencian?, ¿Qué es SKU, qué tipos hay y en qué se diferencian?, ¿Por qué el balanceador de carga necesita una IP pública?
    * Tipos de balanceadores:
        * **Balanceador de carga interno( Privado ):** Este balanceador de carga se encarga de equilibrar la carga de trafico de una red privada ( se utilizan unicamente direcciones ip privadas en la interfaz).

        * **Balanceador de carga publico:** Este balanceador de carga se encarga de equilibrar la carga de trafico de redes publicas, especificamente de la carga proveniente de internet, la dirección ip pública y el puerto son asigandos. 

        * **Balanceador de carga de puerta de enlace:** Es un balanceador que se adapta a escenarios de alto rendimiento y alta disponibilidad con dispositivos virtuales de red (NVA) de terceros. Con las capacidades de Gateway Load Balancer, puede implementar, escalar y administrar NVA fácilmente. Encadenar un balanceador de carga de puerta de enlace a su punto final público solo requiere un clic.
        
    * SKU( Stock Keeping Unit) Azure:
        * Representa una unidad de mantenimiento de existencias, lo cual significa que es un codigo unico asignado a un servicio o producto de Azure, el cual nos permite a nosotros como usuarios la posibilidad de comprar existencias de los mismos.
    * Tipos de SKU:
        *  **Estándar:** Productos estandar los cuales se pueden vender de manera individual o en paquetes, 
        *  **Esamblaje: Aquellos productos que se deben ensamblar antes de un envio, todos los SKU deberan encontrarse dentro de una misma instalacion.** 
        *  **Virtual: Aquellos productos que son virtuales, es decir que no necesitan de una instalacion fisica evitando asi un nivel de inventario** 
        *  **Componente: Productos incluidos en paquetes, esamblajes y colecciones, los cuales no se pueden vender de manera individual** 

* ¿Cuál es el propósito del *Backend Pool*?
   El Backend Pool constituye una parte esencial del equilibrador de carga, siendo responsable de especificar el conjunto de recursos encargados de gestionar el tráfico asociado a una regla de equilibrio determinada. Este conjunto se compone de máquinas virtuales o instancias encargadas de procesar las solicitudes que llegan. Para lograr una expansión eficiente y hacer frente a grandes volúmenes de tráfico entrante, se suele sugerir la adición de más instancias a este grupo.
* ¿Cuál es el propósito del *Health Probe*?
   Al configurar un nuevo equilibrador de carga, se establece una sonda de salud que el balanceador utiliza para evaluar el estado de las instancias dentro del Backend Pool. Si una instancia falla un número predefinido de veces, el balanceador dejará de dirigir tráfico hacia esa instancia hasta que supere nuevamente las pruebas de estado.
* ¿Cuál es el propósito de la *Load Balancing Rule*? ¿Qué tipos de sesión persistente existen, por qué esto es importante y cómo puede afectar la escalabilidad del sistema?.
   Una Load Balancing Rule de un Load Balancer se usa para definir la manera de distibuir el tráfico entrante a todas las instancias dentro del Backend Pool. En Azure existen tres tipos de sesión de persistencia:
   * None (hash-based): Especifica que las solicitudes sucesivas del mismo cliente pueden ser manejadas por cualquier máquina virtual. Los paquetes de la misma sesión TCP o UDP se dirigirán a la misma instancia de IP del Datacenter (DIP), pero cuando el cliente cierra y vuelve a abrir la conexión o inicia una nueva sesión desde la misma IP de origen, el puerto de origen cambia y hace que el tráfico vaya a un DIP diferente. \
     ![image](https://github.com/Tianrojas/Lab09-ARSW_LOAD-BALANCING_AZURE/assets/62759668/54279c48-6047-46d8-a57e-0ab9774893e8)
   *   Client IP (source IP affinity 2-tuple o 3-tuple): Especifica que las peticiones sucesivas de la misma dirección IP del cliente serán gestionadas por la misma máquina virtual. Azure Load Balancer se puede configurar para usar 2 tuplas (IP de origen, IP de destino) o 3 tuplas (IP de origen, IP de destino, Protocolo) para asignar el tráfico a los servidores disponibles. \
     ![image](https://github.com/Tianrojas/Lab09-ARSW_LOAD-BALANCING_AZURE/assets/62759668/02e7f81a-c62c-487e-aeda-21d46e491629) 
 
* ¿Qué es una *Virtual Network*? ¿Qué es una *Subnet*? ¿Para qué sirven los *address space* y *address range*?
    * Red Virtual: Una red virtual es una infraestructura de red que posibilita la interconexión de dispositivos y máquinas virtuales mediante software, sin depender de una ubicación física específica.

    * Subnet: Se trata de una subdivisión de una red física o virtual. Estas divisiones cuentan con su propio rango de direcciones IP, determinado por la forma en que se particiona la dirección IP original.

    * Address Space: Al crear una red virtual, es necesario especificar un conjunto de direcciones IP que no se superpongan entre sí. En otras palabras, el address space es la dirección IP que identifica única y exclusivamente a esa red, por ejemplo, 10.0.0.0/24.

    * Address Range: Define la cantidad de direcciones que están disponibles o pueden estarlo en un address space. Dependiendo de los recursos necesarios en la red virtual, el rango puede aumentar o disminuir, adaptándose a las exigencias específicas de la infraestructura.
* ¿Qué son las *Availability Zone* y por qué seleccionamos 3 diferentes zonas?. ¿Qué significa que una IP sea *zone-redundant*?
    * Zonas de Disponibilidad:
Las Zonas de Disponibilidad son ubicaciones geográficas únicas dentro de una región. Cada zona está compuesta por uno o más centros de datos equipados con sistemas independientes de alimentación, refrigeración y redes. La elección de tres zonas de disponibilidad diferentes se realiza con el objetivo de mejorar la disponibilidad y la tolerancia a fallos del sistema. En caso de que alguno de los centros de datos mencionados falle, el load balancer redirigirá el tráfico hacia otro nodo de la red ubicado en una geolocalización diferente. Este enfoque garantiza resiliencia y reduce la probabilidad de que el sistema esté no disponible.

    * IP Zone-Redundant:
Un gateway con redundancia de zona IP aporta resistencia, escalabilidad y disponibilidad al sistema. Al emplear una IP zone-redundant en Azure, se logra la separación física y lógica del gateway dentro de una región. Este enfoque mejora la conectividad de la red privada y reduce los fallos a nivel de zona de disponibilidad. La redundancia de zona en la dirección IP asegura que, en caso de problemas en una zona específica, la conectividad persista a través de otras zonas, fortaleciendo así la robustez del sistema.
* ¿Cuál es el propósito del *Network Security Group*?
    El Network Security Group cumple con el propósito de filtrar el tráfico que fluye hacia y desde los recursos en una red virtual de Azure. Este grupo de seguridad posibilita la definición de reglas de entrada y/o salida que controlan la autorización o denegación del tráfico de red, tanto entrante como saliente, para diversos tipos de recursos en la plataforma de Azure. En esencia, el NSG ofrece un mecanismo flexible y personalizable para gestionar de manera precisa la seguridad de la red, permitiendo a los usuarios especificar las condiciones bajo las cuales se permite o bloquea el intercambio de datos entre los recursos en la infraestructura de Azure.
* Informe de newman 1 (Punto 2)
* Presente el Diagrama de Despliegue de la solución. Suponiendo que se permite la creacion de las 4 VM
   ![image](https://github.com/Tianrojas/Lab09-ARSW_LOAD-BALANCING_AZURE/assets/62759668/967cea75-ff05-4270-9b64-965b7408e8fa)





