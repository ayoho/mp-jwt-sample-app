# mp-jwt-sample-app

Welcome to the MP JWT Sample Application! This application demonstrates some of the capabilities of the MicroProfile JWT specification, specifically the new features of the MP JWT 1.2 specification and the corresponding feature in the Open Liberty application server.

- [MP JWT 1.2 features](#mp-jwt-12-features)
- [Project overview](#project-overview)
- [Building the application](#building-the-application)
- [Running the application](#running-the-application)
- [Interacting with the application](#interacting-with-the-application)

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

This property allows you to create list of allowable audience values. At least one of these must be found in the `"aud"` claim of the JWT. Previously, this had to be configured in the server.xml file. Now you can configure the audiences in the MicroProfile Config property as follows:

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

This property allows you to control the public key signature algorithm that is supported by the MP JWT endpoint. The default value is `"RS256"` if not specified. Previously, this had to be configured in the server.xml file. Now you can configure the public key algorithm used for verification of the JWT in the MicroProfile Config property as follows:

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

To authenticate requests to this endpoint, we include a JWT in the request. 

### Invoking with a bearer token

Invoking the protected resource with a bearer token is done like so:

```
GET /endp/echo HTTP/1.1
Host: server.example.com
Authorization: Bearer <JWT>

input=Hello
```

Here `<JWT>` is substituted for an appropriate JWT string. Depending on the application configuration, the JWT must be in either JWS or JWE format. This also assumes that the application is configured to look for the JWT in the Authorization header.

### Invoking with a cookie

Invoking the protected resource with a JWT in a cookie is done like so:

```
GET /endp/echo HTTP/1.1
Host: server.example.com
Cookie: <cookie-name>=<JWT>

input=Hello
```

Here `<cookie-name>` is substituted for the expected cookie name and `<JWT>` is substituted for an appropriate JWT string.

### Application response

A successful endpoint response should be an `HTTP 200` and something like the following response body:

```
Hello, jdoe@example.com
```
