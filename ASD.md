# ASDI

## Herhaling ontwerpen

![Associaties](images/associaties.png)

#### Opgave: 

**Mens** definieert wat je met een **mens** kan doen.  
**Ik** ben een **mens**.  
**Ik** heb een **laptop**.  
Als je mij iets vraagt dan gebruik ik een **boek**.  
**Jan** kan mij altijd vervangen (dubbelganger) maar als je iets vraagt is het antwoord nogal kinderlijk.


![Class Diagram](images/CD_HerhalingOntwerpen.png)

## TDD
protected = is private + subklasses toegang geven, package access

### Oefening 1
![](images/CD_Herhaling_Oef1.png)

#### Opgave:

Formaat: 3cijfers-7cijfers-2cijfers  
De rest van de deling eerste tien cijfers door 97 moet gelijk zijn aan de twee laatste cijfers.

#### Code:
Domeinklasse:
```java
package domein;

public class Rekening {
	private String rekeningnummer;

	public Rekening(String rekeningnummer) {
		setRekeningnummer(rekeningnummer);
	}

	private void setRekeningnummer(String rekeningnummer) {
		if (rekeningnummer == null || rekeningnummer.isBlank()) {
			throw new IllegalArgumentException("rekeningnummer niet ingevuld");
		}

		if (!rekeningnummer.matches("\\d{3}-\\d{7}-\\d{2}")) {
			throw new IllegalArgumentException("rekeningnummer in verkeerd formaat");
		}
		
		String[] token = rekeningnummer.split("-");
		long eerste10Cijfers = Long.parseLong(token[0] + token[1]);
		long laatste2Cijfers = Long.parseLong(token[2]);

		if (eerste10Cijfers % 97L != laatste2Cijfers)
			throw new IllegalArgumentException("verkeerde cijfers in rekeningnummer");

		this.rekeningnummer = rekeningnummer;	
	}
}
```
Testklasse:
```java
package testen;

import org.junit.jupiter.api.Assertions;
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.NullAndEmptySource;
import org.junit.jupiter.params.provider.ValueSource;

import domein.Rekening;

class RekeningTest {
	
	@ParameterizedTest
	@ValueSource(strings = {"063-1547563-60", "999-9999999-48", "001-0000001-77"})

	public void geldRekeningnummer(String rekeningnummer) {
		Assertions.assertDoesNotThrow(()->{
			new Rekening(rekeningnummer);
		});
	}
	
	@ParameterizedTest
	@NullAndEmptySource
	@ValueSource(strings = {"063-1547563-61", " ", "063-1547563"})
	
	public void ongeldigeRekeningnummer(String rekeningnummer) {
		Assertions.assertThrows(IllegalArgumentException.class, ()->{
			new Rekening(rekeningnummer);
		});
	}
	
	
	/*@Test
	public void nummerOk() {
		Assertions.assertDoesNotThrow(()->{
			new Rekening("063-1547563-60");
		});
	}
	
	@Test
	public void grootNummerOk() {
		Assertions.assertDoesNotThrow(()-> {
			new Rekening("999-9999999-48");
		});
	}*/
}
```

### Oefening Bowling

#### Opgave

![](images/Bowling.png)  
The game consists of 10 frames as shown above.  In each frame the player hastwo opportunities to knock down 10 pins.  The score for the frame is the totalnumber of pins knocked down, plus bonuses for strikes and spares.  
A spare is when the player knocks down all 10 pins in two tries. The bonus forthat frame is the number of pins knocked down by the next roll.  Soin frame 3above, the score is 10 (the total number knocked down) plus a bonus of 5 (thenumber of pins knocked down on the next roll.)  
A strike is when the player knocks down all 10 pins on his first try.  The bonusfor that frame is the value of the next two balls rolled.  
In the tenth frame a player who rolls a spare or strike is allowed to roll the extraballs to complete the frame.  However no more than three balls can be rolled intenth frame.  

Write a class named "BowlingGame" that has two methods:  
- **roll(pins : int)** is called each time the player rolls a ball.  The argument is the number of pins knocked down.
- **score() : int** is called only at the very end of the game.  It returns the total score for that game.

Create a class **BowlingGame** and test **BowlingGameTest**

#### Ontwerp:
![Class Diagram Bowling](images/CD_OefeningBowling.png)

