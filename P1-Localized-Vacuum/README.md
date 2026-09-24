# Introduccion
This repository shows the development behind the design of a robotic vacuum which counts with localization algorithms and sistematic swipes
# Crear Transformacion de coordenadas a pixeles 
Para este paso se necesita recabar 10 puntos con sus pixeles aproximados obtenidos del png del mapa, para obtener esos valores he creado  un script de python que muestra el mapa en pantalla y va imprimiendo por la terminal las coordenadas de el pixel señalado en la imagen, como se muestra en esta demostracion

https://github.com/user-attachments/assets/e158e50a-f584-44ee-932c-d5da1bcbb27e


El siguiente paso es obtener la matriz de transformacion para relacionar la posicion en coordenadas xy del mapa con los pixeles uv de la imagen, de nuevo, usamos un script de python independiente para obtener la transformada afin de los 10 puntos seleccionados, en el enunciado se recomienda usar una matriz de trnsformacion homogenea distinta, pero como es una transformacion 2D (no necesitamos z) con la matriz afin basta. Nos ayudamos de la biblioteca de CV2 para esto, que cuenta con una funcion *cv2.estimateAffine2D* para esto. Mi matriz de transformacion resultante es la siguiente:

<img width="331" height="80" alt="image" src="https://github.com/user-attachments/assets/1ecb8dde-a6f1-4e9a-b46e-37defcf9b288" />
Que se traduce en:

<img width="313" height="84" alt="image" src="https://github.com/user-attachments/assets/5ad87759-9b5e-4fa5-8da7-eff1ddde9ff4" />


Con estos datos ya se puede relacionar la posición en pixeles 2D a las coordenadas xy del robot resolviendo el sistema de ecuaciones.

# Creación de las casillas del mapa
Primero, el mapa se tiene que dividir en obstaculo (fisico o virtual) y espacio navegable, que se representan como negro y blanco respectivamente. 
En esta etapa me entretuve porque al leer que habia que añadir cierta erosión preventiva al mapa (para no chocar con las paredes o pasar muy cerca etc) decidi añadirlo de golpe a toda la imagen usando OpenCV usando un kernel que se aplicaba a todos los pixeles y "engordaba" las paredes. No resulto muy bien porque dificultaba mucho el cuadrar despues las rejillas, que dejaban pequeños huecos o zonas muy estrechas donde el robot no podria pasar. Tras descartar este metodo, pase con el que me quede finalmente que es: Recorrer toda la imagen (1012x1012) en regiones de 35x35 pixeles que es el tamaño exacto del robot proporcionado por el enunciado. En cada region se hace recuento de los pixeles blancos y los negros y si hay mas de cierto umbral de negros se rellena la casilla entera. Asumiendo que el robot no podria pasar comodamente por el centro de la casilla. 
Una vez recorridas todas las regiones se representan las casillas visualmente con lineas negras, de nuevo con la libreria OpenCV, como se pintan encima de las regiones, las casillas acaban siendo un poco mas pequeñas que la aspiradora, que seria lo ideal.
<img width="1012" height="1012" alt="image" src="https://github.com/user-attachments/assets/c7ddadc3-78b1-4315-a5fe-ae286d447fb4" />

Una vez hecho esto, hay que decidir como se van a representar las casillas dependiendo de su estado durante la ejecucion, en mi caso elegi:

- Naranja: No visitada
- Verde: Visitada (esta no se implementa hasta implementar el movimiento)
- Azul claro: Puntos de retorno
- Rojo: Puntos Criticos
- Azul Oscuro: Casilla de inicio

# Planificacion
Como bien hemos dado en clase para la planificacion se va a usar el Backtracking Spiral Algorithm (BSA), este algoritmo es un algoritmo de cobertura completa, usa barridos sistematicos en forma de espiral siguiendo una prioridad establecida, en mi caso ESWN (Este,Sur,Oeste,Norte). El funcionamiento basico de este algoritmo (usando mi orden de prioridad como ejemplo) es el siguiente:

El robot empieza el movimiento hacia E, mientras no se tope con nada (obstaculo o casilla visitada) sigue en esta dirección. Si se topa con algo cambia la direccion hacia S y hace lo mismo, la diferencia es, segun q direccion lleve, va a comprobar primero una direccion u otra. Es decir, cuando se "choca" en E, primero comprueba S,luego W y por ultimo N, pero si se "choca" en S, primero comprueba W,luego N y por ultimo E. Este procedimiento lo ejecuta hasta que no queden casillas libres esto se denomina punto critico. 
Esta situacion tiene una facil solucion, durante el recorrido de la espiral va anotando que casillas vecinas fuera de la trayectoria estan libres, para por si se queda atascado volver a una de ellas, estos son los puntos de retorno

La función planificacionse ha planteado como una funcion recursiva por cada espiral, en cada llamada se pasa a traves de un bucle for de 4 iteraciones par buscar a que direccion moverse, siguiendo la prioridad, si encuentra una casilla libre, se mueve y vuelve a llamar a la funcion, si no encuentra ninguna casilla libre pasa a la siguiente direccion y pasa a la siguiente iteracion. Cuando termina la espirar la recursividad termina. Para evitar que se pare la planificacion, la llamada a spiral() comienza desde otra plan(), no para de llamar a spiral() hasta que se acaben los puntos de retorno. Durante este proceso se van pintando las casillas.

Durante el desarrollo de esta parte he tenido algunos problemas con versiones anteriores donde no implementaba correctamente del todo el BSA, en un principio el algoritmo no era recursivo sino que se basaba en una unica llamada que aunque hacia un barrido completo y eficiente no era BSA. En esta versión la prioridad se implentaba de forma absoluta, con esto me refiero a que buscaba siempre la opcion de mayor prioridad basandose siempre en el orden ESWN, lo que reslutaba en barridos. Esta opcion no estaba tan lejos de lo que tengo actualmente si hubiera que la prioridad se buscase según la direcciona cual del robot, como he explicado antes. El cambio a una funcion recursiva ha surgido porque como no me funcinaba esta version, me puse a investigar y me salio como ejemplo de explicacion una version recursiva, que decidi implementar a la practica porque me gusto la idea.
## Busqueda de puntos de retorno
