# Threat model Covid-19 Notificatie App. 

## Inleiding 
De Covid-19 Notificatie App brengt grote zorgen bij het publiek en de beveiligingsprofessionals over potentieel misbruik. Om alle mogelijke risico’s en scenario's in beeld te brengen en accuraat in te schatten en daarop weer maatregelen te vinden is veel werk.  Het grootste gevaar echter zijn blinde vlekken dus dat men bepaalde risico’s niet ziet, en daarop geen passende maatregelen heeft genomen. Een bijkomend complicatie is de grote tijdsdruk door de Corona crisis. 
Achteraf zijn veel beveiligings problemen redelijk voor de hand liggend, maar toch worden ze vooraf niet onderkend of de ernst word niet goed ingeschat, zodat niet de juiste maatregelen genomen worden.   
Het volgende document gaat uit van het STRIDE model, zie https://docs.microsoft.com/en-us/azure/security/develop/threat-modeling-tool-threats en
https://insights.sei.cmu.edu/sei_blog/2018/12/threat-modeling-12-available-methods.html. 
De voorbeelden die in dit eerste hoofdstuk staan zijn slechts voorbeelden om de threats te illustreren. 
Er moet niet vanuit gegaan worden dit allemaal realistische bedreigingen zijn,  waar tegen maatregelen genomen moeten worden. 
Er worden in dit document geen gedetaileerde beschrijvingen gegeven van aanvallen dit vanwege de vertrouwelijkheid. 
## Systeem Architectuur 
Voor deze analyse wordt het opgesplitst in 5 delen: 
 1. Contact informatie bepalen. 
Interactie tussen de mobiele telefoons doormiddel van bluetooth of vergelijkebare technologie om contacten binnen een redelijke afstand te bepalen.
Nu wordt voorgesteld om hier voor het GAEN (Google/Apple Exposure Notification)  framework te gebruiken. 
Omdat dit generiek is, en niet voor de Nederlandse  App specifiek, ligt de nadruk in dit document niet op dit gedeelte van de App.  
2. Informatie over besmette contacten verkrijgen. 
Interactie tussen de app en een centrale server om een lijst van geanonimiseerde besmette contacten op te halen. 
3. Het uitvoeren van een test en het doorgeven van een positieve test uitslag. 
Interactie tussen de gebruiker , de app en de gezondheid autoriteiten bij het doen van de test. 
Interactie tussen de gebruiker, de gezondheid autoriteiten, de centrale server en de app voor het doorgeven van een positieve test. 
4. Installatie en onboarding. 
5. Overige app/ gebruiker interacties. 
## Stride Model 
Het STRIDE-model geeft de volgende categorieën 
+ Spoofing 
+ Tampering
+ Non Repudiation 
+ Information Disclosure
+ Denial of Service
+ Elevation of Privilege 
### English explanation 
Category 	Description
Spoofing 	Involves illegally accessing and then using another user's authentication information, such as username and password
Tampering 	Involves the malicious modification of data. Examples include unauthorized changes made to persistent data, such as that held in a database, and the alteration of data as it flows between two computers over an open network, such as the Internet
Repudiation 	Associated with users who deny performing an action without other parties having any way to prove otherwise—for example, a user performs an illegal operation in a system that lacks the ability to trace the prohibited operations. Non-Repudiation refers to the ability of a system to counter repudiation threats. For example, a user who purchases an item might have to sign for the item upon receipt. The vendor can then use the signed receipt as evidence that the user did receive the package
Information Disclosure 	Involves the exposure of information to individuals who are not supposed to have access to it—for example, the ability of users to read a file that they were not granted access to, or the ability of an intruder to read data in transit between two computers
Denial of Service 	Denial of service (DoS) attacks deny service to valid users—for example, by making a Web server temporarily unavailable or unusable. You must protect against certain types of DoS threats simply to improve system availability and reliability
Elevation of Privilege 	An unprivileged user gains privileged access and thereby has sufficient access to compromise or destroy the entire system. Elevation of privilege threats include those situations in which an attacker has effectively penetrated all system defenses and become part of the trusted system itself, a dangerous situation indeed. 
## Matrix
<table
<tr><th> </th><th>   1</th><th>   2</th><th>   3</th><th>    4</th><th>  5<th></tr>
<tr><td>S</td><td>   n</td><td>   n</td><td>   y</td><td>    y</td><td>  n</td></tr>
<tr><td>T</td><td>   y</td><td>   y</td><td>   y</td><td>    n</td><td>  n</td></tr>
<tr><td>R</td><td>   n</td><td>   n</td><td>   y</td><td>    n</td><td>  n</td></tr>
<tr><td>I</td><td>   y</td><td>   n</td><td>   y</td><td>    n</td><td>  y</td></tr>
<tr><td>D</td><td>   y</td><td>   y</td><td>   y</td><td>    n</td><td>  n</td></tr>
<tr><td>E</td><td>   n</td><td>   n</td><td>   y</td><td>    y</td><td>  n</td></tr>
</table>

