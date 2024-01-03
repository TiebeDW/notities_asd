# JAVA 

# Collections & streams

## Reductiebonnen

![Class Diagram Reductiebonnen](images/CD_Reductie.png)

### Reductiebonbeheerder
Gemakzuchtige statische fabrieksmethoden op de List, Set en Map interfaces laten je eenvoudig niet wijzigbare lijsten, sets en maps maken. Een verzameling wordt als niet wijzigbaar beschouwd als elementen niet kunnen worden toegevoegd, verwijderd of vervangen.  
Omzetten naar een niet-wijzigbare lijst
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
        .filter(bon -> bon.getPercentage() > percentage)
        .map(Reductiebon::getReductiebonCode)
        .collect(Collectors.toList());
}
````

#### Vraag 2:
Methode sorteerReductiebonnen: sorteer de lijst met reductiebonnen volgens oplopende percentage (van laag naar hoog), en bij gelijke percentage op reductiebonCode – alfabetisch omgekeerde volgorde. (De originele lijst van reductiebonnen is gewijzigd.)
```java
public void sorteerReductiebonnen() {
		reductiebonLijst.sort(Comparator.comparing(Reductiebon::getPercentage)
				.thenComparing(Comparator.comparing(Reductiebon::getReductiebonCode)
				.reversed()));
}
```
Als er gevraagd werd, maak een nieuwe gesorteerde lijst dan moet er .stream().sorted() staan.

#### Vraag 3:
Methode geefGemPercVanBonnenInToekomst: geef het gemiddelde percentage terug van alle reductiebonnen die in de toekomst liggen (ter info: huidige datum: LocalDate.now() ).
```java
public double geefGemPercVanBonnenInToekomst() {
	return reductiebonLijst.stream() //Stream van reductiebonnen
				.filter(bon -> bon.getEinddatum().isAfter(LocalDate.now()))
				.mapToDouble(Reductiebon::getPercentage)
				.average().getAsDouble();
}
```

#### Vraag 4:
Methode geefUniekeEinddatums: geef alle unieke einddatums terug (m.a.w. geen dubbels), gesorteerd in stijgende volgorde.
```java
public List<LocalDate> geefUniekeEinddatums() {
	return reductiebonLijst.stream()
			.map(Reductiebon::getEinddatum)
			.sorted().distinct()
			.collect(Collectors.toList());
}
```

### Sporterbeheerder
```java

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
```java
public List<Reductiebon> geefAlleReductiebonnenMetKortingsPercentageX(List<Integer> kortingspercentage) {
	return sportersLijst.stream()
			.map(Sporter::getReductiebonLijst)
			.flatMap(Collection::stream)
			.filter(bon -> kortingspercentage.contains(bon.getPercentage()))
			.collect(Collectors.toList());
}
```

#### Extra vraag 2
Methode verwijderAlleSportersMetReductiebonMetPercX : zal alle sporters uit de originele lijst verwijderen die als korting het meegegeven percentage hebben.
Opgelet: Testmethode uncomment
```java
public void verwijderAlleSportersMetReductiebonMetPercX(int perc) {
	sportersLijst.removeIf(sporter -> sporter.getReductiebonLijst().stream()
			.anyMatch(bon -> bon.getPercentage()== perc));	
}
```

## MAP
![Overzicht Collectionframework](images/overzichtCollectionFramework.png)
HashMap<K,V> en Hashtable<K,V> zijn implentatie-klassen van Map.  
TreeMap<K,V> is een implementatie-klasse van SortedMap.  
De klasse properties is een subklasse van HashTable<K,V>  

**Implementaties van interface Map<K,V>:**
- HashMap<K,V>
  - Elementen opgeslagen in een hash-tabel
- Hashtable<K,V>
  - Zoals HashMap maar verouderde versie.
  - Werpt een NullPointerException indien de sleutel of value null is. 
- TreeMap<K,V>
  - Gesorteerd
  - Elementen opgeslagen in boomstructuur
  - Implementatie van SortedMap subinterface van Map. Gebruik de natuurlijke volgorde of een comparator. 

![HashMapVoorbeeld](images/HashMapVoorbeeld.png)
![VbHashMapWerknemers](images/vbHashMapWerknemers.png)

### Java_Map_Oefeningen_start
```Java
class Auteur {

