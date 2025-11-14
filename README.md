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

![](images/lab/1.png)
![](images/lab/2.png)
![](images/lab/3.png)

2. Para conectarse a la VM use el siguiente comando, donde las `x` las debe remplazar por la IP de su propia VM (Revise la sección "Connect" de la virtual machine creada para tener una guía más detallada).

    `ssh scalability_lab@xxx.xxx.xxx.xxx`

    ![](images/lab/4.png)
    ![](images/lab/5.png)

3. Instale node, para ello siga la sección *Installing Node.js and npm using NVM* que encontrará en este [enlace](https://linuxize.com/post/how-to-install-node-js-on-ubuntu-18.04/).

![](images/lab/6.png)

4. Para instalar la aplicación adjunta al Laboratorio, suba la carpeta `FibonacciApp` a un repositorio al cual tenga acceso y ejecute estos comandos dentro de la VM:

    `git clone <your_repo>`

    `cd <your_repo>/FibonacciApp`

    `npm install`

![](images/lab/7.png)   
![](images/lab/8.png) 
![](images/lab/9.png)

5. Para ejecutar la aplicación puede usar el comando `npm FibinacciApp.js`, sin embargo una vez pierda la conexión ssh la aplicación dejará de funcionar. Para evitar ese compartamiento usaremos *forever*. Ejecute los siguientes comando dentro de la VM.

    ` node FibonacciApp.js`

![](images/lab/10.png)

6. Antes de verificar si el endpoint funciona, en Azure vaya a la sección de *Networking* y cree una *Inbound port rule* tal como se muestra en la imágen. Para verificar que la aplicación funciona, use un browser y user el endpoint `http://xxx.xxx.xxx.xxx:3000/fibonacci/6`. La respuesta debe ser `The answer is 8`.

![](images/part1/part1-vm-3000InboudRule.png)

![](images/lab/11.png)
![](images/lab/12.png)

7. La función que calcula en enésimo número de la secuencia de Fibonacci está muy mal construido y consume bastante CPU para obtener la respuesta. Usando la consola del Browser documente los tiempos de respuesta para dicho endpoint usando los siguintes valores:
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

![](images/lab/13.png)
![](images/lab/14.png)
![](images/lab/15.png)
![](images/lab/16.png)
![](images/lab/17.png)
![](images/lab/18.png)
![](images/lab/19.png)
![](images/lab/20.png)
![](images/lab/21.png)
![](images/lab/22.png)

8. Dírijase ahora a Azure y verifique el consumo de CPU para la VM. (Los resultados pueden tardar 5 minutos en aparecer).

![Imágen 2](images/part1/part1-vm-cpu.png)

![](images/lab/23.png)

9. Ahora usaremos Postman para simular una carga concurrente a nuestro sistema. Siga estos pasos.
    * Instale newman con el comando `npm install newman -g`. Para conocer más de Newman consulte el siguiente [enlace](https://learning.getpostman.com/docs/postman/collection-runs/command-line-integration-with-newman/).
    * Diríjase hasta la ruta `FibonacciApp/postman` en una maquina diferente a la VM.
    * Para el archivo `[ARSW_LOAD-BALANCING_AZURE].postman_environment.json` cambie el valor del parámetro `VM1` para que coincida con la IP de su VM.
    * Ejecute el siguiente comando.

    ```
    newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
    newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10
    ```

![](images/lab/24.png)
![](images/lab/25.png)
![](images/lab/41.png)

10. La cantidad de CPU consumida es bastante grande y un conjunto considerable de peticiones concurrentes pueden hacer fallar nuestro servicio. Para solucionarlo usaremos una estrategia de Escalamiento Vertical. En Azure diríjase a la sección *size* y a continuación seleccione el tamaño `B2ms`.

![Imágen 3](images/part1/part1-vm-resize.png)

![](images/lab/26.png)

11. Una vez el cambio se vea reflejado, repita el paso 7, 8 y 9.

![](images/lab/27.png)
![](images/lab/28.png)
![](images/lab/29.png)
![](images/lab/30.png)
![](images/lab/31.png)
![](images/lab/32.png)
![](images/lab/33.png)
![](images/lab/34.png)
![](images/lab/35.png)
![](images/lab/36.png)
![](images/lab/37.png)
![](images/lab/38.png)

12. Evalue el escenario de calidad asociado al requerimiento no funcional de escalabilidad y concluya si usando este modelo de escalabilidad logramos cumplirlo.

    No se cumple ya que todas las peticiones no fueron correctamente repondidas, aunque el consumo de CPU no supero el 70%.

13. Vuelva a dejar la VM en el tamaño inicial para evitar cobros adicionales.

![](images/lab/39.png)

**Preguntas**

1. ¿Cuántos y cuáles recursos crea Azure junto con la VM?

Azure junto con la VM crea 6 recursos adicionales que son:
![](images/lab/40.png)

2. ¿Brevemente describa para qué sirve cada recurso?
    1. VERTICAL-SCALABILITY (Virtual machine):
    Sirve para ejecutar aplicaciones, servicios o laboratorios.

    2. vertical-scalability778_z1 (Network Interface – NIC)
    Permite que la VM tenga una dirección IP privada dentro de la red y se comunique con otros recursos.

    3. VERTICAL-SCALABILITY-nsg (Network Security Group)
    Controla qué tráfico entra y sale mediante reglas (por ejemplo, permitir SSH o HTTP).

    4. VERTICAL-SCALABILITY-ip (Public IP Address)
    Sirve para acceder a ella desde Internet (por ejemplo, mediante SSH o para exponer servicios).

    5. vnet-canadacentral (Virtual Network – VNet)
    Define el espacio de direcciones IP, subredes y la comunicación entre recursos internos.

    6. SCALABILITY_LAB (Resource group)
    Sirve para administrar, organizar y eliminar los recursos asociados a tu laboratorio de escalabilidad.

3. ¿Al cerrar la conexión ssh con la VM, por qué se cae la aplicación que ejecutamos con el comando `npm FibonacciApp.js`? ¿Por qué debemos crear un *Inbound port rule* antes de acceder al servicio?

    El comando npm FibonacciApp.js inicia un proceso que solo funciona si hay una conexión activa. Si la conexión se cierra, la aplicación se cierra y deja de funcionar.
    Debemos crear un Inbound port rule para permitir el tráfico de entrada al puerto 3000, ya que por defecto Azure no permite el tráfico de entrada a ningún puerto, y este es necesario para que la aplicación funcione.

4. Adjunte tabla de tiempos e interprete por qué la función tarda tando tiempo.

    La aplicación tarda tanto porque se calculan todos los números de la secuencia de Fibonacci hasta el número que se desea calcular, y esto toma mucho tiempo.

    Antes del escalamiento:
    ![](images/lab/13.png)
    ![](images/lab/14.png)
    ![](images/lab/15.png)
    ![](images/lab/16.png)
    ![](images/lab/17.png)
    ![](images/lab/18.png)
    ![](images/lab/19.png)
    ![](images/lab/20.png)
    ![](images/lab/21.png)
    ![](images/lab/22.png)

    Después del escalamiento:
    ![](images/lab/27.png)
    ![](images/lab/28.png)
    ![](images/lab/29.png)
    ![](images/lab/30.png)
    ![](images/lab/31.png)
    ![](images/lab/32.png)
    ![](images/lab/33.png)
    ![](images/lab/34.png)
    ![](images/lab/35.png)
    ![](images/lab/36.png)

5. Adjunte imágen del consumo de CPU de la VM e interprete por qué la función consume esa cantidad de CPU.

    La función consume esa cantidad de CPU porque se calculan todos los números de la secuencia de Fibonacci hasta el número que se desea calcular, y esto necesita muchos recursos de CPU.

    Antes del escalamiento:
    ![](images/lab/23.png)

    Después del escalamiento:
    ![](images/lab/37.png)

6. Adjunte la imagen del resumen de la ejecución de Postman. Interprete:
    * Tiempos de ejecución de cada petición.

        Antes de hacer el escalamiento el tiempo fue mayor.

        Antes del escalamiento:
        ![](images/lab/41.png)
        Después del escalamiento:
        ![](images/lab/38.png)

    * Si hubo fallos documentelos y explique.

        El error ECONNRESET significa que la conexión TCP en su cliente Postman fue cerrada inesperadamente por el servidor o algún intermediario como un proxy. En el contexto de su prueba de Postman, parece que la solicitud a la API "fibonacci" en la iteración 2 fue interrumpida antes de que pudiera completarse. Este error puede ser causado por varias razones, como un servidor que se cierra inesperadamente, un tiempo de espera de la conexión, problemas de red, etc.

        ![](images/lab/42.png)

7. ¿Cuál es la diferencia entre los tamaños `B2ms` y `B1ls` (no solo busque especificaciones de infraestructura)?

    La principal diferencia entre los tamaños B2ms y B1ls no es solo técnica, sino en su capacidad real de uso y rendimiento. La B2ms es una máquina pequeña-mediana que permite ejecutar aplicaciones web, servicios con varios hilos, pruebas de carga y procesos que requieren memoria, ofreciendo un desempeño estable y útil para escenarios reales. En cambio, la B1ls es la máquina más básica de Azure: extremadamente limitada, con recursos mínimos que solo sirven para tareas simples, scripts o pruebas muy ligeras, y que se congela fácilmente ante cualquier carga significativa. En resumen, B2ms es adecuada para trabajo real y pruebas de escalabilidad, mientras que B1ls es solo para usos básicos y no soporta aplicaciones exigentes.

8. ¿Aumentar el tamaño de la VM es una buena solución en este escenario?, ¿Qué pasa con la FibonacciApp cuando cambiamos el tamaño de la VM?

    No, porque el aumento del tamaño de la VM no es una solución escalable, ya que si se aumenta el número de peticiones, la VM no podrá procesarlas todas y se volverá a tener el mismo problema de consumo de CPU. Cuando se cambia el tamaño de la VM, la aplicación deja de funcionar y se debe volver a ejecutar.

9. ¿Qué pasa con la infraestructura cuando cambia el tamaño de la VM? ¿Qué efectos negativos implica?

    Al cambiar el tamaño de la VM, se debe reiniciar la máquina, por lo que se pierde la conexión ssh y la aplicación deja de funcionar.

10. ¿Hubo mejora en el consumo de CPU o en los tiempos de respuesta? Si/No ¿Por qué?

    Sí, porque al aumentar el tamaño de la VM, se aumenta la cantidad de CPU y memoria RAM, por lo que la aplicación puede procesar más peticiones en menos tiempo.

11. Aumente la cantidad de ejecuciones paralelas del comando de postman a `4`. ¿El comportamiento del sistema es porcentualmente mejor?

    No, aunque se lograron procesar las peticiones se evidencio un aumento en el uso del CPU. 

    ![](images/lab/43.png)
    ![](images/lab/44.png)

### Parte 2 - Escalabilidad horizontal

#### Crear el Balanceador de Carga

Antes de continuar puede eliminar el grupo de recursos anterior para evitar gastos adicionales y realizar la actividad en un grupo de recursos totalmente limpio.

1. El Balanceador de Carga es un recurso fundamental para habilitar la escalabilidad horizontal de nuestro sistema, por eso en este paso cree un balanceador de carga dentro de Azure tal cual como se muestra en la imágen adjunta.

![](images/part2/part2-lb-create.png)

![](images/lab/parte2/1.png)
![](images/lab/parte2/2.png)
![](images/lab/parte2/3.png)

2. A continuación cree un *Backend Pool*, guiese con la siguiente imágen.

![](images/part2/part2-lb-bp-create.png)

![](images/lab/parte2/4.png)

3. A continuación cree un *Health Probe*, guiese con la siguiente imágen.

![](images/part2/part2-lb-hp-create.png)

![](images/lab/parte2/5.png)

4. A continuación cree un *Load Balancing Rule*, guiese con la siguiente imágen.

![](images/part2/part2-lb-lbr-create.png)

![](images/lab/parte2/6.png)

5. Cree una *Virtual Network* dentro del grupo de recursos, guiese con la siguiente imágen.

![](images/part2/part2-vn-create.png)

![](images/lab/parte2/7.png)
![](images/lab/parte2/8.png)
![](images/lab/parte2/9.png)

#### Crear las maquinas virtuales (Nodos)

Ahora vamos a crear 3 VMs (VM1, VM2 y VM3) con direcciones IP públicas standar en 3 diferentes zonas de disponibilidad. Después las agregaremos al balanceador de carga.

1. En la configuración básica de la VM guíese por la siguiente imágen. Es importante que se fije en la "Avaiability Zone", donde la VM1 será 1, la VM2 será 2 y la VM3 será 3.

![](images/part2/part2-vm-create1.png)

![](images/lab/parte2/10.1.png)
![](images/lab/parte2/10.2.png)
![](images/lab/parte2/10.3.png)

2. En la configuración de networking, verifique que se ha seleccionado la *Virtual Network*  y la *Subnet* creadas anteriormente. Adicionalmente asigne una IP pública y no olvide habilitar la redundancia de zona.

![](images/part2/part2-vm-create2.png)

![](images/lab/parte2/11.1.png)
![](images/lab/parte2/11.2.png)
![](images/lab/parte2/11.3.png)

![](images/lab/parte2/ipvm3.png)

3. Para el Network Security Group seleccione "avanzado" y realice la siguiente configuración. No olvide crear un *Inbound Rule*, en el cual habilite el tráfico por el puerto 3000. Cuando cree la VM2 y la VM3, no necesita volver a crear el *Network Security Group*, sino que puede seleccionar el anteriormente creado.

![](images/part2/part2-vm-create3.png)

![](images/lab/parte2/12.1.png)

4. Ahora asignaremos esta VM a nuestro balanceador de carga, para ello siga la configuración de la siguiente imágen.

![](images/part2/part2-vm-create4.png)

![](images/lab/parte2/13.1.png)

![](images/lab/parte2/14.1.png)
![](images/lab/parte2/14.2.png)
![](images/lab/parte2/14.3.png)

5. Finalmente debemos instalar la aplicación de Fibonacci en la VM. para ello puede ejecutar el conjunto de los siguientes comandos, cambiando el nombre de la VM por el correcto

git clone https://github.com/daprieto1/ARSW_LOAD-BALANCING_AZURE.git

![](images/lab/parte2/15.1.png)
![](images/lab/parte2/15.2.png)
![](images/lab/parte2/15.3.png)

curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.34.0/install.sh | bash

![](images/lab/parte2/16.1.png)
![](images/lab/parte2/16.2.png)
![](images/lab/parte2/16.3.png)

source /home/vm1/.bashrc

![](images/lab/parte2/17.1.png)
![](images/lab/parte2/17.2.png)
![](images/lab/parte2/17.3.png)

nvm install node

![](images/lab/parte2/18.1.png)
![](images/lab/parte2/18.2.png)
![](images/lab/parte2/18.3.png)

cd ARSW_LOAD-BALANCING_AZURE/FibonacciApp

![](images/lab/parte2/19.1.png)
![](images/lab/parte2/19.2.png)
![](images/lab/parte2/19.3.png)

npm install

![](images/lab/parte2/20.1.png)
![](images/lab/parte2/20.2.png)
![](images/lab/parte2/20.3.png)

npm install forever -g

![](images/lab/parte2/21.1.png)
![](images/lab/parte2/21.2.png)
![](images/lab/parte2/21.3.png)

forever start FibonacciApp.js

![](images/lab/parte2/22.1.png)
![](images/lab/parte2/22.2.png)
![](images/lab/parte2/22.3.png)

Realice este proceso para las 3 VMs, por ahora lo haremos a mano una por una, sin embargo es importante que usted sepa que existen herramientas para aumatizar este proceso, entre ellas encontramos Azure Resource Manager, OsDisk Images, Terraform con Vagrant y Paker, Puppet, Ansible entre otras.

#### Probar el resultado final de nuestra infraestructura

1. Porsupuesto el endpoint de acceso a nuestro sistema será la IP pública del balanceador de carga, primero verifiquemos que los servicios básicos están funcionando, consuma los siguientes recursos:

```
http://52.155.223.248/
http://52.155.223.248/fibonacci/1
```

![](images/lab/parte2/23.png)
![](images/lab/parte2/24.png)

2. Realice las pruebas de carga con `newman` que se realizaron en la parte 1 y haga un informe comparativo donde contraste: tiempos de respuesta, cantidad de peticiones respondidas con éxito, costos de las 2 infraestrucruras, es decir, la que desarrollamos con balanceo de carga horizontal y la que se hizo con una maquina virtual escalada.

Escalabilidad Vertical

- Antes de implementación:

![](images/lab/41.png)

-Después de implementación:

![](images/lab/38.png)

Escalabilidad Horizontal

![](images/lab/parte2/25.png)

| Estrategia             | Estado               | Promedio de respuesta | Min    | Max    | Fallos de requests |
| ---------------------- | -------------------- | --------------------- | ------ | ------ | ------------------ |
| **Vertical (Antes)**   | VM sin escalar       | **20.3 s**            | 16.1 s | 32.1 s | **4 fallos**       |
| **Vertical (Después)** | VM escalada          | **15.9 s**            | 11.3 s | 22.5 s | **5 fallos**       |
| **Horizontal**         | 2 VM + Load Balancer | **12.8 s**            | 11.0 s | 19.8 s | **1 fallo**        |

#### Tiempos de Respuesta

- La escalabilidad vertical mejoró los tiempos respecto al estado inicial (20.3 s → 15.9 s).

- La escalabilidad horizontal ofrece el mejor rendimiento, logrando el menor promedio (12.8 s) incluso cuando maneja carga distribuida.

    El tiempo máximo también mejora de forma notable con horizontal (19.8 s vs 32.1 s).

Conclusión: la estrategia horizontal gestiona mejor el cálculo intensivo, ya que reparte la carga entre varias instancias.

#### Cantidad de Peticiones Fallidas

- Vertical antes: 4 fallos.

- Vertical después: sorprendentemente subió a 5 fallos.

- Horizontal: solo 1 fallo.

Conclusión: en vertical, incluso con más recursos la app falló más. La infraestructura horizontal reduce fallos gracias al balanceador que reparte la carga entre nodos disponibles.

#### Consumo y Costos de Infraestructura

Escalabilidad Vertical
- Requiere una sola máquina, pero con más vCPU y memoria.
- Los costos crecen linealmente y tienden a ser más altos al aumentar el tamaño de la máquina.
- Mayor riesgo: si la VM cae, todo el servicio cae.

Escalabilidad Horizontal
- Usa máquinas pequeñas, más baratas.
- Añade un Load Balancer, que tiene un costo adicional bajo.
- Es más flexible: se pueden encender o apagar instancias según demanda.
- Permite alta disponibilidad.

3. Agregue una 4 maquina virtual y realice las pruebas de newman, pero esta vez no lance 2 peticiones en paralelo, sino que incrementelo a 4. Haga un informe donde presente el comportamiento de la CPU de las 4 VM y explique porque la tasa de éxito de las peticiones aumento con este estilo de escalabilidad.

```
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10
```

No se pudo realizar ya que la suscripción solo dejaba tener 3 ip públicas creadas a la vez las cuales fueron:

- Balanceador de carga.
- Maquina Virtual 1.
- Maquina Virtual 2.

![](images/lab/parte2/ipvm3.png)

Aunque igualmente se ejecutó con las VM1 y VM2:

![](images/lab/parte2/26.png)
![](images/lab/parte2/27.png)
![](images/lab/parte2/28.png)

Escalar horizontalmente aumentó la capacidad efectiva para atender peticiones concurrentes porque multiplicó el número de procesos independientes que pueden ejecutar cálculos CPU-intensivos en paralelo, reduciendo bloqueo en el event-loop y evitando colas/tiempos de espera que provocan fallos.

Al aumentar las réplicas (2 VMs) y la concurrencia a 4 peticiones paralelas, la carga se reparte entre más procesos independientes, lo que reduce el bloqueo por CPU en cada VM, baja los picos individuales y mejora la tasa de éxito y la estabilidad del servicio.

**Preguntas**

* ¿Cuáles son los tipos de balanceadores de carga en Azure y en qué se diferencian?

    1. Azure Load Balancer

    - Funciona en la capa 4 (TCP/UDP).
    - Distribuye tráfico interno o externo.
    - Alta velocidad y bajo costo.
    - Ejemplos: balancear tráfico a VMs, gateways o appliances.

    2. Azure Application Gateway
    - Funciona en la capa 7 (HTTP/HTTPS).

    - Puede hacer:
        - Reglas basadas en URL
        - WAF (Web Application Firewall)
        - Redirecciones

        - Enrutamiento basado en hostnames (path-based routing)
    - Ideal para aplicaciones web.

    3. Azure Front Door
    - Servicio global (edge).
    - CDN + balanceo de capa 7.
    - Optimiza latencia mundial.
    - Ideal para apps globales de alto tráfico.

* ¿Qué es SKU, qué tipos hay y en qué se diferencian?

    SKU (Stock Keeping Unit) en Azure significa la categoría del producto, que define:
    - Capacidad
    - Rendimiento
    - Funcionalidades
    - Precio

    Tipos de SKU del Load Balancer:
    1. Basic
        - Para ambientes de prueba.
        - No soporta zonas de disponibilidad.
        - Menor resiliencia.
        - Menos características.

    2. Standard
        - Producción.
        - Incluye alta disponibilidad, seguridad y métricas mejoradas.
        - Soporta Availability Zones.
        - Más capacidad y mejor rendimiento.

    Diferencias clave:
    - Standard es más seguro.
    - Standard permite zonas de disponibilidad.
    - Standard soporta más conexiones simultáneas.
    - Basic es más limitado y no se recomienda para producción.

* ¿Por qué el balanceador de carga necesita una IP pública?

    Porque una IP pública permite que clientes desde Internet puedan:
    - Acceder al servicio detrás del balanceador.
    - Enviar tráfico entrante al Load Balancer.
    - Permitir que Azure asigne tráfico a las VMs conectadas al Backend Pool.

    Sin una IP pública, el servicio solo funcionaría internamente dentro de la red (Virtual Network).

* ¿Cuál es el propósito del *Backend Pool*?

    El Backend Pool es el conjunto de máquinas virtuales o recursos hacia donde el Load Balancer distribuye el tráfico.

    Azure reparte las peticiones dependiendo de:
    - Reglas de balanceo
    - Algoritmo de distribución
    - Estado de salud (Health Probe)

* ¿Cuál es el propósito del *Health Probe*?

    El Health Probe verifica el estado de cada máquina del Backend Pool.

    Su función es:
    - Enviar pings o solicitudes HTTP/TCP a cada VM.
    - Detectar si una instancia está saludable o caída.
    - Sacar temporalmente del pool a las máquinas que no respondan.

    Esto evita que el Load Balancer envíe tráfico a un servidor que:
    - Está apagado
    - Está saturado
    - Tiene errores

* ¿Cuál es el propósito de la *Load Balancing Rule*? ¿Qué tipos de sesión persistente existen, por qué esto es importante y cómo puede afectar la escalabilidad del sistema?.

    La Load Balancing Rule define cómo el tráfico será distribuido:

    - Puerto de entrada (frontend)
    - Puerto de destino (backend)
    - Protocolo (TCP/UDP)
    - Método de distribución
    - Uso de sesión persistente

    Tipos de sesión persistente

    1. None (sin persistencia)
        - Cada solicitud puede ir a una VM distinta (Round Robin).
        - Máxima escalabilidad.
        - Ideal para apps stateless.

    2. Client IP
        - El cliente siempre va a la misma VM si es posible.
        - Evita romper sesiones.

    3. Client IP + Protocol
        - Aún más estricta.
        - El usuario queda “pegado” a la misma VM para una combinación IP+protocolo.

    ¿Por qué es importante?
    - Influye en la escalabilidad y el rendimiento.
    - Mucha persistencia = menos balanceo = posibles saturaciones.

    Impacto en escalabilidad
    - Persistencia None → mejor para horizontal scaling.
    - Persistencia estricta → puede generar cuellos de botella.

* ¿Qué es una *Virtual Network*?

    Una Virtual Network es la red privada en la nube donde viven:
    - Máquinas virtuales
    - Subnets
    - Balanceadores
    - Bases de datos
    - Gateways

* ¿Qué es una *Subnet*?

    Una Subnet es una división lógica dentro de una VNet.

    Sirve para:
    - Organizar recursos por zonas (web, backend, BD)
    - Controlar tráfico y seguridad
    - Asignar políticas diferentes mediante NSGs

* ¿Para qué sirven los *address space* y *address range*?

    Address Space

    - Es el rango total de IPs que tiene la Virtual Network.

    Address Range (o prefijo de Subnet)

    - Es el rango de IP de una Subnet dentro del address space.

    Relación:
    El subnet range debe estar contenido dentro del address space.

* ¿Qué son las *Availability Zone* y por qué seleccionamos 3 diferentes zonas?.

    Son zonas físicas independientes dentro de una misma región de Azure:
    - Tienen energía, red y cooling propios.
    - Están separadas para evitar que un desastre afecte todas las zonas.

    Al seleccionar 2 o 3 zonas:
    - Garantizas alta disponibilidad.
    - El sistema sigue funcionando incluso si una zona completa falla.

    ¿Por qué seleccionamos 3?

    Porque Azure Load Balancer Standard soporta “zone-redundancy”, lo que significa:
    - El tráfico entrante se distribuye entre VMs en diferentes zonas.
    - Mayor tolerancia a fallos.

* ¿Qué significa que una IP sea *zone-redundant*?

    Que la IP pública no pertenece a una zona específica sino a todas las zonas disponibles.

    Ventajas:
    - Si la Zona 1 cae, la IP sigue disponible en Zonas 2 y 3.
    - Alta disponibilidad de nivel regional.

* ¿Cuál es el propósito del *Network Security Group*?

    El NSG actúa como un firewall.

    Su función es:
    - Permitir o bloquear tráfico hacia VMs, subnets o interfaces.
    - Definir reglas de entrada y salida por:
        - IP origen
        - IP destino
        - Protocolo (TCP/UDP)
        - Puerto

    Es fundamental para:
    - Seguridad
    - Segmentación
    - Proteger la aplicación del tráfico no autorizado

* Informe de newman 1 (Punto 2)

![](images/lab/parte2/25.png)

* Presente el Diagrama de Despliegue de la solución.

## Diagrama de Despliegue

El diagrama de despliegue muestra la arquitectura completa de la solución con escalabilidad horizontal implementada en Azure:

![Diagrama de Despliegue](Diagrama%20de%20Despliegue%20-%20Escalabilidad%20Horizontal%20Azure.png)

El diagrama incluye:
- **Azure Load Balancer**: Balanceador de carga público con IP zone-redundant
- **Backend Pool**: 3 máquinas virtuales distribuidas en 3 zonas de disponibilidad
- **Health Probe**: Verificación de salud de las VMs en el puerto 3000
- **Virtual Network**: Red virtual vnet-canadacentral con espacio de direcciones 10.0.0.0/16
- **Network Security Group**: Reglas de seguridad para SSH (22) y aplicación (3000)
- **Aplicación FibonacciApp**: Ejecutándose en Node.js en cada VM

### Cómo generar el diagrama

El diagrama está definido en el archivo `diagrama-despliegue.puml` usando PlantUML. Para regenerarlo:

```bash
# Instalar Java y Graphviz (si no están instalados)
sudo apt-get install -y default-jre graphviz

# Descargar PlantUML
wget https://github.com/plantuml/plantuml/releases/download/v1.2024.7/plantuml-1.2024.7.jar

# Generar el diagrama
java -jar plantuml-1.2024.7.jar diagrama-despliegue.puml
```

El archivo PNG generado mostrará la arquitectura completa de la solución.

