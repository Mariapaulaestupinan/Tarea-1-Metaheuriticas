# **Heurística constructiva para el problema de ruteo en arcos capacitado** 
## **Indice de funciones**  
1. [dijkstra_mas_corto](#dijkstra_mas_corto) - Encuentra el camino más corto en un grafo ponderado utilizando el algoritmo de Dijkstra. 
2. [encontrar_arco_mas_cercano](#encontrar_arco_mas_cercano) - Encuentra el arco mas cercano al nodo actual basado en el menor costo
3. [construir_ruta](#construir_ruta) - Construye la ruta teniendo en cuenta la capacidad de los vehiculos 
## **dijkstra_mas_corto**
```plaintext
INICIO dijkstra_mas_corto(grafo, inicio, destino)

    // Utilizar el algoritmo de Dijkstra para encontrar el camino más corto
    camino_mas_corto ← calcular_camino_mas_corto(grafo, inicio, destino, peso='costo')

    RETORNAR camino_mas_corto

FIN dijkstra_mas_corto
```
## **encontrar_arco_mas_cercano**
```plaintext
INICIO encontrar_arco_mas_cercano(grafo, nodo_actual, arcos_pendientes)

    min_costo ← ∞  // Inicializar el costo mínimo con un valor muy alto
    arcos_candidatos ← []  // Lista para almacenar arcos con el menor costo encontrado

    // Encontrar el menor costo posible desde nodo_actual a los extremos de los arcos pendientes
    PARA CADA (u, v, data) EN arcos_pendientes HACER
        costo ← calcular_costo_mínimo(grafo, nodo_actual, u)
        SI costo < min_costo ENTONCES
            min_costo ← costo
        FIN SI
    FIN PARA

    // Guardar solo los arcos que tienen el menor costo encontrado
    PARA CADA (u, v, data) EN arcos_pendientes HACER
        costo ← calcular_costo_mínimo(grafo, nodo_actual, u)
        SI costo == min_costo ENTONCES
            AGREGAR (u, v, data) A arcos_candidatos
        FIN SI
    FIN PARA

    // Buscar si hay un arco donde el nodo inicial sea nodo_actual
    PARA i, (u, v, data) EN arcos_pendientes HACER
        SI (u, v, data) ESTÁ EN arcos_candidatos Y u == nodo_actual ENTONCES
            RETORNAR (i, (u, v, data))  // Priorizar el arco con nodo_actual como inicio
        FIN SI
    FIN PARA

    // Si no hay un arco con nodo_actual como inicio, elegir cualquiera de los mejores candidatos
    PARA i, (u, v, data) EN arcos_pendientes HACER
        SI (u, v, data) ESTÁ EN arcos_candidatos ENTONCES
            RETORNAR (i, (u, v, data))  // Devuelve el primero encontrado
        FIN SI
    FIN PARA

FIN encontrar_arco_mas_cercano
```
## **construir_ruta**
```plaintext
INICIO construir_rutas(grafo, capacidad, deposito)
    
    inicio_tiempo ← obtener_tiempo_actual()

    // Ordenar arcos por costo ascendente
    arcos_pendientes ← ordenar_arcos_por_costo(grafo)

    rutas ← []  // Lista para almacenar rutas construidas
    costos ← [] // Lista para almacenar costos de cada ruta

    MIENTRAS arcos_pendientes NO esté vacío HACER
        ruta ← [(deposito, deposito, False)] // Inicializa ruta con el depósito
        carga_actual ← 0
        costo_total ← 0
        nodo_actual ← deposito

        MIENTRAS arcos_pendientes NO esté vacío HACER
            arco_seleccionado ← encontrar_arco_mas_cercano(grafo, nodo_actual, arcos_pendientes)

            SI arco_seleccionado ES None ENTONCES
                SALIR DEL BUCLE INTERNO  // No hay más arcos disponibles

            u, v, data ← arco_seleccionado

            // Verificar si la demanda del arco supera la capacidad del vehículo
            SI carga_actual + data["demanda"] > capacidad ENTONCES
                arco_seleccionado ← None
                
                // Buscar un arco con menor demanda que pueda ser atendido
                PARA cada (a, b, d) en arcos_pendientes HACER
                    SI carga_actual + d["demanda"] ≤ capacidad ENTONCES
                        arco_seleccionado ← (a, b, d)
                        ROMPER CICLO  // Selecciona el primer arco viable
                
                SI arco_seleccionado ES None ENTONCES
                    SALIR DEL BUCLE INTERNO  // No hay arco viable, finaliza ruta

                u, v, data ← arco_seleccionado  // Se actualiza con el nuevo arco seleccionado

            // Encontrar el camino más corto desde la posición actual hasta el inicio del arco seleccionado
            camino_hacia_arco ← dijkstra_mas_corto(grafo, nodo_actual, u)

            // Agregar los arcos del camino más corto a la ruta
            PARA cada par (nodo1, nodo2) en camino_hacia_arco HACER
                ruta.agregar((nodo1, nodo2, False))
                costo_total += grafo[nodo1][nodo2]['costo']

            // Agregar el arco seleccionado a la ruta
            ruta.agregar((u, v, True))
            costo_total += data['costo']
            carga_actual += data["demanda"]
            nodo_actual ← v  // Actualizar la posición del vehículo

            // Eliminar ambos sentidos del arco seleccionado de la lista de arcos pendientes
            arcos_pendientes ← eliminar_arco(arcos_pendientes, u, v)

        // Si la ruta no termina en el depósito, encontrar camino de retorno
        SI nodo_actual ≠ deposito ENTONCES
            camino_de_retorno ← dijkstra_mas_corto(grafo, nodo_actual, deposito)

            // Agregar los arcos del camino de retorno a la ruta
            PARA cada par (nodo1, nodo2) en camino_de_retorno HACER
                ruta.agregar((nodo1, nodo2, False))
                costo_total += grafo[nodo1][nodo2]['costo']

        rutas.agregar(ruta)
        costos.agregar(costo_total)

    tiempo_ejecucion ← obtener_tiempo_actual() - inicio_tiempo

    RETORNAR rutas, costos, tiempo_ejecucion
FIN
```
