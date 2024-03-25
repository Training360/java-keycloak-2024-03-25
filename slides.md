# Bevezetés

## Tematika

* OAuth 2.0 és OpenID Connect
* JWT tokenek
* Telepítés
* Indítás Docker konténerben
* Spring Boot integráció
* Spring Security integráció
* Bejelentkezés felhasználónévvel vagy e-mail címmel
* Remember me
* Saját regisztráció
* Account console
* Enable delete account
* Password policies
* User locale
* Új attribútum felvétele
* Consent
* Sessions
* Saját téma használata
* Terms and conditions
* E-mail küldés
* GitHub/Facebook/Google integráció
* Authentication flow
* User federation, LDAP
* Events
* Logging
* REST API
* Admin CLI
* Export/import
* HA

## Referenciák

http://www.keycloak.org/

Stian Thorgersen
https://www.youtube.com/watch?v=duawSV69LDI

Könyv

Stian Thorgersen, Pedro Igor Silva
Keycloak - Identity and Access Management for Modern Applications: Harness the power of Keycloak, OpenID Connect, and OAuth 2.0 to secure applications 2nd Edition
https://www.amazon.com/Keycloak-Identity-Management-Applications-applications-ebook/dp/B0BPY1RDND

Hozzá tartozó videók: https://www.youtube.com/playlist?list=PLeLcvrwLe187DykEKXg-9Urd1Z6MQT61d

# Keycloak bevezetés

# Keycloak lehetőségek

* Open Source Identity and Access Management
* Autorizáció és autentikáció rábízható (pl. felhasználók tárolása, password policy, e-mail küldés)
* Multi-Factor Authentication (MFA)
* Strong Authentication (SA), pl. OTP (One time password), WebAuthn, céleszközök használata anélkül, hogy az alkalmazást fel kéne készítenünk
* SSO, logout
* Identity Brokering
  *  OpenID Connect vagy SAML 2.0 Identity Providers
  *  Social Login
* User federation
  * Active Directory, LDAP, Relational database
* Webes admin console
* CLI és REST API
* Account management console
* Audit naplózás
* Multitenancy - több szeparált felhasználói csoport/szervezet (realm)
* Clusterelhető
* Könnyen integrálható (pl. Spring Security)
* Könnyen kiegészíthető
  * Custom authentication mechanism
  * Custom user stores
  * Custom manipulation of tokens
  * Own custom login protocols

---

## Keycloak verziók

