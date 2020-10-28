# International Interoperability

**Version**: 0.4, 27 October 2020

**Status**: Draft

**Authors**: 

* Ivo Jansch
* Maarten Brugman
* Dirk-Willem van Gulik
* Ryan Barrett

# Introduction

This document describes the architecture for interoperability between the Dutch CoronaMelder app and the apps of other countries in Europe via the European Union’s Federation Gateway Server (EFGS).

Interoperability means: users from one of the participating COVID19 Exposure Notification apps can receive a warning that they have been in close proximity of an infected user, regardless of which participating country or which app the infected user has used.

The basis for this interoperability are the following two EU e-Health network’s documents:

1. "eHealth Network Guidelines to the EU Member States and the European Commission on Interoperability specifications for cross-border transmission chains between approved apps: **Basic** interoperability elements between COVID+ Keys driven solutions" [EFGS/1]

2. "eHealth Network Guidelines to the EU Member States and the European Commission on Interoperability specifications for cross-border transmission chains between approved apps: **Detailed** interoperability elements between COVID+ Keys driven solutions" [EFGS/2]

The premise for interoperability lies in two key aspects:

* Countries that we want to interoperate with use the Google Apple Exposure Notification (GAEN) protocol (NOTE:  Other protocols may be considered later, but are currently out of scope for the initial interop version).

* Countries that we want to interoperate with are connected to the European Union Federation Gateway service (EFGS).

For now, we consider countries that do not use the EFGS out of scope for interoperability.

This document outlines all the technical and non functional details regarding the interoperability. 

# Contents

