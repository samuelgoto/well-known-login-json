# TL;DR;

This is a proposal to allow webites to expose a machine readable `.well-known/login.json` file that tells browsers/tools/agents/software how to construct a mediated login UI for the website.

# Relationship to other proposals

- This is intended to be used inside and/or outside of a browser enviornment too (i.e. outside of a JS/HTML context, e.g. in a native app or in a CLI).
- This is a `.well-known` version of the imperative [`mediation: "immediate"`](https://github.com/w3c/webauthn/blob/main/explainers/immediate-mediation.md) proposal and the HTML equivalent [`<login>` element](https://github.com/fedidcg/login-element) proposal
- This is somewhat related to `WWW-Authenticate` and, for example, `Basic Auth`, but outside of HTTP headers.
- This is intended to be as an input to things like, for example, [agentic federated login](https://github.com/w3c-fedid/agentic-federated-login)

# Proposal

Every website has to create their own login flow, leading to an inconsistent, fragmented, inneficient and cumbersome user experience. 

This is a proposal to allow website authors to expose a `.well-known/login.json` file that provides a machine-readable description of how to build a mediated and unified login flow across all authentication mechanisms (notably passwords, passkeys and federation). 

The `login.json` file describes declarative what an otherwise imperative call to [Credential Manager API](https://developer.mozilla.org/en-US/docs/Web/API/Credential_Management_API) would be.

For example:

```json
{
  // Accepts passwords
  "passwords": true,
  // Accepts passkeys
  "publicKey": {
    "challenge": "...",
    "rpId": "example.com",
    "userVerification": "preferred",
  },
  // Accepts federated login
  "identity": {
     "providers": [{
       "clientId": "1234",
       "configURL": "https://idp.example",
       "params": { "nonce": "..."}
     }]
  }
}
```

Since we don't have a JS/HTML conext, the result of the process gets `POST`-ed to the endpoint:

```http
POST /.well-known/login.json HTTP/1.1
Host: example.com
Content-Type: application/json
Content-Length: 45

{
  // federated identity was selected by the user
  "identity": {
    "token": "...",
  }
}
```

# Open Questions

1. What should we call `/.well-known/login.json`? Should we use a `.json` suffix?
1. How do we deal with `nonces` and `challenges`? Do we need a level of indirection perhaps (e.g. where the .well-known file just points to a URL that anyone can use to generate dynamically these random challenges?)?
1. How does the result get `POST`-ed back? Should we `POST` back to `/.well-known/login.json`? Or maybe there is a URL somewhere there that tells the agent where to `POST`?
