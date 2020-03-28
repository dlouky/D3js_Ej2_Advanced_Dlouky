# COVID-19 in SPAIN (Module 9 Mandatory Exercise)

I focus on Spain affection by community displaying a map pinning affected locations and scaling that pin according to the number of cases affected, something like:

![map affected coronavirus](./content/chart.png "affected coronavirus")


I have to face three challenges here:

- Place pins on a map based on location.
- Scale pin radius based on affected number.
- Create buttons to show different data periods.


# Steps

- I will take as starting example _02-pin-location-scale_, let's copy the content from that folder and execute _npm install_.

```bash
npm install
```

- This time we will Spain topojson info: https://github.com/deldersveld/topojson/blob/master/countries/spain/spain-comunidad-with-canary-islands.json

Let's copy it under the following route _./src/spain.json_

- Now import _spain.json_.

_./src/index.ts_

```diff
import * as d3 from "d3";
import * as topojson from "topojson-client";
+ const spainjson = require("./spain.json");
```

- Let's build the spain map:

_./src/index.ts_

```diff
const geojson = topojson.feature(
+  spainjson,
+  spainjson.objects.ESP_adm1
);
```
> How do I know that we have to use _spainjson.objects.ESP_adm1_ just by examining
> the _spain.json_ file and by debugging and inspecting what's inside _spainjson_ object

- If we run the project, we will get some bitter-sweet feelings, we can see a map of spain,
  but it's too smal, and on the other hand, canary islands are shown far away (that's normal,
  but usually in maps these islands are relocated).

- Let's start by adding the right size to be displayed in our screen.

_./src/index.ts_

```diff
const aProjection = d3
  .geoMercator()
  // Let's make the map bigger to fit in our resolution
+  .scale(2300)
  // Let's center the map
+  .translate([600, 2000]);
```

