# CF Image proxy

> Proxy CDN for caching static assets from third-party domains via [Cloudflare Workers](https://workers.cloudflare.com).

[![Build Status](https://github.com/transitive-bullshit/cf-image-proxy/actions/workflows/build.yml/badge.svg)](https://github.com/transitive-bullshit/cf-image-proxy/actions/workflows/build.yml) [![Prettier Code Formatting](https://img.shields.io/badge/code_style-prettier-brightgreen.svg)](https://prettier.io)

## Features

- Free 💪
- Super simple to setup and self-host
- Perfect lighthouse scores
- Normalizes origin URLs

## Config

**All config is defined in [wrangler.toml](./wrangler.toml).**

0. Create a new blank [Cloudflare Worker](https://workers.cloudflare.com).
1. Fork / clone this repo
2. Update the missing values in [wrangler.toml](./wrangler.toml)
3. `npm install`
4. `npm run dev` to test locally
5. `npm run deploy` to deploy to cloudflare workers 💪

### wrangler.toml

```yaml
name = "cf-image-proxy"
type = "javascript"
webpack_config = "webpack.config.js"
account_id = "TODO"
workers_dev = true

[env.production]
zone_id = "TODO"
route = "TODO"
```

You can find your `account_id` and `zone_id` in your Cloudflare Workers settings.

Your `route` should look like `"exampledomain.com/*"`.

### Cloudflare Polish

You can optionally enable Polish in your Cloudflare zone settings if you want to enable on-the-fly image optimization as part of your CDN. In many cases, this will serve images to supported clients in an optimized `webp` format.

This may increase costs, so it's not recommended for everyone. The CF worker should support both configurations without issue.

## Usage

In the application where you want to consume your proxied iamges, you'll just need to replace your third-party image URLs.

You can replace them with your proxy domain plus a path that contains the URI-encoded version of your original domain. In TypeScript, this looks like the following:

```ts
const imageCDNHost = 'https://exampledomain.com'

export const mapImageUrl = (imageUrl: string) => {
  if (imageUrl.startsWith('data:')) {
    return imageUrl
  }

  if (imageCDNHost) {
    // Our proxy uses Cloudflare's global CDN to cache these image assets
    return `${imageCDNHost}/${encodeURIComponent(imageUrl)}`
  } else {
    return imageUrl
  }
}
```

If you are using this image proxy as part of [nextjs-notion-starter-kit](https://github.com/transitive-bullshit/nextjs-notion-starter-kit), all you need to do is set `imageCDNHost` in your `site.config.js` and your image proxy will be used automatically.

## Technical Notes

A few notes about the implementation:

- It is hosted via Cloudflare (CF) edge [workers](https://workers.cloudflare.com).
- It is transpiled by webpack before uploading to CF.
- CF runs our worker via V8 directly in an environment mimicking [web workers](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API).
- This means that our worker does not have access to Node.js primitives such as `fs`, `dns` and `http`.
- It does have access to a custom [web fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API).

## License

MIT © [Travis Fischer](https://transitivebullsh.it)

Support my OSS work by <a href="https://twitter.com/transitive_bs">following me on twitter <img src="https://storage.googleapis.com/saasify-assets/twitter-logo.svg" alt="twitter" height="24px" align="center"></a>