    private String naam, voornaam;

    public Auteur(String naam, String voornaam) {
        setNaam(naam);
        setVoornaam(voornaam);
    }

    public String getNaam() {
        return naam;
    }

    public String getVoornaam() {
        return voornaam;
    }

    public void setNaam(String naam) {
        this.naam = naam;
    }

    public void setVoornaam(String voornaam) {
        this.voornaam = voornaam;
    }

    @Override
    public String toString() {
        return String.format("%s %s", naam, voornaam);
    }
}

public class OefMap_opgave {
    public OefMap_opgave() {
        // we zullen een hashmap gebruiken waarbij auteursid de sleutel is en
        // de waarde is naam en voornaam van Auteur.
        //Creëer de lege hashMap "auteursMap"; de sleutel is van type Integer, de waarde van type Auteur
        //----------------------------------------------------------------------------------
        Map<Integer, Auteur> auteursMap = new HashMap<>();
        
        //Voeg toe aan de hashmap: auteursID = 9876, naam = Gosling, voornaam = James
        //Voeg toe aan de hashmap: auteursID = 5648, naam = Chapman, voornaam = Steve
        //-------------------------------------------------------------------------------
        auteursMap.put(9876, new Auteur("Gosling", "James"));
        auteursMap.put(5648, new Auteur("Chapman", "Steve"));
        
        //Wijzig de voornaam van Chapman: John ipv Steve
        //----------------------------------------------
        auteursMap.get(5648).setVoornaam("John");
        
        //Komt de auteursID 1234 voor in de hashmap
        //-----------------------------------------
        if (auteursMap.containsKey(1234))
		System.out.println("auteursID 1234 komt voor\n");
        else
		System.out.println("auteursID 1234 komt niet voor\n");
         
        //Toon de naam en voornaam van auteursID 5648
        //-------------------------------------------
        
		Auteur auteur = auteursMap.get(5648);
		if (auteur != null)
			System.out.println(auteur);
         
        toonAlleAuteurs(auteursMap);

        //Alle auteursID's worden in stijgende volgorde weergegeven.
        //  1) de hashMap kopiëren naar een treeMap (= 1 instructie)
        //  2) roep de methode toonAlleSleutels op.
        //---------------------------------------------------------------
        Map<Integer, Auteur> treeMap = new TreeMap<>();
        toonAlleAuteurs(auteursMap);
        
    }

    public void toonAlleSleutels(Map<Integer, Auteur> map) {
        //Alle sleutels van de map worden op het scherm weergegeven.
        //---------------------------------------------------------------
    	map.keySet().forEach(System.out::println);
        System.out.println();
    }

    public void toonAlleAuteurs(Map<Integer, Auteur> map) {
        /*Alle gegevens van de map worden op het scherm weergegeven.
		Per lijn wordt een auteursnr, naam en voornaam weergegeven.*/
        //---------------------------------------------------------------
    	//NIET ZO -> OVERKILL
    	//map.entrySet().stream().forEach(entry -> System.out.printf("%d %s%n", entry.getKey(), entry.getValue()));
        map.forEach((auteursId, auteur) -> System.out.printf("%d %s%n",auteursId, auteur));
    	System.out.println();
    }

    public static void main(String args[]) {
        new OefMap_opgave();
    }
}
```
```Java
class CollectionOperaties {
    
    //methode verwijderOpLetter
    //-------------------------
	public static boolean verwijderOpLetter(List<String> list, char c) {
		return list.removeIf(elem -> elem.charAt(0)==c);
	}

    //methode verwijderSequence
    //-------------------------
	public static boolean verwijderSequence(List<String> list, String grens) {
		int first = list.indexOf(grens);
		if (first==-1) {
			return false;
		}
		int last = list.lastIndexOf(grens);
		list.subList(first, last).clear();
		return true;
		
	}

	//uitbreiding opgave Fruit   addOrdered
	//-------------------------------------
	public static boolean addOrdered(List<String> list, String fruit) {
		int index = Collections.binarySearch(list, fruit);
		if(index>=0) {
			return false;
		}
		list.add(index*-1,fruit);
		return true;
	}
}

public class OefFruit_opgave {

