+++
date = '2025-09-28T20:59:17+02:00'
title = 'JWT wrangling in your terminal'
+++

If you are slinging [JWTs](https://en.wikipedia.org/wiki/JSON_Web_Token) around you'll often want to inspect the claims contained within to check expiry, user identity or whatever else.

I made a little alias to decode and pretty print the header and claims, as well as printing the `exp` and `iat` values as dates:

```sh
alias jwt="jq -R 'split(\".\") | .[0], .[1] | @base64d | fromjson | \
    (.. | select(has(\"exp\") or has(\"iat\")?)) |= \
    with_entries(\
    if .key == \"exp\" then .value=(.value|todateiso8601)\
    elif .key == \"iat\" then .value=(.value|todateiso8601)\
    else . end\
)'"
```

Stick this alias in your `.zshrc` or similar and then you can use it like so:

```sh
$ echo eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWUsImlhdCI6MTUxNjIzOTAyMn0.KMUFsIDTnFmyG3nMiGM6H9FNFUROf3wh7SmqJp-QV30 | jwt
{
  "alg": "PS384",
  "typ": "JWT"
}
{
  "sub": "1234567890",
  "name": "John Doe",
  "admin": true,
  "iat": "2018-01-18T01:30:22Z"
}
```

The only dependency is [jq](https://jqlang.org).
