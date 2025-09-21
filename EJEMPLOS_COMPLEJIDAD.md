# Ejemplos Específicos de Complejidad en el Código

## 1. StatsController - El Problema Principal

### Lo que hace esta clase (demasiadas cosas):
```java
public class StatsController {
    // 1. Maneja estadísticas del Tamagotchi
    public void increasedBoredom() { ... }
    public void increasedHungry() { ... }
    
    // 2. Implementa juegos completos
    public boolean adivinanzasJuego() { ... }  // 40+ líneas
    public void ahorcadoJuego() { ... }        // 50+ líneas
    
    // 3. Maneja persistencia de datos
    public void saveTamagotchi() { ... }
    public void loadTamagotchi() { ... }
    
    // 4. Imprime información (debería estar en Vista)
    public void printTamagotchiData() { ... }
}
```

**Problema**: Una clase con 356 líneas haciendo 4 trabajos diferentes.

## 2. Threading Complejo

### En CreateTamagotView.java:
```java
public void transition(){
    // ... código inicial ...
    
    // Inicia 3 threads simultáneamente
    Thread decrementThread = new Thread(() -> sc.decrementLifeTime());
    decrementThread.start();
    
    Thread incrementThread = new Thread(() -> sc.increasedBoredom());
    incrementThread.start();
    
    Thread incrementHungry = new Thread(() -> sc.increasedHungry());
    incrementHungry.start();
}
```

**Problemas**:
- No hay sincronización entre threads
- Difícil de debuggear
- Posibles condiciones de carrera

## 3. Lógica de Juego Embebida

### Juego de Adivinanzas (StatsController):
```java
public boolean adivinanzasJuego() throws InputMismatchException{
    int intentos = 5;
    boolean resultado = false;
    int num = (int)(Math.random()*20+1);
    
    for(int indi = 0; indi < intentos; indi++){
        numsUsuario = in.nextInt(); 
        if(numsUsuario < num){
            System.out.println("cerca, pero ingresaste un numero menor al mio");
        } else if(numsUsuario > num){
            System.out.println("cerca, pero el numero ingresado es mayor al mio");
        } else if(numsUsuario == num){
            // ... actualiza múltiples stats del Tamagotchi ...
            resultado = true;
            break;
        } 
    }
    return resultado;
}
```

**Problemas**:
- Lógica de juego mezclada con lógica de negocio
- No debería estar en un controlador de stats

## 4. Manejo de Archivos Múltiple

### En EggController.java:
```java
// Guarda en formato serializado
public void saveEgg(){
    try{
        FileOutputStream fileOut = new FileOutputStream("Friends.txt");
        ObjectOutputStream objectOut = new ObjectOutputStream(fileOut);
        objectOut.writeObject(firstEgg);
    } catch (FileNotFoundException e){
        // manejo de error
    }
}

// También guarda en texto plano
public void saveEgg1() {
    try (PrintWriter writer = new PrintWriter(new FileOutputStream("PlainsTextFriends.txt"))) {
        for (Egg gee : firstEgg) {
            writer.println(gee.getName() + "," + gee.getGender());
        }
    }
}
```

**Problema**: Duplicación de lógica de persistencia.

## 5. Mezcla de Idiomas

### Ejemplo en OwnerMenu.java:
```java
public void askQuestions() {
    Scanner scanner = new Scanner(System.in);  // Variable en inglés
    
    System.out.println("¡Hola! Soy tu Tamagotchi. Vamos a conocernos mejor."); // Español
    System.out.print("Cuál es tu comida favorita? ");  // Español
    String food = in.nextLine();  // Variable en inglés
    
    t.setIntelligence(t.getIntelligence() + 45);  // Método en inglés
}
```

**Problema**: Inconsistencia que dificulta el mantenimiento.

## 6. Código Bien Implementado (Ejemplos Positivos)

### Patrón Singleton Correcto:
```java
public class StatsController {
    private static StatsController instance;
    
    public static synchronized StatsController getIn() {
        if (instance == null) {
            instance = new StatsController();
        }
        return instance;
    }
}
```

### Herencia Bien Usada:
```java
public class Tamagotchi extends Egg implements Serializable {
    private int life, happiness, boredom, intelligence, hungry, sleepiness;
    // ... métodos específicos del Tamagotchi
}
```

### Modelo Limpio:
```java
public class Owner {
    private String name, gender;
    private int age;
    
    // Constructor, getters y setters limpios
}
```

## Resumen de Complejidad por Componente

| Componente | Complejidad | Razón |
|------------|-------------|-------|
| StatsController | 🔴 Alta | Demasiadas responsabilidades, threading, juegos |
| OwnerMenu | 🟡 Media | Múltiples opciones, navegación compleja |
| CreateTamagotView | 🟡 Media | Threading, flujo de creación |
| Modelos (Tamagotchi, Egg, Owner) | 🟢 Baja | Simples POJOs bien diseñados |
| EggController | 🟡 Media | Doble persistencia, serialización |
| MenuMain | 🟢 Baja | Navegación simple |

## Recomendaciones Específicas de Refactoring

1. **Dividir StatsController**:
   ```java
   // En lugar de una clase grande:
   class StatsController // 356 líneas
   
   // Crear varias clases pequeñas:
   class TamagotchiStatsService  // Solo stats
   class GameService            // Solo juegos  
   class PersistenceService     // Solo archivos
   class StatsDisplayService    // Solo mostrar datos
   ```

2. **Mejorar Threading**:
   ```java
   // En lugar de:
   Thread thread = new Thread(() -> method());
   thread.start();
   
   // Usar:
   ExecutorService executor = Executors.newFixedThreadPool(3);
   executor.submit(() -> method());
   ```

3. **Extraer Constantes**:
   ```java
   // En lugar de:
   Thread.sleep(8000);
   while(t.getBoredom() <= 99)
   
   // Usar:
   private static final int SLEEP_TIME = 8000;
   private static final int MAX_STAT_VALUE = 99;
   ```

Este análisis muestra que el código tiene una base sólida pero necesita refactoring en áreas específicas para mejorar su mantenibilidad.