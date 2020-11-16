# Despliegue en EC2

## Creación de instancia EC2 en AWS

Nota: si no tiene una cuenta en AWS [Cómo crear una cuenta en AWS Educate](https://www.youtube.com/watch?v=8DrcNddARLo&feature=youtu.be)

1. En la Consola de administración de AWS seleccionar ```Ejecute una máquina virtual
Con EC2``` 

![](/custom-consumer/img/1.jpg)

2. Seleccionar la Amazon Machine Image(AMI) ```Amazon Linux AMI 2018.03.0 (HVM), SSD Volume Type```

![](/custom-consumer/img/2.jpg)

3. Seleccionar tipo de instancia ```t2.micro (- ECUs, 1 vCPUs, 2.5 GHz, -, 1 GiB memory, EBS only)``` (por defecto)

![](/custom-consumer/img/3.jpg)

4. Seleccionar ```Review And Launch``` > ```Launch```
5. Seleccionar ```Create a new key pair``` y definir su nombre. Generará un par de llaves para permitir la conexión a la instancia EC2
6. Seleccionar ```Download key pair```, esto descargará la llave privada de la forma <keyName>.pem

![](/custom-consumer/img/4.jpg)

7. Una vez teniendo el archivo .pem seleccionar ```Launch Inatances```
8. Al ver las instancias estará inicializando la EC2

## Hacer puertos accesibles
Luego de que la instancia de EC2 esté en ejecución y lista para ser accedida:

![](/custom-consumer/img/5.jpg)

Especificar los puertos para hacerlos accesibles y realizar conexiones a la instancia EC2:
1. Luego de seleccionar la instancia creada, ir a la pestaña ```Seguridad```, allí puede ver las reglas de entrada, por defecto solo está habilitado el puerto 22

![](/custom-consumer/img/6.jpg)

2. Para habilitar más puertos, dentro de la misma pestaña hay un link que contiene ```launch-wizard```, seleccionarlo
3. Seleccionar ```Editar reglas de entrada``` > ```Agragar regla```, configure los puertos que desea que sean accesibles desde afuera (puede establecer intervalos o puertos individuales), establecer su origen como 0.0.0.0/0 en todos

![](/custom-consumer/img/7.jpg)

Para este punto es necesario abrir solo el puerto 8083, el cual permitirá hacer solititudes al servicio REST: 

![](/custom-consumer/img/8.jpg)

## Realizar conexión SSH con la EC2 (Linux o subsistema Ubuntu en Windows)
1. Al seleccionar la instancia, ir a ```Acciones``` > ```Conectar```, en la pestaña ```Cliente SSH``` se dan los pasos necesarios para realizar la conexión (usando el subsistema puede ser necesario agregar ```sudo``` al inicio del comando de conexión SSH)

Al conectarse debería ver algo como lo siguiente:

![](/custom-consumer/img/9.jpg)

así tiene acceso y manipulación sobre la instancia EC2

## Instalar Docker en AWS
En la consola de EC2:
1. ```sudo yum update -y```
2. ```sudo yum install docker```
3. ```sudo service docker start```
4. ```sudo usermod -a -G docker ec2-user```

Luego de esto, es necesario cerrar la conexión SSH (```exit``` en la consola EC2) y volver a abrirla

Para poder ejecutar el comando Docker-compose en EC2: [Instalación de docker-compose](https://docs.docker.com/compose/install/)

Para comprobar que Docker fue instalado exitosamente puede ejecutar ```docker --version```, para comprobar que el servicio se esta ejecutando ```docker ps```(no debería mostrar errores) y para comprobar que puede ejecutar el comando docker-compose ```docker-compose --version```:

![](/custom-consumer/img/10.jpg)

## Transferir zip del proyecto
Es recomendable tener el ZIP del proyecto en donde está el archivo .pem para transferirlo con facilidad

![](/custom-consumer/img/11.jpg)

1. Se usará una conexión SFTP para esto, el comando para generar la conexión SFTP es similar al de la conexión SSH pero justamente reemplazando ```ssh``` por ```sftp```

![](/custom-consumer/img/12.jpg)

2. Ya dentro de la consola SFTP, ejecutar ```put <proyecto>.zip```.

![](/custom-consumer/img/13.jpg)

3. Al volver a la consola que tiene la conexión SSH o conectarse nuevamente, se puede ver el archivo .zip en la instancia EC2. Para descomprimir el zip ```unzip <proyecto>.zip```

![](/custom-consumer/img/14.jpg)

## Configuración del conector MQTT
En este punto ya debería tener el archivo del proyecto, docker y docker-compose instalados en la instancia EC2.
1. Ejecutar los contenedores como en el laboratorio 3.

Nota: Al ejecutar el docker-compose dentro de EC2 puede que arroje el error ```there is insufficient memory for the java runtime environment to continue```, para esto se muestra una solución en [JRE out of memory in Docke](https://stackoverflow.com/questions/45129299/jre-out-of-memory-in-docker)

2. Para asegurar que los servicios se ejecutan correctamente, con el comando ```docker ps``` debería ver algo como:

![](/custom-consumer/img/17.jpg)

3. Antes de configurar el conector MQTT, puede hacer una petición GET en cualquier buscador para verificar que esté funcionando correctamente:

![](/custom-consumer/img/18.jpg)

4. Para la configuración del conector MQTT solo debería variar (con respecto al Lab3) que en la ejecución del cliente REST, este debería hacerse a la dirección de la instancia EC2 el lugar de localhost:

![](/custom-consumer/img/15.jpg)

Esta es visible en la pestaña ```Detalles``` de la instancia, específicamente en ```Dirección IPv4 pública``` o ```DNS de IPv4 pública```

![](/custom-consumer/img/16.jpg)

5. Al consultar el path ```/connectors/cloud-mqtt-source/status``` su estado debería ser RUNNING:

![](/custom-consumer/img/19.jpg)

6. Luego de seguir los pasos del lab 3 configurando un nuevo tópico ```feeds/data```

![](/custom-consumer/img/20.jpg)

Al postear información en el broker con el ESP debería ver en la terminal del docker-compose la información que esta posteando:

