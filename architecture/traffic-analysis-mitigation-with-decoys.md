Traffic analysis mitigation with decoy traffic
==============================================

# Introduction

The COVID-19 notification app allows a user who has been tested positive to upload TEKs (Temporary Exchange Keys) to the app’s back end servers. Anybody who can monitor the network traffic of the user’s device (smartphone) could potentially derive from the upload traffic that the user has a COVID-19 infection. This is a major privacy concern. This document describes how to mitigate network traffic analysis by systematically sending ‘decoy’ traffic that is indistinguishable from real TEK uploads.

Persons or entities that can listen in on network traffic include:

* If user is on a public Wifi hotspot, anybody on the same public Wifi hotspot can monitor network traffic.

* If user uses a corporate VPN, the corporate employer can continuously monitor the VPN network traffic to detect possible infections among employees.

* If user is on a home Wifi network, any housemates can monitor the network traffic.


# TEK upload sequence

Genuine TEK uploads are a sequence of 3 calls from app to server:

1. /register, to request a one-time password (labConfirmationID) and an accompanying confirmationKey and bucketID

2. /postkeys, to submit the TEK keys up to today.

3. /postkeys, once more to submit the 14th TEK key.  (NOTE:  It may be that in a next release of the GAEN framework the 14th (current) TEK is delivered together with the last 13 TEKs, followed immediately by a TEK renewal inside the framework, in the interest of privacy.)

In between the first two calls, the user reads the labConfirmationID to the health care worker over the phone. This will take some time, typically from 5 to 60 seconds (this is an estimation; not a result from a user test).

There’s a few hours of delay in between the second and third call, until just after Midnight when the framework allows retrieval of the 14th key.

The following table plots this observable network traffic on a timeline. For this example, a delay of 20 seconds is used between the first and second call, and a delay of 8000 seconds is used between the second and third call.

<table>
  <tr>
    <td>Timestamp
example (sec:ms)</td>
    <td>Duration
example (sec:ms)</td>
    <td>Duration label</td>
    <td>Action</td>
    <td>Network
packet length</td>
    <td>Actor</td>
  </tr>
  <tr>
    <td>0:000</td>
    <td>0:100</td>
    <td>Treg_req</td>
    <td>Send /register request</td>
    <td>Lreg_req</td>
    <td>App</td>
  </tr>
  <tr>
    <td>0:100</td>
    <td>0:300</td>
    <td>Treg_prc</td>
    <td>Process /register request</td>
    <td>-</td>
    <td>Server</td>
  </tr>
  <tr>
    <td>0:400</td>
    <td>0:100</td>
    <td>Treg_rsp</td>
    <td>Send/register response</td>
    <td>Lreg_rsp</td>
    <td>Server</td>
  </tr>
  <tr>
    <td>0:500</td>
    <td>20:000</td>
    <td>Tuser</td>
    <td>Read lab conf code to operator; duration 20s in this example</td>
    <td>-</td>
    <td>User</td>
  </tr>
  <tr>
    <td>20:500</td>
    <td>0:100</td>
    <td>Tkey_req</td>
    <td>Send /postkeys request</td>
    <td>Lkey_req</td>
    <td>App</td>
  </tr>
  <tr>
    <td>20:600</td>
    <td>0:500</td>
    <td>Tkey_prc</td>
    <td>Process /postkeys request</td>
    <td>-</td>
    <td>Server</td>
  </tr>
  <tr>
    <td>21:100</td>
    <td>0:150</td>
    <td>Tkey_rsp</td>
    <td>Send /postkeys response</td>
    <td>Lpst_rsp</td>
    <td>Server</td>
  </tr>
  <tr>
    <td>8021:100</td>
    <td>0:150</td>
    <td>Tp14_req</td>
    <td>Send /postkeys request for 14th TEK; 8000s later in this example</td>
    <td>Lp14_req</td>
    <td>App</td>
  </tr>
  <tr>
    <td>8021:250</td>
    <td>0:200</td>
    <td>Tp14_prc</td>
    <td>Process /postkeys request</td>
    <td>-</td>
    <td>Server</td>
  </tr>
  <tr>
    <td>8021:450</td>
    <td>0:100</td>
    <td>Tp14_rsp</td>
    <td>Send /postkeys response</td>
    <td>Lp14_rsp</td>
    <td>Server</td>
  </tr>
</table>


 

Additional scenarios to take into account are:

* When the user opens the key upload screen, the app does a /register call and caches the labConfirmationID and the confirmationKey until the expiration of these, at 6AM next morning. If the user opens the screen more than once on a day and eventually uploads his keys, the time between the /register call and the /postkeys call may be much longer than some seconds or minutes.

* The first call (to /register) might fail due to communications failure or server failure, resulting in one or even more retries.

* The second call (to /postkeys) might fail due to communications failure or server failure, resulting in one or even more retries.

* The user may change his mind about uploading his keys after the /register call has been made. Then the /postkeys call will not be invoked.

# Equal request and response message lengths

An observer can detect and analyse all the Txxx_xxx request and response times, and all the Lxxx_xxx message lengths in the above table.

A first measure to mitigate traffic analysis is:

All calls to /register and /postkeys will have equal request and response sizes, by using padding.

* Lreg_req must be equal to Lpst_req

* Lreg_rsp must be equal to Lpst_rsp.

This requires both the apps (Android and iOS) and the back end server to agree upon the request and response message lengths for the /register and /postkey implementations, so that the request and response payload will always fit.

