# Introducción
Este documento pretende estandarizar la escritura de código C/C++ en los proyectos desarrollados en SCI. Algunas de las pautas tienen buenas razones, mientras que otras simplemente son preferencias personales. Además con esto se quiere reducir los errores comunes, obtener una base de código más consistente y evitar discusiones sobre detalles insignificantes.

# Nombres
* snake_case para todo lo que no sean definiciones, incluido nombres de archivos
  - Ej: uart.c, state_machine.h, sci_drivers.cpp
  ```Cpp
  void this_is_a_functions(void) {
    uint8_t this_is_a_variable;
  }
  ```
* UPPER_CASE para las definiciones
  ```Cpp
  #define THIS_IS_A_DEFFINITION (0)
  #define PI (3.1416)
  ```
* En caso se desarrolle una librería, cada función que se pueda acceder desde fuera debe estar prefijada con el nombre de la libería o módulo
  - Ej: uart.h/uart.c, i2c.h/i2c.c, state_machine.h/state_machine.c
  ```Cpp
  void uart_init(void);
  void uart_write(void);
  void uart_read(void);
  void i2c_init(void);
  void i2c_write(void);
  void state_machine_next_state(void);
  ```
# Indentación
* 02 espacios de indexado
* Espacios alrededor de operadores
  ```Cpp
  a = b + c;
  ```
* Llaves en la misma línea que el nombre de la función o estructura
  ```Cpp
  if(condicion) {

  }

  void name_function(void) {

  }

  while(0) {

  }
  ```
# Comentarios
* Comentar lo que no es obvio con solo leer el código
* Concéntrece en el por qué, en lugar del como y el qué
* No te excedas
* Prefiere buenos nombres de variables y funciones antes que mas comentarios
* Utilice // para comentarios al final de una línea de código
* Utilice /* */ para comentarios de varias líneas de código y comentarios de una línea que no están al final de una línea de código
  ```Cpp
  /* Este es un comentario
  * de varias
  * líneas
  */
  void this_is_a_function(void) {

    /* Este es un comentario de una línea */
    if( 1 ) { // Este es un comentario al final de una línea de código

    }
  }
  ```
* Seguir formato doxygen para la documentación de funciones, módulos, estructuras y clases
  ```Cpp
  /**
   * @brief Inicializa el módulo UART
   *
   * @param baud_rate Tasa de baudios deseada
   * @param bits_stop Numero de bits de parada
   * 
   * @return Retorna 1 si la inicialización fue exitosa
   */
  uint8_t uart_init(uint32_t baud_rate, uint8_t bits_stop);
  ```
* Usar marcadores TODO/FIXME
  - TODO:
    - Indica una tarea pendiente que se debe realizar mas adelante
    - Funcionalidades por implementar
    - Mejoras u optimizaciones que se planean pero no son urgentes
    - Tareas que requieren más investigación o tiempo
  - FIXME:
    - Indica problemas en el código y deben ser corregidos lo antes posible
    - Código que está roto o no produce los resultados esperados
    - Soluciones provisionales que deben ser reemplazadas por implementaciones más robustas
    - Problemas conocidos que aún no fueron solucionados
  ```Cpp
  void open_tcp_communication(void) {
    // ...
    process();
    //TODO: Implementar un error handler para evitar conflictos
  }

  void get_ring_buffer(void) {
    process();
    if(is_ring_buffer_null()) { // FIXME: Esta función falla cuando el array es muy grande
      // ...
    }
  }
  ```
# Defines
* Usar formate UPPER_CASE
* Defina todos los números constantes y comente si no está claro de dónde vienen
* Utilice siempre paréntesis, incluso para números individuales, para evitar una expansión inesperada
  ```Cpp
  #define CONSTANT_NUMBER (100) //This number ...
  ```
* Use do{}while(0) para macros de instrucciones múltiples, para evitar errores de expansión y que actúen como función, es decir, pueden ir seguidas de punto y coma
  ```Cpp
  #define ASSERT(expression)    \
      do {                      \
        if(!(expression)) {     \
          assert_handler();     \
        }                       \
      }while(0)
  ```
* Evite definir macros, ya que son propensas a errores y menos legibles
# Archivos .h
* Use #ifndef en cada archivo .h para evitar inclusiones duplicadas y recursivas
  ```Cpp
  #ifndef UART_H
  #define UART_H

  #endif /* UART_H */
  ```
* Incluya los archivos de cabecera en orden, desde global hasta local
  ```Cpp
  #include <stdio.h>
  #include <strstr.h>
  #include "drivers/sci_drivers.h"
  #include "app/state_machine.h"
  #include "config.h"
  ```
* Incluir encabezados solo si es necesario, para mejorar la legibilidad y entendimiento del código
* Utilizar **forward declarations** en estructuras si es necesario, para reducir dependencias circulares entre archivos de encabezado
# Sentencias switch
* Si se evalúa una variable tipo ```enum```, maneje todos los casos y omita el caso ```default```
* Sin el caso ```default```, el compilador colocará una advertencia y obligará a actualizar los ```case``` cuando se agrege un nuevo valor de ```enum```
  ```Cpp
  switch(color) {
    case R: 
      // ...
      break;
    case G: 
      // ...
      break;
    case B: 
      // ...
      break;
  }
  ```
# Sentencias if/else
* Use siempre {} incluso para declaraciones de una sola línea
# Funciones
* Las funciones que solo se usen dentro de un único .c deben ser de tipo ```static```
* Utilice ```void``` como parámetro en declaraciones de funciones sin parámetros, porque en C, las funciones con paréntesis vacíos se pueden llamar con cualquier número de parámetros
  ```h
  /* drivers.h */
  void drivers_init(void);
  ```
  ```Cpp
  /* drivers.c */
  static void io_init(void) {
    // ...
  }

  void drivers_init(void) {
    io_init();
    // ...
  }
  ```
* Pase estructuras por puntero cuando corresponda, para reducir el uso de la pila
* Las funciones pequeñas o que se llamen de forma frecuenta pueden ser de tipo ```inline``` para optimizar el rendimiento.
  ```Cpp
  inline void toggle_led(uint8_t pin) {
    // ...
  }

  /* Funcion que solo se usa en este .c */
  static inline void toggle_alarm(void) {
    // ...
  }
  ```
# Tipos de dato
## Typedef
* Siempre escriba ```typedef``` en los ```enum``` para evitar el uso de la palabra clave ```enum``` antes de cada valor de enumeración
* Siempre agregue el sufijo de enumeraciones _e
* Los valores de enumeración deben seguir el formato UPPER_CASE y tener el prefijo de la enumeración
  ```Cpp
  typedef enum {
    COLORS_RED,
    COLORS_GREEN,
    COLORS_BLUE
  }colors_e;
  ```
## Enteros de ancho fijo (stdint.h)
* Utilice enteros de ancho fijo (uint8_t, uint16_t, int32_t, etc)
* Hace que el uso de memoria sea mas obvio y de fácil portabilidad entre tecnologías
## Constantes
* Toda variable que no debería cambiar su valor, debe ser constante
* Protege contra modificación accidental y hace mas legible y explícito el código
# Optimizaciones y seguridad
* Evitar el uso de memoria dinámica y más si se está usando algún RTOS
* Para variables que interactúan con hardware o ISRs (Interrupt Service Routines) es obligatorio usar el prefijo ```volatile```
# Clang format
* Use el siguiente clang-format
  ```Cpp
  { BasedOnStyle: Google, ColumnLimit: 150, BreakBeforeBraces: Attach, AllowShortCaseLabelsOnASingleLine: true,AllowShortFunctionsOnASingleLine: false}
  ```