- [Background](#background)
  * [Travel challenges for exposure notifications](#travel-challenges-for-exposure-notifications)
- [Data Model](#data-model)
  * [Definitions](#definitions)
  * [Required key data for the app user](#required-key-data-for-the-app-user)
  * [Example scenario](#example-scenario)
  * [Determining countries of interest](#determining-countries-of-interest)
  * [Coordinating transmission risk levels between countries](#coordinating-transmission-risk-levels-between-countries)
  * [Avoiding duplicate notifications](#avoiding-duplicate-notifications)
- [High Level Architecture](#high-level-architecture)
  * [Overview](#overview)
    + [Filtering](#filtering)
  * [Impact Analysis](#impact-analysis)
    + [CoronaMelder Existing Back-end](#coronamelder-existing-back-end)
    + [CoronaMelder Interop Server](#coronamelder-interop-server)
    + [Exposure Key Set generator](#exposure-key-set-generator)
    + [GGD Portal](#ggd-portal)
    + [CoronaMelder App](#coronamelder-app)
- [API Design](#api-design)
  * [Interface between Dutch back-end and Federation Gateway](#interface-between-dutch-back-end-and-federation-gateway)
  * [Interface between Dutch back-end and the Dutch app](#interface-between-dutch-back-end-and-the-dutch-app)
    + [Manifest](#manifest)
    + [Exposure Key Set](#exposure-key-set)
- [Security Considerations](#security-considerations)
  * [GAEN Signatures](#gaen-signatures)
  * [CMS Signatures](#cms-signatures)
  * [Trust between Dutch back-end and Federation Gateway](#trust-between-dutch-back-end-and-federation-gateway)
  * [Key Stuffing](#key-stuffing)
- [Privacy Considerations](#privacy-considerations)
  * [Key sharing](#key-sharing)
  * [Key downloads](#key-downloads)
  * [Travel Destinations](#travel-destinations)
- [References](#references)

# Background

## Travel challenges for exposure notifications

Interoperability is not limited to travelers between countries. Residents of a country who do not travel might, unknowingly, still encounter an infected person from another country, who is visiting during a holiday or for work. 

Therefore, we need to look at the risks and features we need for both travelers and non-travelers. The following table describes the various options we have to deal with:

<table>
  <tr>
    <th>Who is at risk</th>
    <th>Potential source of infection</th>
    <th>What does the person at risk need for successful Exposure Notification?</th>
    <th>Remark</th>
  </tr>
  <tr>
    <td rowspan="3">Traveler</td>
    <td>Residents of country of origin (before departure)</td>
    <td>All keys of infected people in origin country</td>
    <td>This scenario is equal to domestic contact tracing, it doesn’t matter if the person is a traveler or not.</td>
  </tr>
  <tr>
    <td>Residents of destination country, when traveler was there </td>
    <td>All keys of infected people in destination country</td>
    <td></td>
  </tr>
  <tr>
    <td>Residents of third party countries that also visited the destination country at the same time as the traveler.</td>
    <td>All keys of infected people who have traveled to destination country, regardless of their origin</td>
    <td>Risk is higher for international events and tourist areas where people from many origins gather.</td>
  </tr>
  <tr>
    <td rowspan="2">Non-traveler</td>
    <td>Residents of country of origin </td>
    <td>All keys of infected people in origin country</td>
    <td>Equal to domestic contact tracing.</td>
  </tr>
  <tr>
    <td>Residents of third party countries that visited country of origin</td>
    <td>All keys of infected people of all countries who have visited origin country</td>
    <td>Risk is higher for international events and tourist areas where people from many origins gather.</td>
  </tr>
</table>


# Data Model

## Definitions

The following definitions are used in this document:

<table>
  <tr>
    <td>Origin</td>
    <td>The country which made the mobile application that the user is using; this includes the back-end the mobile application interacts with. More specifically - the crucial element is the GAEN public key that the GAEN framework on the phone uses to verify the digital signature on any TEK distributions. This is managed and set by Apple and Google as part of the mobile app approval process. </td>
  </tr>
  <tr>
    <td>Traveler</td>
    <td>A user who has a mobile app of a country (A) that is currently traveling in another country (B). I.e. who is traveling in an environment where most phones are originating from that country B; and will get their distributions from that origin B.</td>
  </tr>
  <tr>
    <td>Non traveler</td>
    <td>Person who is in the same country (Origin) as the country whose mobile app he or she is using.</td>
  </tr>
  <tr>
    <td>(Origin) Backend</td>
    <td>The backend that a mobile application interacts with. Is operated by and tied to the same country as to which the mobile app is tied by the digital signature managed by apple/google.</td>
  </tr>
  <tr>
    <td>Federation Gateway</td>
    <td>LUX based exchange service - that communicates securely (and only) with the participating Origin Backends.</td>
  </tr>
</table>


## Required key data for the app user

Deriving the ideal datasets that users of contact tracing apps need, from the above risk profiles,  we come to the following aggregate:

<table>
<tr>
 <td rowspan="4">Traveler’s app requires: 	</td>
 <td>a. All domestic keys of origin</td>  
</tr>
<tr><td>b. Keys of all travelers who traveled to origin </td></tr>
<tr><td>c. All domestic keys of destination </td></tr>
<tr><td>d. Keys of all travelers who traveled to destination</td></tr>
</td>
</tr>
<tr>
<td rowspan="2">Non-traveler’s app requires:</td>
<td>a. All domestic keys of origin</td></tr>
<tr><td>b. Keys of all travelers who traveled to origin</td></tr>
</table>

This maps nicely on the proposal for the Federation Gateway from the e-health initiative. This gateway stores keys for all participating countries and adds 2 important fields:

* The country of origin: which country (app) did the key come from.
* The *countries of interest*: for which countries is the key relevant.

If a traveler from The Netherlands travels to Germany, then the *origin* is ‘NL’ and the *country of interest* is 'DE', so that when the Dutch person is tested positive for COVID-19, their key can be shared with Germany to notify people they have met during their stay.

Mapping the country of interest on the above datasets, we get (for the example of a Dutch traveler to Germany) 4 keysets, marked **a** through **d** below:

<table>
<tr>
<td rowspan="4">Traveler’s app requires:</td>
<td>a. All domestic keys of origin</td><td>keys with origin = NL</td></tr>
<tr><td>b. Keys of all travelers who traveled to origin</td><td>keys with country of interest = NL</td></tr>
<tr><td>c. All domestic keys of destination</td><td>keys with origin = DE</td></tr>
<tr><td>d. Keys of all travelers who traveled to destination</td><td>keys with country of interest = DE</td></tr>	
<tr>
<td rowspan="2">Non-traveler’s app requires:</td>
<td>a. All domestic keys of origin</td><td>keys with origin = NL</td></tr>
<tr><td>b. Keys of all travelers who traveled to origin</td><td>keys with country of interest = NL</td></tr>
</table>

## Key distribution pattern

### Key download patterns
There are three main patterns that countries participating in EFGS are using for distributing EFGS keys to their domestic apps:
1. **One EU pattern**: distribute _all_ keys downloaded from the EFGS to the domestic app.
2. **Traveller pattern**: have a user indicate in the app whether or not they travelled. If so, distribute all keys from the EFGS to the user. If not, distribute only the domestic keys to the user.
3. **Country of interest pattern**: have user indicate in the app the country or countries they are visiting. Have the app fetch key sets for the visited countries accordingly.

### Key upload patterns
Similarly, there are three main patterns for uploading diagnosis keys to EFGS:
1. **One EU pattern**: do not ask a diagnosed user whether or not they travelled. Upload the keys of diagnosed users to EFGS and specify _all_ participating countries as countries of interest.
2. **Traveller pattern**: ask a diagnosed user whether or not they travelled abroad while they were contagious. If so, upload the keys of the diagnosed user to EFGS with _all_ participating countries as countries of interest. If user did not travel, upload with the country of interest = this country's country code.
3. **Country of interest pattern**: ask a diagnosted user which participating countries they visited while they were contagious. Upload the diagnosis keys to EFGS with countries of interest set accordingly.

### Choice of distribution pattern
The Traveller and Country of Interest patterns have some drawbacks:
* All app users need to set travelling modes (domestic/traveling/list of countries) in the app. This is error-prone.
* Diagnosed users need to be asked about their travels abroad. This may cause privacy concerns, since the CoronaMelder system has been presented to the public as a system that does not track location.

On the other hand, the One EU pattern has the drawback that the number of keys shared between all countries participating in EFGS may grow to a point that this results in considerable data usage and potential technical problems on the devices during key matching, such as resource problems due to operating system constraints or higher battery usage.

Assuming 100.000 daily infections across participating countries (which is the case in the 'second Covid-19' wave end of October 2020), about 1.500.000 keys (for maximum 15 contagious days) would be shared to EFGS daily. The upper limit for a TEK file is 750.000 keys. So this amount of keys would have to be shared across two TEK files.

Considering all of the above, and also with privacy maximization in mind, we have chosen for the **'One EU' pattern**. That is, we upload all diagnosis keys to EFGS and we distribute all EFGS keys to the CoronaMelder apps.

## Coordinating transmission risk levels between countries

The Dutch CoronaMelder app uses the recommended values for Transmission Risk Level as specified by Apple (NOTE:  See https://developer.apple.com/documentation/exposurenotification/enexposureconfiguration/3586323-transmissionrisklevelvalues):

0 = Unused/Custom

**1 = Confirmed test: Low transmission risk level**

**2 = Confirmed test: Standard transmission risk level**

**3 = Confirmed test: High transmission risk level**

4 = Confirmed clinical diagnosis

5 = Self report

**6 = Negative case**

7 = Recursive case

Only the bold ones are used in NL. The Negative Case (6) is used only temporarily, when due to days of symptom onset filtering, we want to remove certain keys. These keys get assigned value 6, and will not be exported to the CDN or the EFGS.

If countries use the same definition, interoperability is immediately achieved. If countries use a custom mapping, then our back-end will have to do a conversion between the originating country’s risk levels and our own (or remove them altogether; for example in the case of ‘self diagnosis’, which the Dutch app does not support).

# High Level Architecture

## Overview

The following diagram depicts a high level architecture of our interaction with the Federation Gateway. Here we can see the relationship between the *origin* of the key and the *countries of interest* of the keys, and where keys come from and how they are published.

We publish the foreign keysets on the CDN in a separate folder so that these files can be selectively downloaded by only those residents who have traveled *to* a particular country. Residents who do not travel only need the NL keysets, which include all keys that have ‘countries of interest’ NL (whether they originated from NL or from other countries).

The blue parts are new compared to the existing setup. The yellow parts are external.

![High Level Architecture](images/interop_hla.png)

*Figure 2: Interactions between components*

Note how keys that are uploaded by our app are authorized by the GGD (see Solution Architecture, lab confirmation flow). Keys that are downloaded from the Federation Gateway are *pre-authorized*. We assume that those keys were validated by the health authority in their country of origin.

### Filtering

To ensure adequate day to day operational control and some level of digital sovereignty for the National Health Authorities several filters are introduced in the diagram in the previous chapter. These filters are a compromise with the principle of not irrevocably deleting or modifying data from other countries on the one hand and the desire to keep as little information as possible on the other hand.

The **Ingress** filter is applied prior to any storage and is to filter out any data that does not meet National policy; for example around privacy. This data is simply stopped at the edge and never reaches the database. Criteria of filtering are Country of Interest (CoI), Period, Length and Risk level.

Note that other countries may well use other Risk Level definitions. The Ingress filter needs to store the original value that is retrieved from the EFGS as a separate value, distinct from the Risk Levels we use.

The EFGS interface for fetching TEKs takes a date and a batchTag for keeping track of what our server has downloaded so far. No filtering options are available in this interface. Filtering on the Ingress filter does not interfere with fetching the data.

The **Converter** filter is applied just prior to the data being packaged. It converts things such as risk levels. The reason for having this applied post storage is to allow operational changes and retransmission. This also lets the National Health authority delay or speed up certain data - as to adjust capacity and demand.

The **Output** filter can filter out certain countries operationally; for example to manage capacity. Initially this filter will not filter anything. The need for filtering will be discussed with the Dutch Health Authority.

The **Egress** filter is applied prior to sending data to the Federation Gateway. For example if the National health authority considers a certain country not yet fit to be informed of the CoI of dutch travellers.

### Deduplication

Because we are performing a 2-way exchange of keys between the gateway and our back-end, we must pay attention to not create duplicate notifications (for example because keys we have contributed to the gateway are re-downloaded when we download from the gateway). This can also be a problem if a key has 2 countries of interest and ends up both in the domestic and the foreign set. We want to avoid duplicate notifications, so the Dutch server should publish each key only once for its users. In order to do so, we need to take into account how a country distributes its keys.

** 'One World pattern' **

This pattern means that a country does not track which countries a user has visited. All users are the same, whether they traveled or not. All keys from all infected persons are always marked relevant to all participating countries. Keys that have this pattern should be stored in our domestic set, as they are relevant for people who travel and people who do not travel. 

** 'Traveler pattern' **

This pattern means that a country does distinguish between travelers, but not to which country they traveled. In this case, traveler's keys have all participating countries as 'country of interest', non traveler's keys only have the home country as country of interest. In this case we act the same as with the one world pattern.

** 'Country of Interest pattern' **

A country implementing the desired 'country of interest' will specifically mark keys for all countries that are relevant. In this case, we could optimize how we distribute the keys if we were to allow the user to specify their countries of interest. However, since we do not do this, again we include all keys in our domestic key set. 


# API Design

## Interface between Dutch back-end and Federation Gateway

For the interface between the Dutch backend and the Federation Gateway we follow the specifications of the Federation Gateway [EFGS/2]. The Gateway offers both push and pull models. Our choices here are:

1. Keys that the Dutch backend should share **to** the Gateway will be periodically **pushed** to the Gateway. 

2. Keys that the Dutch backend needs **from** the Gateway will be periodically **pulled** from the Gateway.

This model ensures that the Dutch back-end only needs outgoing connections, and does not need to offer any additional open endpoints. This greatly simplifies the security design.


## Interface between Dutch back-end and the Dutch app

The interface between the Dutch CoronaMelder backend and the apps does not need any modification. Only the number of keys published by the backend and processed by the apps will increase.


# Security Considerations

## GAEN Signatures

The CoronaMelder app has a registered public key with apple and google, that is used to validate keysets. Since our app can only access our national (Dutch) public keys, we can’t re-use foreign GAEN signatures. When generating keysets, the CoronaMelder back-end should sign the packages with the **Dutch** unmanaged GAEN key.

## CMS Signatures

The same holds true for the PKI Overheid CMS signatures that we place on files. We can’t reuse foreign certificates, so the files that we publish on the CDN should be signed with the Dutch PKI Overheids certificates, exactly the same as we do with the domestic keysets.

## Trust between Dutch back-end and Federation Gateway

When exchanging keys, the Federation Gateway and the Dutch backend mutually authenticate with a X.509 client and server certificates, as described in [EFGS/2] (Section 6). The Dutch back-end will verify (pin) the Federation Gateway explicitly. The Federation Gateway will authenticate the Dutch back-end explicitly (page 48 of [EFGS/2])

## Key Stuffing

In theory it could happen that we send only a few files from a few infected persons to the gateway. To ensure that a compromised gateway wouldn’t be able to link keys so it can track persons across multiple days, the same stuffing rules as applied domestically are applied before we upload them to the Gateway.

# Privacy Considerations

A separate DPIA (Data Protection Impact Analysis) of the CoronaMelder app to take interoperability considerations into account will be performed and documented. The chosen architecture by the European eHealth initiative and our connection with it introduce a number of elements that warrant DPIA analysis. 

## Key sharing

Since diagnosis keys rotate on a daily basis and a user may have visited multiple countries on the same day, the chosen data sharing solution means that one temporary daily key will be sent to multiple countries, via the gateway. This means that:

1. The country that receives keys from travelers, can see how many infected people have traveled to that country, and (depending on the implementation), from where.

2. The Federation Gateway has origin and countries of interest of all keys from all participating countries, so has knowledge about travel patterns between countries. 

In both cases only the total number is known; the individual residents can’t be derived from just these databases.

The information is limited to a specific day because daily keys are unlinkable. With unlinkable keys, it isn’t possible to create a ‘trail’ that spans more than a single day.

## Key downloads

Since all keys from all participating countries are published the same way on the CoronaMelder CDN, foreign keys are not distinguishable from this endpoint. It is possible to download the keys from countries that implement the Country of Interest or the Traveller pattern and derive travel patterns of the collected daily keys. These keys however do not by themselves identify individuals. 

The EFGS design ensures that all apps including CoronaMelder only download from their domestic server. The foreign keys are never sourced directly from other countries but only via the Federation Gateway and our own interop server. 

## Use of test data

For all testing purposes (during development, testing, EU pilot etc) we will only supply synthetic, test TEKs that do not originate from real users. Real end user keys will only be used in production. 


# References

[EFGS/1] 	eHealth Network Guidelines to the EU Member States and the European Commission on Interoperability specifications for cross-border transmission chains between approved apps; Basic interoperability elements between COVID+ Keys driven solutions; V1.0 2020-06-16 
[https://ec.europa.eu/health/sites/health/files/ehealth/docs/mobileapps_interoperabilityspecs_en.pdf](https://ec.europa.eu/health/sites/health/files/ehealth/docs/mobileapps_interoperabilityspecs_en.pdf)

[EFGS/2] 	eHealth Network Guidelines to the EU Member States and the European Commission on Interoperability specifications for cross-border transmission chains between approved apps; Detailed interoperability elements between COVID+ Keys driven solutions; V1.0 2020-06-16 [https://ec.europa.eu/health/sites/health/files/ehealth/docs/mobileapps_interoperabilitydetailedelements_en.pdf](https://ec.europa.eu/health/sites/health/files/ehealth/docs/mobileapps_interoperabilitydetailedelements_en.pdf)
