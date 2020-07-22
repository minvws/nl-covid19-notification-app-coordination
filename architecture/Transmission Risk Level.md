# Transmission Risk Level

The GAEN protocol offers a 'transmission risk param' that helps assign a risk value to TEK keys. Each app that implements the protocol can define its own values for this parameter and [assign its own weights](https://developer.apple.com/documentation/exposurenotification/enexposureconfiguration).

To ensure compatibility with future versions of the protocol and to aid international interoperability, we will be using a subset of the [recommended values by Apple](https://developer.apple.com/documentation/exposurenotification/enexposureconfiguration/3586323-transmissionrisklevelvalues):

* 1: Confirmed test: Low transmission risk level
* 2: Confirmed test: Standard transmission risk level
* 3: Confirmed test: High transmission risk level
* 6: Negative case 

We will use the negative case classifier to filter out keys outside of the contagious period. Although we could publish these keys and assign them a zero weight in the client, we have opted for filtering out these keys on the server (data minimisation).

We will base the transmission risk level based on the 'day of symptoms onset' (if known) of the index patient, as provided by the GGD. .


The following pseudocode describes the algorithm. Note that the values may change: these will be provided by the health authority

````
daysSinceSymptomOnset = date(TEK) - dateOfSymptomOnset;

if (daysSinceSymptomOnset <= -4)       transmissionRiskLevel = 6; // not contagious
if (daysSinceSymptomOnset == -3)       transmissionRiskLevel = 1; // low
if (daysSinceSymptomOnset == -2)       transmissionRiskLevel = 2; // medium
if (-1 <= daysSinceSymptomOnset <= 2)  transmissionRiskLevel = 3; // high
if ( 3 <= daysSinceSymptomOnset <= 4)  transmissionRiskLevel = 2; // medium
if ( 5 <= daysSinceSymptomOnset <= 11) transmissionRiskLevel = 1; // low
if (daysSinceSymptomOnset >= 12)       transmissionRiskLevel = 6; // not contagious
```

Note that the risk calculation assigns a *value* to each of these classifications during the matching phase by providing the [risk calculation parameters](https://developer.apple.com/documentation/exposurenotification/enexposureconfiguration).