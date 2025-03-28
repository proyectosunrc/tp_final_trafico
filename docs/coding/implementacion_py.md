## Servidor de respuestas a peticiones HTTP

### Servidor

```python
from fastapi import FastAPI
import random
import time
import asyncio

# Inicializar la aplicación FastAPI
app = FastAPI()

# Definir la tasa media de servicio
SERVICE_RATE = 10  

def intensive_task(duration: float):
    """Realiza una tarea intensiva en CPU durante un tiempo determinado."""
    end_time = time.time() + duration
    while time.time() < end_time:
        _ = sum(x * x for x in range(10000))

@app.get("/")
async def root():
    """Simula una solicitud con retardo y carga en CPU."""
    delay = random.expovariate(SERVICE_RATE)  # Generar tiempo de espera exponencial
    await asyncio.sleep(delay)  # Simular espera asíncrona

    intensive_task(delay)  # Cargar la CPU durante el mismo tiempo

    return {"message": f"Respuesta procesada con retraso y carga de {delay:.4f} segundos"}

```

**Este código crea una API con FastAPI que simula un endpoint con retrasos variables y carga computacional, resultando útil para pruebas de estrés.**

**Endpoint / (Ruta principal):**

* Encargado de crear los valores con promedio de 0.1 segundos (1/mu)
* Libera el servidor para atender otras peticiones
* por medio de la funcion stress_cpu(a) consume recursos durante el tiempo "a"


### Generador de Trafico

```python
import asyncio
import aiohttp
import random
import time
import json

# Configuración
URL = "http://miapp.local:30240/"  # Sustituye con la URL deseada
LAMBDA = 4  # Tasa de arribo (inversa del tiempo medio entre solicitudes)
SIMULATION_DURATION = 480  # Duración de la simulación en segundos
OUTPUT_FILE = "simulation_data.json"  # Archivo de salida

# Almacenamiento de datos
requests_data = []

async def send_request(request_number):
    """Envía una solicitud HTTP y registra la respuesta."""
    start_time = time.time()
    send_timestamp = start_time
    port = 10000 + request_number  # Cada solicitud usa un puerto distinto
    conn = aiohttp.TCPConnector(local_addr=('0.0.0.0', port))
    
    async with aiohttp.ClientSession(connector=conn) as session:
        try:
            async with session.get(URL) as response:
                response_time = time.time() - start_time
                requests_data.append({
                    "request_number": request_number,
                    "response_time": response_time,
                    "status_code": response.status,
                    "send_timestamp": send_timestamp
                })
        except Exception as e:
            requests_data.append({
                "request_number": request_number,
                "response_time": time.time() - start_time,
                "status_code": "error",
                "error": str(e),
                "send_timestamp": send_timestamp
            })

async def request_generator():
    """Genera solicitudes basadas en una distribución exponencial."""
    request_number = 0
    simulation_start_time = time.time()
    
    while time.time() - simulation_start_time < SIMULATION_DURATION:
        request_number += 1
        asyncio.create_task(send_request(request_number))
        await asyncio.sleep(random.expovariate(LAMBDA))
    
    return time.time() - simulation_start_time

async def main():
    """Ejecuta la simulación y guarda los resultados en un archivo JSON."""
    try:
        simulation_time = await request_generator()
    except asyncio.CancelledError:
        simulation_time = None
    
    simulation_output = {
        "total_simulation_time": simulation_time or 0,
        "requests": requests_data
    }
    
    with open(OUTPUT_FILE, "w") as f:
        json.dump(simulation_output, f, indent=4)
    
    return simulation_time

if __name__ == "__main__":
    try:
        simulation_time = asyncio.run(main())
        print(f"Tiempo total de simulación: {simulation_time:.2f} segundos.")
    except KeyboardInterrupt:
        print("\nSimulación interrumpida manualmente.")


```

## Funcionamiento

### Configuracion inicial

El código realiza una simulación de tráfico HTTP enviando solicitudes a una URL específica durante un tiempo determinado.

Primero, se configuran los parámetros de la simulación, como la URL de destino, la tasa de arribo (lambda), la duración total de la simulación y la lista requests_data, donde se almacenan los datos de cada solicitud.

### Funcion para el envio de la solicitud HTTP

La función send_request(request_number) envía solicitudes HTTP de manera asíncrona utilizando la biblioteca aiohttp. Para cada solicitud, se registra el tiempo de inicio y se asigna un puerto dinámico para evitar la congestión en un solo puerto. Luego, se establece una sesión HTTP con aiohttp.ClientSession() y se envía una petición GET a la URL definida. Se calcula el tiempo de respuesta y se almacena junto con el código de estado HTTP en la lista requests_data. Si ocurre un error, se captura la excepción y se registra el tiempo transcurrido junto con el mensaje de error.

### Generador de solicitudes

