# App termination plan

At some point in the future a decision will be taken that the CoronaMelder system be taken out of service. This involves:
- communicating this to the general public.
- remotely deactivating the app functions on the devices.
- Deactivating the server infrastructure.
This document describes the technical design of the latter two actions.

## Remotely disabling the app functions on the Google and Apple devices
The apps periodically refresh their `Appconfig` from the server (as described in https://github.com/minvws/nl-covid19-notification-app-coordination/blob/master/architecture/api/apispec.yaml). This happens several times a day, as a background process of the app.
If this file contains a key titled `coronaMelderDeactivated` with value `deactivated`, the following actions occur in the app:
- When user opens the app, a blocking screen is presented.
- The Exposure Notification framework is disabled.
- All scheduled background tasks are stopped. As a consequence, no further notifications will be triggered.

So to disable the apps on the devices, the Appconfig key `coronaMelderDeactivated` with value `deactivated` needs to be added to the Appconfig on the central server infrastructure and published to the CDN (Content Delivery Network). 

## Deactivating the server infrastructure
In addition to the previous server configuration change, the following actions need to be taken at the server side:
- The periodic background job for processing and publishing Exposure Key Sets is taken out of service.
- The TEK upload endpoints ("/register", "/postkeys", "/stopkeys") are modified to do no processing, and to return an HTTP code 410 ('gone'). Note that this may be done by modifying the application code, or in a load balancer or firewall that precedes the application servers.
As a consequence, if a user tries to perform a TEK upload, the app will show an end-of-service message. 
(Attempts to upload keys at app end-of-life should occur rarely, since as soon as the app receivces the `coronaMelderDeactivated` key described previously, the TEK upload function will be disabled.)
- The periodic refresh job from the European Federation Gateway is taken out of service.
- The CDN needs to remain active for some time, in the order of 30 days to allow all app installations nation-wide to retrieve the Appconfig with the `coronaMelderDeactivated` key.
- All server infrastructure is decommissioned.
