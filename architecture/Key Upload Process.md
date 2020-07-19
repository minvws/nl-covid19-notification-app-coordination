# Key Upload Process

## Background

Up until the GAEN 1.4 version of the framework, retrieving keys from the framework was 'idempotent': every call on the same day would retrieve the same set of keys, and wouldn't change the underlying keys. As of GAEN 1.5 this has changed, in two ways:

* Same day TEK release: The TEK of today is no longer embargo'd until after midnight. To avoid replay attacks by distributing keys that are still 'active', servers should now implement this embargo. 
* TEK reset upon retrieval: If an app reads the TEKS from the device, a new TEK is generated for the device. Caveat: although this new TEK only really is active from the time of the reset, for backward compatibility the 'rolling period' of the TEK is still set at midnight. The interval is shortened though. This essentially mean that you can't derive from a TEK what the real moment was it started to become active. 

These changes lead to a reconsideration in the way we upload keys. Up util 1.4 we simply re-uploaded the last 14 days of keys upon pressing the 'upload' button. Even if the user would repeat the process a number of times, the same set of keys would end up on the server, and when our Public Health AUthority confirms a positive test, that same set of keys would simply be published. As of 1.5, using this same methodology would grow the set by 1 key upon every upload, causing trouble in decoys and additional network traffic, so we have to make a few changes to the way we upload keys.

The updated process should follow the following guiding principles:

1) Security: we should avoid publishing TEKs that may still be active (this could be caused by the device still echo'ing it, due to clock skew)
2) Usability: we should minimize traffic for the user. 
3) Minimize impact: our timelines do not allow a major change in the way we do things.
4) Backward compatibility: Google introduced a 'switch' that allows them to change the TEK release without rolling out a software update. This means our app will have to work with both values of the switch. Caveat: there is no 'getVersion' in the GAEN framework nor a way to see the value of this switch, so the app will have to derive this from the behavior. 

## Proposed process

1. Every time the user requests an upload, the device uploads the keys of the past 14 days MINUS keys that it has already uploaded for the current bucket. (I.e. device keeps trac of keys it has already uploaded to the current bucket)
2. This 'keeping track' does not need to cache the actual key data, just some metadata, so we don't accidentally create a key store outside GAEN.
3. If after reading the keys from GAEN it appears we do not have today's key, we know we are dealing with a 1.4 device (or 1.5 and google has its switch turned off), and we schedule an upload after midnight of the last key.
4. On the server, a bucket silently discards **same day** keys that are uploaded x minutes after the GGD entered the confirmation code. (previous day keys don't have to be discarded). This ensures that in the case of an 1.4 device the key is accepted (it arrives after midnight and is therefor not a 'same day' key). It also ensures in the case of a 1.5 device that it only accepts same day keys from before the GGD call; thwarting any later disgruntled patient keys.

The value of x represents a trade-off. A larger number of minutes gives the user some more time to upload his same day key during/after the call. However it also gives the user more time to use this now 'known infected phone' to generate false positive encounters ('disgruntled patient syndrome'). We start x at 30 minutes, but it should be configurable. Because the server accepts previous days keys but not same day keys, the worst case scenario is: the user uploads too late and we miss the last same day key.


## Edge case validation

### User that's on GAEN 1.4 or 1.5-with-off-switch

Assume the following sequence of events, which contains all edge cases in a single flow. We identify keys by a day number and sequence (e.g. K0908.1 is generated on september 8, and is the first key of that day. Note that real keys don't have this identifier but it helps see what happens.

### September 1-13
> 00.00h    Device generates K0901.1 through K0913.1

## September 14
> 00.00h    Devices generates K0914.1
>           The device has 14 days of keys and an active confirmationKey/labConfirmationID set  
> 10.00h    User uploads a set of keys because of fiddling with the upload button 
            Note: the device doesn't know if the user has a GGD call at this point, so it should honor the upload
            Device now generates K0914.2 because of tek reset.
> 23.59h    No further events this day, no call from GGD.

### September 15
> 00.00h    Device generates K0915.1 and deletes K0901.1 (too old)
> 10.00h    Users uploads a set of keys again.
            Device generates K0915.2 
            
