# Changelog

## UNRELEASED

### Breaking

The config variable `UNSAFE_NO_RESET_BINDING` has been removed in favor of `PASSWORD_RESET_COOKIE_BINDING`.
The logic for this security feature has been reversed. The default behavior until now was to block subsequent
requests to the password reset form if they provided an invalid binding cookie. This created issues for people
that were using evil E-Mail providers. These would scan their users E-Mails and use links inside them.
This link usage however made it impossible for "the real user" to use the link properly, because it has been
used already by its provider.  
In some cases, this hurts the UX more than it is a benefit to the security, so this feature is now an opt-in
hardening instead of opt-out evil provider error fixing.  
Additionally, to improve the UX even further, the additional E-Mail input form has been removed from the password
reset page as well. The security benefits of this were rather small compared to the UX degradation.
#365
[1af7b92](https://github.com/sebadob/rauthy/commit/1af7b92204a99de4883154055bb3081dc196d759)

### Features

#### Dynamic Server Side Search + Pagination

Until now, the Admin UI used client side searching and pagination. This is fine for most endpoints, but
the users can grow quite large depending on the instance while all other endpoints will return rather small
"GET all" data.  
To keep big Rauthy instances with many thousands of users fast and responsive, you can set a threshold for
the total users count at which Rauthy will dynamically switch from client side to server side pagination
and searching for the Admin UI's Users page.

```
# Dynamic server side pagination threshold
# If the total users count exceeds this value, Rauthy will dynamically
# change search and pagination for users in the Admin UI from client
# side to server side to not have a degradation in performance.
# default: 1000
SSP_THRESHOLD=1000
```

For smaller instances, keeping it client side will make the UI a bit more responsive and snappy.
For higher user counts, you should switch to do this on the server though to keep the UI fast and not
send huge payloads each time.

[b4dead3](https://github.com/sebadob/rauthy/commit/b4dead36169cc284c97af5a982cc33fb8a0be02b)
[9f87af3](https://github.com/sebadob/rauthy/commit/9f87af3dfb49b48300b885bf406f852579470193)
[e6d39d1](https://github.com/sebadob/rauthy/commit/e6d39d1e1118e18aeb020fbbb477a944fcd1467a)

#### UX Improvement on Login

The login form now contains a "Home" icon which will appear, if a `client_uri` is registered for the current
client. A user may click this and be redirected to the client, if a login is not desired for whatever reason.
Additionally, if the user registration is configured to be open, a link to the user registration will be shown
at the bottom as well.

[]()

#### Unlink Account from Provider

A new button has been introduced to the account view of federated accounts.  
You can now "Unlink" an account from an upstream provider, if you have set it up with at least
a password or passkey before.
[8b1d9a8](https://github.com/sebadob/rauthy/commit/8b1d9a882b0d4b059f3ed884deaacfcdeb109856)

#### Bootstrap default Admin in production

You can set environment variables either via `rauthy.cfg`, `.env` or as just an env var during
initial setup in production. This makes it possible to create an admin account with the very first
database setup with a custom E-Mail + Password, instead of the default `admin@localhost.de` with
a random password, which you need to pull from the logs.

```
#####################################
############# BOOSTRAP ##############
#####################################

# If set, the email of the default admin will be changed
# during the initialization of an empty production database.
BOOTSTRAP_ADMIN_EMAIL="alfred@batcave.io"

# If set, this plain text password will be used for the
# initial admin password instead of generating a random
# password.
#BOOTSTRAP_ADMIN_PASSWORD_PLAIN="123SuperSafe"

# If set, this will take the argon2id hashed password
# during the initialization of an empty production database.
# If both BOOTSTRAP_ADMIN_PASSWORD_PLAIN and
# BOOTSTRAP_ADMIN_PASSWORD_ARGON2ID are set, the hashed version
# will always be prioritized.
BOOTSTRAP_ADMIN_PASSWORD_ARGON2ID='$argon2id$v=19$m=32768,t=3,p=2$mK+3taI5mnA+Gx8OjjKn5Q$XsOmyvt9fr0V7Dghhv3D0aTe/FjF36BfNS5QlxOPep0'
```

### Bugfixes

- The button for requesting a password reset from inside a federated account view has been
  disabled when it should not be, and therefore did not send out requests.
  [39e585d](https://github.com/sebadob/rauthy/commit/39e585d1d53a2490b273ba5c33b864ec0d7835d5)
- A really hard to reproduce bug where the backend complained about a not-possible mapping
  from postgres `INT4` to Rust `i64` as been fixed. This came with the advantage of hacing
  a few more compile-time checked queries for the `users` table.
  [1740177](https://github.com/sebadob/rauthy/commit/174017736d62d2237a5f00b9d0508bca0a57c8b0)
- A fix for the `/users/register` endpoint in the OpenAPI documentation has been fixed, which
  was referencing the wrong request body
  [463e424](https://github.com/sebadob/rauthy/commit/463e42409d14d84b59a1d3606fa53ca2ecee2b86)
- The page title for a password reset now shows "New Account" if this is a fresh setup and only
  "Password Reset" when it actually is a reset
  [84bbdf7](https://github.com/sebadob/rauthy/commit/84bbdf7bc464e5869285225e446cb56e17f53583)
- The "User Registration" header on the page for an open user registration as only showing up,
  when the domain was restricted.
  []()

## 0.22.1

### Security

This version fixes a [potential DoS in rustls](https://rustsec.org/advisories/RUSTSEC-2024-0336.html) which has
been found yesterday.  
[f4d65a6](https://github.com/sebadob/rauthy/commit/f4d65a6b056183f914075d6047384e2a7a4f0329)

### Features

#### Dedicated `/forward_auth` + Trusted Authn/Authz Headers

In addition to the `/userinfo` endpoint specified in the OIDC spec, Rauthy implements an additional endpoint
specifically for ForwardAuth situations. You can find it at `/auth/v1/oidc/forward_auth` and it can be configured
to append optional Trusted Header with User Information for downstream applications, that do not support OIDC
on their own.

The HeaderNames can be configured to match your environment.
Please keep in mind, that you should only use these, if you legacy application does not support OIDC natively,
because Auth Headers come with a lot of pitfalls, when your environment is not configured properly.

```
# You can enable authn/authz headers which would be added to the response
# of the `/auth/v1/forward_auth` endpoint. With  `AUTH_HEADERS_ENABLE=true`,
# the headers below will be added to authenticated requests. These could
# be used on legacy downstream applications, that don't support OIDC on
# their own.
# However, be careful when using this, since this kind of authn/authz has
# a lot of pitfalls out of the scope of Rauthy.
AUTH_HEADERS_ENABLE=true

# Configure the header names being used for the different values.
# You can change them to your needs, if you cannot easily change your
# downstream apps.
# default: x-forwarded-user
AUTH_HEADER_USER=x-forwarded-user
# default: x-forwarded-user-roles
AUTH_HEADER_ROLES=x-forwarded-user-roles
# default: x-forwarded-user-groups
AUTH_HEADER_GROUPS=x-forwarded-user-groups
# default: x-forwarded-user-email
AUTH_HEADER_EMAIL=x-forwarded-user-email
# default: x-forwarded-user-email-verified
AUTH_HEADER_EMAIL_VERIFIED=x-forwarded-user-email-verified
# default: x-forwarded-user-family-name
AUTH_HEADER_FAMILY_NAME=x-forwarded-user-family-name
# default: x-forwarded-user-given-name
AUTH_HEADER_GIVEN_NAME=x-forwarded-user-given-name
# default: x-forwarded-user-mfa
AUTH_HEADER_MFA=x-forwarded-user-mfa
```

[7d5a44a](https://github.com/sebadob/rauthy/commit/7d5a44a2ff29ec87458c6bf6b979cc4750491391)

### Bugfixes

- allow CORS requests for the GET PoW and the user sign up endpoint's to make it possible to build a custom UI without
  having a server side. At the same time, the method for requesting a PoW **has been changed from `GET` to `POST`**.
  This change has been done because even though only in-memory, a request would create data in the backend, which should
  never be done by a `GET`.
  Technically, this is a breaking change, but since it has only been available from the Rauthy UI itself because of the
  CORS header setting, I decided to only bump the patch, not the minor version.
  [e4d935f](https://github.com/sebadob/rauthy/commit/e4d935f7b51459031a37fb2ec2eb9952bc278f2e)

## v0.22.0

### Breaking

There is one breaking change, which could not have been avoided.  
Because of a complete rewrite of the logic how custom client logos (uploaded via
`Admin UI -> Clients -> Client Config -> Branding`), you will loose custom logos uploaded in the past for a client.
The reason is pretty simple. Just take a look at `Auto Image Optimization` below.

Apart from this, quite a few small internal improvements have been made to make life easier for developers and new
contributors. These changes are not listed in the release notes.

### Changes

#### Upstream Auth Providers

Rauthy v0.22.0 brings (beta) support for upstream authentication providers.  
This is a huge thing. It will basically allow you to set up things like *Sign In with Github* into Rauthy. You could use
your Github account for signup and login, and manage custom groups, scopes, and so on for the users on Rauthy. This
simplifies the whole onboarding and login for normal users a lot.

You can add as many auth providers as you like. They are not statically configured, but actually configurable via the
Admin UI. A user account can only be bound to one auth provider though for security reasons. Additionally, when a user
already exists inside Rauthy's DB, was not linked to an upstream provider and then tries a login but produces an email
conflict, the login will be rejected. It must be handled this way, because Rauthy can not know for sure, if the upstream
email was actually been verified. If this is not the case, simply accepting this login could lead to account takeover,
which is why this will not allow the user to login in that case.  
The only absolutely mandatory information, that Rauthy needs from an upstream provider, is an `email` claim in either
the `id_token` or as response from the userinfo endpoint. If it cannot find any `name` / `given_name` / `family_name`,
it will simply insert `N/A` as values there. The user will get a warning on his next values update to provide that
information.

The supported features (so far) are:

- auto OpenID Connect metadata discovery
- accept invalid TLS certs for upstream clients, for instance inside self-hosted environments
- provide a root certificate for an upstream client for the same reason as above
- choose a template for the config (currently Google and Github exist)
- fully customized endpoint configuration if the provider does not support auto-lookup
- optional mfa claim mapping by providing a json parse regex:  
  If the upstream provider returns information about if the user has actually done at least a 2FA sign in, Rauthy can
  extract this information dynamically from the returned JSON. For instance, Rauthy itself will add an `amr` claim to
  the `id_token` and you can find a value with `mfa` inside it, if the user has done an MFA login.  
  Github returns this information as well (which has been added to the template).
- optional `rauthy_admin` claim mapping:
  If you want to allow full rauthy admin access for a user depending on some value returned by the upstream provider,
  you can do a mapping just like for the mfa claim above.
- upload a logo for upstream providers
  Rauthy does not (and never will do) an automatic logo download from a provider, because this logo will be shown on the
  login page and must be trusted. However, if Rauthy would download any arbitrary logo from a provider, this could
  lead to code injection into the login page. This is why you need to manually upload a logo after configuration.

**Note:**  
If you are testing this feature, please provide some feedback in [#166](https://github.com/sebadob/rauthy/issues/166)
in any case - if you have errors or not. It would be nice to know about providers that do work already and those, that
might need some adoptions. All OIDC providers should work already, because for these we can rely on standards and RFCs,
but all others might produce some edge cases and I simply cannot test all of them myself.  
If we have new providers we know of, that need special values, these values would be helpful as well, because Rauthy
could provide a template in the UI for these in the future, so please let me know.

#### Auto Image Optimization

The whole logic how images are handled has been rewritten.
Up until v0.21.1, custom client logos have been taken as a Javascript `data:` url because of easier handling.
This means however, that we needed to allow `data:` sources in the CSP for `img-src`, which can be a security issue and
should be avoided if possible.

This whole handling and logic has been rewritten. The CSP hardening has been finalized by removing the `data:` allowance
for `img-src`. You can still upload SVG / JPG / PNG images under the client branding (and for the new auth providers).
In the backend, Rauthy will actually parse the image data, convert the images to the optimized `webp` format, scale
the original down and save 2 different versions of it. The first version will be saved internally to fit into 128x128px
for possible later use, the second one even smaller. The smaller version will be the one actually being displayed on
the login page for Clients and Auth Providers.  
This optimization reduces the payload sent to clients during the login by a lot, if the image has not been manually
optimized beforehand. Client Logos will typically be in the range of ~5kB now while the Auth Providers ones will usually
be less than 1kB.

[6ccc541](https://github.com/sebadob/rauthy/commit/6ccc541b0e6d155024aa9ba4d832dcb01f135528)

#### Custom Header Name for extracting Client IPs

With the name config variable `PEER_IP_HEADER_NAME`, you can specify a custom header name which will be used for
extracting the clients IP address. For instance, if you are running Rauthy behind a Cloudflare proxy, you will usually
only see the IP of the proxy itself in the `X-FORWARDED-FOR` header. However, cloudflare adds a custom header called
`CF-Connecting-IP` to the request, which then shows the IP you are looking for.    
Since it is very important for rate limiting and blacklisting that Rauthy knows the clients IP, this can now be
customized.

```
# Can be set to extract the remote client peer IP from a custom header name
# instead of the default mechanisms. This is needed when you are running
# behind a proxy which does not set the `X-REAL-IP` or `X-FORWARDED-FOR` headers
# correctly, or for instance when you proxy your requests through a CDN like
# Cloudflare, which adds custom headers in this case.
# For instance, if your requests are proxied through cloudflare, your would
# set `CF-Connecting-IP`.
PEER_IP_HEADER_NAME="CF-Connecting-IP"
```

[a56c05e](https://github.com/sebadob/rauthy/commit/a56c05e560c4b89f88f34240cb79a29e1e8a746c)

#### Additional Client Data: `contacts` and `client_uri`

For each client, you can now specify contacts and a URI, where the application is hosted. These values might be shown
to users during login in the future. For Rauthy itself, the values will be set with each restart in the internal
anti lockout rule. You can specify the contact via a new config variable:

```
# This contact information will be added to the `rauthy`client
# within the anti lockout rule with each new restart.
RAUTHY_ADMIN_EMAIL="admin@localhost.de"
```

[6ccc541](https://github.com/sebadob/rauthy/commit/6ccc541b0e6d155024aa9ba4d832dcb01f135528)

#### Custom `redirect_uri` During User Registration

If you want to initiate a user registration from a downstream app, you might not want your users to be redirected
to their Rauthy Account page after they have initially set the password. To encounter this, you can redirect them
to the registration page and append a `?redirect_uri=https%3A%2F%2Frauthy.example.com` query param. This will be
saved in the backend state and the user will be redirected to this URL instead of their account after they have set
their password.

#### Password E-Mail Tempalte Overwrites

You can not overwrite the template i18n translations for the NewPassword and ResetPassword E-Mail templates.  
There is a whole nwe section in the config and it can be easily done with environment variables:

```
#####################################
############ TEMPLATES ##############
#####################################

# You can overwrite some default email templating values here.
# If you want to modify the basic templates themselves, this is
# currently only possible with a custom build from source.
# The content however can mostly be set here.
# If the below values are not set, the default will be taken.

# New Password E-Mail
#TPL_EN_PASSWORD_NEW_SUBJECT="New Password"
#TPL_EN_PASSWORD_NEW_HEADER="New password for"
#TPL_EN_PASSWORD_NEW_TEXT=""
#TPL_EN_PASSWORD_NEW_CLICK_LINK="Click the link below to get forwarded to the password form."
#TPL_EN_PASSWORD_NEW_VALIDITY="This link is only valid for a short period of time for security reasons."
#TPL_EN_PASSWORD_NEW_EXPIRES="Link expires:"
#TPL_EN_PASSWORD_NEW_BUTTON="Set Password"
#TPL_EN_PASSWORD_NEW_FOOTER=""

#TPL_DE_PASSWORD_NEW_SUBJECT="Passwort Reset angefordert"
#TPL_DE_PASSWORD_NEW_HEADER="Passwort Reset angefordert für"
#TPL_DE_PASSWORD_NEW_TEXT=""
#TPL_DE_PASSWORD_NEW_CLICK_LINK="Klicken Sie auf den unten stehenden Link für den Passwort Reset."
#TPL_DE_PASSWORD_NEW_VALIDITY="Dieser Link ist aus Sicherheitsgründen nur für kurze Zeit gültig."
#TPL_DE_PASSWORD_NEW_EXPIRES="Link gültig bis:"
#TPL_DE_PASSWORD_NEW_BUTTON="Passwort Setzen"
#TPL_DE_PASSWORD_NEW_FOOTER=""

# Password Reset E-Mail
#TPL_EN_RESET_SUBJECT="Neues Passwort"
#TPL_EN_RESET_HEADER="Neues Passwort für"
#TPL_EN_RESET_TEXT=""
#TPL_EN_RESET_CLICK_LINK="Klicken Sie auf den unten stehenden Link um ein neues Passwort zu setzen."
#TPL_EN_RESET_VALIDITY="This link is only valid for a short period of time for security reasons."
#TPL_EN_RESET_EXPIRES="Link expires:"
#TPL_EN_RESET_BUTTON="Reset Password"
#TPL_EN_RESET_FOOTER=""

#TPL_DE_RESET_SUBJECT="Passwort Reset angefordert"
#TPL_DE_RESET_HEADER="Passwort Reset angefordert für"
#TPL_DE_RESET_TEXT=""
#TPL_DE_RESET_CLICK_LINK="Klicken Sie auf den unten stehenden Link für den Passwort Reset."
#TPL_DE_RESET_VALIDITY="Dieser Link ist aus Sicherheitsgründen nur für kurze Zeit gültig."
#TPL_DE_RESET_EXPIRES="Link gültig bis:"
#TPL_DE_RESET_BUTTON="Passwort Zurücksetzen"
#TPL_DE_RESET_FOOTER=""
```

### Bugfix

- UI: when a client name has been removed and saved, the input could show `undefined` in some cases
  [2600005](https://github.com/sebadob/rauthy/commit/2600005be81649083103051a5bfc7b7ec49c9c3c)
- The default path to TLS certificates inside the container image has been fixed in the deploy cfg template.
  This makes it possible now to start the container for testing with TLS without explicitly specifying the path
  manually.
  [3a04dc0](https://github.com/sebadob/rauthy/commit/3a04dc02a878263cec2d841553747c78a41b7c4a)
- The early Passkey implementations of the Bitwarden browser extension seem to have not provided all correct values,
  which made Rauthy complain because of not RFC-compliant requests during Passkey sign in. This error cannot really be
  reproduced. However, Rauthy tries to show more error information to the user in such a case.
  [b7f94ff](https://github.com/sebadob/rauthy/commit/b7f94ff8ad3cd9aad61baf3710e4f9788c498ca6)
- Don't use the reset password template text for "new-password emails"
  [45b4160](https://github.com/sebadob/rauthy/commit/45b41604b1e0c0c9af423be65021953116b88150)

## v0.21.1

### Changes

- host `.well-known additionally` on root `/`
  [3c594f4](https://github.com/sebadob/rauthy/commit/3c594f4628c8d301ed1556d5dd3ea5a004afdcac)

### Bugfix

- Correctly show the `registration_endpoint` for dynamic client registration in the `openid-configuration`
  if it is enabled.
  [424fdd1](https://github.com/sebadob/rauthy/commit/424fdd10e57639c1d60dde61f551462e67ff4934)

## v0.21.0

### Breaking

The access token's `sub` claim had the email as value beforehand. This was actually a bug.  
The `sub` of access token and id token must be the exact same value. `sub` now correctly contains the user ID,
which is 100% stable, even if the user changes his email address.  
This means, if you used the `sub` from the access token before to get the users email, you need to pay attention now.
The `uid` from the access token has been dropped, because this value is now in `sub`. Additionally, since many
applications need the email anyway, it makes sense to have it inside the access token. For this purpose, if `email`
is in the requested `scope`, it will be mapped to the `email` claim in the access token.

### Features

#### OpenID Core Compatibility

Rauthy should now be compliant with the mandatory part of the OIDC spec.
A lot of additional things were already implemented many versions ago.
The missing thing was respecting some additional params during GET `/authorize`.

#### OpenID Connect Dynamic Client Registration

Rauthy now supports Dynamic Client registration as
defined [here](https://openid.net/specs/openid-connect-registration-1_0.html).

Dynamic clients will always get a random ID, starting with `dyn$`, followed by a random alphanumeric string,
so you can distinguish easily between them in the Admin UI.  
Whenever a dynamic client does a `PUT` on its own modification endpoint with the `registration_token` it
received from the registration, the `client_secret` and the `registration_token` will be rotated and the
response will contain new ones, even if no other value has been modified. This is the only "safe" way to
rotate secrets for dynamic clients in a fully automated manner. The secret expiration is not set on purpose,
because this could easily cause troubles, if not implemented properly on the client side.  
If you have a
badly implemented client that does not catch the secret rotation and only if you cannot fix this on the
client side, maybe because it's not under your control, you may deactivate the auto rotation with
`DYN_CLIENT_SECRET_AUTO_ROTATE=false`. Keep in mind, that this reduces the security level of all dynamic
clients.

Bot and spam protection is built-in as well in the best way I could think of. This is disabled, if you set
the registration endpoint to need a `DYN_CLIENT_REG_TOKEN`. Even though this option exists for completeness,
it does not really make sense to me though. If you need to communicate a token beforehand, you could just
register the client directly. Dynamic clients are a tiny bit less performant than static ones, because we
need one extra database round trip on successful token creation to make the spam protection work.  
However, if you do not set a `DYN_CLIENT_REG_TOKEN`, the registration endpoint would be just open to anyone.
To me, this is the only configuration for dynamic client registration, that makes sense, because only that
is truly dynamic. The problem then are of course bots and spammers, because they can easily fill your
database with junk clients. To counter this, Rauthy includes two mechanisms:

- hard rate limiting - After a dynamic client has been registered, another one can only be registered
  after 60 seconds (default, can be set with `DYN_CLIENT_RATE_LIMIT_SEC`) from the same public IP.
- auto-cleanup of unused clients - All clients, that have been registered but never used, will be deleted
  automatically 60 minutes after the registration (default, can be set with `DYN_CLIENT_CLEANUP_MINUTES`).

There is a whole new section in the config:

```
#####################################
########## DYNAMIC CLIENTS ##########
#####################################

# If set to `true`, dynamic client registration will be enabled.
# Only activate this, if you really need it and you know, what
# you are doing. The dynamic client registration without further
# restriction will allow anyone to register new clients, even
# bots and spammers, and this may create security issues, if not
# handled properly and your users just login blindly to any client
# they get redirected to.
# default: false
#ENABLE_DYN_CLIENT_REG=false

# If specified, this secret token will be expected during
# dynamic client registrations to be given as a
# `Bearer <DYN_CLIENT_REG_TOKEN>` token. Needs to be communicated
# in advance.
# default: <empty>
#DYN_CLIENT_REG_TOKEN=

# The default token lifetime in seconds for a dynamic client,
# that will be set during the registration.
# This value can be modified manually after registration via
# the Admin UI like for any other client.
# default: 1800
#DYN_CLIENT_DEFAULT_TOKEN_LIFETIME=1800

# If set to 'true', client secret and registration token will be
# automatically rotated each time a dynamic client updates itself
# via the PUT endpoint. This is the only way that secret rotation
# could be automated safely.
# However, this is not mandatory by RFC and it may lead to errors,
# if the dynamic clients are not implemented properly to check for
# and update their secrets after they have done a request.
# If you get into secret-problems with dynamic clients, you should
# update the client to check for new secrets, if this is under your
# control. If you cannot do anything about it, you might set this
# value to 'false' to disable secret rotation.
# default: true
#DYN_CLIENT_SECRET_AUTO_ROTATE=true

# This scheduler will be running in the background, if
# `ENABLE_DYN_CLIENT_REG=true`. It will auto-delete dynamic clients,
# that have been registered and not been used in the following
# `DYN_CLIENT_CLEANUP_THRES` hours.
# Since a dynamic client should be used right away, this should never
# be a problem with "real" clients, that are not bots or spammers.
#
# The interval is specified in minutes.
# default: 60
#DYN_CLIENT_CLEANUP_INTERVAL=60

# The threshold for newly registered dynamic clients cleanup, if
# not being used within this timeframe. This is a helper to keep
# the database clean, if you are not using any `DYN_CLIENT_REG_TOKEN`.
# The threshold should be specified in minutes. Any client, that has
# not been used within this time after the registration will be
# automatically deleted.
#
# Note: This scheduler will only run, if you have not set any
# `DYN_CLIENT_REG_TOKEN`.
#
# default: 60
#DYN_CLIENT_CLEANUP_MINUTES=60

# The rate-limiter timeout for dynamic client registration.
# This is the timeout in seconds which will prevent an IP from
# registering another dynamic client, if no `DYN_CLIENT_REG_TOKEN`
# is set. With a `DYN_CLIENT_REG_TOKEN`, the rate-limiter will not
# be applied.
# default: 60
#DYN_CLIENT_RATE_LIMIT_SEC=60
```

#### Better UX with respecting `login_hint`

This is a small UX improvement in some situations. If a downstream client needs a user to log in, and it knows
the users E-Mail address somehow, maybe because of an external initial registration, It may append the correct
value with appending the `login_hint` to the login redirect. If this is present, the login UI will pre-fill the
E-Mail input field with the given value, which make it one less step for the user to log in.

### Changes

- The `/userinfo` endpoint now correctly respects the `scope` claim from withing the given `Bearer` token
  and provides more information. Depending on the `scope`, it will show the additional user values that were
  introduced with v0.20
  [49dd553](https://github.com/sebadob/rauthy/commit/49dd553e1df072f6a0db3b1cfaa130f7146aaf25)
- respect `max_age` during GET `/authorize` and add `auth_time` to the ID token
  [9ca6970](https://github.com/sebadob/rauthy/commit/9ca697091eba2a6297c289162f426a92b28385f4)
- correctly work with `prompt=none` and `prompt=login` during GET `/authorize`
  [9964fa4](https://github.com/sebadob/rauthy/commit/9964fa4aed7fdb6428b518a13dc18cc2761606f3)
- Make it possible to use an insecure SMTP connection
  [ef46414](https://github.com/sebadob/rauthy/commit/ef464145d5ed7f8ffe97fc24667a74485f94c2f1)
- Implement OpenID Connect Dynamic Client Registration
  [b48552e](https://github.com/sebadob/rauthy/commit/b48552e79f2a3aca0c5cefcc25ef7d9f7c21c6d4)
  [12179c9](https://github.com/sebadob/rauthy/commit/12179c9898126e5e78a80a3b49df6ca5a501ff81)
- respect `login_hint` during GET `/authorize`
  [963644c](https://github.com/sebadob/rauthy/commit/963644c36466c5eb9d0ad4d2411198ea71753d59)

### Bugfixes

- Fix the link to the latest version release notes in the UI, if an update is available
  [e66e496](https://github.com/sebadob/rauthy/commit/e66e4962631620ad762a80181e085d3e477ad8d4)
- Fix the access token `sub` claim (see breaking changes above)
  [29dbe26](https://github.com/sebadob/rauthy/commit/29dbe26a5c8f76d9a61931078811192ac2fb782d)
- Fix a short route fallback flash during Admin UI logout
  [6787261](https://github.com/sebadob/rauthy/commit/67872618949bc3b13e95d144d9c434f97174b395)

## v0.20.1

This is a small bugfix release.  
The temp migrations which exist for v0.20 only to migrate existing database secrets to
[cryptr](https://github.com/sebadob/cryptr) were causing a crash at startup for a fresh installation. This is the only
thing that has been fixed
with this version. They are now simply ignored and a warning is logged into the console at the very first startup.

## v0.20.0

### Breaking

This update is not backwards-compatible with any previous version. It will modify the database under the hood
which makes it incompatible with any previous version. If you need to downgrade for whatever reason, you will
only be able to do this by applying a database backup from an older version.
Testing has been done and everything was fine in tests. However, if you are using Rauthy in production, I recommend
taking a database backup, since any version <= v0.19 will not be working with a v0.20+ database.

### IMPORTANT Upgrade Notes

If you are upgrading from any earlier version, there is a manual action you need to perform, before you can
start v0.20.0. If this has not been done, it will simply panic early and not start up. Nothing will get damaged.

The internal encryption of certain values has been changed. Rauthy now uses [cryptr](https://github.com/sebadob/cryptr)
to handle these things,
like mentioned below as well.

However, to make working with encryption keys easier and provide higher entropy, the format has changed.
You need to convert your currently used `ENC_KEYS` to the new format:

#### Option 1: Use `cryptr` CLI

**1. Install cryptr - https://github.com/sebadob/cryptr**

If you have Rust available on your system, just execute:

  ```
  cargo install cryptr --features cli --locked
  ```

Otherwise, pre-built binaries do exist:

Linux: https://github.com/sebadob/cryptr/raw/main/out/cryptr_0.2.2

Windows: https://github.com/sebadob/cryptr/raw/main/out/cryptr_0.2.2.exe

**2. Execute:**

  ```
  cryptr keys convert legacy-string
  ```

**3. Paste your current ENC_KEYS into the command line.**

For instance, if you have

  ```
  ENC_KEYS="bVCyTsGaggVy5yqQ/S9n7oCen53xSJLzcsmfdnBDvNrqQ63r4 q6u26onRvXVG4427/3CEC8RJWBcMkrBMkRXgx65AmJsNTghSA"
  ```

in your config, paste

  ```
  bVCyTsGaggVy5yqQ/S9n7oCen53xSJLzcsmfdnBDvNrqQ63r4 q6u26onRvXVG4427/3CEC8RJWBcMkrBMkRXgx65AmJsNTghSA
  ```

If you provide your ENC_KEYS via a Kubernetes secret, you need to do a base64 decode first.
For instance, if your secret looks something like this

  ```
  ENC_KEYS: YlZDeVRzR2FnZ1Z5NXlxUS9TOW43b0NlbjUzeFNKTHpjc21mZG5CRHZOcnFRNjNyNCBxNnUyNm9uUnZYVkc0NDI3LzNDRUM4UkpXQmNNa3JCTWtSWGd4NjVBbUpzTlRnaFNB
  ```

Then decode via shell or any tool your like:

  ```
  echo -n YlZDeVRzR2FnZ1Z5NXlxUS9TOW43b0NlbjUzeFNKTHpjc21mZG5CRHZOcnFRNjNyNCBxNnUyNm9uUnZYVkc0NDI3LzNDRUM4UkpXQmNNa3JCTWtSWGd4NjVBbUpzTlRnaFNB | base64 -d
  ```

... and paste the decoded value into cryptr

**4. cryptr will output the correct format for either usage in config or as kubernetes secret again**

**5. Paste the new format into your Rauthy config / secret and restart.**

#### Option 2: Manual

Rauthy expects the `ENC_KEYS` now base64 encoded, and instead of separated by whitespace it expects them to
be separated by `\n` instead.  
If you don't want to use `cryptr` you need to convert your current keys manually.

For instance, if you have

```
ENC_KEYS="bVCyTsGaggVy5yqQ/S9n7oCen53xSJLzcsmfdnBDvNrqQ63r4 q6u26onRvXVG4427/3CEC8RJWBcMkrBMkRXgx65AmJsNTghSA"
```

in your config, you need to convert the enc key itself, the value after the `/`, to base64, and then separate
them with `\n`.

For instance, to convert `bVCyTsGaggVy5yqQ/S9n7oCen53xSJLzcsmfdnBDvNrqQ63r4`, split off the enc key part
`S9n7oCen53xSJLzcsmfdnBDvNrqQ63r4` and encode it with base64:

```
echo -n 'S9n7oCen53xSJLzcsmfdnBDvNrqQ63r4' | base64
```

Then combine the result with the key id again to:

```
bVCyTsGaggVy5yqQ/UzluN29DZW41M3hTSkx6Y3NtZmRuQkR2TnJxUTYzcjQ=
```

Do this for every key you have. The `ENC_KEYS` should then look like this in the end:

```
ENC_KEYS="
bVCyTsGaggVy5yqQ/UzluN29DZW41M3hTSkx6Y3NtZmRuQkR2TnJxUTYzcjQ=
q6u26onRvXVG4427/M0NFQzhSSldCY01rckJNa1JYZ3g2NUFtSnNOVGdoU0E=
"
```

**Important:**  
Make sure to not add any newline characters or spaces when copying values around when doing the bas64 encoding!

### Encrypted SQLite backups to S3 storage

Rauthy can now push encrypted SQLite backups to a configured S3 bucket.
The local backups to `data/backups/` do still exist. If configured, Rauthy will now push backups from SQLite
to an S3 storage and encrypt them on the fly. All this happens with the help
of [cryptr](https://github.com/sebadob/cryptr)
which is a new crate of mine. Resource usage is minimal, even if the SQLite file would be multiple GB's big.
The whole operation is done with streaming.

### Auto-Restore SQLite backups

Rauthy can now automatically restore SQLite backups either from a backup inside `data/backups/` locally, or
fetch an encrypted backup from an S3 bucket. You only need to set the new `RESTORE_BACKUP` environment variable
at startup and Rauthy will do the rest. No manually copying files around.  
For instance, a local backup can be restored with setting `RESTORE_BACKUP=file:rauthy-backup-1703243039` and an
S3 backup with `RESTORE_BACKUP=s3:rauthy-0.20.0-1703243039.cryptr`.

### Test S3 config at startup

To not show unexpected behavior at runtime, Rauthy will initialize and test a configured S3 connection
at startup. If anything is not configured correctly, it will panic early. This way, when Rauthy starts
and the tests are successful, you know it will be working during the backup process at night as well, and
it will not crash and throw errors all night long, if you just had a typo somewhere.

### Migration to `spow`

The old (very naive) Proof-of-Work (PoW) mechanism for bot and spam protection has been migrated to make use
of the [spow](https://github.com/sebadob/spow) crate, which is another new project of mine.
With this implementation, the difficulty for PoW's a client must solve can be scaled up almost infinitely,
while the time is takes to verify a PoW on the server side will always be `O(1)`, no matter hoch high the
difficulty was. `spow` uses a modified version of the popular Hashcat PoW algorithm, which is also being used
in the Bitcoin blockchain.

### Separate users cache

A typical Rauthy deployment will have a finite amount of clients, roles, groups, scopes, and so on.
The only thing that might scale endlessly are the users. Because of this, the users are now being cached
inside their own separate cache, which can be configured and customized to fit the deployment's needs.
You can now set the upper limit and the lifespan for cached user's. This is one of the first upcoming
optimizations, since Rauthy gets closer to the first v1.0.0 release:

### E-Mails as lowercase only

Up until now, it was possible to register the same E-Mail address multiple times with using uppercase characters.
E-Mail is case-insensitive by definition though. This version does a migration of all currently existing E-Mail
addresses
in the database to lowercase only characters. From that point on, it will always convert any address to lowercase only
characters to avoid confusion and conflicts.  
This means, if you currently have the same address in your database with different casing, you need to resolve this
issue manually. The migration function will throw an error in the console at startup, if it finds such a conflict.

```
# The max cache size for users. If you can afford it memory-wise, make it possible to fit
# all active users inside the cache.
# The cache size you provide here should roughly match the amount of users you want to be able
# to cache actively. Depending on your setup (WebIDs, custom attributes, ...), this number
# will be multiplied internally  by 3 or 4 to create multiple cache entries for each user.
# default: 100
CACHE_USERS_SIZE=100

# The lifespan of the users cache in seconds. Cache eviction on updates will be handled automatically.
# default: 28800
CACHE_USERS_LIFESPAN=28800
```

### Additional claims available in ID tokens

The scope `profile` now additionally adds the following claims to the ID token (if they exist for the user):

- `locale`
- `birthdate`

The new scope `address` adds:

- `address` in JSON format

  The new scope `phone` adds:
- `phone`

### Changes

- new POST `/events` API endpoint which serves archived events
  [d5d4b01](https://github.com/sebadob/rauthy/commit/d5d4b0145768982f20ebc1dbbfc568a73d7937bd)
- new admin UI section to fetch and filter archived events.
  [ece73bb](https://github.com/sebadob/rauthy/commit/ece73bb38878d8d189d52855845c63fa729cae2a)
- backend + frontend dependencies have been updated to the latest versions everywhere
- The internal encryption handling has been changed to a new project of mine
  called [cryptr](https://github.com/sebadob/cryptr).  
  This makes the whole value encryption way easier, more stable and future-proof, because values have their own
  tiny header data with the minimal amount of information needed. It not only simplifies encryption key rotations,
  but also even encryption algorithm encryptions really easy in the future.
  [d6c224e](https://github.com/sebadob/rauthy/commit/d6c224e98198c155d7df83c25edc5c97ab590d2a)
  [c3df3ce](https://github.com/sebadob/rauthy/commit/c3df3cedbdff4a2a9dd592aac65ae21e5cd67385)
- Push encrypted SQLite backups to S3 storage
  [fa0e496](https://github.com/sebadob/rauthy/commit/fa0e496956769f995e22bc6e388a8cd2a88a34d3)
- S3 connection and config test at startup
  [701c785](https://github.com/sebadob/rauthy/commit/701c7851dd872c337e89227d56a3e46f2a05ac3e)
- Auto-Restore SQLite backups either from file or S3
  [65bbfea](https://github.com/sebadob/rauthy/commit/65bbfea5a1a3b23735b82f3eb05a415ce7c51013)
- Migrate to [spow](https://github.com/sebadob/spow)
  [ff579f6](https://github.com/sebadob/rauthy/commit/ff579f60414cb529d727ae27fd83e9506ad770d5)
- Pre-Compute CSP's for all HTML content at build-time and get rid of the per-request nonce computation
  [8fd2c99](https://github.com/sebadob/rauthy/commit/8fd2c99d25aea2f307e0197f6f91a585b4408dce)
- `noindex, nofollow` globally via headers and meta tag -> Rauthy as an Auth provider should never be indexed
  [38a2a52](https://github.com/sebadob/rauthy/commit/38a2a52fe6530cf4efdedfe96d2b3041959fcd3d)
- push users into their own, separate, configurable cache
  [3137927](https://github.com/sebadob/rauthy/commit/31379278440ec6ddaf1a2288ba3950ab60994963)
- Convert to lowercase E-Mail addresses, always, everywhere
  [a137e96](https://github.com/sebadob/rauthy/commit/a137e963d2c409749b65240ebd9f5b0587c96938)
  [2467227](https://github.com/sebadob/rauthy/commit/24672277e58694d9b23ce12da932ba515eb8674e)
- add additional user values matching OIDC default claims
  [fca0c13](https://github.com/sebadob/rauthy/commit/fca0c1306624bdffa112ad8239e381064cb0b843)
- add `address` and `phone` default OIDC scopes and additional values for `profile`
  [3d497a2](https://github.com/sebadob/rauthy/commit/3d497a2fb7d91952d82d6d0efaea891a8088f523)

### Bugfixes

- A visual bugfix appeared on Apple systems because of the slightly bigger font size. This made
  the live events look a bit ugly and characters jumping in a line where they should never end up.
  [3b56b50](https://github.com/sebadob/rauthy/commit/3b56b50f4f24b7707c522934f9c03714703c64ad)
- An incorrect URL has been returned for the `end_session_endpoint` in the OIDC metadata
  [3caabc9](https://github.com/sebadob/rauthy/commit/3caabc98b24893ecadcd2f9219783a202c12f730)
- Make the `ItemTiles` UI componend used for roles, groups, and so on, wrap nicely on smaller screens
  [6f83e4a](https://github.com/sebadob/rauthy/commit/6f83e4a917ffd4eaec9b543b66170dc5ea76ed6e)
- Show the corresponding E-Mail address for `UserPasswordReset` and `UserEmailChange` events in the UI
  [7dc4794](https://github.com/sebadob/rauthy/commit/7dc47945ec3fdd335ed486465f98cdbb4734d653)

## v0.19.2

### Changes

- Invalidate all user sessions after a password reset to have a more uniform flow and better UX
  [570dea6](https://github.com/sebadob/rauthy/commit/570dea6379d1bba061a8b7acee64d9e36cf52733)
- Add an additional foreign key constraint on the `user_attr_values` table to cascade rows
  on user deletion for enhanced stability during user deletions
  [1dc730c](https://github.com/sebadob/rauthy/commit/1dc730c78c7f4e0a9c6b0b1b3dab8a35a4893b47)

### Bugfixes

- Fix a bug when an existing user with already registered passkeys would not be able
  to use the password reset functionality correctly when opened in a fully fresh browser
  [24af03c](https://github.com/sebadob/rauthy/commit/24af03c0365faad2108516bba3174d208e52d616)
- Fix cache evictions of existing user sessions after a user has been deleted while having
  an active session
  [ed76418](https://github.com/sebadob/rauthy/commit/ed764180fff5ed1f4aa6f497b0fc2362412db7d7)
- Fix some default values in the reference config docs not having the correct default documented
  [a7101b2](https://github.com/sebadob/rauthy/commit/a7101b25d72ced35e7bc22c7a084816ce7fa7635)

## v0.19.1

This is a small bugfix and compatibility release regarding password reset E-Mails.

The main reason for this release are problems with the Password Reset via E-Mail when users
are using Microsoft (the only service provider where this problems can be replicated 100% of the time)
and / or Outlook. These users were unable to use password reset links at all.
The reason is a "Feature" from Microsoft. They fully scan the user's E-Mails and even follow all links
inside it. The problem is, that the binding cookie from Rauthy will go to the Microsoft servers instead
of the user, making is unusable and basically invalidating everything before the user has any chance to
use the link properly.

The usage of this config variable is **highly discouraged,** and you should **avoid it, if you can**.
However, big enterprises are moving slowly (and often not at all). This new config variable can be used
as a last resort, to make it usable by giving up some security.

```
# This value may be set to 'true' to disable the binding cookie checking
# when a user uses the password reset link from an E-Mail.
#
# When using such a link, you will get a so called binding cookie. This
# happens on the very first usage of such a reset link. From that moment on,
# you will only be able to access the password reset form with this very 
# device and browser. This is just another security mechanism and prevents
# someone else who might be passively sniffing network traffic to extract 
# the (unencrypted) URI from the header and just use it, before the user 
# has a change to fill out the form. This is a mechanism to prevent against
# account takeovers during a password reset.
#
# The problem however are companies (e.g. Microsoft) who scan their customers
# E-Mails and even follow links and so on. They call it a "feature". The
# problem is, that their servers get this binding cookie and the user will be
# unable to use this link himself. The usage of this config option is highly
# discouraged, but since everything moves very slow in big enterprises and
# you cannot change your E-Mail provider quickly, you can use it do just make
# it work for the moment and deal with it later.
#
# default: false
#UNSAFE_NO_RESET_BINDING=false
```

### Changes

- implement `UNSAFE_NO_RESET_BINDING` like mentioned above
  [1f4a146](https://github.com/sebadob/rauthy/commit/1f4a1462697e85e068edd6bbd3f670f9d1ed985b)
- prettify the expiry timestamp in some E-Mails
  [1173fa0](https://github.com/sebadob/rauthy/commit/1173fa0f5ac517c7797c34e1240cbd37cb54dae6)

### Bugfixes

- It was possible to get an "Unauthorized Session" error during a password reset, if it has been
  initiated by an admin and / or from another browser.
  [e5d1d9d](https://github.com/sebadob/rauthy/commit/e5d1d9dd30452fdf5c33cc8e1cfac9670a514c74)
- Correctly set `ML_LT_PWD_FIRST` - set the default value in minutes (like documented) instead
  of seconds. New default is `ML_LT_PWD_FIRST=4320`
  [e9d1b56](https://github.com/sebadob/rauthy/commit/e9d1b5627809825241fcb0dbea4935f76d1334f1)

## v0.19.0

### Solid OIDC Support

This is the main new feature for this release.

With the now accepted `RSA` signatures for DPoP tokens, the ephemeral, dynamic clients and
the basic serving of `webid` documents for each user, Rauthy should now fully support Solid OIDC.
This feature just needs some more real world testing with already existing applications though.

These 3 new features are all opt-in, because a default deployment of Rauthy will most probably
not use them at all. There is a whole new section in the [Config](https://sebadob.github.io/rauthy/config/config.html)
called `EPHEMERAL CLIENTS` where you can configure these things. The 3 main variables you need
to set are:

```
# Can be set to 'true' to allow the dynamic client lookup via URLs as
# 'client_id's during authorization_code flow initiation.
# default: false
ENABLE_EPHEMERAL_CLIENTS=true

# Can be set to 'true' to enable WebID functionality like needed
# for things like Solid OIDC.
# default: false
ENABLE_WEB_ID=true

# If set to 'true', 'solid' will be added to the 'aud' claim from the ID token
# for ephemeral clients.
# default: false
ENABLE_SOLID_AUD=true
```

Afterward, the only "manual" thing you need to do is to add a custom scope called `webid`
once via the Admin UI.

### `EVENT_MATRIX_ERROR_NO_PANIC`

This new config variable solves a possible chicken and egg problem, if you use a self-hosted
Matrix server and Rauthy as its OIDC provider at the same time. If both services are offline,
for instance because of a server reboot, you would not be able to start them.

- The Matrix Server would panic because it cannot connect to and verify Rauthy
- Rauthy would panic because it cannot connect to Matrix

Setting this variable to `true` solves this issue and Rauthy would only log an error in that
case instead of panicking. The panic is the preferred behavior though, because this makes
100% sure that Rauthy will actually be able to send out notification to configured endpoints.

### Features

- ~20% smaller binary size by stripping unnecessary symbols
  [680d5e5](https://github.com/sebadob/rauthy/commit/680d5e5926481947324d8e5868a9c4d903ead05a)
- Accept `DPoP` tokens with `RSA` validations
  [daade41](https://github.com/sebadob/rauthy/commit/daade41a4ff22980d41e54570462eef783607766)
- Dynamically build up and serve custom scopes in the `/.well-known/openid-configuration`
  [904cf09](https://github.com/sebadob/rauthy/commit/904cf090a1f3070a33dbbac8c503b7190dd6ee47)
- A much nicer way of generating both DEV and PROD TLS certificates by using [Nioca](https://github.com/sebadob/nioca)
  has been integrated into the project itself, as well as the
  [Rauthy Book](https://sebadob.github.io/rauthy/config/tls.html)
  [463bf8a](https://github.com/sebadob/rauthy/commit/463bf8a40bf71e588a0449d647714acc96c68f83)
  [a14beda](https://github.com/sebadob/rauthy/commit/a14beda84942ecb17ac56d15b52e4668ebb12b41)
- Implement opt-in ephemeral clients
  [52c84c2](https://github.com/sebadob/rauthy/commit/52c84c2343447698978be4e25e5f537aad0070e0)
  [617908b](https://github.com/sebadob/rauthy/commit/617908bada3ba3e16d352e797e25c2d62eb512a6)
- Implement opt-in basic `webid` document serving
  [bca77f5](https://github.com/sebadob/rauthy/commit/bca77f5612a2a374d8e34ca8a717d94832328e7f)
  [1e32f6f](https://github.com/sebadob/rauthy/commit/1e32f6f93dd09aacabd6ea8596d028818691624e)
  [79cb836](https://github.com/sebadob/rauthy/commit/79cb83622fe508b3d296e9a6a71f5d6761a6b83a)
  [55433f4](https://github.com/sebadob/rauthy/commit/55433f4c614b7660dad32ad2321ff86367d8892e)
  [3cdf81c](https://github.com/sebadob/rauthy/commit/3cdf81c03cf9b78177412069264fa76f4d770ecc)
- For developers, a new [CONTRIBUTING.md](https://github.com/sebadob/rauthy/blob/main/CONTRIBUTING.md)
  guide has been added to get people started quickly
  [7c38142](https://github.com/sebadob/rauthy/commit/7c381428f74210a2dc56a5d995ab76485a3686ad)
  [411393f](https://github.com/sebadob/rauthy/commit/411393faab2d4f0242ea6fc1414d501c1260d50c)
- Add a new config variable `EVENT_MATRIX_ERROR_NO_PANIC` to only throw an error instead of
  panic on Matrix connection errors
  [4fc3382](https://github.com/sebadob/rauthy/commit/4fc3382929e65780fb20a78994233357423f0aab)
- Not really a bug nor a feature, but the "App Version Update" watcher now remembers a
  sent notification for an update and will only notify after a restart again.
  [be19735](https://github.com/sebadob/rauthy/commit/be197355437d0338041cc3206421ec638ca938d7)

### Bugfixes

- In a HA deployment, the new integrated health watcher from v0.17.0 could return false positives
  [93d75d5](https://github.com/sebadob/rauthy/commit/93d75d5d97e92d20a54f4781cea0f0b186b1098d)
  [9bbaeb2](https://github.com/sebadob/rauthy/commit/9bbaeb2f0b582838398547ddb477b7f8ab537a30)
- In v0.18.0 a bug has been introduced because of internal JWKS optimizations. This produced
  cache errors when trying to deserialize cached JWKS after multiple requests.
  [3808423](https://github.com/sebadob/rauthy/commit/3808423c8c13c06cdd82f6d97a9ef01486561a79)

## v0.18.0

This is a rather small release.
The main reason it is coming so early is the license change.

### License Change To Apache 2.0

With this release, the license of Rauthy is changed from the AGPLv3 to an Apache 2.0.
The Apache is way more permissive and make the integration with other open source projects and software a lot easier.

### DPoP Token Support (Experimental)

The first steps towards DPoP Token support have been made.
It is marked as experimental though, because the other authentication methods have been tested and verified with
various real world applications already. This is not the case for DPoP yet.
Additionally, the only supported alg for DPoP proofs is EdDSA for now. The main reason being that I am using Jetbrains
IDE's and the Rust plugin for both IDEA and RustRover are currently broken in conjunction with the `rsa` crate
(and some others) which makes writing code with them a nightmare. RSA support is prepared as much as possible
though and I hope they will fix this bug soon, so it can be included.

If you have or use a DPoP application, I would really appreciate testing with Rauthy and to get some feedback, so I
can make the whole DPoP flow more resilient as well.

Please note that Authorization Code binding to a DPoP key is also not yet supported, only the `/token` endpoint accepts
and validates the `DPoP` header for now.

### Changes

- Experimental DPoP support
  [803e312](https://github.com/sebadob/rauthy/commit/803e312194b67a20e475e685bd1fecf69d8a98fa)
  [51dc320](https://github.com/sebadob/rauthy/commit/51dc320291b125cee5c56e97eda54a12149caf33)
  [4329303](https://github.com/sebadob/rauthy/commit/43293030bf575175706357dda7ebed0beb6684dd)
  [29dce66](https://github.com/sebadob/rauthy/commit/29dce66aa167e789fab689fe680d07201b1126c7)

### Bugfixes

- Typos have been changed in docs and config
  [51dc320](https://github.com/sebadob/rauthy/commit/51dc320291b125cee5c56e97eda54a12149caf33)
- Listen Scheme was not properly set when only HTTP was selected exclusively
  [c002fbe](https://github.com/sebadob/rauthy/commit/c002fbed506e9dae2024e6e3ea5af33b7429cf11)
- Resource links in default error HTML template did not work properly in all locations
  [5965d9a](https://github.com/sebadob/rauthy/commit/5965d9aa1f7d7144786a2f6d73ce966570da5467)

### New Contributors

[twistedfall](https://github.com/twistedfall)

## v0.17.0

This is a pretty huge update with a lot of new features.

### New Features

#### Support for `linux/arm64`

With the release of v0.17.0, Rauthy's container images are now multi-platform.

Both a `linux/amd64` and a `linux/arm64` are supported. This means you can "just use it" now on Raspberry Pi and
others, or on Ampere architecture from Cloud providers without the need to compile it yourself.

#### Events and Auditing

Rauthy now produces events in all different kinds of situations. These can be used for auditing, monitoring, and so on.
You can configure quite a lot for them in the new `EVENTS / AUDIT` section in the
[Rauthy Config](https://sebadob.github.io/rauthy/config/config.html).

These events are persisted in the database, and they can be fetched in real time via a new Server Sent Events(SSE)
endpoint `/auth/v1/events/stream`. There is a new UI component in the Admin UI that uses the same events stream.
In case of a HA deployment, Rauthy will use one additional DB connection (all the time) from the connection pool
to distribute these events via pg listen / notify to the other members. This makes a much simpler deployment and
there is no real need to deploy additional resources like Nats or something like that. This keeps the setup easier
and therefore more fault-tolerant.

> You should at least set `EVENT_EMAIL` now, if you update from an older version.

#### Slack and Matrix Integrations

The new Events can be sent to a Slack Webhook or Matrix Server.

The Slack integration uses the simple (legacy) Slack Webhooks and can be configured with `EVENT_SLACK_WEBHOOK`:

```
# The Webhook for Slack Notifications.
# If left empty, no messages will be sent to Slack.
#EVENT_SLACK_WEBHOOK=
```

The Matrix integration can connect to a Matrix server and room. This setup requires you to provide a few more
variables:

```
# Matrix variables for event notifications.
# `EVENT_MATRIX_USER_ID` and `EVENT_MATRIX_ROOM_ID` are mandatory.
# Depending on your Matrix setup, additionally one of
# `EVENT_MATRIX_ACCESS_TOKEN` or `EVENT_MATRIX_USER_PASSWORD` is needed.
# If you log in to Matrix with User + Password, you may use `EVENT_MATRIX_USER_PASSWORD`.
# If you log in via OIDC SSO (or just want to use a session token you can revoke),
# you should provide `EVENT_MATRIX_ACCESS_TOKEN`.
# If both are given, the `EVENT_MATRIX_ACCESS_TOKEN` will be preferred.
#
# If left empty, no messages will be sent to Slack.
# Format: `@<user_id>:<server address>`
#EVENT_MATRIX_USER_ID=
# Format: `!<random string>:<server address>`
#EVENT_MATRIX_ROOM_ID=
#EVENT_MATRIX_ACCESS_TOKEN=
#EVENT_MATRIX_USER_PASSWORD=
# Optional path to a PEM Root CA certificate file for the Matrix client.
#EVENT_MATRIX_ROOT_CA_PATH=tls/root.cert.pem
# May be set to disable the TLS validation for the Matrix client.
# default: false
#EVENT_MATRIX_DANGER_DISABLE_TLS_VALIDATION=false
```

You can configure the minimum event level which would trigger it to be sent:

```
# The notification level for events. Works the same way as a logging level. For instance:
# 'notice' means send out a notifications for all events with the info level or higher.
# Possible values:
# - info
# - notice
# - warning
# - critical
#
# default: 'warning'
EVENT_NOTIFY_LEVEL_EMAIL=warning
# default: 'notice'
EVENT_NOTIFY_LEVEL_MATRIX=notice
# default: 'notice'
EVENT_NOTIFY_LEVEL_SLACK=notice
```

#### Increasing Login Timeouts

Up until version 0.16, a failed login would extend the time the client needed to wait for the result
artificially until it ended up in the region of the median time to log in successfully.
This was already a good thing to do to prevent username enumeration.
However, this has been improved a lot now.

When a client does too many invalid logins, the time he needs to wait until he may do another try
increases with each failed attempt. The important thing here is, that this is not bound to a user,
but instead to the clients IP.  
This makes sure, that an attacker cannot just lock a users account by doing invalid logins and therefore
kind of DoS the user. Additionally, Rauthy can detect Brute-Force or DoS attempts independently of
a users account.

There are certain thresholds at 7, 10, 15, 20, 25 invalid logins, when a clients IP will get fully
blacklisted (explained below) for a certain amount of time. This is a good DoS and even DDoS prevention.

#### Ip Blacklist Middleware

This is a new HTTP middleware which checks the clients IP against an internal blacklist.

This middleware is the very first thing that is being executed and just returns an HTML page
to a blacklisted client with the information about the blacklisting and the expiry.  
This blacklist is in-memory only to be as fast as possible to actually be able to handle brute
force and DoS attacks in the best way possible while consuming the least amount of resources
to do this.

Currently, IP's may get blacklisted in two ways:

- Automatically when exceeding the above-mentioned thresholds for invalid logins in a row
- Manually via the Admin UI

Blacklisted IP's always have an expiry and will get removed from the blacklist automatically.
Both actions will trigger one of the new Rauthy Events and send out notifications.

#### JWKS Auto-Rotate Scheduler

This is a simple new cron job which rotates the JSON Web Key Set (JWKS) automatically for
enhanced security, just in case one of the keys may get leaked at some point.

By default, it runs every first day of the month. This can be adjusted in the config:

```
# JWKS auto rotate cronjob. This will (by default) rotate all JWKs every
# 1. day of the month. If you need smaller intervals, you may adjust this
# value. For security reasons, you cannot fully disable it.
# In a HA deployment, this job will only be executed on the current cache
# leader at that time.
# Format: "sec min hour day_of_month month day_of_week year"
# default: "0 30 3 1 * * *"
JWK_AUTOROTATE_CRON="0 30 3 1 * * *"
```

#### Fully Reworked Authentication Middleware

The authentication and authorization system has been fully reworked and improved.

The new middleware and way of checking the client's access rights in each endpoint is way
less error-prone than before. The whole process has been much simplified which indirectly
improves the security:

- CSRF Tokens are now checked automatically if the request method is any other than a `GET`
- `Bearer` Tokens are not allowed anymore to access the Admin API
- A new `ApiKey` token type has been added (explained below)
- Only a single authn/authz struct is needed to validate each endpoint
- The old permission extractor middleware was removed which also increases the performance a bit

#### New API-Key Type

This new API-Key type may be used, if you need to access Rauthy API from other applications.

Beforehand, you needed to create a "user" for an application, if you wanted to access the API,
which is kind of counter-intuitive and cumbersome.  
These new API-Keys can be used to handle this task now. These are static keys with an
optional expiry date and fine-grained access rights. You should only give them permissions
to the resources you actually need to further improve your backend security.

They can be easily created, configured and revoked / deleted in the Admin UI at any time.

> IMPORTANt: The API now cannot be accessed anymore with Bearer tokens! If you used this
> method until now, you need to switch to the new API Keys

#### OIDC Client FORCE_MFA feature

In the configuration for each individual OIDC client, you can find a new `FORCE MFA` switch.
It this new option is activated for a client, it will only issue authentication codes for
those users, that have at least one Passkey registered.  
This makes it possible to force MFA for all your different applications from Rauthy directly
without the need to check for the `amr` claim in the ID token and do or configure all of
this manually downstream. Most of the time, you may not even have control over the client
itself, and you are basically screwed, if the client does not have its own "force mfa integration".

> CAUTION: This mentioned in the UI as well, but when you check this new force mfa option,
> it can only force MFA for the `authorization_code` flow of course! If you use other flows,
> there just is no MFA that could be checked / forced.

#### Rauthy Version Checker

Since we do have an Events system now, there is a new scheduled cron job, which checks the
latest available Rauthy Version.

This Job runs once every 8 hours and does a single poll to the Github Releases API. It looks
for the latest available Rauthy Version that is not a prerelease or anything unstable.
If it finds a version higher than the currently running one, a new Event will be generated.
Additionally,
you will see the current Rauthy Version in the UI now and a small indicator just next to it,
if there is a stable update available.

### Changes

- Support for `linux/arm64`
  [2abb071](https://github.com/sebadob/rauthy/commit/2abb071afd6e9379fa3deca233c649bf62d33032)
- New events and auditing
  [758dda6](https://github.com/sebadob/rauthy/commit/758dda631734c0c8e5baddf79ff2b0aa67947929)
  [488f9de](https://github.com/sebadob/rauthy/commit/488f9de03653c5eb2c673644deb188599763afbb)
  [7b95acc](https://github.com/sebadob/rauthy/commit/7b95acc22470bc49af6cd1812d87a414d7b4176b)
  [34d8888](https://github.com/sebadob/rauthy/commit/34d8888faa3ee6740d633814bf29920d2c2856a6)
  [f70f0b2](https://github.com/sebadob/rauthy/commit/f70f0b2a20e9e3fc53958959238b9f8136b5c9f0)
  [5f0c9c9](https://github.com/sebadob/rauthy/commit/5f0c9c979c0ba3cb4d3a75cd78a899eee7d13464)
  [a9af494](https://github.com/sebadob/rauthy/commit/a9af494bba788e462bb22eb31131d19b5ffaeaed)
  [797dad5](https://github.com/sebadob/rauthy/commit/797dad564ff190b8739393c0405486b8f55b057e)
  [b338f26](https://github.com/sebadob/rauthy/commit/b338f2613e9d19581677915c5ceb1996653709d7)
- `rauthy-notify` crate has been added which implements the above-mentioned Slack and
  Matrix integrations for Event notifications.
  [8767389](https://github.com/sebadob/rauthy/commit/8767389dafe3dc392910135d8cfc7f6a63bf3cd5)
- Increasing login timeouts and delays after invalid logins
  [7f7a675](https://github.com/sebadob/rauthy/commit/7f7a675102f21aabe9f4cd2a5eef95d4947134d4)
  [5d19d2d](https://github.com/sebadob/rauthy/commit/5d19d2d6d4434b2192eb3c8860eb8111ffdb8d79)
- IpBlacklist Middleware
  [d69845e](https://github.com/sebadob/rauthy/commit/d69845ed772e1454e8ac902fdffb253416cbac14)
- IPBlacklist Admin UI Component
  [c76c208](https://github.com/sebadob/rauthy/commit/c76c20871a303cbf6cfc587483113cd544d65424)
- JWKS Auto-Rotate
  [cd087eb](https://github.com/sebadob/rauthy/commit/cd087eb8b475c9b8ae409e7a34bc880e76f2ee69)
- New Authentication Middleware
  [a097a5d](https://github.com/sebadob/rauthy/commit/a097a5d2d0d75940cc192b2b1a86f59e5a732a88)
- ApiKey Admin UI Component
  [53ffe49](https://github.com/sebadob/rauthy/commit/53ffe4938a8f5a2fd6d30188ac0a819f3eae2f96)
- OIDC Client `FORCE_MFA`
  [3efdcce](https://github.com/sebadob/rauthy/commit/3efdcceb2277b2f8b28572485cdd8137115d8b66)
- Rauthy Version Checker
  [aea7794](https://github.com/sebadob/rauthy/commit/aea77942b2488a26d12e7d191582996637079918)
  [41b4c9c](https://github.com/sebadob/rauthy/commit/41b4c9c19c061f36f3ed5fa97d75ddcf803e0614)
- Show a Link to the accounts page after a password reset if the redirection does not work
  [ace4daf](https://github.com/sebadob/rauthy/commit/ace4daf180b09f0cc8677bf4cf85e792496f9ee0)
- Send out E-Mail change confirmations E-Mails to both old and new address when an admin changes the address
  [8e97e31](https://github.com/sebadob/rauthy/commit/8e97e310a6369bd5150d087cd5cd402d8edc221e)
  [97197db](https://github.com/sebadob/rauthy/commit/97197dbcdf3268647a3269c5cf5648176f0000d7)
- Allow CORS requests to the `.well-known` endpoints to make the oidc config lookup from an external UI possible
  [b57656f](https://github.com/sebadob/rauthy/commit/b57656ffaf7e822527d2d8c9d74b7c948193c220)
- include the `alg` in the JWKS response for the `openid-configuration`
  [c9073cb](https://github.com/sebadob/rauthy/commit/c9073cb473be04092e326c95ca7a1b3502379f40)
- The E-Mail HTML templates have been optically adjusted a bit to make them "prettier"
  [926de6e](https://github.com/sebadob/rauthy/commit/926de6e7348a8294fe284500cb41381a56a5cd2b)

### Bugfixes

- A User may have not been updated correctly in the cache when the E-Mail was changed.
  [8d9cdce](https://github.com/sebadob/rauthy/commit/8d9cdce61992ee59e381876b71b18168e7e3ce31)
- With v0.16, it was possible to not be able to switch back to a password account type from passkey,
  when it was a password account before already which did update its password in the past and therefore
  would have entries in the DB for `last_recent_passwords` if you had the password policy correctly.
  [7a965a2](https://github.com/sebadob/rauthy/commit/7a965a276b2a127057dfd589afde23cf5ec981d1)
- When you were using a password manager that filled out the username 'again' in the login form,
  after the additional password request input showed up, it could reset the form on some browser.
  [09d1d3a](https://github.com/sebadob/rauthy/commit/09d1d3a7446d5f636b69046c8066d69b09537411)

### Removed

- The `ADMIN_ACCESS_SESSION_ONLY` config variable was removed. This was obsolete now with
  the introduction of the new ApiKey type.
  [b28d8ba](https://github.com/sebadob/rauthy/commit/b28d8baa50f778867647535cf204cf87a896968b)

## v0.16.1

This is a small bugfix release.  
If you had an active and valid session from a v0.15.0, did an update to v0.16.0 and your session was still valid,
it did not have valid information about the peer IP. This is mandatory for a new feature introduced with v0.16.0
though and the warning logging in that case had an unwrap for the remote IP (which can never be null from v0.16.0 on),
which then would panic.

This is a tiny bugfix release that just gets rid of the possible panic and prints `UNKNOWN` into the logs instead.

### Changes

- print `UNKNOWN` into the logs for non-readable / -existing peer IPs
  [6dfd0f4](https://github.com/sebadob/rauthy/commit/6dfd0f4299b90c03d9b5a2fb4106d72f153146af)

## v0.16.0

### Breaking

This version does modify the database and is therefore not backwards compatible with any previous version.
If you need to downgrade vom v0.15 and above, you will only be able to do this via by applying a DB Backup.

### New Features

#### User Expiry

It is now possible to limit the lifetime of a user.  
You can set an optional expiry date and time for each user. This enables temporary access, for instance in
a support case where an external person needs access for a limited time.

Once a user has expired, a few things will happen:

- The user will not be able to log in anymore.
- As soon as the scheduler notices the expiry, it will automatically invalidate all possibly existing
  sessions and refresh tokens for this user. How often the scheduler will run can be configured with the
  `SCHED_USER_EXP_MINS` variable. The default is 'every 60 minutes' to have a good balance between security
  and resource usage. However, if you want this to be very strict, you can adjust this down to something like
  '5 minutes' for instance.
- If configured, expired users can be cleaned up automatically after the configured time.
  By default, expired users will not be cleaned up automatically. You can enable this feature with the
  ´SCHED_USER_EXP_DELETE_MINS` variable.

#### `WEBAUTHN_NO_PASSWORD_EXPIRY`

With this new config variable, you can define, if users with at least one valid registered passkey will
have expiring passwords (depending on the current password policy), or not.  
By default, these users do not need to renew their passwords like it is defined in the password policy.

#### Peer IP's for sessions -> `SESSION_VALIDATE_IP`

When a new session is being created, the peer / remote IP will be extracted and saved with the session
information. This peer IP can be checked with each access and the session can be rejected, if this IP
has changed, which will force the user to do a new login.

This will of course happen if a user is "on the road" and uses different wireless networks on the way,
but it prevents a session hijack and usage from another machine, if an attacker has full access to the
victims machine and even can steal the encrypted session cookie and(!) the csrf token saved inside the
local storage. This is very unlikely, since the attacker would need to have full access to the machine
anyway already, but it is just another security mechanism.

If this IP should be validated each time can be configured with the new `SESSION_VALIDATE_IP` variable.
By default, peer IP's will be validated and a different IP for an existing session will be rejected.

#### Prometheus metrics exporter

Rauthy starts up a second HTTP Server for prometheus metrics endpoint and (optional) SwaggerUI.
By default, the SwaggerUI from the `Docs` link in the Admin UI will not work anymore, unless you
specify the SwaggerUI via config to be publicly available. This just reduces the possible attack
surface by default.

New config variables are:

```
# To enable or disable the additional HTTP server to expose the /metrics endpoint
# default: true
#METRICS_ENABLE=true

# The IP address to listen on for the /metrics endpoint.
# You do not want to expose your metrics on a publicly reachable endpoint!
# default: 0.0.0.0
#METRICS_ADDR=0.0.0.0

# The post to listen on for the /metrics endpoint.
# You do not want to expose your metrics on a publicly reachable endpoint!
# default: 9090
#METRICS_PORT=9090

# If the Swagger UI should be served together with the /metrics route on the internal server.
# It it then reachable via:
# http://METRICS_ADDR:METRICS_PORT/docs/v1/swagger-ui/
# (default: true)
#SWAGGER_UI_INTERNAL=true

# If the Swagger UI should be served externally as well. This makes the link in the Admin UI work.
#
# CAUTION: The Swagger UI is open and does not require any login to be seen!
# Rauthy is open source, which means anyone could just download it and see on their own,
# but it may be a security concern to just expose less information.
# (default: false)
#SWAGGER_UI_EXTERNAL=false
```

#### User Verification status for Passkeys

For all registered passkeys, the User Verification (UV) state is now being saved and optionally checked.
You can see the status for each device with the new fingerprint icon behind its name in the UI.

New config variable:

```
# This feature can be set to 'true' to force User verification during the Webauthn ceremony.
# UV will be true, if the user does not only need to verify its presence by touching the key,
# but by also providing proof that he knows (or is) some secret via a PIN or biometric key for
# instance. With UV, we have a true MFA scenario where UV == false (user presence only) would
# be a 2FA scenario (with password).
#
# Be careful with this option, since Android and some special combinations of OS + browser to
# not support UV yet.
# (default: false)
#WEBAUTHN_FORCE_UV=true
```

#### Passkey Only Accounts

This is the biggest new feature for this release. It allows user accounts to be "passkey only".

A passkey only account does not have a password. It works only with registered passkeys with
forced additional User Verification (UV).

Take a look at the updated documentation for further information:  
[Passkey Only Accounts]https://sebadob.github.io/rauthy/config/fido.html#passkey-only-accounts

#### New E-Mail verification flow

If an already existing user decides to change the E-Mail linked to the account, a new verification
flow will be started:

- A user changes the E-Mail in the Account view.
- The E-Mail will not be updated immediately, but:
    - A verification mail will be sent to the new address with an expiring magic link.
    - After the user clicked the link in the mail, the new address will be verified.
- Once a user verifies the new E-Mail:
    - The address will finally be updated in the users profile.
    - Information E-Mails about the change will be sent to the old and the new address

#### `EMAIL_SUB_PREFIX` config variable

This new variable allows you to add an E-Mail subject prefix to each mail that Rauthy sends out.
This makes it easier for external users to identify the email, what it is about and what it is
doing, in case the name 'Rauthy' does not mean anything to them.

```
# Will be used as the prefix for the E-Mail subject for each E-Mail that will be sent out to a client.
# This can be used to further customize your deployment.
# default: "Rauthy IAM"
EMAIL_SUB_PREFIX="Rauthy IAM"
```

#### New nicely looking error page template

In a few scenarios, for instance wrong client information for the authorization code flow or
a non-existing or expired magic link, Rauthy now does not return the generic JSON error response,
but actually a translated HTML page which informs the user in a nicer looking way about the problem.
This provides a way better user experience especially for all Magic Link related requests.

#### Rauhty DB Version check

This is an additional internal check which compares the version of the DB during startup and
the App version of Rauthy itself. This makes it possible to have way more stable and secure
migrations between versions in the future and helps prevent user error during upgrades.

### Changes

- legacy MFA app pieces and DB columns have been cleaned up
  [dce148a](https://github.com/sebadob/rauthy/commit/dce148a2d665836ec733b8d0904e9d3892c40b2f)
  [423db7a](https://github.com/sebadob/rauthy/commit/423db7a66c412ce9d04ec317cd0a237e278653bb)
- user expiry feature
  [bab6bfc](https://github.com/sebadob/rauthy/commit/bab6bfcecd62e44d23b9527009904c12a04e567a)
  [e63d1ce](https://github.com/sebadob/rauthy/commit/e63d1ce746dd27ed78acebb688d44fbf7a0b50dc)
  [566fff1](https://github.com/sebadob/rauthy/commit/566fff12e0a3e8ee2b867ecfd590cbe0ef24378d)
- `WEBAUTHN_NO_PASSWORD_EXPIRY` config variable
  [7e16b6e](https://github.com/sebadob/rauthy/commit/7e16b6e7c5e5038c653f55807e6aeed35e7536cb)
- `SESSION_VALIDATE_IP` config variable
  [828bcd2](https://github.com/sebadob/rauthy/commit/828bcd285b844d846f23e22e488a372dba0c59ff)
  [26924ff](https://github.com/sebadob/rauthy/commit/26924ffbc2054abc537bba4ef6cd98b50ad594c3)
- `/metrics` endpoint
  [085d412](https://github.com/sebadob/rauthy/commit/085d412e51c3f77449c39449206d296b710c1d9b)
- `WEBAUTHN_FORCE_UV` config variable
- Passkey Only Accounts
  [6c2406c](https://github.com/sebadob/rauthy/commit/6c2406c4f261ec0e784df6ee50efd837a86160f7)
- New E-Mail verification flow
  [260169d](https://github.com/sebadob/rauthy/commit/260169d289731c44289cd6fce1c642c6d26786ae)
- New nicely looking error page template
  [0e476ab](https://github.com/sebadob/rauthy/commit/0e476ab0ec8728450d83a315240dd870ac34cddf)
- Rust v1.73 update
  [43f5b61](https://github.com/sebadob/rauthy/commit/43f5b61c86d563cefaf2ea2ee832e3317bcfcfb7)
- `EMAIL_SUB_PREFIX` config variable
  [af85839](https://github.com/sebadob/rauthy/commit/af858398d61b2bea347dabcd4b7adf6a8c60f86d)
- Rauthy DB Version check
  [d2d9271](https://github.com/sebadob/rauthy/commit/d2d92716b93a9ed1e6dc02ce7906f0741044d464)
- Cleanup schedulers in HA_MODE run on leader only
  [4881dd8](https://github.com/sebadob/rauthy/commit/4881dd83a9b7d298fc998c635face7feb55549ff)
- Updated documentation and book
  [5bbaae9](https://github.com/sebadob/rauthy/commit/5bbaae9c24ce49dec0c10908d351257519279c9a)
- Updated dependencies
  [0f11923](https://github.com/sebadob/rauthy/commit/0f1192354c805832d8f252d3204f624deb9e6561)

## v0.15.0

### Breaking

This version does modify the database and is therefore not backwards compatible with any previous version.
If you need to downgrade vom v0.15 and above, you will only be able to do this via by applying a DB Backup.

### Changes

This release is all about new Passkey Features.

- A user is not limited to just 2 keys anymore
- During registration, you can (and must) provide a name for the passkey, which helps you identify and distinguish
  your keys, when you register multiple ones.
- The `exclude_credentials` feature is now properly used and working. This makes sure, that you cannot register the
  same Passkey multiple times.
- The Passkeys / MFA section in the Admin UI has been split from the User Password section to be more convenient to use

Commits:

- New Passkey Features  
  [d317d90](https://github.com/sebadob/rauthy/commit/d317d902f66613b743955117dbede3d52efa74b6)
  [cd5d086](https://github.com/sebadob/rauthy/commit/cd5d086e0b38d8d906d3ce698ed9860bec3d52ab)
  [61464bf](https://github.com/sebadob/rauthy/commit/61464bfcc69921fae0c826fe8c3bac73e290a242)
  [e70c5a7](https://github.com/sebadob/rauthy/commit/e70c5a7bbf8f40f90a9d8d3d6570736e5322a0c5)
  [bc75610](https://github.com/sebadob/rauthy/commit/bc75610d7aa4cfc35abf1e0ee67af3b4dc2eea82)
  [49e9630](https://github.com/sebadob/rauthy/commit/49e9630bb3ffc25be1523492ef47d2fc66dc041c)
- New config option `WEBAUTHN_FORCE_UV` to optionally reject Passkeys without user verification
  [c35ecc0](https://github.com/sebadob/rauthy/commit/c35ecc090b222196cf1ac095aa2b340edbeec0f5)
- Better `/token` endpoint debugging capabilities
  [87f7969](https://github.com/sebadob/rauthy/commit/87f79695b86f9b7199dbad244c61e2eb1a0c6bcb)

## v0.14.5

This is the last v0.14 release.  
The next v0.15 will be an "in-between-release" which will do some migration preparations for Webauthn
/ FIDO 2 updates and features coming in the near future.

- Removed duplicate `sub` claims from JWT ID Tokens
  [a35db33](https://github.com/sebadob/rauthy/commit/a35db330ff7c6ee680a7d834f08a3db077e08073)
- Small UI improvements:
    - Show loading indicator when doing a password change
    - The Loading animation was changes from JS to a CSS animation
      [abd0a06](https://github.com/sebadob/rauthy/commit/abd0a06280de4fedef9028f142b6e844bf132d80)
- Upgrades to actix-web 4.4 + rustls 0.21 (and all other minor upgrades)
  [070a453](https://github.com/sebadob/rauthy/commit/070a453aaa584ff8d024284de91477626fe5ea6c)

## v0.14.4

This release mostly finishes the translation / i18n part of rauthy for now and adds some other
smaller improvements.  
Container Images will be published with ghcr.io as well from now on. Since I am on the free plan
here, storage is limited and too old versions will be deleted at some point in the future.
However, I will keep pushing all of them to docker hub as well, where you then should be able
to find older versions too. ghcr.io is just preferred, because it is not so hardly rate limited
than the docker hub free tier is.

- Added translations for E-Mails
  [11544ac](https://github.com/sebadob/rauthy/commit/11544ac46fcddeb53a511b8ad702b1ad2868148e)
- Made all UI parts work on mobile (except for the Admin UI itself)
  [a4f31f2](https://github.com/sebadob/rauthy/commit/a4f31f22396b5767c5a2c20e0253171910296447)
  [4ee3540](https://github.com/sebadob/rauthy/commit/4ee3540d9b32e6f0e2fbd5cf5eabcd7736179da8)
- Images will be published on Github Container Registry as well from now on
  [cc15ea9](https://github.com/sebadob/rauthy/commit/cc15ea9cb5d7c0bd6d3cad1fea908e656f488e50)
- All dependencies have been updates in various places. This just keeps everything up to date
  and fixed some potential security issues in third party libraries

## v0.14.3

- UI: UX Improvements to Webauthn Login when the user lets the request time out
  [7683133](https://github.com/sebadob/rauthy/commit/76831338e88a36cc3166039493318c38cc7a1e49)
- UI: i18n for password reset page
  [27e620e](https://github.com/sebadob/rauthy/commit/27e620edb34d803259a000d837513920905d2332)
- Keep track of users chosen language in the database
  [7517693](https://github.com/sebadob/rauthy/commit/7517693ddec4f5e0215fcced9dde6322e665ff10)
- Make user language editable in the admin ui
  [77886a9](https://github.com/sebadob/rauthy/commit/77886a958655b721e96c092df739627d0d5d9172)
  [1061fc2](https://github.com/sebadob/rauthy/commit/1061fc212bb684b153ae99f1439ea312091e32cd)
- Update the users language in different places:
    - Language switch in the Account page
    - Fetch users chosen language from User Registration
    - Selector from Registration in Admin UI
      [5ade849](https://github.com/sebadob/rauthy/commit/5ade849dd5a0b139c73a61b8fefde03eff3036bf)

## v0.14.2

- Fix for the new LangSelector component on mobile view
- Add default translations (english) for the PasswordPolicy component for cases when it is used
  in a non-translated context
  [2f8a627](https://github.com/sebadob/rauthy/commit/2f8a6270f97df075d52507c0aa4e6850e5ef8edc)

## v0.14.1

Bugfix release for the Dockerfiles and Pagination in some places

- Split the Dockerfiles into separate files because the `ARG` introduced access rights problems
  [25e3918](https://github.com/sebadob/rauthy/commit/25e39189f95d21e84e24f6d21670709a7ee1effd)
- Small bugfix for the pagination component in some places
  [317dbad](https://github.com/sebadob/rauthy/commit/317dbadb0391b78aca0cdadc65dff79d9bf74717)

## v0.14.0

This release is mostly about UI / UX improvements and some smaller bugfixes.

- UI: Client side pagination added
  [60a499a](https://github.com/sebadob/rauthy/commit/60a499aee0ff071937a03a78759e447f0d477c90)
- Browsers' native language detection
  [884f599](https://github.com/sebadob/rauthy/commit/884f5995b71b8faf6ebcacf4331fdda6ccd78d57)
- `sqlx` v0.7 migration
  [7c7a380](https://github.com/sebadob/rauthy/commit/7c7a380bdac520df7c29d0d0f4b3b7d3d48be943)
- Docker container image split
  [adb3971](https://github.com/sebadob/rauthy/commit/adb397139627a4e3b3f72682457bdfd24b6ce9f4)
- Target database validation before startup
  [a68c652](https://github.com/sebadob/rauthy/commit/a68c6527a68b970221f3c48982cabc418aa81d39)
- UI: I18n (english and german currently) for: Index, Login, Logout, Account, User Registration
  [99e454e](https://github.com/sebadob/rauthy/commit/99e454ee459ac041c6975df99d481f0145cf7fa4)
  [dd2e9ae](https://github.com/sebadob/rauthy/commit/dd2e9ae579444359dc04db76bc1e13d3d0753fe6)
  [7b401f6](https://github.com/sebadob/rauthy/commit/7b401f6f0639c053b0e9475121f9ec814f80ef65)
- UI: Custom component to overwrite the browsers' native language
  [4208fdb](https://github.com/sebadob/rauthy/commit/4208fdb9904044f9c5b9ddfb5eb2c42c1264481a)
- Some Readme and Docs updates
  [e2ebef9](https://github.com/sebadob/rauthy/commit/e2ebef9c72d0f212ac5a4b39241b6d5d486bd8b0)
  [d0a71d6](https://github.com/sebadob/rauthy/commit/d0a71d641c3c829b33191cc2c0ff04e5f7d27017)
- The `sub` claim was added to the ID token and will contain the Users UID
  [6b0a8b0](https://github.com/sebadob/rauthy/commit/6b0a8b0679484c5515a068911810159a5a39e07d)

## v0.13.3

- UI: small visual bugfixes and improvements in different places
  [459bdbd](https://github.com/sebadob/rauthy/commit/459bdbd55ca60bdb0076908131c569a4dc653086)
  [57a5600](https://github.com/sebadob/rauthy/commit/57a56000f6ffecf46bd1d202a3bea5a2ded4985f)
- UI: All navigation routes can be reached via their own link now. This means a refresh of
  the page does not return to the default anymore
  [4999995](https://github.com/sebadob/rauthy/commit/49999950ac1ade24e433e911df84c99256a7f4d0)
  [7f0ac0b](https://github.com/sebadob/rauthy/commit/7f0ac0b0d1cf1e2c53881c4a4e010ce43cc2ec11)
  [cadaa40](https://github.com/sebadob/rauthy/commit/cadaa407efa9b70b5159e6ec42b5151f8ef79997)
- UI: added an index to the users table to prevent a rendering bug after changes
  [e35ffbe](https://github.com/sebadob/rauthy/commit/e35ffbe9cb4e14785c61249d141895c1a7fb4921)

## v0.13.2

- General code and project cleanup
  [4531ae9](https://github.com/sebadob/rauthy/commit/4531ae93d453429a54198211b7d122dada452ae4)
  [782bb9a](https://github.com/sebadob/rauthy/commit/782bb9adbbb12f77232b1820e7dd05265c0fdf00)
  [0c5ad02](https://github.com/sebadob/rauthy/commit/0c5ad02e369935b01aac46988a2242c859737e24)
  [e453142](https://github.com/sebadob/rauthy/commit/e45314269234612a3eec046073e988e260a7ca31)
  [85fbafe](https://github.com/sebadob/rauthy/commit/85fbafe5ef6b8f124af6af1508b6e2bab067a8ff)
- Created a `justfile` for easier development handling
  [4aa5b99](https://github.com/sebadob/rauthy/commit/4aa5b9993897e43dfc765eb2849172bc087ea34c)
  [1489efe](https://github.com/sebadob/rauthy/commit/1489efe139c0a0c79169f47ba4fc964cdc6b6e3e)
- UI: fixed some visual bugs and improved the rendering with larger default browser fonts
  [45334fd](https://github.com/sebadob/rauthy/commit/45334fd65049f2950dae3a2bc28c5667c275aa1d)

## v0.13.1

This is just a small bugfix release.

- UI Bugfix: Client flow updates were not applied via UI
  [6fe8fbc](https://github.com/sebadob/rauthy/commit/6fe8fbc1440498ea126a5aee5bed9dfe34e367d4)

## v0.13.0

- Improved container security: Rauthy is based off a Scratch container image by default now. This improved the security
  quite a lot, since you cannot even get a shell into the container anymore, and it reduced the image size by another
  ~4MB.  
  This makes it difficult however if you need to debug something, for instance when you use a SQLite deployment. For
  this reason, you can append `-debug` to a tag
  and you will get an Alpine based version just like before.
  [1a7e79d](https://github.com/sebadob/rauthy/commit/1a7e79dc96d27d8d180d1e4394644c8851cbdf70)
- More stable HA deployment: In some specific K8s HA deployments, the default HTTP2 keep-alive's from
  [redhac](https://github.com/sebadob/redhac) were not good enough and we got broken pipes in some environments which
  caused the leader to change often. This has been fixed
  in [redhac-0.6.0](https://github.com/sebadob/redhac/releases/tag/v0.6.0)
  too, which at the same time makes Rauthy HA really stable now.
- The client branding section in the UI has better responsiveness for smaller screens
  [dfaa23a](https://github.com/sebadob/rauthy/commit/dfaa23a30ccf77da2b29654c7dd3b41a4ca78168)
- For a HA deployment, cache modifications are now using proper HA cache functions. These default back to the single
  instance functions in non-HA mode since [redhac-0.6.0](https://github.com/sebadob/redhac/releases/tag/v0.6.0)
  [7dae043](https://github.com/sebadob/rauthy/commit/7dae043d7b42724adad85b5ed54f1dcd9d143d27)
- All static UI files are now precompressed with gzip and brotli to use even fewer resources
  [10ad51a](https://github.com/sebadob/rauthy/commit/10ad51a296c5a7596b34f9c726fe87480b6ec42c)
- CSP script-src unsafe-inline was removed in favor of custom nonce's
  [7de918d](https://github.com/sebadob/rauthy/commit/7de918d601007d2807701a096d6403bf2b3274c9)
- UI migrated to Svelte 4
  [21f73ab](https://github.com/sebadob/rauthy/commit/21f73abfb0332be3fc391b9d108655a0cd5a3cec)

## v0.12.0

Rauthy goes open source