#### Code
Domeinklasse:
```java
package domain;
public class BowlingGame {
	
	private int rolls[] = new int[21];
	private int currentRoll = 0;

	public void roll(int pins) {
		rolls[currentRoll++] = pins;
		
	}

	public int score() {
		int score = 0;
		for(int frameIndex = 0, frame = 0;frame<10; frameIndex+=2, frame++) 
			if(isStrike(frameIndex)) {
				score += 10 + strikeBonus(frameIndex);
				frameIndex--;
			}
			else if(isSpare(frameIndex))
				score += 10 + spareBonus(frameIndex);
			else
				score += sumOfPinsInFrame(frameIndex);
		return score;
	}
	
	private int sumOfPinsInFrame(int frameIndex) {
		return  rolls[frameIndex] + rolls[frameIndex+1];
	}
	
	private int strikeBonus(int frameIndex) {
		return rolls[frameIndex+1] + rolls[frameIndex+2];
	}
	
	private int spareBonus(int frameIndex) {
		return rolls[frameIndex+2];
	}
	
	private boolean isStrike(int frameIndex) {
		return rolls[frameIndex]==10;
	}

	private boolean isSpare(int frameIndex) {
		return rolls[frameIndex] + rolls[frameIndex + 1] == 10;
	}
}
```
Testklasse:
```java
package tests;

import org.junit.jupiter.api.Assertions;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.CsvSource;

import domain.BowlingGame;

class BowlingGameTest {
	
	private BowlingGame game;
	
	@BeforeEach
	public void before() {
		game = new BowlingGame();
	}
	
	private void rollMany(int n, int pins) {
		for(int i=0;i<n;i++) game.roll(pins);
	}
	
	private void rollSpare() {
		game.roll(5);
		game.roll(5);
	}
	
	private void rollStrike() {
		game.roll(10);
	}
	
	@ParameterizedTest
	@CsvSource({"0,0","1,20"})
	public void testSameNumberOfPins(int number, int expexted) {
		rollMany(20, number);
		Assertions.assertEquals(expexted, game.score());
	}
	
	@Test
	public void testOneSpare() {
		rollSpare();
		game.roll(3);
		rollMany(17, 0);
		Assertions.assertEquals(16, game.score());
	}
	
	@Test
	public void testTwoSpare() {
		rollSpare();
		rollSpare();
		game.roll(3);
		rollMany(15, 0);
		Assertions.assertEquals(31, game.score());
	}

	@Test
	public void testOneStrike() {
		rollStrike();
		game.roll(3);
		game.roll(4);
		rollMany(16, 0);
		Assertions.assertEquals(24, game.score());
	}
	
	@Test
	public void testAllStrikes() {
		for(int i = 0; i<12; i++) rollStrike();
		Assertions.assertEquals(300, game.score());
	}
	
	@Test 
	void testAllSpares_5_5() {
		for(int i = 0; i<10; i++) rollSpare();
		game.roll(5);
		Assertions.assertEquals(150, game.score());
	}
	
	@Test
	public void testScenario() {
		int[] pins = {1,4,4,5,6,4,5,5,10,0,1,7,3,6,4,10,2,8,6};
		for(int i=0; i<pins.length;i++) game.roll(pins[i]);
		Assertions.assertEquals(133, game.score());
	}
	
}
```

## Collections

### Reductiebonnen

![Class Diagram Reductiebonnen](images/CD_Reductie.png)

Gemakzuchtige statische fabrieksmethoden op de List, Set en Map interfaces laten je eenvoudig niet wijzigbare lijsten, sets en maps maken. Een verzameling wordt als niet wijzigbaar beschouwd als elementen niet kunnen worden toegevoegd, verwijderd of vervangen.
```java
public List<Reductiebon> getReductiebonLijst() {
	//return reductiebonLijst;
	return Collections.unmodifiableList(reductiebonLijst);
}
```

#### Vraag 1:  
Methode geefReductiebonCodes: een lijst van reductiebonCodes wordt teruggegeven waarvan de percentage hoger ligt dan het meegegeven percentage.
````java
public List<String> geefReductiebonCodes(int percentage) {
    return reductiebonLijst.stream()
        .filter(r -> r.getPercentage()>percentage)
        .map(Reductiebon::getReductiebonCode)
        .collect(Collectors.toList());
}
````

#### Vraag 2:  
Methode sorteerReductiebonnen: sorteer de lijst met reductiebonnen volgens oplopende percentage (van laag naar hoog), en bij gelijke percentage op reductiebonCode – alfabetisch omgekeerde volgorde. (De originele lijst van reductiebonnen is gewijzigd.)
````java
public void sorteerReductiebonnen() {
		reductiebonLijst.sort(Comparator.comparing(Reductiebon::getPercentage)
				.thenComparing(Comparator.comparing(Reductiebon::getReductiebonCode)
				.reversed()));
}
````

