# Java újdonságok

A JEP 440: Record Patterns teszi lehetővé, hogy egy record típusú változót
nem kell létrehozni, és meghívni az accessor-okat, hanem a komponensek értékeit
közvetlenül a változóknak lehet értékül adni.

Ez csak recorddal működik, és a record fejlécében szereplő komponensek sorrendje 
alapján működik, nem kell névegyezés, és nem veszi figyelembe a konstruktor(oka)t.

pl.

```java
public record Rectangle(Point x, Point y) implements Shape {}

public record Triangle(Point x, Point y, Point z) implements Shape {}
```

```java
public Point getCenter(Shape shape) {
    return switch (shape) {
        // ...
        case Rectangle(Point topLeft, Point bottomRight) ->
            // ...
        case Triangle(Point p1, Point p2, Point p3) ->
            // ...
    };
}
```

`topLeft`, `bottomRight`, `p1`, `p2`, `p3` a record definícóban lévő
sorrend alapján kapnak értéket.