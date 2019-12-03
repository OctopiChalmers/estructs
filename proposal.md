
## Extending `imperative-edsl` with struct types

### Simple struct values

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

```Haskell

data Point = Point {
  x :: Int32 ,
  y :: Int32
}

mainC = do

  p <- initSt "p" (Point 1 2)
  q <- initSt "q" (Point 3 4)
  
  sx  <- fmap (+) (get @"x" p) <*> (get @"x" q)
  sy  <- fmap (+) (get @"y" p) <*> (get @"y" q)
  mid <- initSt "mid" $ Point (sx / 2) (sy / 2)
  
  return ()
```

### Pointers to struct values

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

## Interfacing with struct APIs

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
