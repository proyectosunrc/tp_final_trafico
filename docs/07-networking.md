
En la PC anfitriona donde se aloja el clúster, varios niveles de red gestionan el tráfico interno y externo. Dentro del clúster, los pods obtienen direcciones IP privadas asignadas por el plugin de red (por ejemplo, 10.244.0.0/16), accesibles solo desde dentro del clúster. Los servicios de tipo ClusterIP crean IPs virtuales internas para permitir la comunicación entre pods y la distribución de carga, pero no son accesibles desde el exterior.

El recurso Ingress, con el controlador NGINX, se despliega en uno o varios pods y expone los puertos 80 y 443 para recibir tráfico externo. A través de reglas basadas en host y path, redirige las solicitudes a los servicios internos correspondientes.

Minikube, que generalmente se ejecuta en una máquina virtual o contenedor según el driver utilizado, cuenta con su propia red interna para conectar nodos, pods e ingress. Sin embargo, esta red no es accesible directamente desde el exterior.

Para solucionar esto, se utilizó la herramienta socat en la PC anfitriona, redirigiendo el tráfico externo desde un puerto local hacia el puerto correspondiente del ingress dentro del clúster. Esto permite canalizar el tráfico externo hacia los pods, incluso cuando estos y sus servicios operan con IPs internas.
***

&nbsp;


Solicitud:

Se observan las IPs de origen y destino asi como tambien los puertos que forman parte entre el host cliente y la interfaz de red de la PC anfitrion. Luego, el socat se encarga de realizar el port forwarding mapeando el puerto 31042 de la PC host con el del ingress para tener acceso al cluster.

![Solicitud HTTP](img/http_wireshark.png)

&nbsp;

Respuesta:

Después de que los pods procesan la solicitud y generan una respuesta, esta es
enviada de regreso al Ingress, que se encarga de transferirla desde la PC anfitrion hacia el cliente nuevamente con la información para almacenar en un archivo JSON.

![Respuesta HTTP](img/http_respuesta.png)