#### Vraag 3:
Methode geefGemPercVanBonnenInToekomst: geef het gemiddelde percentage terug van alle reductiebonnen die in de toekomst liggen (ter info: huidige datum: LocalDate.now() ).
````java
// VRAAG3
public double geefGemPercVanBonnenInToekomst() {
	return reductiebonLijst.stream() //Stream van reductiebonnen
			.filter(bon -> bon.getEinddatum().isAfter(LocalDate.now()))
			.mapToInt(Reductiebon::getPercentage) // IntStream
			.average().getAsDouble();
}
````

#### Vraag 4:
Methode geefUniekeEinddatums: geef alle unieke einddatums terug (m.a.w. geen dubbels), gesorteerd in stijgende volgorde.
```java
// VRAAG4
public List<LocalDate> geefUniekeEinddatums() {
	return reductiebonLijst.stream()
			.map(Reductiebon::getEinddatum)
			.sorted()
			.distinct()
			.collect(Collectors.toList());
}
```

#### Vraag 5:
Bekijk deze klasse en pas aan. Er is nog een encapsulatie lek.
```java
public Collection<Sporter> getSportersLijst() {
	return Collections.unmodifiableCollection(sportersLijst);
}
```

#### Vraag 6:
Methode geefEenSporterMetGegevenReductiebon: geeft een willekeurige sporter terug die de gegeven reductiebon bevat. Indien geen sporter aanwezig, dan wordt null teruggegeven.
```java
public Sporter geefEenSporterMetGegevenReductiebon(Reductiebon bon) {
	return sportersLijst.stream()
			.filter(sporter -> sporter.getReductiebonLijst().contains(bon)) //Stream<Sporters>
			.findAny() //Optional<Sporter>
			.orElse(null); 
}
```

#### Extra vraag 1
Methode geefAlleReductiebonnenMetKortingsPercentageX: geeft een lijst van alle Reductiebonnen met die één van de meegegeven kortingspercentages hebben.
Opgelet: Testmethode uncomment + aanvullen resultaatSet (ophalen uit bonnen)
```java
public List<Reductiebon> geefAlleReductiebonnenMetKortingsPercentageX(List<Integer> kortingspercentage) {
	return sportersLijst.stream()
			.map(Sporter::getReductiebonLijst)
			.flatMap(Collection::stream)
			.filter(bon -> kortingspercentage.contains(bon.getPercentage()))
			.collect(Collectors.toList());
}
```
```java
@Test
public void testVraagExtra1_reductiebonnenMetKorting() {
	// TODO	uncomment
	List<Reductiebon> bonnen = sporterBeheerder.geefAlleReductiebonnenMetKortingsPercentageX(List.of(10, 40));

	Set<String> resultaat = //TODO verzamel dereductieboncodes in een Set
		bonnen.stream().map(Reductiebon::getReductiebonCode)
		.collect(Collectors.toSet());
		
	Set<String> verwachteResultaat = new HashSet<>(Arrays.asList("R31", "R34", "R36", "R322"));
	//Assertions.assertEquals(verwachteResultaat, resultaat);
}
```

#### Extra vraag 2
Methode verwijderAlleSportersMetReductiebonMetPercX : zal alle sporters uit de originele lijst verwijderen die als korting het meegegeven percentage hebben.
Opgelet: Testmethode uncomment
```java
public void verwijderAlleSportersMetReductiebonMetPercX(int perc) {
	sportersLijst.removeIf(s -> s.getReductiebonLijst().stream()
			.anyMatch(bon -> bon.getPercentage()== perc));	
}
```

## Map


## Design Patterns

### Strategy Pattern
Het Strategy Patterndefinieert een familie algoritmen, isoleert ze en maakt ze uitwisselbaar. Strategy maakt het mogelijk om het algoritme los van de client die deze gebruikt, te veranderen.

#### Waarom
Indien het gedrag varieert, dan halen we het eruit. 

#### Voorbeeld
Eenden: niet alle eenden vliegen op dezelfde manier. Sommige vliegen niet. Het vlieggedrag varieert, dus we halen het eruit.

#### Stappen
1. Maak een interface, zet het gedrag erin
2. Maak de implementatieklassen. Zet het gedrag erin. Stippenlijn tekenen
3. De client klasse bevat de interface: pijl tekenen naar de interface
4. In de client klasse schrijven we
   1. De injectie: we kiezen hier bv voor een setter injectie
   2. Het gedrag

#### UML
![Strategy pattern](images/UML_Strategy.png)

#### Code
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