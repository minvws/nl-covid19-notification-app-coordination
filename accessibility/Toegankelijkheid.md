## Toegankelijkheid

Een van de uitgangspunten bij het ontwikkelen van deze app is digitale toegankelijkheid. IN dit document wordt dit uitgangspunt uitgewerkt tot concrete richtlijnen en handvatten voor zowel de ontwerpers, ontwikkelaars en testers. Het document wordt aangevuld op basis van vragen en feedback vanuit het team en de community.

Net als bijvoorbeeld privacy en beveiliging is toegankelijkheid iets dat gedurende het hele traject aandacht verdiend. Iets achteraf toegankelijk proberen te maken is veel werk en levert vaak geen optimaal resultaat. Zeker gezien de korte doorlooptijd van dit proces is het van belang om toegankelijkheid in één keer zo goed mogelijk te doen.

In de requirements staat dat we ervoor zorgen dat de app voldoet aan de [Web Accessibility Guidelines (WCAG)](https://www.w3.org/TR/WCAG21/) op niveau AA. Deze richtlijnen zijn voornamelijk voor webpagina's opgesteld, maar een groot deel is ook toepasbaar op apps. Doordat de richtlijnen nog niet veel voor apps worden toegepast, is het niet altijd duidelijk hoe een succescriterium in deze context te interpreteren. Het voldoen aan WCAG geeft een basisniveau van toegankelijkheid. Daarnaast probere nwe er met expertise in het team en feedback vanuit mensen met een functiebeperking een zo bruikbaar mogelijke app voor iedereen van te maken. Eigenlijk is WCAG-compliance dus niet het einddoel, maar het startpunt. Het einddoel is een prettige gebruikservaring voor iedereen.

### Procesmatige uitgangspunten

* We bouwen iets dat voldoet aan WCAG 2.1 niveau AA en als dit niet gaat
  * Documenteren we waarom
  * Beschrijven we de impact voor gebruikers
  * Geven we een duidelijk tijdspad of en wanneer we dit alsnog toegankelijk maken
  * Komen we, waar mogelijk, met een tussenoplossing of workaround
* Als iets al WCAG 2.1 compliant is, maar uit gebruikersfeedback blijkt dat er voor bepaalde doelgroepen een niet-optimale gebruikservaring is
  * Documenteren we dit en maken we een inschatting van de impact op gebruikers
  * Lossen we het waar mogelijk structureel op of besluiten we dit niet te doen op basis van de impact. Ook dit besluit leggen we vast
* Bij release bundelen we de gemaakte keuzes in een toegankelijkheidsverklaring. Deze verklaring stellen we op volgens het voorlopige model voor toegankelijkheidsverklaringen via <www.toegankelijkheidsverklaring.nl>
  * De verklaring moet in ieder geval vindbaar zijn in de Appstore-beschrijving van de app
  * Mogelijk nemen we ook een link hiernaar op in de app zelf

### Design

Voor ontwerpers zijn de volgende uitgangspunten relevant. Tussen haakjes staat het succescriterium uit WCAG vermeld waar dit uitgangpunt aan bijdraagt:

* Voor elke afbeelding wordt een afweging gemaakt of deze decoratief is. Als dit het geval is, wordt dit in het design vermeld Als dit niet het geval is, voorziet het design in een tekstalternatief. (1.1.1)
* Gebruikte kleuren hebben voldoende contrast (zie criterium voor details). Dit geldt voor tekst, niet-decoratieve grafische elementen en visuele informatie die nodig is om onderdele nvan de interface te herkennen. (1.4.3 en 1.4.11)
* Ontwerp waar nodig aparte schermen voor portret/landschap (1.3.4)
* Meldingen moeten voor iedereen leesbaar zijn, ook als je daar meer tijd voor nodig hebt. Ontwerp meldingen dus altijd met een knop om de melding te sluiten (Sluiten/OK/Verder etc) en laat ze niet vanzelf verdwijnen. (2.2.1)
* Denk bij het implementeren van animaties langer dan 5 seconden aan een mogelijkheid deze te pauzeren (2.2.2)
  * Dit geldt niet als de animatie het enige element in de viewport is, dus bijvoorbeeld een loader/spinner
* Maak geen elementen die meer dan drie keer per seconde knipperen, zie ook hier het succescriterium voor meer details en nuancering (2.3.1)
* Ontwerp foutteksten met de volgende kenmerken (3.3.1, 3.3.2, 3.3.3)
  * Maak duidelijk wat er fout ging
  * Vertel wat de gebruiker moet doen
  * Geef een suggestie hoe het op te lossen
  * Voorbeeld: "Er is een ongeldige code ingevoerd, voer een geldige code in. Een code bestaat uit 6 cijfers, bijvoorbeeld 123456"

### Ontwikkeling

In de designs is al rekening gehouden met bovenstaande eisen. Echter is het bij de ontwikkeling ook van belang om op een aantal zaken te letten.

* Gebruik native componenten
  * Als dit niet gaat, baseer het aangepaste component dan op het native component dat het dichtst in de buurt komt
* Neem de tekstomschrijvingen voro afbeeldingen, iconen en knoppen over uit de designs. Decoratieve afbeeldingen krijgen expliciet geen tekstalternatief
  * Als duidelijkheid hierover ontbreekt in de designs, leg de vraag dan terug bij de designer

### Testen

Door de handvatten voor design en ontwikkeling zal er wat toegankelijkheid betreft een hoop goed gaan. Toch is het belangrijk om ook de toegankelijkheidsaspecten bij het testen mee te nemen.

#### Android

* Test met een toetsenbord (2.1.1)
  * Doorloop de app een keer volledig met de pijltoetsen en een keer volledig met de tabtoets, activeer elementen met de entertoets
  * Alle interactieve elementen, die dus aanklikbaar of invulbaar zijn, moeten hierbij de focus krijgen. Alle niet-aanklikbare elementen niet. Ook moet de volgorde logisch zijn en overeenkomen met de visuele volgorde op het scherm
* Test met de ingebouwde screenreader Talkback (1.1.1, 1.3.1, 4.1.2)
  * Controleer of alle elementen een duidelijk label hebben
  * Controleer of alle elementen met het juiste type worden benoemd. Dus "knop", "tekstvak" etc
* Controleer de weergave van de ap met de opties "kleurinversie", "donkere mode" en "kleurcorrectie"
  * Leveren deze instellingen een te lag contrast op?
* Controleer zowel de functie tekst vergroten als vergroting
  * Is de app nog bruikbaar met de grootste vergrotingsfactor
* Wellicht kunnen we een klein deel van de tests automatiseren, dit moet nog worden uitgezocht

#### iOS

* Test met de ingebouwde screenreader VoiceOver (1.1.1, 1.3.1, 4.1.2)
  * Controleer of alle elementen een duidelijk label hebben
  * Controleer of alle elementen met het juiste type worden benoemd. Dus "knop", "tekstvak" etc
* Test met de zoomfunctie
  * Is de app nog bruikbaar op de grootste vergrotingsfactor
* Test met de verschillende functies voor kleuraanpassing zoals donkere mode, hoog contrast enkleuren omkeren
  * Levert dit een te laag contrast op?


### Ondersteuning en toegankelijkheidsklachten

* Er moet een plaats zijn waar mensen terecht kunnen met toegankelijkheidsproblemen, dit komt ook in de toegankelijkheidsverklaring
* Mensen die dergelijke klachten behandelen of gebruikers ondersteunen bij het gebruik van de ap moeten op de hoogte zijn van basale toegankelijkheidsfuncties