La función request_generator() genera y lanza solicitudes de manera continua mientras dure la simulación. Se inicializa un contador de solicitudes y se ejecuta un bucle que se mantiene activo hasta alcanzar la duración total de la simulación. Los tiempos entre solicitudes se generan de manera aleatoria siguiendo una distribución exponencial con la tasa lambda. Cada solicitud se lanza de manera asíncrona sin bloquear la ejecución de otras solicitudes, y el generador espera el tiempo calculado antes de enviar la siguiente.

### Funcion main

La función main() ejecuta la simulación llamando a request_generator() y guarda los datos obtenidos en un archivo JSON. Este archivo contiene el tiempo total de simulación y la información detallada de cada solicitud, incluyendo número, tiempo de respuesta, código de estado y posibles errores.

En conjunto, el código permite realizar pruebas de carga y evaluar el rendimiento de un servidor simulando tráfico HTTP con intervalos de llegada aleatorios.


# Scripts de Graficas

## Tiempo de respuesta del sistema en funcion de la carga y el uso de CPU:

```python
import numpy as np
import matplotlib.pyplot as plt
import math

# ESCENARIO - 1 POD
#TEORICO
mu10 = 10  # Tasa de servicio fija
rho_values_t10 = np.linspace(0, 0.99, 100)
factor_delay10 = 1 / (mu10 * (1 - rho_values_t10))

#PRÁCTICO
# Valores experimentales de λ (tasa de llegada) y μ (tasa de servicio)
lambda_values1 = [1.04, 3.08, 6.01, 8.66, 9.7]   # Valores de lambda obtenidos
mu_values1 = [9.45, 9.56, 9.55, 9.84, 9.8]    # Valores de mu calculados 
# Calcular ρ (carga del sistema) y factor de delay W
rho_values_p1 = [lam / mu for lam, mu in zip(lambda_values1, mu_values1)]
factor_delay1 = [1 / (mu - lam) if mu > lam else np.inf  # Evita división por 0 o inestabilidad
for lam, mu in zip(lambda_values1, mu_values1)]

# Graficar la curva práctica ρ vs W
plt.figure(figsize=(8, 6))
plt.xlim(0,1)  # Ajusta el eje X entre 0 y 1
plt.ylim(0,3)  # Ajusta el eje Y entre 0 y 10
plt.plot(rho_values_t10, factor_delay10, color="b", label=f"pods=1", linestyle="--")
plt.plot(rho_values_p1, factor_delay1, marker='o', linestyle='none', color='r', label="Puntos Prácticos")
plt.axvline(x=1, color='r', linestyle='--', label="Límite ρ=1")
plt.xlabel("Carga del Sistema ρ")
plt.ylabel("Factor de Delay (W)")
plt.title("Curva Práctica del Factor Delay vs Carga del Sistema")
plt.grid(True)
plt.legend()
plt.show()
plt.savefig("Factor_Delay_rho.png")

# Gráfico 2: Uso de CPU vs Tasa de Arribo
plt.figure(figsize=(8, 6))
plt.plot(lambda_values1, rho_values_p1, marker='s', color='m', linestyle='-', label="λ vs ρ")
plt.xlabel("Tasa de Arribo (λ)")
plt.ylabel("Uso del CPU (%")
plt.title("Uso de CPU (%) vs Tasa de Arribo")
plt.grid(True)
plt.legend()
plt.savefig("Cpu_vs_Lambda.png")
plt.show()

```

## Número de perdidas en funcion de la tasa de arrivo

```python
import numpy as np
import matplotlib.pyplot as plt
import math

def loss_rate(lam, mu, servers=1):
    """
    Calcula la tasa de pérdida de solicitudes en un sistema M/M/k/k (Erlang-B).
    
    Parámetros:
    - lam: tasa de llegada (λ)
    - mu: tasa de servicio (μ)
    - servers: número de servidores disponibles (k)

    Retorna:
    - Probabilidad de que una solicitud sea rechazada (P_loss)
    """
    rho = lam / (servers * mu)  # Carga del sistema
    if rho >= 1:
        return 1  # Si la carga es >=1, todas las solicitudes nuevas serán rechazadas
    
    # Fórmula de Erlang-B para pérdida de solicitudes
    erlang_b = ( (lam/mu)**servers / math.factorial(servers) ) / sum((lam/mu)**n / math.factorial(n) for n in range(servers + 1))
    
    return erlang_b

# Definir valores de λ y μ para la simulación
lambda_values = [1.04, 3.08, 6.01, 8.66, 9.7]
mu_values = [9.45, 9.56, 9.55, 9.84, 9.8]
servers = 1  # Número de pods o servidores

# Calcular tasa de pérdida
loss_rates = [loss_rate(lam, mu, servers) for lam, mu in zip(lambda_values, mu_values)]

# Graficar la tasa de pérdida en función de la tasa de arribo (λ)
plt.figure(figsize=(8, 6))
plt.plot(lambda_values, loss_rates, marker='s', color='r', linestyle='-', label="Tasa de Pérdida vs λ")
plt.xlabel("Tasa de Arribo (λ)")
plt.ylabel("Tasa de Pérdida de Solicitudes")
plt.title("Tasa de Pérdida vs Tasa de Arribo")
plt.grid(True)
plt.legend()
plt.savefig("Loss_Rate_vs_Lambda.png")
plt.show()

```

