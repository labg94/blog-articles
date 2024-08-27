# La Revolución de los Records en Java: Comparaciones, Limitaciones y Patrones de Copia
¿Sigues en Java 8 u 11? Con Java 23 a punto de salir, es crucial mantenerse al día con las últimas características. Una de las más emocionantes es la introducción de los records. Pero, ¿qué son exactamente los **records** y cómo se comparan con los tradicionales POJOs (Plain Old Java Objects)? En este artículo, exploraremos estas preguntas y más.

## Comparación entre POJOs y Records
Los POJOs han sido la base de la programación en Java durante años. Son simples objetos que contienen datos y métodos para manipular esos datos. Sin embargo, la creación de POJOs puede ser tediosa, ya que requiere la escritura de mucho código boilerplate, como constructores, getters, setters, y métodos `equals`, `hashCode` y `toString`.

Aquí es donde los records entran en juego. Introducidos en Java 14 como una característica preliminar y estabilizados en Java 16, los records son una forma concisa de declarar clases que son principalmente portadoras de datos.

### POJO inmutable
Para hacer una comparación justa (los records son inmutables) aquí está un ejemplo de un **POJO** inmutable:

```Java
public class Persona {
    private final String nombre;
    private final int edad;

    public Persona(String nombre, int edad) {
        this.nombre = nombre;
        this.edad = edad;
    }

    public String getNombre() {
        return this.nombre;
    }

    public int getEdad() {
        return this.edad;
    }

    @Override
    public boolean equals(Object o) {
        if (o == this) return true;
        if (!(o instanceof Persona)) return false;
        Persona p = (Persona) o;
        return edad == p.edad && Objects.equals(nombre, p.nombre);
    }

    @Override
    public int hashCode() {
        return Objects.hash(nombre, edad);
    }

    @Override
    public String toString() {
        return "Persona[nombre=" + nombre + ", edad=" + edad + "]";
    }
}
```
Al ser inmutable dejamos todos los campos como **finales** y no agregamos los **setters**.


### Record
Un record en Java se define de la siguiente manera:
```Java
public record Persona(String nombre, int edad) {}
```
Con esta simple declaración, Java genera automáticamente el constructor, getters, y los métodos `equals`, `hashCode` y `toString`. Esto reduce significativamente el código boilerplate y hace que el código sea más legible y mantenible (comparando 36 líneas contra 1).

## Constructores en Records: Canónico vs. Compacto
En los records, existen dos tipos de constructores: el **canónico** y el **compacto**.

- **Constructor Canónico**: Es el constructor que Java genera automáticamente y que toma un argumento por cada campo del record. Este constructor inicializa todos los campos del record.
```Java
public record Persona(String nombre, int edad) {
   public Persona(String nombre, int edad){ 
    //este caso es redundante al constructor de arriba, 
    //por lo que realmente no es necesario,
    //a menos que quieras hacer alguna validación
        this.edad=edad;
        this.nombre=nombre;
    }
}
```
En este caso, el constructor canónico sería public `Persona(String nombre, int edad)`.


- **Constructor Compacto**: Es un constructor que permite inicializar los campos del record sin tener que declarar explícitamente los parámetros. Es útil para agregar lógica de validación o transformación de datos.
```Java
public record Persona(String nombre, int edad) {
    public Persona {
        if (edad < 0) {
            throw new IllegalArgumentException("La edad no puede ser negativa");
        }
        if(nombre !=null){
            nombre= nombre.toUpperCase();
        }
    }
}
```
En este ejemplo, el constructor compacto valida que la edad no sea negativa y transforma el nombre para que siempre esté en mayúsculas.

## Limitaciones de los Records
A pesar de sus ventajas, los records tienen algunas limitaciones que los desarrolladores deben tener en cuenta:

1. **Inmutabilidad**: Los records son inmutables por defecto. Una vez que se crea una instancia de un record, sus campos no pueden ser modificados. Sin embargo, esto no significa que no puedas trabajar con datos mutables. Puedes crear nuevas instancias modificadas utilizando métodos como `withNombre` o `withEdad` que veremos más adelante.
2. **Herencia**: Los records no pueden extender otras clases. Sin embargo, pueden implementar interfaces.
3. **Flexibilidad**: Los records están diseñados para ser simples portadores de datos. Si necesitas lógica compleja o comportamiento adicional, un POJO tradicional puede ser más adecuado.
4. **Constructores y Campos**: En los records, los campos solo pueden ser definidos dentro del constructor. No puedes tener campos adicionales a menos que sean estáticos.
5. **Getters**: Los getters en los records no tienen el prefijo `get`. En su lugar, el nombre del método getter es el mismo que el nombre del campo.
```Java
public record Persona(String nombre, int edad) {}

// Uso de getters
Persona persona = new Persona("Juan", 30);
String nombre = persona.nombre(); // No persona.getNombre()
int edad = persona.edad(); // No persona.getEdad()
```