- If we run the project we can check that the map is now renders in a proper size and position, let's
  go for the next challenge, we want to reposition Canary Islands, in order to do that we can build a
  map projection that positions that piece of land in another place, for instance for the USA you can
  find Albers USA projection: https://bl.ocks.org/mbostock/2869946, there's a great project created by
  [Roger Veciana](https://github.com/rveciana) that implements a lot of projections for several
  maps:

  - [Project site](https://geoexamples.com/d3-composite-projections/)
  - [Github project](https://github.com/rveciana/d3-composite-projections)

Let's install the library that contains this projections:

```bash
npm install d3-composite-projections --save
```

- Let's import it in our _index.ts_ (we will use require since we don't have typings).

```diff
import * as d3 from "d3";
import * as topojson from "topojson-client";
const spainjson = require("./spain.json");
+ const d3Composite = require("d3-composite-projections");
```

- Let's change the projection we are using (we will need to tweak as well the
  _scale_ and _translate_ values):

_./index.ts_

```diff
const aProjection =
-   d3
-  .geoMercator()
+  d3Composite
+  .geoConicConformalSpain()
  // Let's make the map bigger to fit in our resolution
-  .scale(2300)
+  .scale(3300)
  // Let's center the map
-  .translate([600, 2000]);
+  .translate([500, 400]);
```

- If we run the project, voila ! we got the map just the way we want it.

- Now we want to display a circle in the middle of each community,
  we have collected the latitude and longitude for each community, let's add them to our
  project.

_./src/communities.ts_

```typescript
export const latLongCommunities = [
  {
    name: "Madrid",
    long: -3.70256,
    lat: 40.4165
  },
  {
    name: "Andalucía",
    long: -4.5,
    lat: 37.6
  },
  {
    name: "Valencia",
    long: -0.37739,
    lat: 39.45975
  },
  {
    name: "Murcia",
    long: -1.13004,
    lat: 37.98704
  },
  {
    name: "Extremadura",
    long: -6.16667,
    lat: 39.16667
  },
  {
    name: "Cataluña",
    long: 1.86768,
    lat: 41.82046
  },
  {
    name: "País Vasco",
    long: -2.75,
    lat: 43.0
  },
  {
    name: "Cantabria",
    long: -4.03333,
    lat: 43.2
  },
  {
    name: "Asturias",
    long: -5.86112,
    lat: 43.36662
  },
  {
    name: "Galicia",
    long: -7.86621,
    lat: 42.75508
  },
  {
    name: "Aragón",
    long: -1.0,
    lat: 41.0
  },
  {
    name: "Castilla y León",
    long: -4.45,
    lat: 41.383333
  },
  {
    name: "Castilla La Mancha",
    long: -3.000033,
    lat: 39.500011
  },
  {
    name: "Islas Canarias",
    long: -15.5,
    lat: 28.0
  },
  {
    name: "Islas Baleares",
    long: 2.52136,
    lat: 39.18969
  },
  {
    name: "La Rioja",
    long: -2.44373,
    lat: 36.97706
  }
];
```

- Let's import it:

_./src/index.ts_

```diff
import * as d3 from "d3";
import * as topojson from "topojson-client";
+ import { latLongCommunities } from "./communities";
```
- let' define a function to calculate the radius of each circle bases on infected cases:

```diff
const calculateRadiusBasedOnAffectedCases = (comunidad: string, data: ResultEntry[]) => {
  const entry = data.find(item => item.name === comunidad);
  const maxAffected = 10000

  const affectedRadiusScale = d3
  .scaleLinear()
  .domain([0, maxAffected])
  .range([0, 50]); // 50 pixel max radius, we could calculate it relative to width and height
  
  return entry ? affectedRadiusScale(entry.value) : 0;
};

```

- And let's append at the bottom of the _index_ file a
  code to render a circle on top of each community:

_./src/index.ts_

```typescript
svg
  .selectAll("circle")
  .data(latLongCommunities)
  .enter()
  .append("circle")
  .attr("class", "affected-marker")
  .attr("r", d => calculateRadiusBasedOnAffectedCases(d.name, initial))
  .attr("cx", d => aProjection([d.long, d.lat])[0])
  .attr("cy", d => aProjection([d.long, d.lat])[1])
  ;
```

- Nice ! we got an spot on top of each community, now is time to
  make this spot size relative to the number of affected cases per community.

- We will add the stats that we need to display (affected persons per community):

_./stats.ts_

```typescript
export interface ResultEntry {
  name: string;
  value: number;
}

export const initial : ResultEntry[] = [
  {
    name: "Madrid",
    value: 587
  },
  {
    name: "La Rioja",
    value: 102
  },
  {
    name: "Andalucía",
    value: 54
  },
  {
    name: "Cataluña",
    value: 101
  },
  {
    name: "Valencia",
    value: 50
  },
  {
    name: "Murcia",
    value: 5
  },
  {
    name: "Extremadura",
    value: 7
  },
  {
    name: "Castilla La Mancha",
    value: 26
  },
  {
    name: "País Vasco",
    value: 148
  },
  {
    name: "Cantabria",
    value: 12
  },
  {
    name: "Asturias",
    value: 10
  },
  {
    name: "Galicia",
    value: 18
  },
  {
    name: "Aragón",
    value: 32
  },
  {
    name: "Castilla y León",
    value: 40
  },
  {
    name: "Islas Canarias",
    value: 24
  },
  {
    name: "Islas Baleares",
    value: 11
  },
  {
    name: "Navarra",
    value: 13
  }
];

export const final : ResultEntry[] = [
  {
    name: "Madrid",
    value: 9702
  },
  {
    name: "La Rioja",
    value: 654
  },
  {
    name: "Andalucía",
    value: 1725
  },
  {
    name: "Cataluña",
    value: 4704
  },
  {
    name: "Valencia",
    value: 1604
  },
  {
    name: "Murcia",
    value: 296
  },
  {
    name: "Extremadura",
    value: 384
  },
  {
    name: "Castilla La Mancha",
    value: 1819
  },
  {
    name: "País Vasco",
    value: 2097
  },
  {
    name: "Cantabria",
    value: 282
  },
  {
    name: "Asturias",
    value: 545
  },
  {
    name: "Galicia",
    value: 915
  },
  {
    name: "Aragón",
    value: 532
  },
  {
    name: "Castilla y León",
    value: 1744
  },
  {
    name: "Islas Canarias",
    value: 414
  },
  {
    name: "Islas Baleares",
    value: 331
  },
  {
    name: "Navarra",
    value: 794
  }
];
```

- Let's import it into our index.ts

_./src/index.ts_

```diff
import * as d3 from "d3";
import * as topojson from "topojson-client";
const spainjson = require("./spain.json");
const d3Composite = require("d3-composite-projections");
import { latLongCommunities } from "./communities";
+ import { stats } from "./stats";
```


- If we run the example we can check that know circles are shonw in the right size:

- Black circles are ugly let's add some styles, we will just use a red background and
  add some transparency to let the user see the spot and the map under that spot.


- Let's apply this style to the black circles tha we are rendering:

_./src/index.ts_

```diff
svg
  .selectAll("circle")
  .data(latLongCommunities)
  .enter()
  .append("circle")
+  .attr("class", "affected-marker")
  .attr("r", d => calculateRadiusBasedOnAffectedCases(d.name))
  .attr("cx", d => aProjection([d.long, d.lat])[0])
  .attr("cy", d => aProjection([d.long, d.lat])[1]);
```

- now I enter the svg.

_./src/index.ts_

```diff
svg
  .selectAll("path")
  .data(geojson["features"])
  .enter()
  .append("path")
  .attr("class", "country")
  // data loaded from json file
  .attr("d", geoPath as any)
```

_./src/map.css

```diff
.country {
  stroke-width: 1;
  stroke: #2f4858;
  fill: #008c86;
}


.affected-marker {
  stroke-width: 1;
  stroke: #bc5b40;
  fill: #f88f70;
  fill-opacity: 0.7;
}
```

- Create buttons to see past or actual info.

_./src/index.html

```diff
<html>
  <head>
    <link rel="stylesheet" type="text/css" href="./map.css" />
    <link rel="stylesheet" type="text/css" href="./base.css" />
  </head>
  <body>
    <div>
      <button id="initial">Initial</button>
      <button id="final">Final</button>
    </div>
    <script src="./index.ts"></script>
  </body>
</html>
```

- Asociate actions to those buttons.

_./src/index.ts

```diff
document
.getElementById("initial")
.addEventListener("click", function handleResultsInitial() {
  updateCircles(initial);
});

document
.getElementById("final")
.addEventListener("click", function handleResultsFinal() {
  updateCircles(final);
});
```


# Knowledge
- Spain topojson info:
https://github.com/deldersveld/topojson/blob/master/countries/spain/spain-comunidad-with-canary-islands.json
- d3js-typescript-examples
https://github.com/Lemoncode/d3js-typescript-examples/tree/master/02-maps/02-pin-location-scale
