# Design Patterns

## Herhaling ontwerpen

![Associaties](images/associaties.png)

### Opgave:

**Mens** definieert wat je met een **mens** kan doen.  
**Ik** ben een **mens**.  
**Ik** heb een **laptop**.  
Als je mij iets vraagt dan gebruik ik een **boek**.  
**Jan** kan mij altijd vervangen (dubbelganger) maar als je iets vraagt is het antwoord nogal kinderlijk.


![Class Diagram](images/CD_HerhalingOntwerpen.png)

## Strategy Pattern
Het Strategy Patterndefinieert een familie algoritmen, isoleert ze en maakt ze uitwisselbaar. Strategy maakt het mogelijk om het algoritme los van de client die deze gebruikt, te veranderen.

### Waarom
Indien het gedrag varieert, dan halen we het eruit.

### Voorbeeld
Eenden: niet alle eenden vliegen op dezelfde manier. Sommige vliegen niet. Het vlieggedrag varieert, dus we halen het eruit.

### Stappen
1. Maak een interface, zet het gedrag erin
2. Maak de implementatieklassen. Zet het gedrag erin. Stippenlijn tekenen
3. De client klasse bevat de interface: pijl tekenen naar de interface
4. In de client klasse schrijven we
    1. De injectie: we kiezen hier bv voor een setter injectie
    2. Het gedrag

### UML
![Strategy pattern](images/UML_Strategy.png)

### Code
```java
//STAP 1
public interface FlyBehavior {
   public String fly();
}
```
```java
//STAP 2
public class FlyNoWay implements FlyBehavior{
   @Override
   public String fly() {
      return "Ik kan niet vliegen";
   }
}
```
```java
public class Duck {
   //STAP 3
   private FlyBehavior flyBehavior;
   
   //STAP4
   public void setFlyBehavior(FlyBehavior flyBehavior) {
      this.flyBehavior = flyBehavior;
   }
   
   public String performFly() {
      return flyBehavior.fly();
   }//einde stap 4
}  
```
### Oefening Strategy pattern
#### Opgave

- Het spel draait rond een held
- Een held kan wapens gebruiken om aan te vallen. De verschillende wapens zijn: mes, geweer en handen. hij kan ook vechten met zijn handen, vandaar dat het ook als wapen kan worden beschouwd.
- Indien hij noch mes of geweer heeft, dan vecht hij met zijn handen.
- Een held kan slechts één wapen tegelijk gebruiken.
- Een held kan tijdens het spel van wapen veranderen.
- Het resultaat van een aanval is afhankelijk van het gebruikte wapen.

Maak voor deze opgave een klassendiagram
Maak ook een voorbeeldimplementatie in Java

#### Ontwerp
![CD_Held](images/CD_Held.png)

#### Code

```java
public class Held {

	private Wapens wapen;
	
	public Held() {
		wapen = new Handen();
	}

	public void valAan() {
		wapen.valAan();
	}

	public void setWapen(Wapens wapen) {
		if (wapen!=null) {
			this.wapen = wapen;
		} else {
			this.wapen = new Handen();
		}
	}
}

```
```java
public interface Wapens {

	void valAan();
}
```
```java
//Dit is hetzelfde voor Mes en handen
public class Geweer implements Wapens {

	public void valAan() {
		System.out.println("Schiet!");
	}
}
```

## Simple factory pattern

### UML

![](images/UML_Factory.png)

### Code
```Java
public class PizzaStore {

    private PizzaFactory factory;

    public PizzaStore(PizzaFactory factory) {
        this.factory = factory;
    }

    public Pizza orderPizza(String type) {
        Pizza pizza;

        pizza = factory.createPizza(type);

        //if (pizza != null) {
            pizza.prepare();
            pizza.bake();
            pizza.cut();
            pizza.box();
        //}
        return pizza;
    }
}
```
```Java
public class PizzaFactory {

    public Pizza createPizza(String type) {

        return switch (type.toLowerCase()) {
            case "cheese" ->
                new CheesePizza();
            case "pepperoni" ->
                new PepperoniPizza();
            case "clam" ->
                new ClamPizza();
            case "veggie" ->
                new VeggiePizza();
            default ->
                //null;
            	new NoPizza();
        };
    }

}
```
```Java
public class CheesePizza extends Pizza {

    public CheesePizza() {
        super("Cheese Pizza", "Regular Crust", "Marinara Pizza Sauce",
                new ArrayList<>(Arrays.asList(new String[]{
            "Fresh Mozzarella",
            "Parmesan"})));
    }

}
```
```Java
public class NoPizza extends Pizza {

	public NoPizza() {
		super("No Pizza", "", "", new ArrayList<>());
	}

	@Override
	public void prepare() {}

	@Override
	public void bake() {}

	@Override
	public void cut() {}

	@Override
	public void box() {}

	@Override
	public String toString() {
		return "";
	}
}
```
### Oefening

