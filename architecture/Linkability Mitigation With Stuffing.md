# Linkability Mitigation with Key Stuffing

Users upload multiple keys to the server. While individual TEKs are already unlinkable, linkability could be achieved if the volume of keys is low. E.g. if a user typically uploads 14 keys and only one user would upload keys, the batch would contain only those 14 keys and that makes the keys linkable. 

To avoid such linkability, batches should be stuffed with random keys if they do not contain sufficient keys. 

A TEK is 128 bits (16 bytes). Therefor there are 2^128 = 340.282.366.920.938.463.463.374.607.431.768.211.456 possible keys. The chance of a random key actually having a match to a real key is equal to the chance of two devices generating the same key. The chance of this happening is sufficiently low to not worry about collisions.

## Stuffing rules

To be able to adjust the level of stuffing, we introduce a configuration value **eksMinimumBatchSize**. If, after collecting the amount of keys to put into the next batch, the number of real keys is lower than eksMinimumBatchSize, additional random keys will be generated to stuff the archive. 

We will use 150 as initial value for eksMinimumBatchSize (roughly 10-15 times a single key set).

Note that we also define an **eksMaximumBatchSize** as suggested by Apple and Google. If the number of keys exceeds a batch, the EKS engine will split the keys into multiple files belonging to the same batch. It is not necessary to stuff the keys in that case, as the keys are randomized across all files belonging to the batch, and even if the last file of the batch is small, no linkability can be derived from it.

## Realistically random keys

To make it hard to distinguish real keys from stuffed keys, the random keys should be generated using a secure random number generator.

## Random key sort

To make sure keys can't be linked by the order in which they appear, all keys (real and stuffed) should be sorted randomly. Since TEKs are generated randomly, an alphabetic sort on the key itself will achieve this.
