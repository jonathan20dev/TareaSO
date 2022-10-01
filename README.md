###  Ejercicio 2: Lectores y escritores

***Definición del problema***


> hay un área de datos compartida entre varios procesos. El área de datos puede ser un archivo, un bloque de memoria principal o incluso un banco de registros del procesador. Hay varios procesos que solo leen el área de datos (lectores) y otros que solo escriben en el área de datos (escritores). 


***Restricciones del problema***


	1. Cualquier número de lectores puede leer simultáneamente el archivo.
	2. Solo un escritor a la vez puede escribir en el archivo
	3. Si un escritor está escribiendo en el archivo, ningún lector puede leerlo.


#### ***Analisis del código***


##### Solución cuando lectura tiene prioridad sobre escritura


		Se utilizan tres variables: x, wsem, readCount


> **Semaphore x:** se utiliza para garantizar la exclusión mutua cuando se actualiza readCount, es decir, cuando cualquier lector entra o sale de la sección crítica 
>
>**semaphore wsem:** es utilizado tanto por lectores como por escritores. 
> 
> **int readCount**; indica el número de procesos que realizan la lectura en la sección crítica, inicialmente 0


Funciones del semaforo:


> – **semWait()** : disminuye el valor del semáforo.
>
> – **semSignal()** : incrementa el valor del semáforo.


#### **Proceso de lectura: **


1. El lector solicita la entrada a la sección crítica
1. Si está permitido:
	- incrementa el número de lectores dentro de la sección crítica. Si este lector es el primer lector que entra, bloquea el semáforo wsem para restringir la entrada de escritores si algún lector está dentro.
	- Cualquier otro lector puede entrar mientras otros ya están leyendo.
	- Después de realizar la lectura, sale de la sección crítica. Al salir, comprueba si no hay más lector dentro, señala el semáforo "wsem" ya que ahora, el escritor puede ingresar a la sección crítica.
1. Sale de la sección crítica.


----

```javascript
int readCount; 
semaphore x = 1, wsem = 1;

void reader()
{
    while (true){
			//incrementa el número de lectores dentro de la sección crítica
			semWait(x);
			readCount++;

			//No permite que un escritor entre a la sesión 🚫✍
						if(readCount == 1)
							semWait(wsem);

			//Permite la entrada a otros lectores, mientras haya uno en la sesión🙍‍♂️🙍‍♀️.
					semSignal(x);

			//El lector realiza la lectura 📖
					READUNIT();

			//El lector actual realizó la lectura y ahora se retira 👋
					semWait(x);
					readCount--;

			//Permite que se escriba sobre la unidad, cuando no hay lectores. 🆗
						if (readCount == 0)
							semSignal(wsem);
			semSignal(x);
    }
}
```


##### Proceso de escritura


1. El escritor solicita la entrada a la sección crítica.
1. Si está permitido, es decir, semWait() da un valor verdadero, ingresa y realiza la escritura. Si no 	está permitido, sigue esperando.
1. Sale de la sección crítica.


----

```javascript
int readCount;
semaphore x = 1, wsem = 1;

void writer()
{
    while (true){
			semWait(wsem); //Un escritor solicita entrar a la sesión crítica
			WRITEUNIT(); //El escritor realiza la escritura ✍
			semSignal(wsem); //El escritor abandona la sesión crítica
    }
}
```

##### Iniciando la ejecución simultanea

```javascript
void main()
{
		readCount = 0;
		parbegin (reader, writer);
}
```