1) Verbeter het ontwerp: bv.
```Java
public MallardDuck(){
   setQuackBehavior(new Quack());
   setFlyBehavoir(new FlyWithWings());
}
```
We mogen NIET naar een implementatie programmeren!

Programmeer naar een interface, niet naar een implemntatie.

+ testklasse verder aanvullen

```Java
public class DecoyDuck extends Duck {

    /*public DecoyDuck() {
        setQuackBehavior(new MuteQuack());
        setFlyBehavior(new FlyNoWay());
    }*/
	
	public DecoyDuck(QuackBehavior quack, FlyBehavior fly) { //MEEGEVEN --> voor factory
		super(quack, fly); //NORMAAL GEBRUIK JE DE CTOR VAN DE SUPER, NIET SETTER
	}
    
    @Override
    public String display() {
        return "Ik ben een lokeend";
    }

}
```
```Java
public abstract class Duck {

    private QuackBehavior quackBehavior;

    private FlyBehavior flyBehavior;
    
    public Duck(QuackBehavior quack, FlyBehavior fly) {
    	setFlyBehavior(fly);
    	setQuackBehavior(quack);
    }
    ...
}
```
```Java
public class DuckFactory {

	public Duck createDuck(DuckSpecies specie) {

		return switch (specie) {
			case REDHEAD -> new RedheadDuck(new Quack(), new FlyWithWings());
			case MALLARD -> new MallardDuck(new Quack(), new FlyWithWings());
			case RUBBER -> new RubberDuck(new Squeak(), new FlyNoWay());
			case DECOY -> new DecoyDuck(new MuteQuack(), new FlyNoWay());
		};
        
	}
}
```
```Java
public enum DuckSpecies {
	DECOY, MALLARD, REDHEAD, RUBBER
}
```
```Java
class DuckTest {
    ...
	
	private DuckFactory duckFactory;
	
	@BeforeEach
	public void before() {
		duckFactory = new DuckFactory();
	}
	
	@ParameterizedTest
	@MethodSource("duckProvider")
	public void testDuck(DuckSpecies kind, String expectedDisplay, String expectedQuack, String expectedFly) {
		Duck duck = duckFactory.createDuck(kind);
		Assertions.assertEquals(expectedDisplay, duck.display());
		Assertions.assertEquals(expectedQuack, duck.performQuack());
		Assertions.assertEquals(expectedFly, duck.performFly());
	}
	
	...
}
```

2) Voeg het vlieggedrag "FlyRocketPowered" toe

Toon aan dat je het gedrag dynamisch kan wijzigen (testklasse verder aanvullen).
```Java
        @ParameterizedTest
	@MethodSource("duckProvider")
	public void wijzigAtRuntime(DuckSpecies kind) {
		Duck duck = duckFactory.createDuck(kind);
		duck.setFlyBehavior(new FlyRocketPowered());
		Assertions.assertEquals(flyRocketPowered, duck.performFly());
	}
```

## State pattern

### Theorie

### Oefening

### Opdracht
Een document wordt enkel bewaard indien nodig: wanneer wijzigingen werden aangebracht.
Indien er wijzigingen werden aangebracht en het document werd nog niet bewaard -> toestand ‘Dirty’
Indien het document werd bewaard en er werden geen wijzigingen meer aangebracht -> toestand ‘Clean’

#### Code

## Design patterns door elkaar

### Oefening 1

![](images/DP_oef1.png)

#### Oplossing

![](images/DP_oef1_opl.png)

```java
public class ScrollBar extends VisualDecorator {

	public void draw() {
		System.out.println("scrollbar");
		visualComponent.draw();
	}

	public ScrollBar(VisualComponent visualComponent) {
		super(visualComponent);
	}
}
```

```java
public class Border extends VisualDecorator {

	private int width;

	public void draw() {
		System.out.printf("kader van %d dik%n", width);
		visualComponent.draw();
	}

	public Border(VisualComponent visualComponent, int width) {
		super(visualComponent);
		this.width=width;
	}
}
```

```java
public interface VisualComponent {

	void draw();
}
```

```java
public abstract class VisualDecorator implements VisualComponent {

	protected VisualComponent visualComponent;

	public VisualDecorator(VisualComponent visualComponent) {
		this.visualComponent = visualComponent;
	}
}
```

```java
public class TextView implements VisualComponent {

	public void draw() {
		System.out.println("textview");
	}
}
```

```java
public class main {

	public static void main(String[] args) {
		VisualComponent textView = new TextView();
		textView.draw();
		VisualComponent textView2 = new ScrollBar(new Border(new TextView(), 10));
		textView2.draw();
	}

}
```

### Oefening 2

![](images/DP_oef2.png)

#### Oplossing
![](images/DP_oef2_opl.png)

### Oefening 6

![PDF oefenining](extras/oef6_opgave.pdf)

#### Oplossing

![](images/DP_oef6_opl.png)
![](images/DP_oef6_opl2.png)

### Oefening 7

![](images/DP_oef7.png)

#### Oplossing
![](images/DP_oef7_opl.png)