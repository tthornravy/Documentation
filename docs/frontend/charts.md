---
sidebarDepth: 3
---

# Charts

![NPM License](https://img.shields.io/npm/l/@enso-ui/charts.svg)
![npm download](https://img.shields.io/npm/dm/@enso-ui/charts.svg)
![GitHub top language](https://img.shields.io/github/languages/top/enso-ui/charts.svg)
![GitHub issues](https://img.shields.io/github/issues/enso-ui/charts.svg)
![npm version](https://img.shields.io/npm/v/@enso-ui/charts.svg)

Vue Charts Package

Can be used outside of the Enso ecosystem.

Should be used with its backend [sibling](https://github.com/laravel-enso/Charts)

For live examples and demos, you may visit [laravel-enso.com](https://www.laravel-enso.com)

## Installation

Install the package:
```
yarn add @enso-ui/charts
```

## Usage
Import the desired component(s):
```js
import { EnsoChartCard as ChartCard } from '@enso-ui/charts/bulma';
```

```js
import { Chart, colors } from '@enso-ui/charts';
```

### ChartCard.vue

Bulma Chart Card implementation on top of the Card component.

#### Example:
```vue
<chart-card class="is-rounded raises-on-hover has-margin-bottom-large"
    source="/api/dashboard/pie"/>
```

#### Props:
- `errorHandler` - `function`, optional - provides a custom error handler for axios
- `i18n` - `function`, optional - provides localisation
- `params` - `Object`, optional - parameter bag that is sent to the backend when fetching data
- `source` - `string`, required - the URI used for fetching the chart data
 
#### Methods:
- `fetch()` - fetches data from the backend
- `resize()` - resizes the charts

### EnsoChartCard

The above component designed to be used within the Enso ecosystem, requiring less configuration from the dev. 

#### Example:
```vue
<chart-card class="is-rounded raises-on-hover has-margin-bottom-large"
    source="/api/dashboard/pie"/>
```

#### Props:
- `params` - `Object`, optional, default `null`, extra parameters that get sent to the backend when fetching data
- `source` - `string`, required, the URI used for fetching the chart data

#### Methods:
- `fetch()` - fetches data from the backend
- `resize()` - resizes the charts

### Chart

CSS framework agnostic Chart implementation.

#### Example:
```vue
<chart class="has-padding-medium"
    :data="data"
    :options="options"
    :type="type"/>
```

#### Props:
- `data` - `Object`, required - the data for the chart
- `formatter` - `function`, optional - datalabels values formatter function
- `options` - `Object`, optional - options object for chart.js
- `type` - `string`, required - specifies the type of chart.  Valid types are 'line', 'bar', 'radar', 'polarArea', 'pie', 'doughnut', 'bubble'.
 
#### Methods:
- `resize()` - resizes the charts

## Questions & Issues

For questions and support please use the issues functionality
for this package's github repository.

Please make sure to search for existing issues before creating a new issue,
and when opening a new issue, fill the required information in the issue template.

Issues not conforming to the guidelines may be closed immediately.

## Depends on

- `@fortawesome/fontawesome-svg-core`
- `@fortawesome/free-solid-svg-icons`
- `@fortawesome/vue-fontawesome`
- `bulma`
- `chart.js`
- `chartjs-plugin-datalabels`
- `file-saver`
- `vue`

## Contributions

are welcome. Pull requests are great, but issues are good too.

Thank you to all the people who already contributed to Enso!

## License

[ISC](https://opensource.org/licenses/ISC)