## Patrones de Copia: with y Builder
A pesar de la inmutabilidad, hay algunos trucos para mantener los valores de una instancia cambiando solo los necesarios. Esto se puede lograr utilizando el patrón `with` o un patrón `Builder`.

### Patrón `with`
El patrón `with` permite crear una nueva instancia de un record con uno o más campos modificados. Aunque Java no proporciona un método `with` por defecto, puedes implementarlo fácilmente:
```Java
public record Persona(String nombre, int edad) {
    public Persona withNombre(String nuevoNombre) {
        return new Persona(nuevoNombre, this.edad);
    }

    public Persona withEdad(int nuevaEdad) {
        return new Persona(this.nombre, nuevaEdad);
    }
}
```
Es recomendable utilizar el patrón `with` cuando se trata de records con pocos campos (menos de **3** o **4**), ya que es más sencillo y directo.

### Patrón `Builder`
El patrón `Builder` es otra forma popular de crear copias modificadas de objetos. Aunque es más comúnmente utilizado con POJOs, también se puede aplicar a records:
```Java
public class PersonaBuilder {
    private String nombre;
    private int edad;

    public PersonaBuilder(Persona persona) {
        this.nombre = persona.nombre();
        this.edad = persona.edad();
    }

    public PersonaBuilder nombre(String nombre) {
        this.nombre = nombre;
        return this;
    }

    public PersonaBuilder edad(int edad) {
        this.edad = edad;
        return this;
    }

    public Persona build() {
        return new Persona(nombre, edad);
    }
}
```
Es recomendable utilizar el patrón `Builder` cuando se trata de records con más de **3** o **4** campos, ya que proporciona una forma más estructurada y legible de modificar múltiples campos. Por ejemplo:

```Java
Persona personaOriginal = new Persona("Juan", 30);
PersonaBuilder builder = new PersonaBuilder(personaOriginal);
Persona personaModificada = builder.nombre("Carlos").edad(35).build();
```

En este ejemplo, `personaModificada` es una nueva instancia de `Persona` con los campos `nombre` y `edad` modificados.

## Ventajas de los Records para Value Objects
Una de las grandes ventajas de los records es su idoneidad para representar **Value Objects**. En el diseño orientado a objetos, un Value Object es un objeto que no tiene identidad propia y se define únicamente por sus atributos. Los records, con su inmutabilidad y generación automática de métodos `equals` y `hashCode`, son perfectos para este propósito. Esto asegura que dos instancias de un record con los mismos datos sean consideradas iguales, lo cual es esencial para la correcta implementación de Value Objects.

### Ejemplo de Value Object: Email
Consideremos un ejemplo de un Value Object para representar un email:

```Java
public record Email(String direccion) {
    public Email {
        if (!direccion.matches("^[A-Za-z0-9+_.-]+@(.+)$")) {
            throw new IllegalArgumentException("Dirección de email inválida");
        }
    }

    public String dominio() {
        return direccion.substring(direccion.indexOf("@") + 1);
    }
}
```

En este ejemplo, el record `Email` asegura que la dirección de email sea válida al momento de la creación, y cualquier instancia de `Email` con la misma dirección será considerada igual a otra. Además, hemos añadido un método `dominio` que devuelve el dominio del correo electrónico, demostrando que los records también pueden tener comportamiento basado en sus campos.

## Conclusión
Los records en Java representan un avance significativo en la simplificación de la creación de clases portadoras de datos. Aunque tienen algunas limitaciones, su capacidad para reducir el código boilerplate y mejorar la legibilidad del código los convierte en una herramienta valiosa para los desarrolladores. Al comprender las diferencias entre POJOs y records, y al utilizar patrones de copia como `with` y `Builder`, puedes aprovechar al máximo esta poderosa característica de Java. Además, su idoneidad para representar Value Objects los hace aún más útiles en el diseño de software moderno.