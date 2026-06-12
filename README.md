# lightning-mpp-extension-sdk

Browser-side [Lightning MPP](https://www.npmjs.com/package/@buildonspark/lightning-mpp-sdk) client that automatically routes `402 Payment Required` approvals through a compatible browser extension (e.g. TIPT) via a `window` event bridge.

It wraps `Mppx.create` so that when a server responds with a `402` Lightning payment challenge, the SDK:

1. Probes the page for a compatible extension using the `mpp:extension` event (`type: 'request'`, `paymentMethods: ['lightning']`, `intents: ['charge']`).
2. Forwards the Lightning `Payment` challenge to the extension via `mpp:challenge`.
3. Waits for the extension's `mpp:credential` response and transparently retries the original request with the returned credential.

## Installation

```bash
npm install lightning-mpp-extension-sdk @buildonspark/lightning-mpp-sdk mppx
```

`@buildonspark/lightning-mpp-sdk` and `mppx` are **peer dependencies** — install them alongside this package.

## Usage

```ts
import { createTiptLightningClient } from 'lightning-mpp-extension-sdk';

const client = createTiptLightningClient({
  polyfill: false,
});

const response = await client.fetch('https://api.example.com/paid-endpoint');
const data = await response.json();
```

### Options

```ts
interface CreateTiptLightningClientOptions {
  /** Custom fetch implementation. Defaults to the global fetch. */
  fetch?: typeof globalThis.fetch;
  /** Patch the global fetch (Mppx.create polyfill). Defaults to true. */
  polyfill?: boolean;
  /** Timeout for the extension payment approval, in ms. Defaults to 90000. */
  paymentTimeoutMs?: number;
  /** Probe for the extension before requesting credentials. Defaults to true. */
  probeExtension?: boolean;
  /** Timeout for the extension probe, in ms. Defaults to 1500. */
  extensionProbeTimeoutMs?: number;
}
```

### Restoring global fetch

If you used the global `fetch` polyfill, restore the original implementation with:

```ts
import { restoreTiptLightningClientFetch } from 'lightning-mpp-extension-sdk';

restoreTiptLightningClientFetch();
```

## Requirements

- A browser `window` context (the SDK throws if `window` is unavailable).
- A compatible extension installed on the page that responds to the `mpp:extension`, `mpp:challenge`, and `mpp:credential` events.

## Development

```bash
npm install      # install peer + dev dependencies
npm run build    # bundle ESM + CJS + type declarations into dist/
npm run typecheck
```

## Publishing

```bash
npm version <patch|minor|major>
npm publish
```

`prepublishOnly` runs the build automatically, and only the `dist/`, `README.md`, and `LICENSE` files are included in the published package.

## License

[MIT](./LICENSE)
