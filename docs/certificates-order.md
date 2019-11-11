Certificates and their order of deployment
---

One common question that developers ask, "What is the sequence of specifying certificates, when using nginx?" and "Does the order really matter?"

Should it be the `rootCA`, followed by `intermediate CA` and then the actual certificate? Or the other way around?

The order in which the request is verified by server is, the incoming request certificate signature is first verified with the issuer CA, and then the verification of the signature continues down the chain, till the rootCA signature is verified.

Now if the order is reversed, then verification of signature is out of order, and will cause browsers, mobile apps, desktop apps to fail.

So always make sure when the certificates are place on the nginx server, or any other webserver, the sequence matches:
**SERVER CERTIFICATE > INTERMEDIATE CERTIFICATE/S > ROOT CA**

For more info:
https://www.digicert.com/ssl-support/pem-ssl-creation.htm
https://knowledge.digicert.com/solution/SO16297.html
