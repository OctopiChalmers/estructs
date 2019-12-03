
## Extending `imperative-edsl` with struct types

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
