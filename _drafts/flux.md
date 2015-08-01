Encontrar una arquitectura para aplicaciones Android no es tarea fácil. 

La tendencia actual es adaptar "Clean Architecture", una propuesta de 2012 de Uncle Bob para aplicaciones web. 

Definir una arquitectura para tu aplicación es importante porque, quieras o no, toda aplicación va a tener una arquitectura. Así que más te vale ser tú quien la defina y no dejar que vaya simplemente surgiendo. 

La arquitectura nos define quién es responsable de qué. Cómo se organizan las capas y quién habla con quién. 
El problema viene cuando la arquitectura no nos deja ver la aplicación. 

Tener una separación de responsabilidad por capas es bueno. Que tu aplicación funcione y puedas acabarla en un tiempo finito es mejor. 

No pretendo decir Clean Architecture es un mal concepto. No existen las soluciones perfectas para todos los casos. Clean Architecture es posiblemente la mejor solución para un tipo de problema concreto. 
Y la realidad es que la mayoría de apps Android hoy día no son ese caso.

La sobreingeniería necesaria para montar Entities, Interactors, Presenters, Gateways, Repositories, etc. no se justifica en proyectos de dos o tres meses. 
Hay casos concretos en los que una app se va a desarrollar durante más de 6 meses y va a vivir durante años. Si ese es tu caso, adelante, Clean Architecture es tu modelo.

Pero si no es así hay alternativas más ligeras y con curvas de aprendizaje mucho mucho más suaves.

## Introducing Flux Architecture

Flux es la architectura utilizada por Facebook para construir sus client-side web applications. Al igual que "Clean Architecture" no está pensada para apps móviles, pero sus características y simpleza nos van a permitir adaptarla mucho más facilmente.

Las dos características clave para entender Flux son:

- La aplicación se divide en tres partes principales: dispatcher, stores y vistas
- El flujo de datos es siempre unidireccional.


[flux architecture]

Un flujo de datos unidireccional es el núcleo de la arquitectura Flux y es lo que hace que sea tan fácil de adoptar. Además proporciona grandes ventajas a la hora de testear la aplicación como veremos más adelante.

- Vista: interfaz de la aplicación. Originan acciones en respuesta a la interacción con el usuario.
- Acción: Son objetos simples que se identifican por un _tipo_ y contienen los datos relativos a esa acción.
- Dispatcher: Hub central por el que pasan todas las acciones y cuya responsabilidad es hacerlas llegar a todas las Stores.
- Store: Mantienen el estado para un dominio concreto de la aplicación. Responden a las acciones según ese estado y emiten un evento _change_ cuando han terminado. Este evento es utilizado por las vistas para actualizar su interfaz.

[más información]

## Flux Android Architecture

El objetivo es montar una architectura con un buen balance entre simpleza y facilidad de escalar y testear. 

El primer paso es mapear los elementos de Flux con componentes de una aplicación Android

Hay dos elementos de Flux cuya implementación es inmediata:

- Vista: una Activity o Fragment.
- Dispatcher: un bus de eventos.

## Actions

Las acciones tampoco son complejas. Las implementaremos como simples POJOs que serán fáciles de enviar usando el Bus.

[ejemplo de Action]


### Stores

Es quizá el concepto más complejo de asimilar de Flux. Si has trabajado antes con Clean Architecture, además te resultará incómodo de aceptar, pues las Stores asumen responsabilidades que antes estaban separadas en distintas capas.

Las Stores contienen el estado de la aplicación y la lógica de negocio. Son algo similar a _rich data models_ pero pueden gestionar el estado de varios objetos, no sólo uno. 
Reaccionan a Acciones emitidas por el Dispatcher y ejecutan lógica de negocio.

Ningún otro componente del sistema debería necesitar saber nada sobre el estado de la aplicación.

Por ejemplo, en una aplicación de búsqueda de establecimientos se utilizaría una SearchStore para llevar un registro del elemento buscado, los resultados de la búsqueda y el historial de búsquedas pasadas. En esa misma aplicación un ReviewedStore contendría los establecimientos comentados y contendría la logica necesaria para, por ejemplo, ordenarlos según el tipo de review

Las Stores sin embargo *no* son Repositories. Su responsabilidad no es obtener datos de una fuente externar (API o DB) sino gestionar los datos que se le proveen. 

Entonces, ¿cómo obtiene datos una aplicación Flux?

### Network requests and asynchronous calls

En el gráfico inicial sobre Flux falta una parte importante a propósito: llamadas de red. 
El siguiente gráfico completa al inicial añadiendo más detalles.

Las llamadas asíncronas de red para obtener son lanzadas desde el Action Creator. 
Un Adaptador de Red hace la llamada asíncrona al API correspondiente y devuelve el resultado. El Action Creator lanza entonces una Acción con el resultado.

Esto tiene dos ventajas fundamentales:

- *Tus Stores son completamente síncronas* Esto hace que la lógica de la Store sea mucho más simple de seguir. Los bugs serán mucho más fáciles de tracear y testear una Store se convierte en un trabajo muy sencillo. Todos los cambios de estado van a ser *síncronos*. Testear Stores se convierte en: lanzar acciones y comprobar que el estado final es el esperado.
- *Todas las acciones se lanzan desde un Action Creator* Tener un único punto en el que se crean y lanzan todas las acciones del usuario simplifica enormemente encontrar errores. Olvídate de navegar por clases intentando descubrir dónde se origina una acción. Todo ocurre aquí. Y como las llamadas asíncronas ocurren _antes_ todo lo que sale del ActionCreator es síncrono lo que mejora significativamente trazabilidad y testabilidad.

## Show me the code: To-Do App




