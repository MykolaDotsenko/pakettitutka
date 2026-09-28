# Pakettitutka — Finnish Parcel Price Intelligence

[![Quality](https://github.com/MykolaDotsenko/pakettitutka/actions/workflows/quality.yml/badge.svg)](https://github.com/MykolaDotsenko/pakettitutka/actions/workflows/quality.yml)

**A Finland-specific parcel comparison engine built around carrier tariff rules instead of generic delivery estimates.**

[**Open the live app →**](https://pakettitutka.vercel.app/) · [Architecture](./ARCHITECTURE.md)

![Pakettitutka brand](./public/brand/visuals/brand-board.webp)

## Why this is not a “price per kilometre” calculator

Parcel carriers use different commercial models.

Examples include:

- fixed consumer parcel sizes;
- weight bands;
- pickup/door-service additions;
- actual vs volumetric chargeable weight;
- fuel surcharges;
- VAT and handling fees;
- island/ferry surcharges;
- account/contract pricing that cannot be represented honestly by a public static number.

Pakettitutka models each supported carrier according to its own published rules and shows an **accuracy/confidence label** rather than pretending every result has the same certainty.

## Product rules

- official tariff/source links are visible in the UI;
- public/no-contract and published business list-rate modes stay separate;
- mainland/Åland routing is requested only when it changes pricing;
- destination postcode is optional and used only where a published surcharge rule actually needs it;
- unsupported dynamic/account pricing is explained instead of approximated;
- final carrier checkout/invoice remains authoritative;
- no service-worker tariff caching — freshness matters more than offline pricing.

## Pricing model

A carrier implementation can combine:

```text
package dimensions
actual weight
volumetric weight
service/pickup mode
route/region rules
published surcharge rules
tax
        ↓
carrier-specific quote
        ↓
price + coverage/accuracy explanation
```

For example, a chargeable-weight service can use:

```text
volumetric weight = volume × carrier factor
chargeable weight = max(actual weight, volumetric weight)
```

That logic is carrier-specific; it is not shared blindly across every provider.

## Data vs calculation logic

Mutable tariff data lives in:

`src/data/finlandTariffData.ts`

Pricing behaviour lives in:

`src/domain/finlandTariffs.ts`

This separation keeps routine tariff updates reviewable without mixing them with the algorithms that decide eligibility, chargeable weight and surcharge composition.

## Engineering rules

- fixed-size boxes are rotation-aware;
- provider minimum and maximum dimensions are enforced;
- girth-based services use the carrier's longest-side/circumference rule;
- Åland is handled separately when the official tariff differs;
- fuel surcharge is applied only to the components the carrier defines;
- current public prices are not backfilled from an old list when a current value is unavailable;
- external provider/account pricing is never presented as a calculated exact quote.

## Carrier coverage

The project contains calculated public/list-rate support where published rules make deterministic calculation possible, and source-linked directory coverage where pricing is dynamic or contract-specific.

Current carrier data and source URLs are maintained in the repository rather than duplicated in this README so pricing updates do not require rewriting the project overview.

## Stack

- React 19
- TypeScript 6
- Vite 8
- Vitest
- Playwright
- axe-core
- ESLint
- GitHub Actions
- Vercel

The runtime dependency surface is intentionally small: React + React DOM.

## Quality

```bash
npm ci
npm run check
npx playwright install chromium
npm run test:e2e
```

CI covers linting, type checking, unit tests, production build, dependency audit and browser/accessibility checks across desktop, Pixel-sized mobile and narrow mobile layouts.

## Run locally

```bash
npm ci
npm run dev
```

## Accuracy policy

Pakettitutka distinguishes:

- deterministic calculations from published rules;
- partial/public list-rate coverage;
- carriers that require a live quote or account agreement.

That distinction is a product feature, not an implementation detail: a missing exact price is preferable to a confident-looking invented one.

## License

MIT.