* Keycloak 17-től kezdve Quarkus
* [https://www.keycloak.org/migration/migrating-to-quarkus](https://www.keycloak.org/migration/migrating-to-quarkus)
* Inkompatibilis változtatások
  * Sokkal egyszerűbb konfigurációs fájlok
  * Quarkus keretrendszer
    * Gyorsabb indítás
    * Futás közben nem telepíthető komponensek
    * Optimalizációs lépés az elején: saját image-et érdemes buildelni
  * Környezeti változó alapján létrehozza a kezdő felhasználót, nem kell a `add-user-keycloak.sh` script futtatása
  * Context path-ból eltávolításra került a `/auth` előtag
  * Custom provider
    * Nincs saját classpath, ezért vigyázni kell a függőségekkel
    * Újraindítás szükséges
    * Ha voltak WildFly hivatkozások benne, akkor azokat módosítani kell
  * Új Kubernetes Operator
  * `X-Forwarded-Port` headernek prioritása van a `X-Forwarded-Host` headerrel szemben
* Keycloak Adapters deprecatedek lettek
  * Helyette érdemes az adott programnyelven, keretrendszerben elterjedt szabványos OAuth 2 megvalósításokat alkalmazni. Pl. `org.keycloak:keycloak-spring-boot-starter` helyett
    Spring Security

---

## Keycloak support

* CNCF tagja: https://www.cncf.io/projects/
* Sponsored by Red Hat
* Red Hat Single Sign-On, based on the Keycloak


# Elméleti fogalmak

## OAuth 2.0

* Nyílt szabány erőforrás-hozzáférés kezelésére (Open Authorization)
* 2012 óta
* Fő használati területe a web, de kitér az asztali alkalmazásokra, mobil eszközökre, okos eszközökre, stb.
* Autorizációra koncentrál (hozzáférés erőforráshoz, pl. API-hoz), és nem az autentikációra
* Access Tokent használ arra, hogy leírja, mire van a felhasználónak jogosultsága
  * Formátumát nem specifikálja, de gyakran JSON Web Token (JWT), melybe adat menthető
* Elválik, hogy a felhasználó mit is akar igénybe venni, <br /> és az, hogy hol jelentkezik be
  * Google, GitHub, Facebook vagy saját szerver

## OAuth 2.0 szereplők

* Resource owner: aki rendelkezik a védett erőforrásokkal, és ezekhez képes hozzáférést adni (felhasználó vagy alkalmazás)
* Client: a szoftver, ami hozzá akar férni a védett erőforrásokhoz, ehhez Access Tokennel kell rendelkeznie
* Authorization Server: a felhasználó itt tud hozzájutni az Access Tokenhez sikeres autentikáció után. Két végponttal rendelkezik:
  * Autorizációs végpont: itt autentikál a felhasználó
  * Token végpont: gép-gép kommunikációnál
* Resource Server: védi és hozzáférést biztosít az erőforrásokhoz, a megfelelő Access Token meglétekor

## OAuth 2.0 kiegészítések

* Bearer Token (RFC 6750)
* Token Introspection (RFC 7662) - access token opaque, azaz az alkalmazás nem tudja értelmezni, token introspection endpointon lehet lekérdezni
* Token Revocation (RFC 7009) - token visszavonás

## Bearer Token

* OAuth 2.0 az access token formátumát nem specifikálja
* Gyakran Bearer Token (RFC 6750)
  * Struktúrálatlan, csak egy karaktersor
  * Tipikusan `Authorization` fejlécben
    * Lehet form-encoded body-ban, vagy URL paraméterben (, de ez utóbbi biztonsági kockázat)
  * Gyakran JSON Web Token (JWT), melybe adat menthető
  * Non-opaque token: az alkalmazás közvetlenül ki tudja olvasni az adatot

# OpenID Connect

* OAuth 2.0 csak autorizáció, OpenID Connect authentication
* Single sign-on megoldás
* OpenID Connect 1.0 is a simple identity layer on top of the OAuth 2.0 protocol
* Identity: Set of attributes related to entity.
* Egy élő emberhez több identity tartozik (attribútumok halmaza, másnak mást mutat, pl. barátoknak, főnöknek)
* OpenID Connect Core a magja

Más elnevezéstant használ:

* End User: This is the equivalent of the resource owner in OAuth 2.0. It is, of course, the human being that is authenticating.
* Relying Party (RP): A somewhat confusing term for the application that would like to authenticate the end user. It is called the RP as it is a party that relies on the OpenID Provider (OP) to verify the identity of the user.
* OpenID Provider (OP): The identity provider that is authenticating the user, which is the role of Keycloak.
* Flow-ként hivatkozik az OAuth 2.0 grant type-okra
* Authorization Code grant type-ra épül, csak `scope=openid`paraméterrel jelzi, hogy ez authentication request

## Tokenek

* Access token: rövid lejárat
* Refresh Token: ha lejár az Access Token, segítségével új Access Token kérhető le 
* Identity Token: felhasználó adatai

## OAuth 2.0 Grant Types

* Authorization Code: klasszikus mód, ahogy egy webes alkalmazásba <br /> lépünk Facebook vagy a Google segítségével,
Authorization Code kerül vissza, mellyel lekérhető háttérben az Access Token
* Implicit: mobil alkalmazások, vagy SPA-k használják, Access Tokent kap azonnal (deprecated)
* Authorization Code Grant with Proof Key for Code Exchange (PKCE): mint az Authorization Code, de egy plusz lépéssel, hogy biztonságosabb legyen mobil/SPA alkalmazásoknál
* Resource Owner Password Credentials: ezt olyan megbízható <br /> alkalmazások használják, melyek maguk kérik be a jelszót, nem kell átirányítás (deprecated)
* Client Credentials: ebben az esetben <br /> nem a felhasználó kerül azonosításra, <br /> hanem az alkalmazás önmaga
* Device Authorization Flow: korlátozott eszközökön, pl. okostévé
* Refresh Token: új Access Token lekérése Refresh Tokennel

## Authorization Code

* A felhasználó (Resource Owner) elmegy az alkalmazás (Client) oldalára, ami pl. egy weboldal
* Az átirányít a Authorization Serverre (pl. Google vagy Facebook), megadva a saját azonosítóját és jelszavát (client id és client secret), hogy Authorization Code-ot kérjen. Átadja a scope-ot is, hogy mire van szüksége, valamint egy endpoint (redirect) URI-t, melyre az Authorization Code-ot várja
* Authorization Server ellenőrzi a client secretet, és a scope-ot
* Az Authorization Serveren a felhasználó bejelentkezik, ad jogot (consent), hogy a Client elérje az erőforrást
* Az Authorization Server visszairányítja a felhasználót <br /> az alkalmazás oldalára, url paraméterként átadva neki <br /> az Authorization Code-ot
* Az Authorization Code-dal az alkalmazás lekéri a tokeneket
* Az alkalmazás elindítja a felhasználói sessiont
* Az Access Tokennel hozzáfér az alkalmazás a Resource Serveren lévő erőforráshoz

## További specifikációk

* Discovery: Allows clients to dynamically discover information about the OP.
* Dynamic Registration: Allows clients to dynamically register themselves with the OP.
* Session Management: Defines how to monitor the end user’s authentication session with the OP, and how the client can initiate a logout.
* Front-Channel Logout: Defines a mechanism for single sign-out of multiple applications using embedded iframes.
* Back-Channel Logout: Defines a mechanism for single sign-out for multiple applications using a back-channel request mechanism.

## További koncepciók

* Id token az OAuth 2.0-val ellentétben not opaque, JWT formátumú
  * Alaposan specifikált, mezői a claimek, melyeket az alkalmazás közvetlenül tud olvasni
* userinfo endpoint - access tokennel hívható, és visszaadja az id tokenben lévő claimeket

## Access token

* Formátumát a OpenID Connect nem definiálja
* Keycloak itt is JWT-t használ
  * Így nem kell a Resource Servernek a Keycloak felé annyi kérést intéznie (az OAuth 2.0 token introspection endpoint, or the OpenID Connect UserInfo endpoint felé)

## Financial-Grade API (FAPI) 

Financial-Grade API (FAPI) : best practice-ek nagyon védett környezetben

## JWT

JWT comes from a family of specifications known as JOSE, which stands for JavaScript Object Signing and Encryption. The related specifications are as follows:

JSON Web Token (JWT, RFC 7519): Consists of two base64url-encoded JSON documents separated by a dot, a header, and a set of claims.
JSON Web Signature (JWS, RFC 7515): Adds a digital signature of the header and the claims.
JSON Web Encryption (JWE, RFC 7516): Encrypts the claims.
JSON Web Algorithms (JWA, RFC 7518): Defines the cryptographic algorithms that should be leveraged for JWS and JWE.
JSON Web Key (JWK, RFC 7517): Defines a format to represent cryptographic keys in JSON format.

OpenID Connect Discovery endpoint advertises an endpoint where the JSON Web Key Set (JWKS) can be retrieved, as well as what signing and encryption mechanisms from the JWA specification are supported.

OpenID Provider Metadata from a standard endpoint, namely <base URL>/.well-known/openid-configuration

---

# Működő alkalmazás

---

## Architektúra indítása

```shell
cd employees-infra
docker compose up
```

A Keycloak elérhető a `http://localhost:8080` címen. Felhasználónév / jelszó: `admin` / `admin` 

## Realm

* Létre kell hozni egy realmet (`employees`)
* Létre kell hozni egy klienst, amihez meg kell adni annak azonosítóját, <br /> és hogy milyen url-en érhető el 
  * _Client ID_: `employees-frontend`
  * _Name_: `Employees Frontend` - pl. a Consent felületen jelenik meg
  * _Root URL_: `http://localhost:8082`
  * _Home URL_: `http://localhost:8082` - pl. az Account Console-on jelenik meg
  * _Valid Redirect URIs_: `http://localhost:8082/*`
* Létre kell hozni a szerepköröket (`employees_user`, `employees_admin`)
* Létre kell hozni egy felhasználót 
  * Username: `johndoe`
  * Ki kell tölteni az _Email_, _First name_, _Last name_ mezőket is
  * _Email Verified_: _On_
  * Beállítani a jelszavát (a _Temporary_ értéke legyen _Off_, hogy ne kelljen jelszót módosítani)
  * Hozzáadni a szerepkört a _Role Mappings_ fülön


  * Ki kell tölteni az _Email_, _First name_, _Last name_ mezőket is, különben nem teljes a regisztráció. Ha a felhasználó be akar jelentkezni, akkor a felületen meg kell adni az adatait.

Ha nincs megadva az _Email_, _First name_, _Last name_, nem teljes a regisztráció. Ekkor ha a felhasználó be akar jelentkezni az űrlappal, akkor a felületen meg kell adni az adatait. Token lekéréskor a következő választ adja:

```json
{
    "error": "invalid_grant",
    "error_description": "Account is not fully set up"
}
```

## Token lekérése

```http
### OpenID configuration
GET http://localhost:8080/realms/employees/.well-known/openid-configuration
```

```http
### Certificates
GET http://localhost:8080/realms/employees/protocol/openid-connect/certs
```

```http
### Get token with resource owner password credentials
POST http://localhost:8080/realms/employees/protocol/openid-connect/token
Content-Type: application/x-www-form-urlencoded

grant_type=password&client_id=employees-frontend&username=johndoe&password=johndoe
```

* Token a [https://jwt.io/](https://jwt.io/) címen kibontható

Mezők:

* `exp`: lejárat
* `iss`: kiadó, URL of the Keycloak realm
* `sub`: felhasználó egyedi azonosítója
* `name`: felhasználó neve
* `preferred_username`: felhasználónév, mivel változhat, mindig a `sub` értéket érdemes tárolni
* `iat`: token kiadásának ideje
* `jti`: token egyedi azonosítója
* `aud`: kinek a számára lett kiállítva a token
* `azp`: amelyik alkalmazásnak a tokent kiadták, `employees-frontend`
* `sub`: a felhasználó egyedi azonosítója, ezt érdemes használni, mert a felhasználónév és az e-mail cím változhat
* `realm_access`: a felhasználó szerepkörei és a kliens által hozzáférhető szerepkörök metszete
* `resource_access`: kliens által hozzáférhető szerepkörök
* `scope`: mechanizmus arra, hogy az alkalmazás a felhasználó mely adataihoz férhet hozzá

```http
### Get token - OpenID
POST http://localhost:8080/realms/employees/protocol/openid-connect/token
Content-Type: application/x-www-form-urlencoded

grant_type=password&client_id=employees-frontend&username=johndoe&password=johndoe&scope=openid
```

* Belekerül az `id_token` is

Nincs benne a szerepkör az id tokenben, ezért hozzá kell adni: _Client Scopes_/`roles`/_Mappers_/`realm roles`/_Add to ID token_: _On_

`realm_access/roles` path-on kerül bele

UserInfo endpoint meghívása

```http
GET http://localhost:8080/realms/employees/protocol/openid-connect/userinfo
Authorization: Bearer eyJhb...
```

## Alkalmazások indítása

```shell
cd employees-backend
mvnw spring-boot:run
```

```shell
cd employees-frontend
mvnw spring-boot:run
```

A backend címe: `http://localhost:8081`

A frontend címe: `http://localhost:8082`


## Backend hívása tokennel

```http
### Create employee without token
POST http://localhost:8081/api/employees
Content-Type: application/json
Accept: application/json

{
  "name": "Jack Doe"
}
```

* `401 Unauthorized`

```http
### Create employee
POST http://localhost:8081/api/employees
Content-Type: application/json
Accept: application/json
Authorization: Bearer eyJhb...

{
  "name": "Jack Doe"
}
```

## Bejelentkezés menete

* Átirányítás a Keycloakra

```
GET http://localhost:8080/realms/employees/protocol/openid-connect/auth?response_type=code&client_id=employees-frontend&scope=openid%20email%20profile&state=X7QxVPcXSNoqylplsx1ScyL6zIaWIsxMybWIFXtJacM%3D&redirect_uri=http://localhost:8082/login/oauth2/code/&nonce=ZsvjELtbFK_SE-zkgkX1Jmg0CLNy6x5jZb4P6aSq1XQ
```

URL paraméterek:

* `response_type=code` - authorization code-ot kér vissza
* `client_id=employees-frontend`
* `scope=openid email profile`
* `state`: CSRF támadás ellen, átirányítás előtt elmenti (pl. session), majd visszairányításnál visszakapja és ellenőrzi
* `redirect_uri` - ide kéri a visszairányítást

* Bejelentkezés

```
POST http://localhost:8080/realms/EmployeesRealm/login-actions/authenticate?session_code=iFsJsXinyZy6parjQjz2dAWDqJATR-njQoHs-Z6Ug1g&execution=971d1195-045a-4f2f-813d-75c4799f5ac7&client_id=employees-frontend&tab_id=Zd8-5rWuBJ8

username=johndoe&password=johndoe&credentialId=
```

* `session_code`: Keycloak session azonosításhoz

* Átirányítás

```GET
http://localhost:8082/login/oauth2/code/?state=X7QxVPcXSNoqylplsx1ScyL6zIaWIsxMybWIFXtJacM%3D&session_state=156d2eca-2221-4644-bfe1-bd2677f2a1f8&iss=http%3A%2F%2Flocalhost%3A8080%2Frealms%2Femployees&code=7a49f249-11fc-4add-8b93-a6081b5f9dd3.156d2eca-2221-4644-bfe1-bd2677f2a1f8.73218982-1a49-4076-9ca1-fa875c16f64d
```

* `state` - kliens által küldöttet adja vissza
* `session_state` - OpenID Connect Session Management része, itt definiált, hogy iframe-ben lehet ellenőrizni, hogy a felhasználó kijelentkezett-e
* `iss` - "mix-up attacks" elleni védelem, kiadó urlje
* `code` - authorization code

* Backend log: HTTP requestben az `Authorization` header a Bearer tokennel

## Kijelentkezés

```
http://localhost:8080/realms/employees/protocol/openid-connect/logout
```

# Forráskód

## Frontend

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-oauth2-client</artifactId>
</dependency>
```

```yaml
spring:
  security:
    oauth2:
      client:
        registration:
          keycloak:
            client-id: employees-frontend
            authorization-grant-type: authorization_code
            scope: openid,email,profile
        provider:
          keycloak:
            issuer-uri: http://localhost:8080/realms/employees
            user-name-attribute: preferred_username

```

* `SecurityConfig` `filterChain()` metódusa
* `userAuthoritiesMapper()` role-ok beolvasása

* Token továbbküldése, `ClientConfig` osztályban

```java
var oauth2 = new ServletOAuth2AuthorizedClientExchangeFilterFunction(authorizedClientManager);
oauth2.setDefaultOAuth2AuthorizedClient(true);

var webClient = builder
        .baseUrl(employeesProperties.getBackendUrl())
        .apply(oauth2.oauth2Configuration())
        .build();

```

## Backend

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-oauth2-resource-server</artifactId>
</dependency>
```

```yaml
spring:
  security:
    oauth2:
      resourceserver:
        jwt:
          issuer-uri: http://localhost:8080/realms/employees
```

* `SecurityConfig` osztály
* `jwtDecoderByIssuerUri()` metódus és a `UsernameSubClaimAdapter` osztály - felhasználónév lekérdezése
* `jwtAuthenticationConverter()` és a `KeycloakRealmRoleConverter` oszály - role-ok lekérdezése

# Postman

Authorization/ OAuth 2.0
_Grant Type_: Password Credentials
_Access Token URL_: `http://localhost:8080/realms/employees/protocol/openid-connect/token`
_Client ID_: `employees-frontend`
_Scope_: `openid`

# Logout

## OpenID Connect 1.0 Client-Initiated Logout

A kliens alkalmazás behív az `end_session_endpoint` URL-re, melyet az OpenID Provider’s Discovery Metadata-ból olvas ki

Forráskód szinten:

https://docs.spring.io/spring-security/reference/servlet/oauth2/login/logout.html#configure-client-initiated-oidc-logout

## OpenID Connect 1.0 Back-Channel Logout

A Keycloak szól be az alkalmazásnak

Forráskód szinten:

https://docs.spring.io/spring-security/reference/servlet/oauth2/login/logout.html#configure-provider-initiated-oidc-logout

# Bejelentkezés felhasználónévvel vagy e-mail címmel

Be lehet jelentkezni e-mail címmel is

# Remember me

Böngésző bezárása után is megmarad a bejelentkezés

Bekapcsolható: _Realm settings_ / _Login_ / _Remember me_: _On_

(Kipróbálható másik Chrome profillal)

`KEYCLOAK_REMEMBER_ME` HTTP only cookie

# Saját regisztráció

_Realm settings_ / _Login_ / _User registration_

Nem kapja meg a jogosultságot

`default-roles-employees` role-ba beletenni az `employees_user` role-t

# Account console

http://localhost:8080/realms/employees/account/

* Updating their user profile
* Updating their password
* Enabling second factor authentication
* Viewing applications, including what applications they have authenticated to
* Viewing open sessions, including remotely signing out of other sessions

# Enable delete account

https://www.keycloak.org/docs/latest/server_admin/#proc-allow-user-to-delete-account_server_administration_guide

_Authentication_ / _Required Actions_ / _Delete Account_: _On_

Felhasználónál:

_Role Mappings_ / _Assign role_ / _Filter by clients_ / _account_ _delete_account_

# User locale

_Realm settings_ / _Localization_ / Internationalization Enabled
Default Locale
Felhasználónál kiválaszható: _Select a locale_
Megjelenik az Account console-on is
Átmegy a JWT-ben is, `locale` claimben
Megjelenik a login formon is

# Roles and groups

* _Realm roles_ fülön globális role-ok
* _Clients_ / _Roles_ fülön role-ok kliensenként
* Hozzárendelés: _Users_ / _Role mapping_ / _Assign role_: itt a szűrőben lehet kiválasztani, hogy globális, vagy klienshez tartozót választunk
* Default role: regisztrációkor megkapja
* Groups - koncepcionális fogalom, érdemes a szervezetnek megfelelően bevezetni
  * Hasonlít a composite role-okhoz, de inkább ezt használjuk
* Default group: regisztrációkor megkapja

# Új attribútum felvétele

_Realm settings_ / _User profile_ / _Attributes_ / _Create attribute_

* _Name_: `avatar_url`
* _Display name_: `Avatar URL`
* _Enabled when_: _Always_
* _Required field_: _Off_
* _Permission_: mind
* _Validations_: _uri_

Felhasználónál beállítandó: `https://avatars.githubusercontent.com/training360`

_Client scopes_ / _Create client scope_

* _Name_: `avatar_url`
* _Description_: `Custom scope: avatar_url`
* _Type_: _Optional_
* _Display on consent_: _On_
* _Consent screen text_: `Avatar URL`

_Mappers_

_Add mapper_ / _By configuration_ / _User Attribute_

_Create Protocol Mapper_
_User attribute_

_Name_: `Avatar URL`
_User Attribute_: `avatar_url`
_Token Claim Name_: `avatar_url`
_Claim JSON Type_: `String`
_Add to ID token_: `On`

## Client scopes, protocol mappers

Client scope

* Kliensek között megosztható konfiguráció
* Konfigurál:
  * Protocol mapper
  * Role scope mappings
* Támogatja az OAuth 2.0 `scope` paramétert, ezzel kér a kliens claimet vagy role-okat
* Protocol: OpenID vagy SAML
* Előre gyártott scope-ok: `profile`, `email`, `address` és `phone` OpenID Connect specifikáció részei
* `roles` nem a szabvány része
* _Client scopes_ fülön types
  * Default - új klienshez automatikusan hozzáadja
  * Optional - ki kell választani
* _Client_ / _Client scopes_ fülön types
  * Default - ekkor automatikusan megkapja a kliens
  * Optional - csak akkor, ha kéri a kliens

Billentsük át az `avatar_url` type-ját Optional értékre a kliens _Client scopes_ fülön!

Ez utóbbi esetben kérni kell a `scope` paraméterben

```http
### Get token - OpenID
POST http://localhost:8080/realms/employees/protocol/openid-connect/token
Content-Type: application/x-www-form-urlencoded

grant_type=password&client_id=employees-frontend&username=johndoe&password=johndoe&scope=openid%20avatar_url
```

Kipróbálható az _Evaluate_ / _Generated ID token_ fülön

Kiválasztandó `johndoe` felhasználó, csak `openid` scope-pal nincs `avatar_url` claim, `avatar_url` scope-pal van

Kliens alkalmazásban `spring.security.oauth2.client.registration.keycloak.scope` property

Mappings:

* User Attribute - felhasználók attribútumai alapján
* Hardcoded claim - egy konkrét érték, minden felhasználónál ugyanaz

`-dedicated` scope: adott kliensre specifikus

# Roles és scope kapcsolata: role scope mappings

Alapvetően a kliens megkapja a tokenben az összes role-t
Be lehet állítani, hogy a kliens csak azokat a role-okat kaphassa meg, melyek a klienshez tartoznak. _Clients_ / _Client scopes_ / `-dedicated` / _Scope_ / _Full scope allowed_: _Off_

# Roles és scope kapcsolata: Client scopes permissions

Alapesetben mindegyik felhasználó használhatja a scope-ot
Ha van megadva a Client scope-nál a _Scope_ fülön role (role mapping), akkor csak az a felhasználó kaphatja meg azt a scope-ot, akinek van az a role-ja

# Audience

* Keycloak oldalon bele kell tenni az `aud` mezőbe az alkalmazás nevét, mely fel fogja használni az access tokent
  * Több megoldás is van rá
    * Pl. protocol mapperrel kitöltjük az értékét, pl. `aud` értéke legyen `employees-backend`

Kliens oldalon pedig ellenőrizni kell

Forráskód szinten: https://docs.spring.io/spring-security/reference/servlet/oauth2/resource-server/jwt.html#_supplying_audiences

# Token felhasználásának korlátozása

* Audience használatával, a resource server nem fogadja el a tokent
* Role scope mappings: tokenbe nem kerül bele az összes role
* Saját scope, Client scopes permissions használatával
  * Kell egy saját scope protocol mapper nélkül
  * Kell Client scopes permissions, csak akkor kapja meg a felhasználó, ha megvan a megfelelő role-ja
  * Kliens oldalon le kell kérni a `scope`-pal
  * (Felhasználói consent)
  * Kliens oldalon scope-ok ellenőrzése

Pl. `employees:create` scope

* `employees_admin_scope` _Realm roles_ létrehozása
* `employees:create` Client scope létrehozása, Optional
  * _Scope_-nál beállítani, hogy csak a `employees_admin_scope` role-lal kapja meg
* _Clients_-nél beállítani a `employees:create` Client scope-ot, _Optional_ type-pal

Lekérni a klienssel, új scope-ot hozzáadni

```http
### Get token - OpenID
POST http://localhost:8080/realms/employees/protocol/openid-connect/token
Content-Type: application/x-www-form-urlencoded

grant_type=password&client_id=employees-frontend&username=johndoe&password=johndoe&scope=openid%20avatar_url%20employees:create
```

Access tokenben csak akkor lesz benne, ha a felhasználóhoz felvesszük a `employees_admin_scope` szerepkört

Alkalmazásban: `scope`-ot hozzáadni, az alapján ellenőrizni, nem a role alapján

# Token ellenőrzése

Csak akkor megy, ha a kliens típusa confidental. Ennek beállítása

* _Clients_ / _Client authentication_: _On_
* Mentés után megjelenik új fül: _Keys_, _Credentials_
* _Credentials_ fülön _Client Secret_
* Le kell kérni új tokent már a client secrettel

```http
### Get token - OpenID
POST http://localhost:8080/realms/employees/protocol/openid-connect/token
Content-Type: application/x-www-form-urlencoded

grant_type=password&client_id=employees-frontend&client_secret=p9hmYULlzyeNE4RIbNaeYII8KYSSRZP5&username=johndoe&password=johndoe&scope=openid%20avatar_url%20employees:create
```

```http
### Validate token
POST http://localhost:8080/realms/employees/protocol/openid-connect/token/introspect
Content-Type: application/x-www-form-urlencoded

client_id=employees-frontend&client_secret=p9hmYULlzyeNE4RIbNaeYII8KYSSRZP5&username=johndoe&token=eyJhb...
```

Spring `spring.security.oauth2.client.registration.keycloak.client-secret` property

Utána kapcsoljuk ki

# Consent

* Client-nél Consent Required
* Avatar URL is megjelenik
* Account Console-on vissza lehet vonni

Ez után a token lekérés jelszóval nem fog működni, mert a Resource Owner Password Credentials grant type esetén nincs consent.

# Sessions nyomon követése

Globálisan: _Sessions_ menüpont

Felhasználónál: _Sessions_ / _Logout all sessions_

Kliensnél: _Sessions_ fül

Account Console-on, _Account Security_ / _Device activity_

* Signed in devices - ki lehet jelentkeztetni

# Saját téma

Kiindulni: `/opt/keycloak/lib/lib/main/org.keycloak.keycloak-themes-24.0.1.jar`

* `account` - Account Console
* `admin` - Admin Console
* `email` - Emailek
* `login` - Login űrlapok
* `welcome` - Üdvözlő oldal

Téma részei:

* HTML templates (Freemarker Templates) (`*.ftl`)
* Images
* Message bundles (`*.properties`)
* Stylesheets
* Scripts

Theme properties

`/opt/keycloak/themes` könyvtárba másolni

```plain
mytheme/
├─ login/
│  ├─ resources/
│  │  ├─ css/
│  │  │  ├─ mylogin.css
│  ├─ theme.properties
```

`theme.properties`

```properties
parent=keycloak

styles=css/login.css css/mylogin.css
```

Fontos, hogy a `css/mylogin.css` legyen később, hogy felülírja

`mylogin.css`

```css
#kc-header {
    color: #ff0000;
    overflow: visible;
    white-space: nowrap;
}
```

_Realm settings_ / _Themes_ / _Login theme_ váltani

Developer módban nincs cache-elés

# Terms and conditions

_Authentication_/ _Required actions_ / _Terms and conditions_ / _Enabled_, _Default Action_

# GitHub integráció

* Először fel kell venni egy appot a GitHubon: _Settings_ / _Developer settings_ / _OAuth Apps_
* _Application name_: `keycloak-on-localhost`
* Homepage URL: `http://localhost:8080/`
* Auth callback URL: `http://localhost:8080/realms/employees/broker/github/endpoint`
* Itt ki kell másolni a client id értékét
* Kell generálni egy client secret-et, és kimásolni az értékét

Keycloak

* _Identity providers_ / _Github_
* _Client ID_, _Client Secret_ bemásolandó
* Kell Mappers / Attribute importer
  * _ID_: `avatar_url`
  * _Name_: `avatar_url`
  * _Mapper type_: _Attribute Importer_
  * _JSON Field Path_: `avatar_url`
  * _User Attribute Name_: `avatar_url`

# Authentication flows

* Böngészős bejelentkezés (Browser flow)
* Regisztráció (Registration flow)
* Password reset (Reset credentials flow)
* Token lekérés felhasználónévvel és jelszóval (Direct grant)
* Kliens autentikáció (Client secret)

Tartozik hozzájuk egy flow definition, mely a lépéseket tartalmazza

Új flow: legjobb gyakorlat, duplikálni egyet: `browser` -> `my browser`
Subflow-kból áll

* _Required_: hiba nélkül végig kell futnia
* _Alternative_: nem baj, ha nem fut végig hiba nélkül

Módosítsuk:

* _Username Password Form_ eltávolítása
* Helyére külön _Username Form_ és _Password Form_ (_Required_ _Requirement_-re kell állítani)
* _Action_ / _Bind flow_ / _Browser flow_

# Password policies

_Authentication_ / _Policies_

# E-mail küldés

_Realm settings_ / _Email_

* _From_: `keycloak@keycloak.hu`

* _Host_: `mailhog`
* _Port_: `1025`

Configure `admin` e-mail address, `admin@keycloak`

Tesztelni: [http://localhost:8025/](http://localhost:8025/)

_Realm settings_ / _Login_ / _Verify email_: _On_
_Forgot password_: _On_

# OTP

* 2FA leggyakrabban használt megoldása az OTP (one time password)

_Authentication_ / _Policies_ / _OTP Policy_

_Account Console_ / _Authenticator application_

Képernyőről leolvasni a QR kódot pl. a Microsoft Authenticatorral, beírni az egyszeri jelszót, elnevezni az eszközt

Utána bejelentkezéskor automatikusan kérni fogja az egyszeri jelszót

Kikényszeríteni az OLTP-t: Conditional OTP átállítása _Conditional_ értékről _Required_ értékre

# Webauthn

* PKI-n alapuló megoldás
* Megbízható eszközzel, mely megfelel a FIDO2 követelményeknek
  * Mobiltelefon, USB kulcs, NFC eszköz
* JElszó, OTP kiváltható

Saját flow módosítása:

* forms alatt új subflow, neve `my_webauthn`, _Conditional_
* új condition, _user configured_, _Required_
* új step, _Webauthn Authenticator_, _Required_

_Account Console_: _Set up Passkey_
iPhone
(iPhone Jelszavak alkalmazás)
Bejelentkezés

# User federation, LDAP

```shell
ldapsearch -H ldap://localhost:1389 -D "cn=admin,dc=example,dc=org" -w admin -b "ou=users,dc=example,dc=org"
```

* _Vendor_: _other_
* _Connection URL_: `ldap://openldap:1389`
* _Bind DN_: `cn=admin,dc=example,dc=org`
* _Bind credential_: `admin`

Test authentication

* _Edit mode_: _WRITABLE_
* _Users DN_: `ou=users,dc=example,dc=org`

_Users_ menüre átváltva be kell írni a keresőmezőbe egy `*` karaktert.
User federation _Disabled_ állapotba kapcsolható

# Events

_Realm settings_ / _Events_ / _User events settings_ / _Save Events_: _On_

_Events_ menüpont

Adatbázisban: `event_entity` tábla

Konzol logon is megjelenik a `type="LOGIN_ERROR"`

```
2024-03-21 21:28:50 2024-03-21 20:28:50,956 WARN  [org.keycloak.events] (executor-thread-243) type="LOGIN_ERROR", realmId="d339b1f6-7c48-4430-94f5-da3026e00dae", clientId="employees-frontend", userId="afb56636-ef61-4c93-af95-dd5d92442b9e", ipAddress="172.18.0.1", error="invalid_user_credentials", auth_method="openid-connect", redirect_uri="http://localhost:8082/authorize/oauth2/code/keycloak", code_id="863d70fd-7373-4b24-98a8-371565cb21c5", username="johndoe"
```

(Felhasználó nem látja a saját logját)


# X.509 client certificate user authentication

https://www.keycloak.org/docs/16.1/server_admin/index.html#_x509

# Health and metrics

https://www.keycloak.org/server/health

https://www.keycloak.org/server/configuration-metrics

A Service Provider that adds a metrics endpoint to Keycloak. The endpoint returns metrics data ready to be scraped by Prometheus.

https://github.com/aerogear/keycloak-metrics-spi

# Logging

https://www.keycloak.org/server/logging

# REST API

[https://www.keycloak.org/documentation](https://www.keycloak.org/documentation)

Administration REST API

```http
### Get token for admin
POST http://localhost:8080/realms/master/protocol/openid-connect/token
Content-Type: application/x-www-form-urlencoded

grant_type=password&client_id=admin-cli&username=admin&password=admin
```

```http
### List users
GET http://localhost:8080/admin/realms/employees/users
Authorization: Bearer eyJhb...
```

# Admin CLI

```shell
/opt/keycloak/bin/kcadm.sh
/opt/keycloak/bin/kcadm.sh config credentials --server http://127.0.0.1:8080 --realm master get users --user admin

/opt/keycloak/bin/kcadm.sh get users -r employees
/opt/keycloak/bin/kcadm.sh get events -r employees

/opt/keycloak/bin/kcadm.sh create users -s username=johnsmith -s enabled=true -r employees
/opt/keycloak/bin/kcadm.sh set-password --username johnsmith --new-password johnsmith  -r employees
```

# Export/import

Felületről: Partial export/import

* Include groups and roles
* Include clients

https://www.keycloak.org/server/importExport

```shell
cd /opt/employees
/opt/keycloak/bin/kc.sh export --file employees-keycloak-export.json --realm employees
```

* Ha könyvtárat adunk meg, a realmeket külön fájlba exportálja

Felhasználók exportálása:

* `different_files`: Users export into different json files, depending on the maximum number of users per file set by --users-per-file. This is the default value.
* `skip`: Skips exporting users.
* `realm_file`: Users will be exported to the same file as the realm settings. For a realm named "foo", this would be "foo-realm.json" with realm data and users.
* `same_file`: All users are exported to one explicit file. So you will get two json files for a realm, one with realm data and one with users.

Importálás elvégezhető parancssorból, de a Keycloak indításakor is

* CLI szkriptek
* Configuration as Code for Keycloak realms

https://github.com/adorsys/keycloak-config-cli

`test-user.json`

```json
{
  "realm" : "employees",
  "users": [

{
    "username" : "janeimported",
    "firstName" : "Jane",
    "lastName" : "Imported",
    "email" : "johnimported@localhost",
    "enabled" : true,
    "credentials" : [ {
      "type" : "password",
      "value": "janeimported"
    }],
    "realmRoles" : [ "employees_user" ]
  }

   ]
}
```

* Figyeljük meg, hogy a jelszó nincs titkosított formában

```shell
java -jar keycloak-config-cli-24.0.1.jar --keycloak.url=http://localhost:8080 --keycloak.ssl-verify=false --keycloak.user=admin --keycloak.password=admin --import.files.locations=test-user.json
```

# HA

## Clustering

Több példányban futtatható
Beállításához:

* Infinispan elosztott cache, node-ok közötti kommunikáció KGroups használatával UDP-n
* Reverse proxy, session affinity-vel

https://www.keycloak.org/server/caching

Cache-ek típusai

* Distributed - alapesetben 2 példányon, ez feljebb emelhető
* Local - csak az adott node-on
* Replicated - minden

## Reverse proxy

https://www.keycloak.org/server/reverseproxy

A következőkre kell figyelni:

* Load balancing
* TLS termination and re-encryption

`$KC_HOME/conf/keycloak.conf` fájlban

```conf
proxy=reencrypt
```

Proxy és a Keycloak különböző tanúsítványokat használnak

* Forwarding headers

* `Forwarded`: A standard header containing all the information about the client making a request. For more details, look at https://www.rfc-editor.org/rfc/rfc7239.html.
* `X-Forward-For`: A non-standard header indicating the address of the client where the request originated from
* `X-Forward-Proto`: A non-standard header indicating the protocol (for example, HTTPS) that the client is using to communicate with the proxy
* `X-Forward-Host`: A non-standard header indicating the original host and port number requested by the client

* Session affinity

Ez akkor fontos, mikor a kliens és a Keycloak között több interakció is van, local cache-ek miatt
Bízzuk a proxy-ra a `KC_ROUTE` alapján, és ekkor a Keycloak saját mechanizmusát kapcsoljuk ki a `$KC_HOME/conf/keycloak.conf` fájlban

```conf
spi-sticky-session-encoder-infinispan-should-attach-route=false
```

# Extend

https://www.keycloak.org/docs/latest/server_development/

[User Storage SPI](https://www.keycloak.org/docs/latest/server_development/index.html#_user-storage-spi)

Properties fájl alapján:

https://github.com/keycloak/keycloak-quickstarts/tree/latest/extension/user-storage-simple

JPA-val:

https://github.com/keycloak/keycloak-quickstarts/tree/latest/extension/user-storage-jpa

# Optimized container

The following are some optimizations performed by the build command:

* A new closed-world assumption about installed providers is created, meaning that no need exists to re-create the registry and initialize the factories at every Keycloak startup.
* Configuration files are pre-parsed to reduce I/O when starting the server.
* Database specific resources are configured and prepared to run against a certain database vendor.
* By persisting build options into the server image, the server does not perform any additional step to interpret configuration options and (re)configure itself.

https://www.keycloak.org/server/containers
https://www.keycloak.org/server/configuration