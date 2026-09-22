# Introduccion
This repository shows the development behind the design of a robotic vacuum which counts with localization algorithms and sistematic swipes
# Crear Transformacion de coordenadas a pixeles 
Para este paso se necesita recabar 10 puntos con sus pixeles aproximados obtenidos del png del mapa, para obtener esos valores he creado  un script de python que muestra el mapa en pantalla y va imprimiendo por la terminal las coordenadas de el pixel señalado en la imagen, como se muestra en esta demostracion

https://github.com/user-attachments/assets/e158e50a-f584-44ee-932c-d5da1bcbb27e


El siguiente paso es obtener la matriz de transformacion para relacionar la posicion en coordenadas xy del mapa con los pixeles uv de la imagen, de nuevo, usamos un script de python independiente para obtener la transformada afin de los 10 puntos seleccionados, en el enunciado se recomienda usar una matriz de trnsformacion homogenea distinta, pero como es una transformacion 2D (no necesitamos z) con la matriz afin basta. Nos ayudamos de la biblioteca de CV2 para esto, que cuenta con una funcion 'cv2.estimateAffine2D' para esto