Conclusie deel 3 is het meest relevante stuk, waarin bijna alle categorien van het STRIDE model van toepassing zijn.
Omdat dit stuk nog in ontwerp en hier nauwelijks gebruik gemaakt wordt van de GAEN programmatuur maar van eigen code, is de analyse nog niet compleet. 
## Specifieke gevaren buiten de bovenstaande : 
1. Privacy Schendingen 
Bijvoorbeeld de anonimisatie. Men probeert de identiteit van personen de contacten te achterhalen en daarmee extra persoons gegevens te krijgen van de contacten. 
Dit kan richting de app zelf, waarin de status van de gebruiker wordt bepaald. Met voldoende extra data kan de identiteit van gebruikers worden bepaald.  
Voorbeeld: Camera's registreren beelden van gebruikers van de corona app terwijl ze ook de bluetooth signalen opvangen. 
Hierdoor kan men in het tijds interval dat de signalen het zelfde zijn de gebruikers verder volgen en koppelen aan plaatje van de gebruiker. 
Ander voorbeeld extra traceer code is aanwezig binnen de telefoon (buiten de corona app) bijvoorbeeld als malware, die probeert de data van de corona app met de contacten gegevenste achterhalen en te koppelen aan gebruikers informatie zoals mobiele telefoon nummers en email adressen.   
2. Misbruik van de app door de makers / beheerders van de app en de gezondheids authoriteiten. Dat is het gebruiken van de app voor andere toepassingen dan waar de gebruikers toestemming hebben gegeven en wettelijk is toegestaan. 
Voorbeeld: De  GGD gaat in de toekomst de corona app ook gebruiken voor andere besmettelijke zieken zoals griep. Terwijl dit niet is waar de gebruikers toestemming voor hebben gegeven. 
Voorbeeld: Terwijl de app makers claimen het veilige google/apple protocol te gebruiken wordt in werkelijkheid een ander protocol gebruikt dat niet veilig is.  
Open source kan een gedeelte van deze zorg wegnemen maar als men wilt controleren of de gebruikte source code overeenkomt met de gepubliceerde vereist het gebruik van reproducible builds. 
Bij de server ligt het lastiger omdat de beheerders volledige toegang tot het systeem hebben en altijd de configuratie en programmatuur kunnen aanpassen en de server iets anders laten doen dan de broncode aangeeft. 

3. Gebruik van de app voor het traceren van niet zijnde COVID-19 patienten door andere overige overheid en niet overheid instanties.  
Voorbeeld:

