# Analytics Dashboard

A responsive, single-page analytics dashboard prototype built for the Protogen Series 200 Capstone. The capstone asks for an executive dashboard for a fictional logistics company. This implementation follows the project plan's narrower monthly business-metrics scope: revenue, visitors, conversion rate, and orders. It does not report shipment, delivery, regional, or exception metrics.

The dashboard uses synthetic data only. It has no API, backend, or connection to company or client systems.

## Product Decisions

- Keep the leadership view focused: four KPI cards, two side-by-side charts, and one full-width conversion chart.
- Use a single month picker to keep the KPI cards and charts in sync. The KPI cards show month values when a month is selected and annual totals or the average conversion rate for **All months**.
- Preserve the full year of chart context when a month is selected. The selected bar or point is highlighted, with a value callout and its change from the prior month. January has no prior-month chart delta.
- Use a reusable `MetricCard` component and Vuetify layout and card components to keep repeated UI consistent.
- Keep the sample data local and deterministic so interactions work without network access.
- Default to dark mode, offer a light-mode toggle, and use a light gray light-mode background to distinguish the white cards.

These choices reflect the capstone's emphasis on planning a clear scope, reviewing generated work, and refining it through iteration. See [PLAN.md](PLAN.md) for the original dashboard requirements and [Protogen - 200s Capstone Instructions.docx](Protogen%20-%20200s%20Capstone%20Instructions.docx) for the exercise context.

## Data

[`src/data/metrics.json`](src/data/metrics.json) contains one synthetic record for each month from January through December 2025. Each record has a date, revenue in dollars, visitor count, conversion percentage, and order count.

For **All months**, revenue, visitors, and orders are summed across the year; conversion rate is the average of the monthly rates. Selecting a specific month displays that month's values. KPI trend pills show percentage movement, while chart callouts show the absolute change from the preceding month.

## Tech Stack

- Vue 3 with TypeScript and Vue Router
- Vite for local development and production builds
- Vuetify 4 for theming, layout, and UI components
- Chart.js with `vue-chartjs` for revenue, visitor, and conversion charts
- Material Design Icons for dashboard icons

The stack reflects the current project dependencies; the original plan's request for Vuetify 3 is superseded by the installed Vuetify 4 package.

## Project Structure

```text
src/
	assets/main.css             Global styles and shared card typography
	components/MetricCard.vue   Reusable KPI card
	data/metrics.json           Synthetic monthly data
	router/index.ts             Home route
	views/HomeView.vue          Dashboard, interactions, and charts
```

## Development

Requires Node.js `^22.18.0` or `>=24.12.0`.

```sh
npm install
npm run dev
```

## Checks and Build

```sh
npm run type-check
npm run build
```

`npm run build` runs the TypeScript check and creates the production bundle in `dist/`. Use `npm run preview` to serve that bundle locally. Automated tests and lint scripts are not configured in this capstone project.
