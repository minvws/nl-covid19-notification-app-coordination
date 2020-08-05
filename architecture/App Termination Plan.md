App termination plan

At some point in the future a decision will be taken that the CoronaMelder system be taken out of service. This involves:
    - communicating this to the general public.
    - remotely deactivating the app functions on the devices.
    - Deactivating the server infrastructure.
This document describes the technical design of the latter two actions.

# Remotely disabling the app functions on the Google and Apple devices
The apps periodically refresh their `Appconfig` from the server (as described in https://github.com/minvws/nl-covid19-notification-app-coordination/blob/master/architecture/api/apispec.yaml). This happens several times a day, as a background process of the app.
If this file contains a key titled `coronaMelderDeactivated` with value `deactivated`, the following actions occur in the app:
    - When user opens the app, a blocking screen is presented.
    - The Exposure Notification framework is disabled.
    - All scheduled background tasks are stopped. As a consequence, no further notifications will be triggered.

# Deactivating the server infrastructure
The following actions need to be taken at the server side:
    - The server Appconfig key `coronaMelderDeactivated` with value `deactivated` is added to the Appconfig and published to the CDN (Content Delivery Network).
    - The periodic background job for processing and publishing Exposure Key Sets is taken out of service.
    - The TEK upload endpoints are taken out of service.
    - The periodic refresh job from the European Federation Gateway is taken out of service.
    - The CDN needs to remain active for some time, in the order of 30 days to allow all app installations nation-wide to retrieve the Appconfig with the `deactivated` key.
