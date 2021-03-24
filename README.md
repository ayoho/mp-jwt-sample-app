# mp-jwt-sample-app

Welcome to the MP JWT Sample Application! This application demonstrates some of the capabilities of the MicroProfile JWT specification, specifically the new features of the MP JWT 1.2 specification and the corresponding feature in the Open Liberty application server.

- [Run it now](#run-it-now)
- [MP JWT 1.2 features](#mp-jwt-12-features)
- [Project overview](#project-overview)
- [Application overview](#application-overview)
- [Building the application](#building-the-application)
- [Running the application](#running-the-application)
- [Interacting with the application](#interacting-with-the-application)
- [Samples](#samples)

Much of the source for this sample application (e.g. Java classes and PEM files) is taken from the [MicroProfile JWT TCK project](https://github.com/eclipse/microprofile-jwt-auth/tree/master/tck) under the Apache 2.0 license.

## Run it now

1. Clone this repository
1. In the project root, run
    1. `$ mvn package`
    1. `$ mvn liberty:dev`
1. Interact with the app
1. Type `Ctrl + C` to stop the server.

## MP JWT 1.2 features

The MicroProfile JWT specification allows using a JSON Web Token for authenticating and authorizing requests to a service. MicroProfile JWT 1.2 adds some new capabilities that were not in previous releases:

- [New MicroProfile Config properties](#new-microprofile-config-properties)
- [Support for JSON Web Encryption (JWE) tokens](#json-web-encryption-jwe-tokens)
- [Enhanced signature algorithms](#enhanced-signature-algorithms)

### New MicroProfile Config properties

Version 1.2 of the MicroProfile JWT specification adds the following MicroProfile Config properties to control different aspects of JWT validation:

- [mp.jwt.token.header](#mpjwttokenheader)
- [mp.jwt.token.cookie](#mpjwttokencookie)
- [mp.jwt.verify.audiences](#mpjwtverifyaudiences)
- [mp.jwt.decrypt.key.location](#mpjwtdecryptkeylocation)
- [mp.jwt.verify.publickey.algorithm](#mpjwtverifypublickeyalgorithm)

#### `mp.jwt.token.header`

This property allows you to control the HTTP request header that is expected to contain a JWT. You can specify either `"Authorization"` (default) or `"Cookie"`.

#### `mp.jwt.token.cookie`

This property allows you to specify the name of the cookie that is expected to contain a JWT. The default value is `"Bearer"` if not specified.

**Note:** The `mp.jwt.token.header` MP Config property must be set to `"Cookie"` for the `mp.jwt.token.cookie` property to be recognized.

#### `mp.jwt.verify.audiences`

This property allows you to create list of allowable audience values. At least one of these must be found in the `"aud"` claim of the JWT. Previously, this had to be configured in the `server.xml` file. Now you can configure the audiences in the MicroProfile Config property as follows:

```
mp.jwt.verify.audiences=conferenceService,adminService
```

#### `mp.jwt.decrypt.key.location`

To support JSON Web Encryption tokens, this property allows you to specify the location of the Key Management Key. This is the private key that is used to decrypt the Content Encryption Key, which is then used to decrypt the JWE ciphertext. The private key must correspond to the public key that is used to encrypt the Content Encryption Key.

```
mp.jwt.decrypt.key.location=/path/to/privatekey.pem
```

**Note:** When the `mp.jwt.decrypt.key.location` property is specified, the application will only accept tokens in JWE format; tokens in JWS format will be rejected. Inversely, if this property is not specified, the application will only accept tokens in JWS format.

**Also note:** The payload of JWE tokens must be a nested JWT in JSON Web Signature (JWS) format. JWE tokens whose payload is not a nested JWS will be rejected.

#### `mp.jwt.verify.publickey.algorithm`

This property allows you to control the public key signature algorithm that is supported by the MP JWT endpoint. The default value is `"RS256"` if not specified. Previously, this had to be configured in the `server.xml` file. Now you can configure the public key algorithm used for verification of the JWT in the MicroProfile Config property as follows:

```
mp.jwt.verify.publickey.algorithm=ES256
```

### JSON Web Encryption (JWE) tokens

JSON Web Encryption tokens (defined in [RFC 7516](https://tools.ietf.org/html/rfc7516)) represent "encrypted content using JSON-based data structures". They are JWT structures that have encrypted payloads, and therefore offer more security over JWS structures whose payloads are simply Base64-encoded.

### Enhanced signature algorithms

The MP JWT 1.2 specification adds support for the ES256 signature algorithm, while Open Liberty also adds support for using the RS384, RS512, HS384, HS512, ES256, ES384, and the ES512 signature algorithms.

## Project overview

The project makes use of the following technologies:

- Maven
- JAX-RS
- [Eclipse MicroProfile](#eclipse-microprofile)
- [Open Liberty](#open-liberty)
- [Liberty Maven plugin](#liberty-maven-plugin)

### Eclipse MicroProfile

[Eclipse MicroProfile](https://microprofile.io/) is an Enterprise Java environment tailor-made for microservice architectures. The project is a collection of various specifications for technologies such as CDI, Metrics, Health, JAX-RS, JWT, Fault Tolerance, and others.

### Open Liberty

[Open Liberty](https://openliberty.io/) is IBM's open source Java application server. The server is a lightweight, open framework specifically designed for cloud-native environments. The product is [freely available on GitHub](https://github.com/OpenLiberty/open-liberty).

### Liberty Maven plugin

The [Liberty Maven plugin](https://github.com/OpenLiberty/ci.maven) allows you to manage a Liberty server as part of your Maven build lifecycle. This sample application makes use of the plugin to package the app and deploy it onto a Liberty server with as little as one command.

## Application overview

The highlights of the application are:

- [JAX-RS resource](#jax-rs-resource)
- [Server configuration](#server-configuration)
- [MP Config source](#mp-config-source)

### JAX-RS resource

The JAX-RS resource is a Java class that defines the endpoint and security constraints for that resource:
```java
@Path("/endp")
@DenyAll
@RequestScoped
public class RolesEndpoint {

    @GET
    @Path("/echo")
    @RolesAllowed("Echoer")
    public String echoInput(@Context SecurityContext sec, @QueryParam("input") String input) {
        Principal user = sec.getUserPrincipal();
        return input + ", user="+user.getName();
    }
}
```

The configuration of the resource is pretty simple. In a nutshell, this resource provides a `/endp/echo` endpoint that serves HTTP `GET` requests to users in the `"Echoer"` role. If authorized, the endpoint returns a string that contains the Principal name of the authenticated user alongside the value of the `"input"` query parameter that is sent in the request. If the user is not authorized, a 401 will be returned.

With the `mpJwt-1.2` or `microProfile-4.0` feature enabled, authorization will be determined based on the MP JWT configuration in the server and application. Any authorized request to this endpoint must therefore include a valid JWT in accordance with the MP JWT and application configurations.

### Server configuration

With Open Liberty, the server configuration isn't much more than enabling a feature:

```xml
<featureManager>
    <feature>microProfile-4.0</feature>
</featureManager>
```

This is the only server configuration typically necessary to use MP JWT 1.2 functionality. The `mpJwt-1.2` feature can also be specified on its own if the other MicroProfile 4.0 features aren’t needed.

### MP Config source

The MP Config source contains the remaining configuration properties needed for the application to work:

```
mp.jwt.decrypt.key.location=/privateKey.pem
mp.jwt.verify.audiences=s6BhdRkqt3,testing
mp.jwt.verify.publickey.location=/publicKey.pem
mp.jwt.verify.issuer=https://server.example.com
```

In our case, these properties are defined in the `META-INF/microprofile-config.properties` file.

The [New MicroProfile Config properties](#new-microprofile-config-properties) section provides some detail about these properties and their usage.

## Building the application

This is a Maven application, so building it should simply require the `mvn package` command:

```
$ mvn package
```

## Running the application

The application can be run a number of ways using the Liberty Maven plugin:
- [Dev mode](#dev-mode)
- [Liberty in the background](#run-liberty-in-the-background)
- [Liberty in the foreground](#run-liberty-in-the-foreground)

### Dev mode

Dev mode allows you to make hot changes to your application code while the app and server are still running.

To run dev mode, run the `liberty:dev` goal:

```
$ mvn liberty:dev
```

This goal will compile and package your application, download an Open Liberty environment if this is the first time running the command, deploy the application onto the server, and start the server.

To use the hot change capabilities of the plugin, all that is required is to modify and save a file within the application. As soon as the file is saved, the Liberty Maven plugin will recompile, repackage, and redeploy the app onto the running server. This is all done automatically without the need for any other interaction other than saving the file. This includes changes to Java source, MP Config source files, Liberty server config, etc.

This command will run the Liberty server in the foreground of the command line interface. To stop the server and exit dev mode, type `Ctrl + C`.

### Run Liberty in the background

If you want to start the Liberty server in the background so you can continue to use your existing shell, you can run the `liberty:start` goal:

```
$ mvn liberty:start
```

Once the server is started, the `liberty:stop` goal must be run to stop the server:

```
$ mvn liberty:stop
```

### Run Liberty in the foreground

The `liberty:run` goal will run the Liberty server in the foreground of the current shell:

```
$ mvn liberty:run
```

This is similar to the `liberty:dev` goal, however any changes to the application will not be picked up and deployed automatically. If you make any changes to the application, you will need to use separate Maven commands to repackage and redeploy the app.

Similar to the `liberty:dev` goal, type `Ctrl + C` to stop the server.

## Interacting with the application

The JAX-RS resource in this application defines a `/endp/echo` endpoint. The endpoint has the following constraints:

- Only serves HTTP `GET` requests
- Requires authentication
- Authenticated user must be in the `"Echoer"` role

By default the application will be available at `http://localhost:9080/endp/echo`.

To authenticate requests to this endpoint, we include a JWT in the request. You can find [sample JWTs to try out with the application in the Samples section](#samples).

**Recommendation:** I recommend interacting with the application using [Postman](https://www.postman.com/). It provides a pretty clear UI for executing HTTP requests, and lets you easily manipulate headers, parameters, and the like. You could also interact with the application in a browser or cURL, depending on your preference.

### Invoking with a bearer token

Invoking the protected resource with a bearer token is done like so:

```
GET /endp/echo HTTP/1.1
Host: localhost:9080
Authorization: Bearer <JWT>

input=Hello
```

Here `<JWT>` is substituted for an appropriate JWT string. Depending on the application configuration, the JWT must be in either JWS or JWE format. This also assumes that the application is configured to look for the JWT in the Authorization header.

### Invoking with a cookie

Invoking the protected resource with a JWT in a cookie is done like so:

```
GET /endp/echo HTTP/1.1
Host: localhost:9080
Cookie: <cookie-name>=<JWT>

input=Hello
```

Here `<cookie-name>` is substituted for the expected cookie name and `<JWT>` is substituted for an appropriate JWT string.

### Application response

A successful endpoint response should be an `HTTP 200` and something like the following response body:

```
Hello, jdoe@example.com
```

## Samples

You can use these sample JWTs to try out requests to the protected resource.

**Note:** These JWTs will be considered expired by the Liberty server without modification. If you want to use these JWTs, you will need to add the following to the `server.xml` file:

```xml
<mpJwt id="myMpJwt" clockSkew="99999h" >
```

Doing so will allow old tokens to be allowed through (or at least not rejected because they were issued too long ago).

Sample JWE:
```
eyJjdHkiOiJKV1QiLCJlbmMiOiJBMjU2R0NNIiwiYWxnIjoiUlNBLU9BRVAifQ.LgNRqCK6_IyB2gvCGehNx2GMivq1GP5zPeyTJHNvgaNBAKCF2C1Rv8gtibzbHTrVV6zSiwS_beyQJ3rY0t3fb_cfZu1zEstPJ5xO3YIJPOBP0kHniLr49PC5YYwBMcoH3pmnmE11w5_nbHHvSiHnrUT6OCOfZIfVqi0Qw5wy_Cd3W_AMbr1k0ZOEna3lfq8Lyi9vKNxhyBhtNeVcY6MFBe6bJTYDstPQG1QVu6T-NNIp7YyIcfxzNUjvQTH2OCzrGCnb9OyUsz0CxuzKK4yBdSoaE0y_LDHDhiSijQdz9pXl8Bvlqs5fCXbrZxH842sePhV4f21S0GC1Fr9VZCWrvQ.d0vXIob289IEZwMX.nnXIEjQCWPZ3-NPy1O3rzlv79Q4-OPi3JAM3DRT_33E1NFoFYJRLzqqTYwP9r8tcUVNDwgsK0IJ7LBMCvhuyBFFzUwcx9btzr-cKIhQDNl9lG9YuKS50JaRXc3kfi7NaTL4alJuwRUS4ZcvVu637ldFqssfWWqzrd78GIt7dz9YqYAIySjSbGHRzslcKaJjPBS3SjtmfeYEiY2e-cXyBsGusvBMpuOxRKbfKuIQn0CFrP5U-dPvxDQdH09sPHkMaQqGgJHs0bvR8RXTVPn_7UpNmi4cfLbOs6tJ9MqKk5g2Ar8nyWTpYYahAjk77kLEeLXhiMbJPoIQzDJ4dOGLHr06lIGrKjTaSi660BzObSO4-DMX42WKxU7CGlITgpQQp04fw3LRXp56ovhcy0B4vVDqpl12Fo3D5SEE6itiT50cwJnCYRrjxSuJsNaTawCF4hUrQ3WNeM64q2zEeI3Qh5Vsob3gy1dEdDnmikGpdowDNGxYNzrKwqQZc6WMU7_s-Fakj4GhJloxxfj0ulokPlxrRyLh9mdPRbGVDi6RMOjRAdGS2JLEpZMAYFyaTPrE37wJAOuxQ1bUKKzr1QJrcLn4MFtGH8gsJ7aTmbV0Ly1WF1UwlVD6de4vsPQZWG7HP8HbQgn7ZVSo7PddO4xtnTWaKt4TyXVw0bzXt4lw9DiupoBSyKEdxB7vhqSvXRIJ0pnxXfprmO_JXjkBcpMK98r0km0XOG4cPetiqwdMFxUieM1SSDyQUcq2n1zIPhjLOHHWCEpy8QolLu_mCl5N-U-mgpAPQIe-uUgRMGxcPAbiEf5uo-nf2eEMm_0-LnjwJGIUGRU3fKVyBfsITGihTvNRNp2jTFsa8_G-aN7V0QJWZyvWXcZuT7ec0y7XZcH4GCGfmspvk8v3k6c1tIzliSkJ_u5vHYF6E8WGi93i8YJIQFuFn9QbTXn234J9TfGvCjhjWju0_bEDYSHZFGrZffOxbTNaBX0h0TqV7NXHxAy6I2u65uFOWqr1HjQQIdqw-R2Rcvu_skGK9e_yON1RaWWF3zI-_t-1XMWapyl1P-KxyFAB7Ffo-et_S4zkfz9Uam6gxCzkrXFmvpICOyW4dyaS7uGApqYwtCMdZuSAcR7auboi_hKMKJpI8pglWvnMs5fhFaB-zJA6nc4D8xG6ITlBCOnIWHXOFVODlfA9Yb_AnF5OtSIrCQL9DLEqBWH6f2u1ughUPu9niuzOyp8_FkH_ljiI7D305FKQG9TeahyhAZb68o4REHLzxpFHYTHsuxiJnwmwRZx4LxadRyluMa59TajlP_zLA51lGAmSaWrvAXeR_2GhOr1YZw2dB_X2E9apy8wCwQwfLeXjLq_ucGnxd7Uaj__lI-1R4XwEfM2ipQKy5K3AbLnxQsUzDB2ebeDd7-mZ7rZMVGW7NSJsnqu0QCQ2gEnQM5swt5fSw6iMlDlJH-u017QcRPHV5I3e9pFaVUht36DLSudpFu2nKjNSpQ0Qr3muMS5orO2rZgMidOIBmydbiet67EOk6D0-GQY7c0wzRBHsn4ti-fQrNj_k7HJ_wtEBeZ_z4W5Yq8mkGhW0dw97yxl5_wX4rTY-7CjghFZbCqG5VqZwzdJTtN6OQKcD2rqdv_tJHqi8s8ZQK1DGgOqiHVXukJmY6c_FpmK-vlGcBcL06BasutlU8FbMVAUO0mfe7EmIgYqFsnurAmhOduz_krNFKgCVTReXs12ljiJLArbxANatkRaaUC3QhWigkwzJKtIIzmNp42Gk1CXPNA9TRggVlDfHipIrvrKbQ84rISh6X0QtL_hkvE1UPHOihLMvUt2_fSDhqyCpRPBFe53fHaUFQpklnlVfRx0_61wNuMkX4w3apBbD_daIBp2hDQPAiAFVctHIBECJx7pYRWeeXqiu67eI2oEt9iMJwAFjqQn_tLxSSG6rRNJuHiGY4Xu197dLggKA1PwWgGLfr-n472KR_qfTh-4Yo7-mgNmQCOMDzoCYwZbIks1fBA0pKfvL1A0q_AIz2k9b16Uv2BDmPDbKKoocijpFcO2XTwfIGlNqTWlJll7Ba-J5J-oUfNTVc4omPJZNAtSclQsrsqqk7E5HWOMi-YqO5NuCEzABgXnt1wGrbUEwtM--w3OockWAtZrC1AyEl-A-A.MdMK8N5TlroFsT5X9x6vMw
```

- Requires `privateKey.pem` to decrypt the JWE
- Nested JWS is signed using `RS256`
- Requires `publicKey.pem` for signature verification of the nested JWS

Sample JWS (RS256):
```
eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJodHRwczovL3NlcnZlci5leGFtcGxlLmNvbSIsImp0aSI6ImEtMTIzIiwic3ViIjoiMjQ0MDAzMjAiLCJ1cG4iOiJqZG9lQGV4YW1wbGUuY29tIiwicHJlZmVycmVkX3VzZXJuYW1lIjoiamRvZSIsImF1ZCI6InM2QmhkUmtxdDMiLCJleHAiOjE2MTY0NDY2ODksImlhdCI6MTYxNjQ0NjM4OSwiYXV0aF90aW1lIjoxNjE2NDQ2Mzg5LCJyb2xlcyI6WyJFY2hvZXIiXSwiZ3JvdXBzIjpbIkVjaG9lciIsIlRlc3RlciIsImdyb3VwMSIsImdyb3VwMiJdLCJjdXN0b21TdHJpbmciOiJjdXN0b21TdHJpbmdWYWx1ZSIsImN1c3RvbUludGVnZXIiOjEyMzQ1Njc4OSwiY3VzdG9tRG91YmxlIjozLjE0MTU5MjY1MzU4OTc5MywiY3VzdG9tQm9vbGVhbiI6dHJ1ZSwiY3VzdG9tU3RyaW5nQXJyYXkiOlsidmFsdWUwIiwidmFsdWUxIiwidmFsdWUyIl0sImN1c3RvbUludGVnZXJBcnJheSI6WzAsMSwyLDNdLCJjdXN0b21Eb3VibGVBcnJheSI6WzAuMSwxLjEsMi4yLDMuMyw0LjRdLCJjdXN0b21PYmplY3QiOnsibXktc2VydmljZSI6eyJncm91cHMiOlsiZ3JvdXAxIiwiZ3JvdXAyIl0sInJvbGVzIjpbInJvbGUtaW4tbXktc2VydmljZSJdfSwic2VydmljZS1CIjp7InJvbGVzIjpbInJvbGUtaW4tQiJdfSwic2VydmljZS1DIjp7Imdyb3VwcyI6WyJncm91cEMiLCJ3ZWItdGllciJdfX19.pN7F04bM9UTqYzF3Ex7Dd-WgX8BxzdUVdaRtqhLW90AeLw5Rmu6m6erHpGeBNmol0iPfQjYB2vxmu_Syt-5v84SR0SNeA23msrO7LPJVr63TKPkiAAVla4JjOh3wHKDwFX7GLqm4G22ht6YMRy3qhFfuEJPKJxfFv2g1XfhW5-A2Sa6tTnjMFsN50dIQNg2NuBeA8WRF383mJVSp6V30OhV2bjj-E_hHWEvnsbQiUko37Z25U_3Z2mLb82_Xz2j33S2keHTggE43n0fCeWWy-7-d4EuUzDIymdXN3VKvZd7svSolau_WWhOvqpTp0g9ITsc1nILj--yxZNeQjrLUXTfEomgrwcwnWtV3fx1UDsDOWrY3T5gorE5KILW8XXyeHZ4X7jpkXZLSBtlD7BvPq0DYrlye8jHvCjXRBgPHUdaq-jfFpexo_JweR8oNJDYL1K2megNixFnq56tiu3lnZyVHyA_ZLnWSLqPbuK6pP2NpMSk-l5g389zTHf13vAnNXgGtsGqsyCQZvp6G-qIlJRC2bi-3AWcObV4BPHFV1EBQVBiTYB6kzILd9uBPvM3vNte-zSZZOTHuzFO-pTGCef8nzKBYRyOEsXYdwlH5fBHgGxrBQi4YjJ8QN_25CqcH4_atKRJ335KXpY9QVkrV7e2FDU-J73-HrYKjXrTh87I
```
- Requires `publicKey.pem` for signature verification

Sample JWS (ES256):
```
eyJraWQiOiIvZWNQcml2YXRlS2V5LnBlbSIsInR5cCI6IkpXVCIsImFsZyI6IkVTMjU2In0.eyJpc3MiOiJodHRwczovL3NlcnZlci5leGFtcGxlLmNvbSIsImp0aSI6ImEtMTIzIiwic3ViIjoiMjQ0MDAzMjAiLCJ1cG4iOiJqZG9lQGV4YW1wbGUuY29tIiwicHJlZmVycmVkX3VzZXJuYW1lIjoiamRvZSIsImF1ZCI6InM2QmhkUmtxdDMiLCJleHAiOjE2MTY0NDY1ODAsImlhdCI6MTYxNjQ0NjI4MCwiYXV0aF90aW1lIjoxNjE2NDQ2MjgwLCJyb2xlcyI6WyJFY2hvZXIiXSwiZ3JvdXBzIjpbIkVjaG9lciIsIlRlc3RlciIsImdyb3VwMSIsImdyb3VwMiJdLCJjdXN0b21TdHJpbmciOiJjdXN0b21TdHJpbmdWYWx1ZSIsImN1c3RvbUludGVnZXIiOjEyMzQ1Njc4OSwiY3VzdG9tRG91YmxlIjozLjE0MTU5MjY1MzU4OTc5MywiY3VzdG9tQm9vbGVhbiI6dHJ1ZSwiY3VzdG9tU3RyaW5nQXJyYXkiOlsidmFsdWUwIiwidmFsdWUxIiwidmFsdWUyIl0sImN1c3RvbUludGVnZXJBcnJheSI6WzAsMSwyLDNdLCJjdXN0b21Eb3VibGVBcnJheSI6WzAuMSwxLjEsMi4yLDMuMyw0LjRdLCJjdXN0b21PYmplY3QiOnsibXktc2VydmljZSI6eyJncm91cHMiOlsiZ3JvdXAxIiwiZ3JvdXAyIl0sInJvbGVzIjpbInJvbGUtaW4tbXktc2VydmljZSJdfSwic2VydmljZS1CIjp7InJvbGVzIjpbInJvbGUtaW4tQiJdfSwic2VydmljZS1DIjp7Imdyb3VwcyI6WyJncm91cEMiLCJ3ZWItdGllciJdfX19.brc6WPUEFs4B-S7_71XnahvacXO7O1yq_wTREMJd1Gfu-Jpd9WKbcK01ephofvD7virYlhrd-JG9IUWz9RZkvg
```
- Requires the `mp.jwt.verify.publickey.algorithm` property to be set to `ES256` because `RS256` is the default signature algorithm
- Requires `ecPublicKey.pem` for signature verification