As a consequence of this measure, an observer cannot distinguish between /register calls and /postkeys calls.

# TEK upload observation

Without any decoy traffic, an observer can deduce the following:

* 0 calls on a day: no upload 

* 1 call on a day and no call on day after: no upload

* 1 call on a day and 1 call on following day: possible upload (/register may have been called twice; or genuine /register and a delayed /postkeys after Midnight)

* 1 call on a day and 2 calls on following day: possible upload (/register may have been called twice; or /register, 1st /postkeys delayed until after Midnight, and another /register)

* 2 calls on a day with interval < 5 min: very likely upload (/register and /postkeys have been invoked, or /register has been retried, which is unlikely)

* 2 calls on a day with interval > 5 min: very likely upload (/register and /postkeys have been invoked, and for some reason the user opened upload screen twice or more that day.)

* more than 2 calls in a day: very likely upload (/register and/or /postkeys has been retried. /postkeys might have failed, but unlikely.)

## Combination of TEK upload and phone call

Another datum that may be observed is that during a genuine TEK upload (/register and /postkeys calls) is having a phone call with the health care worker. If this is a VoIP call, this will be visible on the WiFi network. If this is a cell phone call, its call metadata could be analysed by e.g. corporate device management software, or by a nearby observer.

Decoy traffic as described below will make this analysis less valuable.

# Decoy traffic design

We take two measures to introduce decoy traffic:

1. Generate decoy traffic when user opens upload screen, to make genuine and simulated TEK uploads indistinguishable

2. Generate simulated TEK uploads on random days.

These two measures are described in the following two paragraphs.

## Upload screen decoy traffic

Whenever a user opens upload screen and /register is invoked:

1. Schedule a first decoy message after random (5 .. 120) seconds (emulating a real upload)

2. Determine random number of additional decoy messages to send: rand (0 .. 3), negative exponential distribution: 0 additional decoys very likely, 3 additional decoys very unlikely, e.g. probability 1/10000.

3. Schedule those randomly across the rest of the day, until Midnight.

4. If the real upload (call to /postkeys) is done, cancel the first remaining decoy message if it has not been sent yet.

This ensures that number of calls done across the day does not reveal whether a real upload has been done, except when all of the following conditions hold: 

* the user has opened the upload screen twice or more times this day

* the user did a real upload

* the random number of decoy messages has been set at 3 (chance of 1/10000)

* the real upload occurred after the 10 random decoy messages have been sent, towards the end of the day.

This scenario is unlikely, but it is possible. The risk that this scenario occurs and an observer thus can detect a real key upload, has to be accepted.

## Random simulated TEK upload

In terms of decoy traffic volume, we will generate about 20 times the expected genuine TEK upload traffic. This is a number that we expect will hide the genuine traffic well, and at the same time does not burden the back end servers too much.

Let us calculate how much this 20x decoy traffic will be on average.

The daily infection numbers of the pandemic over the course of March to June 2020 is shown in the following graph (retrieved 22 Jun 2020 from [https://www.rivm.nl/coronavirus-covid-19/grafieken](https://www.rivm.nl/coronavirus-covid-19/grafieken)).

![Graph Dutch daily infection numbers march-june 2020](images/image_0.png)

So for the calculation of the required decoy traffic volume let us take a number of daily infections of 1000. 

And let us assume (optimistically) that half of these infected persons use the app and are willing to do the key upload. That would mean 500 genuine uploads per day. And that would mean 20x 500 = 10000 decoy uploads per day.

From our assumption that half of the infected persons use the app it follows that about half of the Dutch population has the app, which is about 8M app users.

This means that each app installation needs to generate 10000 / 8M = 0,00125 average decoy TEK upload sequence per day, or 0,0375 per 30 days. Let us round this to 4% chance per 30 days.

Note that the total amount of decoy traffic is proportional with the number of app users and with the key uploads that those app users perform. So a lower number of app users will maintain the 20:1 rate of decoy traffic versus genuine traffic.

The decoy traffic is then generated by the app as follows:

1. Each 30 days, determine whether a decoy traffic sequence is scheduled with a probability of X. X is a configuration parameter that is part of the server’s /appconfig response. X is a decimal between 0 and 1. The default value of X when the app has not successfully retrieved /appconfig yet, is 0.04 (the aforementioned 4%).
So take a number R = random(0..1) and only if R < X, continue with the next step.
Otherwise, stop this procedure and wait for the next round (in 30 days).

2. Pick a random day in the next 30 days to schedule the decoy upload sequence.
Note: we do not take into account Sundays and holidays that may have lower genuine upload traffic. We simply distribute randomly over 30 days.

3. Pick a random time between 7AM and 7PM to schedule the decoy sequence start.

4. Schedule a decoy TEK upload sequence for this random day and time.

Each decoy TEK upload sequence must behave as follows:

1. The decoy TEK upload sequence mimics the decoy traffic sequence as described in the previous paragraph, including the 14th TEK upload the following day..

2. The app maintains a counter DECOYCOUNTER of decoy traffic sent during the day by the scheduled decoy TEK upload sequence. This counter is set to 0 at the start of each decoy TEK upload sequence. Each time a decoy message is sent, the counter is increased by 1.

3. Whenever the user opens the upload screen and the /register call is made as described in the previous paragraph, AND decoy traffic is scheduled for this day:

    1. any remaining scheduled decoy traffic for this day is canceled.

    2. the decoy traffic to be scheduled after this genuine /register call is subtracted with DECOYCOUNTER.