De politie gebruikt de corona app om een alibi te controleren. De andere persoon waarbij de verdachte claimt te zijn geweest ten tijde van het misdrijf, wordt gevraagd zich als corona besmet op te geven. Als de verdachte dan geen 'corona alert' haalt dit zijn alibi onderuit. 
Voorbeeld:  Een geheime dienst ontdekt een zwakheid in het google/apple protocol, houden dit geheim en gebruikt dit om spionnen te traceren maar de technologie die dit kan lekt uit en wordt daarna op grote schaal door prive detectives gebruikt, met een groot schandaal als gevolg.   
## 1. Contact informatie bepalen Risico’s 
Hiervoor wordt verwezen naar het document Crypto Raamwerk in https://github.com/minvws/nl-covid19-notification-app-coordination/tree/master/architecture
Een kleine opmerking valt wel te maken over de test optie van het GAEN protocol. Deze is noodzakelijk om een snelle feedback te krijgen tijdens de test van de programmatuur, maar is niet wenselijk tijdens productie. De vraag is hoe toegang tot een systeemtest in de App wordt geregeld.
## 2. Informatie over besmette contacten verkrijgen Risco's 
De app moet regelmatig informatie van een centrale server ophalen met daarin de Temporary Exposure keys van besmette personen. 
### Spoofing: 
Niet van toepassing. 
### Tampering 
Aangenomen wordt dat alleen een proces op de server(s) de nieuwe bestanden met daarin Temporary Exposure Keys kan generen. Gebruikers toegang tot deze bestanden behalve voor backup doeleinden dient te worden vermeden. 
### Non Repudiation 
Niet van Toepassing 
###  Information Disclosure
De verbinding tussen de App en de centrale server beveiligd te zijn, zodat deze niet ontsleuteld kan worden door tussenliggende partijen. TLS met daarbij Certificate pinning wordt aangeraden.  
De server dient ook beschermt te zijn tegen ongeauthorizeerde toegang via andere kanalen dan de download faciliteit die de app gebruikt. De gebruikelijke OS permissie, OS hardening, software update en authenticatie maatregelen dienen genomen te worden. 
### Denial of Service
Omdat elke App de bestanden download kan men eenvoudig de adressen van de servers achterhalen door de netwerk activiteit van de app te observeren. Dus het is een risico dat deze servers blootgesteld worden aan Denial of Service Attacks. 
### Elevation of Privilege
Niet van toepassing privileges/authenticatie worden in dit proces zoals nu bekend niet gebruikt.

