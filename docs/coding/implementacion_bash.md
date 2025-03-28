

### Calcula el factor de utilidad en funcion del consumo de CPU de los PODs dentro del cluster cuando se realizan pruebas de carga:

```bash
#!/bin/bash

# Intervalo de recolección en segundos y número máximo de muestras
interval=5
max_samples=100

suma_total=0
contador=0

# Capacidad total del clúster en milicores (1950m + 2020m)
cpu_total_cluster=1950

# Función para calcular y mostrar el promedio y el factor de utilización
calcular_promedio() {
    if [ $contador -eq 0 ]; then
        echo "No se ha recolectado ninguna muestra."
    else
        promedio=$(echo "scale=2; $suma_total / $contador" | bc)
        factor_utilizacion=$(echo "scale=4; $promedio / $cpu_total_cluster" | bc)
        echo "Promedio de la suma de milicores de los nodos: ${promedio}m"
        echo "Factor de utilización promedio del clúster: ${factor_utilizacion} (o $(echo "scale=2; $factor_utilizacion * 100" | bc)% de uso)"
    fi
}

# Capturamos la señal de interrupción (Ctrl+C)
trap 'echo; echo "Ejecución interrumpida."; calcular_promedio; exit 0' SIGINT

echo "Iniciando recolección de métricas de CPU (milicores) en dos nodos..."
echo "Presiona Ctrl+C para interrumpir la ejecución y ver el promedio y el factor de utilización."

while [ $contador -lt $max_samples ]; do
    # Se extrae y suma la utilización en milicores de los nodos
    suma_muestra=$(kubectl top nodes --no-headers | awk '{gsub("m","",$2); sum+=$2} END {print sum}')
    
    contador=$((contador + 1))
    echo "Muestra $contador: Suma de utilización = ${suma_muestra}m"
    
    suma_total=$(echo "$suma_total + $suma_muestra" | bc)
    sleep $interval
done

calcular_promedio
```