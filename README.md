# Practica gTests
# dps-laboratory2

La instalación de Google Test y el lanzamiento de los tests se realizará sobre Ubuntu 20.04

Paso 1: Instalación de Librería de Google Test 
	Se utiliza el comando sudo apt-get install libgtest-dev

Paso 2: Instalación de CMake para compilar la librería
	Se utilizan los siguientes comandos
  
	sudo sudo apt-get install cmake

	sudo cmake CMakeLists.txt
	
	sudo make
	
Paso 3: Copiar los archivos libgtest.a y libgtest_main.a a la carpeta /usr/lib

Paso 4: En la carpeta donde se encuentra el programa principal y los tests, compilar y lanzar los tests. Se utilizan los siguientes comandos
	
	cmake CMakeLists.txt
	
	make
	
	./runTests
	
	
Posteriormente, saldrán los resultados de los tests realizados indicando si se han realizado correctamente o no.

~~~
[==========] Running 6 tests from 3 test suites.
[----------] Global test environment set-up.
[----------] 2 tests from wrapAddFunctionTest
[ RUN      ] wrapAddFunctionTest.NonWrappingNums
[       OK ] wrapAddFunctionTest.NonWrappingNums (0 ms)
[ RUN      ] wrapAddFunctionTest.WrappingNums
[       OK ] wrapAddFunctionTest.WrappingNums (0 ms)
[----------] 2 tests from wrapAddFunctionTest (0 ms total)

[----------] 2 tests from wrapMulFunctionTest
[ RUN      ] wrapMulFunctionTest.NonWrappingMulNums
[       OK ] wrapMulFunctionTest.NonWrappingMulNums (0 ms)
[ RUN      ] wrapMulFunctionTest.WrappingMulNums
[       OK ] wrapMulFunctionTest.WrappingMulNums (0 ms)
[----------] 2 tests from wrapMulFunctionTest (0 ms total)

[----------] 2 tests from wrapShiftFunctionTest
[ RUN      ] wrapShiftFunctionTest.NonWrappingMulBNums
[       OK ] wrapShiftFunctionTest.NonWrappingMulBNums (0 ms)
[ RUN      ] wrapShiftFunctionTest.WrappingMulBNums
[       OK ] wrapShiftFunctionTest.WrappingMulBNums (0 ms)
[----------] 2 tests from wrapShiftFunctionTest (0 ms total)

[----------] Global test environment tear-down
[==========] 6 tests from 3 test suites ran. (0 ms total)
[  PASSED  ] 6 tests.
~~~

# NOTA
Dado que los Google Test están orientados principalmente a C++ y puesto que, tanto el programa principal como los tests son archivos C++ (extensión .cpp), es importante modificar las librerías.
Inicialmente se daban las librerías de C (*.h), por lo que, utilizando el cppreference se modifican a sus análogas en C++.
Se modifican las siguientes librerías:

math.h se cambia a cmath
	
limits.h se cambia a climits
	
stdio.h se cambia a cstdio
	
stdint.h se cambia a cstdint

# ANÁLISIS FUNCIÓN WrapFunctionShift

La función WrapFunctionShift puede incumplir la siguiente regla: 
# INT34-C. Do not shift an expression by a negative number of bits or by greater than or equal to the number of bits that exist in the operand

Wrapfunctionshift le pasamos dos variables, uint32_t( unsigned integer de 32 bytes) y y un unsigned int.

uint32_t, al ser de 32 bytes, cuando en el test se la asigna la variable UINT_MAX (valor máximo que puede tomar) equivale a 4294967295. 

Para desplazar a la izquierda, el esquema es el siguiente

b7 b6 b5 b4 b3 b2 b1 b0	-------------------- b6 b5 b4 b3 b2 b1 b0 0 (Left Logical Shift)

Para desplazar a la derecha, el esquema es el siguiente

b7 b6 b5 b4 b3 b2 b1 b0	-------------------- 0 b7 b6 b5 b4 b3 b2 b1(Right Logical Shift)

Si realizamos el test definido, donde a UINT_MAX lo desplazamos 12 unidades

UINT_MAX = 11111111111111111111111111111111 (32 1s)

Left Shift (se desplaza a la izquierda 12)

RESULT Left Shift = 11111111111111111111000000000000 (20 1s + 12 0s)

Right Shift (se desplaza a la derecha 32-12 = 20)

RESULT Right Shift = 00000000000000000000111111111111 (20 0s + 12 1s)

Al realizar la operación OR (se comparan los bits, siendo el resultado 1 si hay algún 1 y siendo el resultado 0 si ambos bits son 0)

RESULT OR = 11111111111111111111111111111111 (32 1s)

Se puede deducir que al aplicar WrapfunctionShift sobre UINT_MAX, si las unidades desplazadas son menor a 32, no importa cuántas unidades se desplace que siempre el resultado va a ser el mismo valor (los 1s que se pierden por la izquierda se compensan por la derecha y al hacer operación OR queda igual).

CASO CUANDO ui_b ES MAYOR QUE 32 

Tras hacer pruebas con diferentes valores, se deduce que, cuando se introduce un número de desplazamiento mayor que 32, se fuerza a que el número se desplace menos de 32. Para que quede más claro, se adjunta un ejemplo.

	- Si aplicamos la función wrapFunctionShift(2,6) el resultado es 128
	- Si aplicamos la función wrapFunctionShift(2,38) el resultado también es 128 (38 - 32 = 6)
	- Si aplicamos la función wrapFunctionShift(2,70) el resultado también es 128 (70 -32*2 = 6)