    public static void main(String args[]) {
        String kist[][] = {{"appel", "peer", "citroen", "kiwi", "perzik"},
        {"banaan", "mango", "citroen", "kiwi", "zespri", "pruim"},
        {"peche", "lichi", "kriek", "kers", "papaya"}};

        List<String> list = Stream.of(kist).flatMap(Arrays::stream).collect(Collectors.toList());
        String mand[];

        //Toon de inhoud van de array "kist"
        //----------------------------------
        System.out.println(Arrays.deepToString(kist));
        //Arrays.toString werkt voor 1 dim array
        
        
        //Voeg de verschillende kisten samen in een ArrayList list.
        //--------------------------------------------------------


        CollectionOperaties.verwijderOpLetter(list, 'p');
        System.out.println("na verwijder letter ('p') :  " + list + "\n");

        CollectionOperaties.verwijderSequence(list, "kiwi");
        System.out.println("na verwijder sequence (kiwi) : " + list + "\n");
        
        //UITBREIDING
        list.sort(null);
        CollectionOperaties.addOrdered(list, "sapodilla");

        //Plaats het resultaat terug in een array mand en sorteer die oplopend.
        //---------------------------------------------------------------------
        mand = list.toArray(new String[0]);
        Arrays.sort(mand);

        //Toon de inhoud van de array "mand"
        //----------------------------------
        System.out.println(Arrays.toString(mand));
    }
}
```
```Java
public class OefFruitMap_opgave {

    public static void main(String args[]) {
        String kist[][] = {{"appel", "peer", "citroen", "kiwi", "perzik"},
        {"banaan", "mango", "citroen", "kiwi", "zespri", "pruim"},
        {"peche", "lichi", "kriek", "kers", "papaya"}};

        List<String> list = Stream.of(kist).flatMap(Arrays::stream).collect(Collectors.toList());
        Scanner in = new Scanner(System.in);

        //declaratie + creatie map
        //------------------------------
        Map<String, Double> fruitMap = new TreeMap<>();
                            
        /*Berg de fruit list van vorige oefeningen in een boom
        op zodat dubbels ge�limineerd worden.
        Er moet ook de mogelijkheid zijn de bijhorende prijs
        (decimale waarde) bij te houden.*/
        //------------------------------------------------------------
        list.forEach(fruit -> fruitMap.put(fruit, null));
        
        /*Doorloop de boom in lexicaal oplopende volgorde en vraag
        telkens de bijhorende prijs, die je mee in de boom opbergt.*/
        //------------------------------------------------------------
        //fruitMap.forEach(...);
        fruitMap.entrySet().stream().forEach(entry -> {
        	System.out.printf("Prijs van %s : ", entry.getKey());
        	double prijs = in.nextDouble();
        	entry.setValue(prijs);
        });
        
        
        /*Druk vervolgens de volledige lijst in twee
        kolommen (naam : prijs) in lexicaal oplopende volgorde af
        op het scherm.*/
        //------------------------------------------------------------
        fruitMap.forEach((fruit, prijs) -> System.out.printf("%s\t%.2f%n", fruit, prijs));       
    }
}
```

### Java_Oefeningen2_start
#### Opgave
![Map oefeningen 2](images/Map_Oef2.png)
![Map oefeningen 2 Opgave](images/Map_Oef2Opgave.png)
#### Code
```java 
public class BierWinkel {

    private final List<Bier> bieren;
    private PersistentieController pc = new PersistentieController();

    public BierWinkel() {
        bieren = pc.inlezenBieren("bieren.txt");
    }
    
    //OEF
    public Map<String, Bier> opzettenOverzichtBierPerNaam() {
    	return bieren.stream().collect(Collectors.toMap(Bier::getNaam, Function.identity()));
    }
    
    //OEF
    public Map<String, List<Bier>> opzettenOverzichtBierenPerSoort() {
    	//return bieren.stream().collect(Collectors.groupingBy(Bier::getSoort));
    	return bieren.stream().collect(Collectors.groupingBy(Bier::getSoort, TreeMap::new, Collectors.toList()));
    }

    //OEF
    public Map<String, Long> opzettenAantalBierenPerSoort() {
    	return bieren.stream().collect(Collectors.groupingBy(Bier::getSoort, TreeMap::new, Collectors.counting()));
    }
}
```