## 3. Het uitvoeren van een test en het doorgeven van een positieve test uitslag.
### Spoofing
De test en uitslagen worden verwerkt via een web applicatie. TLS encryptie dient te worden uitgevoerd. De OWASP richtlijnen voor veilige web applicaties, om risico's zoals sql injection, cross site scripting en dergelijke uit te sluiten.   
### Tampering
Toegang tot de databases waarin tests worden opgelagen dient goed te worden geregeld. Daarnaast dient het systeem van de TAN code's ook cryptografisch geanalyseerd te worden. 
### Non Repudiation 
Gebruikers kunnen een behoorlijk belang hebben bij een negatieve test uitslag omdat een postieve uitslag verplicht tot quarantaine van niet alleen de persoon zelf maar ook van recente contacten. Dit kan bijvoorbeeld negatieve gevolgen hebben voor het werk. 
### Information Disclosure
Informatie over de test uitslag is privacy gevoelig en dient te worden afgeschermd. 
### Denial of Service
Het doen van de test en dan het weer doorgeven van een positieve test gebruiken ook een webapplicatie. Een denial of serive attack zou hier mogelijk kunnen zijn. 
### Elevation of Privilege 
Het stuk waarin informatie wordt opgeslagen dient gebruikers te authoriseren en kent mogelijk meerde privileges. Ook een functionaliteit om mogelijk een fout-positieve of fout-negatieve test later weer te corrigeren is een stuk van de applicatie waarin risico's worden gelopen. Er kan druk worden uitgeoefend om test resultaten later onterecht van uitslag te veranderen.  
## 4. Installatie en onboarding Risco's 
###  Information Disclosure
Informatie over de installatie van uit de App Store/ PlayStore is bekend bij Apple/Google en statistieken zijn hierover te verkrijgen.
### Non Repudiation 
Een druk om de app te installeren in bepaalde iets meer riskante bijeenkomsten (denk aan horeca, sport of OV)  is te verwachten.  
### Denial of Service
Niet van toepassing, bescherming tegen een denial of service attack (dus dat je de app niet meer kan installeren)  ligt vooral bij App Store/ Playstore beheerders. 
### Elevation of Privilege
De App moet niet meer privileges gebruiken dan nodig. De privileges waar toestemming voor gevraagd wordt, moeten worden uitgelegd.   
## 5. Overige app/ gebruiker interacties. 
Dit is afhanklijk van de invulling van deze interacties. Te denken valt aan een test modus, extra informatie over Corona die getoond wordt aan de gebruiken.
## Tegenstanders 
### Script Kiddies. 
Te denken valt aan jongeren die gebruiken maken van standaard programmatuur om zwakheden te ontdekken. Ze zijn vaak(maar niet altijd) afkomstig uit Nederland, en zijn vaak nog minderjarig. Een bekende klasse aanvallen die gebruikt worden zijn Denial of Service Attacks. Dit om de service te verstoren. Een voorbeeld hiervan zijn DDOS aanvallen op de internet betaal sites van de banken in 2018. 
### Cyber criminelen 
Dit zijn meestal in het buitenland operende criminelen die via computer hacks en afpersing geld proberen te verdienen. Je kan het zien als de geprofessionaliseerde en gecriminaliseerde script kiddies. Om vervolging moeilijker werken ze internationaal.  Ransomware aanvallen zijn  de meest gebruikte gereedschappen voor dit soort criminelen. Vooral de centrale servers maar ook de code repository kunnen doelwit zijn van een afpersings zaak waar de  gegevens worden versleuteld of op een andere manier ontoegankelijk worden gemaakt, en alleen tegen betaling worden ontsleuteld.   
### Buitenlandse mogendheden.
Een contacten traceer app geeft ongekende mogelijkheden om mensen te volgen en om groepen (clusters) van mensen te ontdekken. In vele gevallen worden al mobiele telefoons in het buitenland bij grenspassage onderzocht. 
### Corrupte medewerkers 
Medewerkers kunnen rechten misbruiken en/ of informatie (bijvoorbeeld wachtwoorden) doorspelen naar derden en hierdoor gevoelige informatie lekken. Ook is het denkbaar dat bepaalde configuraties of code moedwillig wordt aangepast  om informatie te lekken.  
### Politieke druk 
Het huidige systeem is behoorlijk anoniem. Vanuit justitie (opsporing authoriteiten) kan er een druk verwacht worden om vooral in ernstige zaken toch de anonimiteit maatregelen op te heffen. Vergelijkbaar is bijvoorbeeld de discussie rond de whatsapp encryptie. Als dat eenmaal in een zaak is gedaan is de techniek van het doorbreken van de maatregelen bekend en ontstaat er een gldende schaal van ernst van zaken, en zou het binnen enkele jaren gebruikt kunnen worden voor relatief lichte vergrijpen. Het is van groot belang voor de volksgezondheid dat een zo groot mogelijk aantal Nederlanders meedoet. Bij het gebruiken van contacten informatie voor andere doeleinden gaan de groepen die zich zorgen maken over de privacy de app niet meer gebruiken of gebruikmaken van tegen maatregelen zoals stoorzenders of elektromagnetische afscherming van ruimtes die ook vele andere nadelen hebben.  
### Profiterende Programmeurs 
De COVID-19 notificatie app is opensource. Andere programmeurs zouden in de verleiding kunnen komen om een andere versies van de COVID-19 app te maken gebaseerd op de open source code met een aantal veranderingen bijvoorbeeld een versie die advertenties laat zien. Het tonen van advertenties geeft natuurlijk allerlei extra risco's qua privacy en geeft een extra partij voor de AVG. 

## Vervolg 
In het verdere verloop als ook de architectuur van de app meer bekend is kan een gedetailleerder beeld gegeven worden van elk van de 6 punten van elk deel systeem. 
Daarnaast is een ander handvat om te gaan zoeken naar vergelijkbare analyses en beveiliging problemen van andere vergelijkbare mobiele apps.
