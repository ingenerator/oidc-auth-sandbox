# OIDC Auth Sandbox

This is a very simple github pages repo that serves a publicly available set of keys, certs etc
for generating and validating OIDC (JWKS) tokens.

To generate a token, using  do something like:

```php
use Firebase\JWT\JWT;

$keys = json_decode(
  \file_get_contents('https://ingenerator.github.io/oidc-auth-sandbox/jwks-private-keys.json'),
  TRUE
);
$key = $keys['keys'][0];

$token = JWT::encode(
    [
        'aud'            => 'http://url.of.your.site.or.whatever',
        'email'          => 'dummy@some-fake-serviceaccount.com',
        'email_verified' => TRUE,
        'exp'            => time() + 30,
        'iat'            => time(),
        'iss'            => 'https://ingenerator.github.io/oidc-auth-sandbox',
    ],
    $key['private_key'],
    'RS256',
    $key['kid']
);
```

This will produce a JWT signed by this sandbox issuer.

On the server-side, you can then use the standard OIDC verfication flow to validate the key:
 * Fetch the discovery document from https://ingenerator.github.io/oidc-auth-sandbox/.well-known/openid-configuration -
   note that this has JSON content but is not served with an application/json content-type header due to the limitations
   of github pages.
 * Read the value of the `jwks_uri` in that file
 * Fetch the certificates from the `jwks_uri` (https://ingenerator.github.io/oidc-auth-sandbox/openid-jwks-certs.json)
 * Use the key with the specified ID to validate the JWT
 
It should go without saying that tokens signed by this issuer are **not in any way secure** so you should not be
using them for actual auth on anything whatsoever.
