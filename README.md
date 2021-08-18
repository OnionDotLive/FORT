# FORT
Fingerprinted Onion Response Tag [FORT] is a new HTML based standard implemented by onion.live and several darknet services to fight phishing attacks.
Why FORT?
As many of us know various implementations exist to achieve the same objective which is fighting phishing mirrors/hostnames.

Most of them rely on using a signed mirror file that's either stored in a separate .txt file on the webserver or behind a captcha. We have 2 problems with that:

a. They are not standardized. Every site implements its own captcha or uses a different algorithm to sign the key. with no central key authority. This can be easily faked as we have analyzed several phishing mirrors with forged signatures. This results in poor user experience and makes it even harder for programmers to automate the verification process.

b. TOR is slow as a network. If several requests have to be made in order to reach the signature and the public key. Crypto-verification would be more of a roadblock than a useful cryptographic analysis tool.

FORT is a non-executable HTML <script> tags that are designed to be present in the head section of the HTML itself. It provides a way of verifying the page without additional requests while allowing for standardization to take place so developers can build tools and systems on top of it.

How does FORT work in theory?
A script tag [/mirrsig] containing a detached clear signed message which is a newline-separated list of all valid hostnames or just the hostname that the user is visiting.

Another script tag [/rsakey] containing a LONG fingerprint keyid of the public key used to sign the mirrors list.

A script that performs the verification will extract the clear signed message, extracting that key (hkey) and making sure the signed message is valid.

If the signed message is valid it would proceed to match the key with a previously stored/cached fingerprint for the same site.

If both fingerprints matched it would then move to the final step which is extracting the hostnames and making sure the current host the user is visiting exists in that list.

Supported Algorithms:
Currently, we only support SHA256 and SHA512. We may add support to other Algorithms in the future.

Implementing FORT:
1. Create your key if you don't have one. Enter your email address and your organization name:

gpg --gen-key

2. Identify your key LONG fingerprint ID:

gpg --keyid-format LONG --list-secret-keys your@email.address

Output should look like this:

sec rsa3072/XXXXXXXXXXXXXXXX 20xx-xx-xx [SC] [expires: 20xx-xx-xx] YYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYY uid [ultimate] Organization Name <your@email.address> ssb rsa3072/XXXXXXXXXXXXXXXX 2020-xx-xx [E] [expires: 20xx-xx-xx]

The fingerprint ID is the second line, the string in place of YYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYY.

3. Publish your key to keyservers. You can use torsocks to hide your real IP when publishing.

torsocks gpg --keyserver pgp.mit.edu --send-key YYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYY

You can use other keyservers as well.

torsocks gpg --keyserver hkp://ha.pool.sks-keyservers.net --send-key YYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYY

torsocks gpg --keyserver keyserver.ubuntu.com --send-key YYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYY

4. Create a file containing all valid hostnames to your service separated by newlines. Note: sites with anti-DDoS mirrors rotation landing pages shall only list the addresses of the anti-DDoS gateway on their landing pages. Beware that for your backend rotated mirrors you might only list the mirror the user is visiting. Thus you benefit from protecting your users without exposing your backend mirrors to scripts/bots.

echo -en "xxxxxxxxxxxxxxxx.onion\nyyyyyyyyyyyyyyyy.onion\nzzzzzzzzzzzzzzzz.onion" > mirrors

5. Sign your mirrors file:

gpg --clearsign mirrors

This should result in a mirrors.asc file with the contents similar to this:

-----BEGIN PGP SIGNED MESSAGE-----

Hash: SHA512

xxxxxxxxxxxxxxxx.onion

yyyyyyyyyyyyyyyy.onion

zzzzzzzzzzzzzzzz.onion

-----BEGIN PGP SIGNATURE-----

iQGzBAEBCgAdFiEEeHblV6ahopB1Gx32qNpRORtkFdQFAl6CeQoACgkQqNpRORtk FdRh/AwAsIJLeAuuCT/YF1jn8vK4LZEdjtG7PXsn7wa9Q4HvR1gFd8QvgfEd2QWM qvHVhSEBCGFwVFD7yyEaxi5Hku/tlls15YKXI0QNGRXJvxqnZF41Mz/XoXF4onT3 RoFZ4Bx9jtlSIFM/KnQGJdCKZvNN2gvdYdp/3uBEnGCvnD+T5OcUZ5ss2Zp9g9Rq llrI9VRTrfIO5Lgvg/W7WRNlN4yjkWNlOy9HoOacELKQEd/cLTdqm2tmzhrXO0h4 ljdGcvzOzD8pnmglxCrYklinLDgypws4RincWc5U60GSsvT2gVIx/lKgDy2CffK7 jjq3f2pvSZQtLyX142oYTc5o2YsprhQHuG1TiCZjZ+tJJxkHxn5+NP2NmHVfFG8V YvX9xZNma/sd50eZS6tBA3bxQAH/QSkYapEz+zziymKiw5ZWSkxd96qtVf6cJTG2 DHJGXye+H+atmX6qOAQl5V64HoRI6LfneCOprLnHL6q4MGTYxKLRJ9OqBBEk2WRc cbCBGP8I =Z5vk

-----END PGP SIGNATURE-----

Note: Remove any comments or unnecessary output in the signed message if found.

6. Place your long fingerprint ID in the rsakey tag and insert that anywhere in your HTML [Preferably the section]:

<script type="text/rsakey">YYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYY</script>

7. Place the contents of mirrors.asc in the mirrsig tag and insert that anywhere in your HTML [Preferably the section]:

<script type="text/mirrsig">-----BEGIN PGP SIGNED MESSAGE----- ...</script>

8. Final output should look something like this: 

A good example could be seen live at onion.live - View page source

Anti-DDoS Landing Pages:
If your site implements landing page addresses (we call those rotation gateways) with rotated backend mirrors and you worry about exposing your backend mirrors then you'd need two different signatures, one for all rotation gateways and the other contains only the backend mirror your user is visiting in the browser.

Example:

Assuming you have the following gateway mirrors:

gateway1.onion

gateway2.onion

gateway3.onion

They would all share the same signature like this:

-----BEGIN PGP SIGNED MESSAGE-----

Hash: SHA512

gateway1.onion

gateway2.onion

gateway3.onion

-----BEGIN PGP SIGNATURE-----

And assuming you have the following backend mirrors:

backend1.onion

backend2.onion

backend3.onion

Each one of them should have its unique signature with only its hostname present in the mirrsig tag.

They can as well share the same signature that contains a list of all backend mirrors. Both approaches are valid and depend on your site architecture and how far you are willing to go to isolate mirrors addresses.

Contribution:
Feel free to contribute to this standard. We're open to your suggestions and inquiries on admin@onion.live