## Numero de tareas en funcion de la tasa de arrivo

```python
import numpy as np
import math
import matplotlib.pyplot as plt

# Parámetros del sistema
mu = 10  # Tasa de servicio de cada pod
k = 1  # Número de pods (servidores)

# Rango de tasas de arribo (lambda)
lambda_values = np.linspace(1, 49, 100)  # De 1 a 49 solicitudes/segundo

def p0(lambda_val, mu, k):
    """ Calcula la probabilidad de que no haya solicitudes en el sistema """
    rho = lambda_val / (k * mu)
    sumatoria = sum((lambda_val / mu) ** n / math.factorial(n) for n in range(k + 1))
    sumatoria += ((lambda_val / mu) ** k) / (math.factorial(k) * (1 - rho))
    return 1 / sumatoria

def p_q(lambda_val, mu, k):
    """ Calcula la probabilidad de que una solicitud tenga que esperar en cola """
    rho = lambda_val / (k * mu)
    if rho >= 1:  # Evitar inestabilidad matemática
        return 1
    p0_val = p0(lambda_val, mu, k)
    numerator = (p0_val * (lambda_val / mu) ** k) / (math.factorial(k) * (1 - rho))
    denominator = 1 + ((1 - rho) / rho) * (1 - numerator)
    return numerator / denominator

def l_q(lambda_val, mu, k):
    """ Calcula el número esperado de tareas en la cola """
    rho = lambda_val / (k * mu)
    if rho >= 1:  # Evitar inestabilidad
        return np.inf
    return (p_q(lambda_val, mu, k) * (lambda_val / mu)) / (1 - rho)

def l_s(lambda_val, mu, k):
    """ Calcula el número esperado de tareas en servicio """
    return lambda_val / mu

def l_total(lambda_val, mu, k):
    """ Calcula el número total esperado de tareas en el sistema """
    return l_q(lambda_val, mu, k) + l_s(lambda_val, mu, k)

# Calcular L (número de tareas en el sistema) para cada lambda
l_values = [l_total(lam, mu, k) for lam in lambda_values]

# Graficar número de tareas en el sistema vs tasa de arribo
plt.figure(figsize=(8, 6))
plt.plot(lambda_values, l_values, marker='o', linestyle='-', color='b', label="Número de Tareas (L)")

plt.xlabel("Tasa de Arribo λ (Solicitudes/seg)")
plt.ylabel("Número de Tareas en el Sistema (L)")
plt.title(f"Número de Tareas vs Tasa de Arribo (k={k} pods, μ={mu})")
plt.legend()
plt.grid(True)

# Guardar y mostrar el gráfico
plt.savefig("Numero_Tareas_vs_Lambda.png")
plt.show()
```

## Tiempo de respuesta del sistema en funcion de la cantidad de PODs

```python

#RESPUESTA DEL SISTEMA EN FUNCION DE la cantidad de servidores (PODS)

import numpy as np
import matplotlib.pyplot as plt

# Definir los valores de la tasa de servicio (μ) y la tasa de llegada (λ)
mu = 10  # Tasa de servicio de cada pod (solicitudes por segundo)
lambda_fixed = 15  # Tasa de llegada total de solicitudes (solicitudes por segundo)

# Rango de pods (cantidad de servidores en paralelo)
k_values = np.arange(1, 11, 1)  # De 1 a 10 pods

# Calcular la carga del sistema (ρ) para cada k
rho_values = lambda_fixed / (k_values * mu)

# Calcular el tiempo de respuesta teórico W = 1 / (mu * (1 - rho)), evitando división por 0
W_theoretical = np.where(rho_values < 1, 1 / (mu * (1 - rho_values)), np.inf)

# Simulación de valores prácticos con pequeñas variaciones aleatorias
W_practical = W_theoretical * (1 + np.random.uniform(-0.1, 0.1, size=len(k_values)))

# Graficar los resultados
plt.figure(figsize=(8, 6))
plt.plot(k_values, W_theoretical, label="Tiempo de Respuesta Teórico", color="b", linestyle='--')
plt.scatter(k_values, W_practical, label="Datos Prácticos", color="r", marker='o')

plt.xlabel("Cantidad de Pods (Servidores en el Clúster)")
plt.ylabel("Tiempo de Respuesta Promedio (W)")
plt.title("Tiempo de Respuesta del Sistema vs. Cantidad de Pods")
plt.legend()
plt.grid(True)

# Guardar y mostrar
plt.savefig("Tiempo_Respuesta_vs_Pods.png")
plt.show()


```