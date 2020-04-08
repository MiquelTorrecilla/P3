PAV - P3: detección de pitch
============================

Esta práctica se distribuye a través del repositorio GitHub [Práctica 3](https://github.com/albino-pav/P3).
Siga las instrucciones de la [Práctica 2](https://github.com/albino-pav/P2) para realizar un `fork` de la
misma y distribuir copias locales (*clones*) del mismo a los distintos integrantes del grupo de prácticas.

Recuerde realizar el *pull request* al repositorio original una vez completada la práctica.

Ejercicios básicos
------------------

- Complete el código de los ficheros necesarios para realizar la detección de pitch usando el programa
  `get_pitch`.

   * Complete el cálculo de la autocorrelación e inserte a continuación el código correspondiente.
```c
   void PitchAnalyzer::autocorrelation(const vector<float> &x, vector<float> &r) const {

    for (unsigned int l=0; l<r.size(); ++l) {
  		/* \TODO Compute the autocorrelation r[l] */
      r[l] = 0;
      for(unsigned int j=l; j < x.size(); j++)
        r[l] += (x[j]*x[j-l]);
      r[l] /= x.size();
    }
    if (r[0] == 0.0F) //to avoid log() and divide zero 
      r[0] = 1e-10; 
   }
``` 

   * Inserte una gŕafica donde, en un *subplot*, se vea con claridad la señal temporal de un sonido sonoro
     y su periodo de pitch; y, en otro *subplot*, se vea con claridad la autocorrelación de la señal y la
	 posición del primer máximo secundario.

	 NOTA: es más que probable que tenga que usar Python, Octave/MATLAB u otro programa semejante para
	 hacerlo. Se valorará la utilización de la librería matplotlib de Python.

	<img src ="ej1.png" witdh="640" align="center">

	*Observamos arriba la pronunciación de una vocal sonora, en este caso la 'a', y su periodo de pitch
	 indicado por los marcadores. Abajo, su autocorrelació, y la posición del segundo máximo respecto al
	 principal.*

   * Determine el mejor candidato para el periodo de pitch localizando el primer máximo secundario de la
     autocorrelación. Inserte a continuación el código correspondiente.
```c
   float PitchAnalyzer::compute_pitch(vector<float> & x) const {
    if (x.size() != frameLen)
      return -1.0F;

    //Window input frame
    for (unsigned int i=0; i<x.size(); ++i)
      x[i] *= window[i];

    vector<float> r(npitch_max);

    //Compute correlation
    autocorrelation(x, r);

    vector<float>::const_iterator iR = r.begin(), iRMax = iR+npitch_min, iRref;
  
  	for (iRref=iR+npitch_min; iRref<=iR+npitch_max; ++iRref){
  		if (*iRref > *iRMax){
				iRMax = iRref;
  		}
    }
    unsigned int lag = iRMax - r.begin();
```

   * Implemente la regla de decisión sonoro o sordo e inserte el código correspondiente.
```c
   bool PitchAnalyzer::unvoiced(float pot, float r1norm, float rmaxnorm) const {
    /// \TODO Implement a rule to decide whether the sound is voiced or not.
    /// * You can use the standard features (pot, r1norm, rmaxnorm),
    ///   or compute and use other ones.
    if(r1norm < 0.98 || rmaxnorm < 0.5 || pot < -50){
      return true;
    }else return false;
   }
```
- Una vez completados los puntos anteriores, dispondrá de una primera versión del detector de pitch. El 
  resto del trabajo consiste, básicamente, en obtener las mejores prestaciones posibles con él.

  * Utilice el programa `wavesurfer` para analizar las condiciones apropiadas para determinar si un
    segmento es sonoro o sordo. 
	
	  - Inserte una gráfica con la detección de pitch incorporada a `wavesurfer` y, junto a ella, los 
	    principales candidatos para determinar la sonoridad de la voz: el nivel de potencia de la señal
		(r[0]), la autocorrelación normalizada de uno (r1norm = r[1] / r[0]) y el valor de la
		autocorrelación en su máximo secundario (rmaxnorm = r[lag] / r[0]).

		Puede considerar, también, la conveniencia de usar la tasa de cruces por cero.

	    Recuerde configurar los paneles de datos para que el desplazamiento de ventana sea el adecuado, que
		en esta práctica es de 15 ms.
		
		<img src ="ej2.png" witdh="640" align="center">

      - Use el detector de pitch implementado en el programa `wavesurfer` en una señal de prueba y compare
	su resultado con el obtenido por la mejor versión de su propio sistema.  Inserte una gráfica
	ilustrativa del resultado de ambos detectores.

		<img src ="ej2-2.png" witdh="640" align="center">

	*Podemos obersvar el contorno de pitch dado por el Wavesurfer debajo de la representación de la señal
	 en tiempo. A bajo del todo, el contorno de pitch detectado por nuestro algoritmo, con una precisión
	 muy baja, la cual nos disponemos a mejorar analizando la mejoria de éste según la variación de sus
	 parámetros principales.*
  
  * Optimice los parámetros de su sistema de detección de pitch e inserte una tabla con las tasas de error
    y el *score* TOTAL proporcionados por `pitch_evaluate` en la evaluación de la base de datos 
	`pitch_db/train`..

		<img src ="taula.png" witdh="640" align="center">

	*Nos quedamos con aquellos valores de r1norm, rmaxnorm y pot que nos dan un fscore mas alto. En este caso,
	 respectivamente, 0.9, 0.4, y -38.*

   * Inserte una gráfica en la que se vea con claridad el resultado de su detector de pitch junto al del
     detector de Wavesurfer. Aunque puede usarse Wavesurfer para obtener la representación, se valorará
	 el uso de alternativas de mayor calidad (particularmente Python).

		<img src ="ej2-3.png" witdh="640" align="center">


     	*Observando el resultado actual, el contorno de pitch de abajo que es el nuestro, es considerablemente mejor
     	 que el del apartado anterior.*

Ejercicios de ampliación
------------------------

- Usando la librería `docopt_cpp`, modifique el fichero `get_pitch.cpp` para incorporar los parámetros del
  detector a los argumentos de la línea de comandos.
  
  Esta técnica le resultará especialmente útil para optimizar los parámetros del detector. Recuerde que
  una parte importante de la evaluación recaerá en el resultado obtenido en la detección de pitch en la
  base de datos.

  * Inserte un *pantallazo* en el que se vea el mensaje de ayuda del programa y un ejemplo de utilización
    con los argumentos añadidos.

- Implemente las técnicas que considere oportunas para optimizar las prestaciones del sistema de detección
  de pitch.

  Entre las posibles mejoras, puede escoger una o más de las siguientes:

  * Técnicas de preprocesado: filtrado paso bajo, *center clipping*, etc.
  * Técnicas de postprocesado: filtro de mediana, *dynamic time warping*, etc.
  * Métodos alternativos a la autocorrelación: procesado cepstral, *average magnitude difference function*
    (AMDF), etc.
  * Optimización **demostrable** de los parámetros que gobiernan el detector, en concreto, de los que
    gobiernan la decisión sonoro/sordo.
  * Cualquier otra técnica que se le pueda ocurrir o encuentre en la literatura.

  Encontrará más información acerca de estas técnicas en las [Transparencias del Curso](https://atenea.upc.edu/pluginfile.php/2908770/mod_resource/content/3/2b_PS Techniques.pdf)
  y en [Spoken Language Processing](https://discovery.upc.edu/iii/encore/record/C__Rb1233593?lang=cat).
  También encontrará más información en los anexos del enunciado de esta práctica.

  Incluya, a continuación, una explicación de las técnicas incorporadas al detector. Se valorará la
  inclusión de gráficas, tablas, código o cualquier otra cosa que ayude a comprender el trabajo realizado.

  También se valorará la realización de un estudio de los parámetros involucrados. Por ejemplo, si se opta
  por implementar el filtro de mediana, se valorará el análisis de los resultados obtenidos en función de
  la longitud del filtro.

*En primer lugar, realizamos un preprocesado de la señal con la técnica center clipping. Ésta consiste en poner
 a valor ‘0’ aquellas muestras cuyo valor sea inferior a un mínimo que nosotros establecemos. Con ésto conseguimos
 que aquellas muestras de voz pertenecientes a sonidos sordos queden eliminadas quedándonos sólo con los sonidos sonoros,
 que son los que determinan el pitch de la señal. Veamos cómo implementa dicha técnica nuestro algoritmo:*

```c
   float max_x = 0.0;
   unsigned int i;

   for(i = 0; i<x.size();i++){
     if(x[i] > max_x)
       max_x =x[i];
   }
   float center_x = 0.015*max_x;

   for(i=0; i<x.size(); i++){
     x[i] /= max_x;
     if (x[i] > center_x)
       x[i] -= center_x; 
     else if ( x[i] < -center_x)
       x[i] += center_x;
     else x[i] = 0;
   }
```

*Una vez detectado el pitch de cada una de las muestras de voz sonora, aplicaremos un filtro de mediana a nuestro
 resultado para eliminar errores puntuales en alguna muestra y así minimizar el error total de nuestro sistema. Este
 filtro se basa en la observación de las muestras vecinas, en este caso la anterior y la posterior a la actual, de
 manera que si detectamos una gran discontinuidad de la muestra respecto a sus vecinas, vamos a considerarlo un error
 y le daremos un nuevo valor. Veámos cómo se realiza y se aplica este filtro:*

```c
   std::vector<float> aux(f0);
   unsigned int j =0;
   float maximo, minimo;

   for (j=2;j<aux.size() -1;++j){
     minimo = min(min(aux[j-1], aux[j]),aux[j+1]);
     maximo = max(max(aux[j-1], aux[j]),aux[j+1]);
     f0[j] = aux[j-1] + aux[j] + aux[j+1] -minimo -maximo;
   }
```
   

Evaluación *ciega* del detector
-------------------------------

Antes de realizar el *pull request* debe asegurarse de que su repositorio contiene los ficheros necesarios
para compilar los programas correctamente ejecutando `make release`.

Con los ejecutables construidos de esta manera, los profesores de la asignatura procederán a evaluar el
detector con la parte de test de la base de datos (desconocida para los alumnos). Una parte importante de
la nota de la práctica recaerá en el resultado de esta evaluación.
