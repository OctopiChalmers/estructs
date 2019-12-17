The software of IoT devices are built with low-level languages as C or C++, which opens the door to often introduce vulnerabilities by programming errors. To remedy this situation, the Octopi SSF project at Chalmers (https://octopi.chalmers.se/) researchs the utilization of high-level languages for programming IoT embedded devices in order to reason about security properties of the built software. In this goal, there are many challenges ahead and one of them is to efficiently generate C-code from Haskell. In this light, the goal of this master thesis is to generate C programs which interface with struct-based APIs using the [imperative-edsl](http://hackage.haskell.org/package/imperative-edsl-0.8) Haskell library. Currently the library supports generation of C code manipulating basic data types (int, bool, etc.), arrays and pointers. In this project, we aim to extend it with struct types, and further use this extension to program _existing_ struct-based C APIs. One such API of interest to the project is the [device driver API](https://docs.zephyrproject.org/latest/reference/drivers/index.html) offered by Zephyr operating system for IoT.

## Extending `imperative-edsl` with struct types

The first phase of the project would be dedicated to extending the imperative-edsl library with primitives which support code generation for struct types.

### Simple struct values

We would like to generate a C program, such as the following, by writing a Haskell program which uses the library (see below). In the following example, the programs compute the distance between two _points_, where a point is implemented as a struct type.

_Desired C code_
```C

/* Define a struct type called "point". */
struct point {
   int x;
   int y;
};

/* Declare an alias "point" to "struct point". */
typedef struct point point;

int main(){

  /* Define two points */
  point p = { .x = 1, .y = 2 };
  point q = { .x = 3, .y = 4 };

  /* Compute the midpoint */
  int   sx  = p.x + q.x;
  int   sy  = p.y + q.y;
  point mid = { .x = sx / 2, .y = sy / 2 };

  return 0;
}
```

_Haskell program which uses imperative-edsl_
```Haskell

data Point = Point {
  x :: Int32 ,
  y :: Int32
}

mainC :: Program StructCMD (Param2 exp CType) ()
mainC = do

  p <- initSt "p" (Point 1 2)
  q <- initSt "q" (Point 3 4)
  
  sx  <- fmap (+) (get @"x" p) <*> (get @"x" q)
  sy  <- fmap (+) (get @"y" p) <*> (get @"y" q)
  mid <- initSt "mid" $ Point (sx / 2) (sy / 2)
  
  return ()
```

Ideally, this phase involves the following subgoals:

- Designing a Haskell interface for programming structs containing fields of basic types
- Generating struct definitions from the Haskell interface
- Generating code for creating and accessing struct values from the Haskell interface

### Pointers to struct values

The library currently [supports pointers](http://hackage.haskell.org/package/imperative-edsl-0.8/docs/Language-Embedded-Imperative-CMD.html#g:4) for basic data types. The next step is to extend this pointer API for creating and dereferencing struct pointers. For example, we would like to generate C code such as the following: 

_Desired C code_
```C

/* Swap x and y */
void swap(point *r){
  int temp = r->x;
  r->x = r->y;
  r->y = temp;
}

int main(){
  ... 
  point *mp = &mid;
  swap(mp);
  ...
}

```

With the ability to manipulate struct pointers, the next step would be to support struct definitions with self-referential fields, such as the following:

_Desired C code_
```C
struct Node { 
  int data; 
  struct Node *next; 
}; 

```

## Interfacing with struct APIs using `imperative-edsl`

In the second phase, we seek to use our extension from the first phase to generate programs which use struct APIs, such as the following: 

_Desired C code_
```C

#include <zephyr.h>
#include <device.h>
#include <gpio.h>

#define LED_PORT LED0_GPIO_CONTROLLER
#define LED     LED0_GPIO_PIN

/* 1000 msec = 1 sec */
#define SLEEP_TIME      1000

void main(void){
  int cnt = 0;
  struct device *dev;

  dev = device_get_binding(LED_PORT);
  /* Set LED pin as output */
  gpio_pin_configure(dev, LED, GPIO_DIR_OUT);

  while (1){
    /* Set pin to HIGH/LOW every 1 second */
    gpio_pin_write(dev, LED, cnt % 2);
    cnt++;
    k_sleep(SLEEP_TIME);
  }
}
```

This program (called _blinky_) uses the device driver API provided by Zephyr OS to blink the LED on a device supported by Zephyr. Note that, unlike the previous examples, the struct definition is not generated, but instead simply imported using the appropriate header files. The main challenge here is to interact with an _existing_ API which uses structs, from programs written using imperative-edsl. What should the Haskell program look like? :)
