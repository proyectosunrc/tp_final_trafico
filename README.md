# Proyecto: Escalabilidad Dinámica de servicios en la nube

## Introducción
En el ámbito del cloud computing, la gestión eficiente de recursos, la escalabilidad dinámica y la optimización del rendimiento en entornos basados en contenedores representan desafíos críticos que demandan soluciones teórico-prácticas innovadoras. Este proyecto integra conceptos de teoría de colas, orquestación de contenedores y programación moderna para evaluar y desplegar un modelo de escalabilidad dinámica en servicios cloud. Combina fundamentos de teoría de colas, como el análisis de sistemas bajo procesos de Poisson y distribuciones exponenciales, con tecnologías como Kubernetes, Docker y FastAPI, ofreciendo un enfoque completo para la gestión de tráfico y recursos en entornos virtualizados.

## Objetivos
El trabajo se estructuró en tres ejes principales:

1. **Despliegue de servicios en Kubernetes**: Mediante Minikube, se configuró un clúster local que simuló un entorno cloud realista. En este, se desplegaron servicios web y una API desarrollada con FastAPI, contenerizada con Docker, para garantizar portabilidad y consistencia.

2. **Modelado y análisis de redes de colas**: Se estudió el comportamiento de los servidores bajo cargas de tráfico modeladas con procesos estocásticos (arribos de Poisson y tiempos de servicio exponenciales), vinculando la teoría de colas con métricas prácticas de rendimiento.

3. **Escalado dinámico y automatización**: Se implementaron mecanismos de autoescalado horizontal en Kubernetes (HPA), optimizando la asignación de capacidad según la demanda. Adicionalmente, se empleó un NGINX Ingress Controller como balanceador de carga HTTP, asegurando distribución eficiente del tráfico.

Para evaluar el sistema, se desarrollaron scripts en Python que generaron pruebas de carga y estrés, permitiendo medir el impacto del tráfico en la API y validar la eficacia del escalado automático. La combinación de contenedores ligeros (con ventajas en portabilidad y eficiencia de recursos) y herramientas de orquestación como Kubernetes resulta útil para lograr sistemas resilientes, capaces de adaptarse a fluctuaciones de demanda de tráfico sin comprometer la estabilidad.

Este proyecto no solo valida modelos teóricos en un entorno controlado, sino que también ofrece insights prácticos sobre cómo integrar tecnologías modernas para optimizar infraestructuras en la nube. Los resultados destacan la importancia de automatizar la escalabilidad y priorizar el análisis de tráfico en el diseño de sistemas distribuidos, contribuyendo a la construcción de arquitecturas más robustas y eficientes en el ecosistema de la nube.

## Documentación
Para más detalles sobre la implementación y los resultados obtenidos, consulta la documentación completa en el siguiente enlace:

[Documentación del Proyecto](https://proyectosunrc.github.io/tp_final_trafico/)

