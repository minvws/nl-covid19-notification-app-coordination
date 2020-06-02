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
Elevation of Privilege 	An unprivileged user gains privileged access and thereby has sufficient access to compromise or destroy the entire system. Elevation of privilege threats include those situations in which an attacker has effectively penetrated all system defenses and become part of the trusted system itself, a dangerous situation indeed
## Matrix
    1   2   3  4  5   
S   n   y   y  n   n
T   y   y   y  n   n
R   n   n   y    n  n
I   y   n   y    n  y
D   y   y   y    n  n
E   n   n   y  y y  n

Conclusie deel is het meest relevante stuk, waarin bijna alle categorien van het STRIDE model van toepassing zijn. 
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
3. Gebruik van de app voor het traceren van niet zijnde COVID-19 patienten door andere overige overheid en niet overheid instanties.  
Voorbeeld:

De politie gebruikt de corona app om een alibi te controleren. De andere persoon waarbij de verdachte claimt te zijn geweest ten tijde van het misdrijf, wordt gevraagd zich als corona besmet op te geven. Als de verdachte dan geen 'corona alert' haalt dit zijn alibi onderuit. 
Voorbeeld:  Een geheime dienst ontdekt een zwakheid in het google/apple protocol, houden dit geheim en gebruikt dit om spionnen te traceren maar de technologie die dit kan lekt uit en wordt daarna op grote schaal door prive detectives gebruikt, met een groot schandaal als gevolg.   
## Tegenstanders 
### Script Kiddies. 
Te denken valt aan jongeren die gebruiken maken van standaard programmatuur om zwakheden te ontdekken. Ze zijn meestal uit Nederland, vaak nog minderjarig. Een bekende klasse aanvallen die gebruikt worden zijn Denial of Service Attacks. Dit om de service te verstoren. Een voorbeeld hiervan zijn DDOS aanvallen op de internet betaal sites van de banken in 2018. 
### Cyber criminelen 
Dit zijn meestal in het buitenland operende criminelen die via computer hacks en afpersing geld proberen te verdienen. Het zijn de geprofessionaliseerde en gecriminaliseerde script kiddies. Ransomware aanvallen zijn de meest gebruikte gereedschappen voor dit soort criminelen. Vooral de centrale servers maar ook de code repository kunnen doelwit zijn van een afpersings zaak waar de  gegevens worden versleuteld of op een andere manier ontoegankelijk worden gemaakt, en alleen tegen betaling worden ontsleuteld.   
### Buitenlandse mogendheden.
Een contacten traceer app geeft ongekende mogelijkheden om mensen te volgen en om groepen (clusters) te ontdekken. In vele gevallen worden al mobiele telefoons in het buitenland bij grenspassage onderzocht. 
### Corrupte medewerkers 
Medewerkers kunnen rechten misbruiken en/ of informatie (bijvoorbeeld wachtwoorden) doorspelen naar derden en hierdoor gevoelige informatie lekken. Ook is het denkbaar dat bepaalde configuraties of code moedwillig wordt aangepast  om informatie te lekken.  
### Politieke druk 
Het huidige systeem is behoorlijk anoniem. Vanuit justitie (opsporing authoriteiten) kan er een druk verwacht worden om vooral in ernstige zaken toch de anonimiteit maatregelen op te heffen. Vergelijkbaar is bijvoorbeeld de discussie rond de whatsapp encryptie. Als dat eenmaal in een zaak is gedaan is de techniek van het doorbreken van de maatregelen bekend en ontstaat er een geleidende schaal van ernst van zaken, en zou het binnen enkele jaren gebruikt kunnen worden voor relatief lichte vergrijpen. Het is van groot belang voor de volksgezondheid dat een zo groot mogelijk aantal Nederlanders meedoet. Bij het gebruiken van contacten informatie voor andere doeleinden gaan de groepen die zich zorgen maken over de privacy de app niet meer gebruiken of gebruikmaken van tegen maatregelen zoals stoorzenders of elektromagnetische afscherming van ruimtes die ook vele andere nadelen hebben.  
 ### Profiterende Programmeurs 
De COVID-19 notificatie app is opensource. Andere programmeurs zouden in de verleiding kunnen komen om andere versies van de COVID-19 app te maken gebaseerd op de open source code met een aantal veranderingen. Hierdoor zou de privacy niet meer gewaarborgd worden of de AVG niet meer gevolgd bijvoorbeeld door een versie die advertenties laat zien. 
##  Contact informatie bepalen Risico’s 
Hiervoor wordt verwezen naar het document Crypto Raamwerk in https://github.com/minvws/nl-covid19-notification-app-coordination/tree/master/architecture
Een kleine opmerking valt wel te maken over de test optie van het GAEN protocol. Deze is noodzakelijk om een snelle feedback te krijgen tijdens de test van de programmatuur, maar is niet wenselijk tijdens productie. De vraag is hoe toegang tot een systeemtest in de App wordt geregeld.
## Informatie over besmette contacten verkrijgen. 
De app moet regelmatig informatie van een centrale server ophalen met daarin de Temporary Exposure keys van besmette personen. 
### Spoofing: 
Niet van toepassing. 
### Tampering 
Aangenomen wordt dat alleen een proces op de server(s) de nieuwe bestanden met daarin Temporary Exposure Keys kan generen. Gebruikers toegang tot deze bestanden behalve voor backup doeleinden dient te worden vermeden. 
### Non Repudiation 
Niet van Toepassing 
###  Information Disclosure
Hierbij kunnen andere clients dan de officiele app ook de Temporary exposure keys downloaden.  Om dit te vermijden dienen er maatregelen genomen worden zodat de informatie alleen beschikbaar is voor de app. 
Ook dient de verbinding tussen de App en de centrale server beveiligd te zijn, zodat deze niet ontsleuteld kan worden door tussenliggende partijen. Certificate pinning wordt aangeraden. 
De server dient ook beschermt te zijn tegen ongeauthorizeerde toegang via andere kanalen dan de download faciliteit die de app gebruikt. De gebruikelijke OS permissie, OS hardening, software update en authenticatie maatregelen dienen genomen te worden. 
### Denial of Service
Omdat elke App de bestanden download kan men eenvoudig de adressen van de servers achterhalen door de netwerk activiteit van de app te observeren. Dus het is een risico dat deze servers blootgesteld worden aan Denial of Service Attacks. 
### Elevation of Privilege
Niet van toepassing privileges/authenticatie worden in dit proces zoals nu bekend niet gebruikt.  
## Installatie en onboarding. 
###  Information Disclosure
Informatie over de installatie van uit de App Store/ PlayStore is bekend bij Apple/Google en statistieken zijn hierover te verkrijgen.
### Non Repudiation 
Een druk om de app te installeren in bepaalde iets meer riskante bijeenkomsten (denk aan horeca, sport of OV)  is te verwachten.  
  ### Denial of Service
Niet van toepassing, dit licht vooral bij App Store/ Playstore beheerders. 
### Elevation of Privilege
De APP moet niet meer privileges gebruiken dan nodig. De privileges waar toestemming voor gevraagd wordt, moeten worden uitgelegd.   
## Overige app/ gebruiker interacties. 
Dit is afhanklijk van de invulling van deze interacties. Te denken valt aan een test modus, extra informatie over Corona die getoond wordt aan de gebruiken. 
## Vervolg 
In het verdere verloop als ook de architectuur van de app meer bekend is kan een gedetailleerder beeld gegeven worden van elk van de 6 punten van elk deel systeem. 
Daarnaast is een ander handvat om te gaan zoeken naar vergelijkbare analyses en beveiliging problemen van andere vergelijkbare mobiele apps.
