# Introduccion
This repository shows the development behind the design of a robotic vacuum which counts with localization algorithms and sistematic swipes
# Crear Transformacion de coordenadas a pixeles 
Para este paso se necesita recabar 10 puntos con sus pixeles aproximados obtenidos del png del mapa, para obtener esos valores he creado  un script de python que muestra el mapa en pantalla y va imprimiendo por la terminal las coordenadas de el pixel señalado en la imagen, como se muestra en esta demostracion

https://github.com/user-attachments/assets/e158e50a-f584-44ee-932c-d5da1bcbb27e


El siguiente paso es obtener la matriz de transformacion para relacionar la posicion en coordenadas xy del mapa con los pixeles uv de la imagen, de nuevo, usamos un script de python independiente para obtener la transformada afin de los 10 puntos seleccionados, en el enunciado se recomienda usar una matriz de trnsformacion homogenea distinta, pero como es una transformacion 2D (no necesitamos z) con la matriz afin basta. Nos ayudamos de la biblioteca de CV2 para esto, que cuenta con una funcion *cv2.estimateAffine2D* para esto. Mi matriz de transformacion resultante es la siguiente:

<img width="331" height="80" alt="image" src="https://github.com/user-attachments/assets/1ecb8dde-a6f1-4e9a-b46e-37defcf9b288" />
Que se traduce en:

<img width="313" height="84" alt="image" src="https://github.com/user-attachments/assets/5ad87759-9b5e-4fa5-8da7-eff1ddde9ff4" />


Con estos datos ya se puede relacionar la posicion en pixeles 2D a las coordenadas xy del robot.

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
Como bien hemos dado en clase para la planificacion se va a usar el Backtracking Spiral Algorithm (BSA), este algoritmo es un algoritmo de cobertura completa, usa barridos sistematicos en forma de espiral siguiendo una prioridad establecida, en mi caso ESWN. El funcionamiento basico de este algoritmo (usando mi orden de prioridad como ejemplo) es el siguiente:
