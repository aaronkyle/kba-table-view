# Key Biodiversity Areas

```html
<link rel='stylesheet' href='https://unpkg.com/maplibre-gl@4.3.2/dist/maplibre-gl.css' />
<style>
  .maplibregl-popup-content{
    color: black;
}
</style>
```


The [World Database of Key Biodiversity Areas](https://wdkba.keybiodiversityareas.org/) is managed by [BirdLife International](https://www.birdlife.org/?gad_source=1&gclid=CjwKCAjw_Na1BhAlEiwAM-dm7KWRcYTD7g6NyuMKjliQLOCOBX18cuRseeMqS4x9gHeN-unck_h9yxoC4B0QAvD_BwE) on behalf of the [KBA Partnership](https://www.keybiodiversityareas.org/working-with-kbas/programme/partnership).<sup>[[*]](https://www.keybiodiversityareas.org/kba-news/wdkba)</sup> It hosts data on global and regional [Key Biodiversity Areas](https://www.keybiodiversityareas.org/) (KBAs), including Important Bird and Biodiversity Areas identified by the [BirdLife International Partnership](https://www.birdlife.org/partnership/), [Alliance for Zero Extinction](https://zeroextinction.org/) sites, KBAs identified through hotspot ecosystem profiles supported by the [Critical Ecosystem Partnership Fund](https://www.cepf.net/), and a small number of other KBAs. The database was developed from the [World Bird and Biodiversity Database (WBDB)](https://www.globalconservation.info/) managed by BirdLife International.

```js
const container = display(html`<div style="width:800px; height:600px"></div>`)
```

```js
const image_selection = view(Inputs.radio(["CartoDB Voyager", "NASA Blue Marble", "ESRI World Imagery", "ESRI World Terrain Base", "ESRI Topographic", "USGS Topographic"], {
  label: "Select Basemap",
  value: "CartoDB Voyager"
}))
```

```js
const tableOfContents = view(Inputs.checkbox(toc, {
  label: "Toggle Visibility",
  value: toc.values()
}))
```

---

### Explore Visible Features

_The table below automatically adjusts to the map's viewing area._


```js
display(visibleFeatures)
```


```js
Inputs.table(visibleFeatures.features.map(d => d.properties))
```


---

### KBA Source Data

```js
const kba_wms_view = (mapRef) => {
    mapRef.addSource('kba-wms', {
                'type': 'raster',
                'tiles': [
                    proxyUrl + 'https://birdlaa8.birdlife.org/geoserver/gwc/service/wms?service=WMS&request=GetMap&layers=birdlife_dz:ibas_global_2024_wm&styles=&format=image/png&transparent=true&version=1.1.1&height=256&width=256&srs=EPSG:3857&bbox={bbox-epsg-3857}'
                ],
                'tileSize': 256
            });

    mapRef.addLayer({
                'id': 'kba-wms',
                'type': 'raster',
                'source': 'kba-wms',
                'paint': { 'raster-opacity': 1 }
            });
            }
```

```js
// archival and unofficial copy of "https://birdlaa8.birdlife.org/geoserver/wfs?SERVICE=WFS&VERSION=1.0.0&REQUEST=GetFeature&TYPENAME=birdlife_dz:ibas_pol_20240515_wm_selected&OUTPUTFORMAT=application/json"

const kba_2022_10_POL = display(await FileAttachment("kba-2022-10-poly-simp.geojson").json())
```

```js
const kba_2022_10_POL_view = (mapRef, kba_2022_10_POL) => {
  mapRef.addSource("kba_2022_10_POL", {
    type: "geojson",
    data: kba_2022_10_POL
  });
  mapRef.addLayer({
    id: "kba_2022_10_POL_fill",
    type: "fill",
    source: "kba_2022_10_POL",
    layout: {
      visibility: "visible"
    },
    paint: {
      "fill-color": "yellow",
      "fill-opacity": 0.05
    }
  });

  mapRef.addLayer({
    id: "kba_2022_10_POL_line",
    type: "line",
    source: "kba_2022_10_POL",
    layout: {
      visibility: "visible"
    },
    paint: {
      "line-color": "yellow",
      "line-width": 0.01,
      "line-opacity": 0.3
    }
  });

  // Add popup functionality
  mapRef.on('click', 'kba_2022_10_POL_fill', function (e) {
    const feature = e.features[0];
    const properties = feature.properties;

    // Create the popup content
    const popupContent = `
      <p>
        <strong>${properties['IntName']}</strong><br />
        <strong>Site ID:</strong> ${properties['SitRecID']}<br />
        <strong>Triggers:</strong> ${properties['triggers']}<br />
        <strong>Country:</strong> ${properties['Country']}<br />
        <strong>Region:</strong> ${properties['Region']}
      </p>
    `;

    // Create a new popup and set its coordinates and content
    new maplibregl.Popup()
      .setLngLat(e.lngLat)
      .setHTML(popupContent)
      .addTo(mapRef);
  });

  // Change the cursor to a pointer when the mouse is over the polygon layer
  mapRef.on('mouseenter', 'kba_2022_10_POL_fill', function () {
    mapRef.getCanvas().style.cursor = 'pointer';
  });

  // Change the cursor back to default when the mouse leaves the polygon layer
  mapRef.on('mouseleave', 'kba_2022_10_POL_fill', function () {
    mapRef.getCanvas().style.cursor = '';
  });
};
```

```js
const nat_geo = (mapRef) => {
  mapRef.addSource("nat_geo", {
    type: "raster",
    tiles: [
      "https://server.arcgisonline.com/ArcGIS/rest/services/NatGeo_World_Map/MapServer/tile/{z}/{y}/{x}"
    ],
    minzoom: 0,
    maxzoom: 16
  });
  mapRef.addLayer({
    id: "nat_geo",
    type: "raster",
    source: "nat_geo",
    layout: {
      visibility: "none"
    }
  });
}
```

```js
const nat_geo_visibility = (() => {
  // Check if the map style is loaded
  if (map.isStyleLoaded()) {
    // If loaded, set the visibility
    map.setLayoutProperty(
      "nat_geo",
      "visibility",
      (image_selection.includes("NatGeo World Map")) ? "visible" : "none"
    );
  } else {
    // If not loaded, wait for the 'load' event
    map.on('load', () => {
      map.setLayoutProperty(
        "nat_geo",
        "visibility",
        (image_selection.includes("NatGeo World Map")) ? "visible" : "none"
      );
    });
  }
  return `nat_geo_visibility`;
})();
```

```js
const usgs_topo = (mapRef) => {
  mapRef.addSource("usgs_topo", {
    type: "raster",
    tiles: [
      "https://server.arcgisonline.com/ArcGIS/rest/services/NatGeo_World_Map/MapServer/tile/{z}/{y}/{x}"
    ],
    minzoom: 0,
    maxzoom: 16
  });
  mapRef.addLayer({
    id: "usgs_topo",
    type: "raster",
    source: "usgs_topo",
    layout: {
      visibility: "none"
    }
  });
}
```

```js
const usgs_topo_visibility = (() => {
  // Check if the map style is loaded
  if (map.isStyleLoaded()) {
    // If loaded, set the visibility
    map.setLayoutProperty(
      "usgs_topo",
      "visibility",
      (image_selection.includes("USGS Topographic")) ? "visible" : "none"
    );
  } else {
    // If not loaded, wait for the 'load' event
    map.on('load', () => {
      map.setLayoutProperty(
        "usgs_topo",
        "visibility",
        (image_selection.includes("USGS Topographic")) ? "visible" : "none"
      );
    });
  }
  return `usgs_topo_visibility`;
})();
```

```js
const world_imagery = (mapRef) => {
  mapRef.addSource("world_imagery", {
    type: "raster",
    tiles: [
      "https://server.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/{z}/{y}/{x}"
    ],
    minzoom: 0,
    maxzoom: 16
  });
  mapRef.addLayer({
    id: "world_imagery",
    type: "raster",
    source: "world_imagery",
    layout: {
      visibility: "none"
    }
  });
}
```

```js
const world_imagery_visibility = (() => {
  // Check if the map style is loaded
  if (map.isStyleLoaded()) {
    // If loaded, set the visibility
    map.setLayoutProperty(
      "world_imagery",
      "visibility",
      (image_selection.includes("World Imagery")) ? "visible" : "none"
    );
  } else {
    // If not loaded, wait for the 'load' event
    map.on('load', () => {
      map.setLayoutProperty(
        "world_imagery",
        "visibility",
        (image_selection.includes("World Imagery")) ? "visible" : "none"
      );
    });
  }
  return `world_imagery_visibility`;
})();
```

```js
const terrain = (mapRef) => {
  mapRef.addSource("terrain", {
    type: "raster",
    tiles: [
      "https://server.arcgisonline.com/ArcGIS/rest/services/World_Terrain_Base/MapServer/tile/{z}/{y}/{x}"
    ],
    minzoom: 0,
    maxzoom: 16
  });
  mapRef.addLayer({
    id: "terrain",
    type: "raster",
    source: "terrain",
    layout: {
      visibility: "none"
    }
  });
}
```

```js
const terrain_visibility = (() => {
  // Check if the map style is loaded
  if (map.isStyleLoaded()) {
    // If loaded, set the visibility
    map.setLayoutProperty(
      "terrain",
      "visibility",
      (image_selection.includes("World Terrain Base")) ? "visible" : "none"
    );
  } else {
    // If not loaded, wait for the 'load' event
    map.on('load', () => {
      map.setLayoutProperty(
        "terrain",
        "visibility",
        (image_selection.includes("World Terrain Base")) ? "visible" : "none"
      );
    });
  }
  return `terrain_visibility`;
})();
```

```js
const topo = (mapRef) => {
  mapRef.addSource("topo", {
    type: "raster",
    tiles: [
      "https://server.arcgisonline.com/ArcGIS/rest/services/World_Topo_Map/MapServer/tile/{z}/{y}/{x}"
    ],
    minzoom: 0,
    maxzoom: 16
  });
  mapRef.addLayer({
    id: "topo",
    type: "raster",
    source: "topo",
    layout: {
      visibility: "none"
    }
  });
}
```

```js
const topo_visibility = (() => {
  // Check if the map style is loaded
  if (map.isStyleLoaded()) {
    // If loaded, set the visibility
    map.setLayoutProperty(
      "topo",
      "visibility",
      (image_selection.includes("ESRI Topographic")) ? "visible" : "none"
    );
  } else {
    // If not loaded, wait for the 'load' event
    map.on('load', () => {
      map.setLayoutProperty(
        "topo",
        "visibility",
        (image_selection.includes("ESRI Topographic")) ? "visible" : "none"
      );
    });
  }
   return `topo_visibility`;
})();
```

```js
const blue_marble = (mapRef) => {
  mapRef.addSource("blue_marble", {
    type: "raster",
    tiles: [
        "https://gibs.earthdata.nasa.gov/wmts/epsg3857/best/BlueMarble_NextGeneration/default/GoogleMapsCompatible_Level8/{z}/{y}/{x}.jpeg"
      ],
    tileSize: 256,
    minzoom: 0,
    maxzoom: 16
  });
  mapRef.addLayer({
    id: "blue_marble",
    type: "raster",
    source: "blue_marble",
    layout: {
      visibility: "none"
    }
  });
}
```

```js
const blue_marble_visibility = (() => {
  // Check if the map style is loaded
  if (map.isStyleLoaded()) {
    // If loaded, set the visibility
    map.setLayoutProperty(
      "blue_marble",
      "visibility",
      (image_selection.includes("NASA Blue Marble")) ? "visible" : "none"
    );
  } else {
    // If not loaded, wait for the 'load' event
    map.on('load', () => {
      map.setLayoutProperty(
        "blue_marble",
        "visibility",
        (image_selection.includes("NASA Blue Marble")) ? "visible" : "none"
      );
    });
  }
  return `blue_marble_visibility`;
})();
```

---

```js
const toc = new Map([
  [
    "Show / Hide KBA Layer",
    {
      label: "kba_2022_10_POL",
      layers: ["kba_2022_10_POL_fill", "kba_2022_10_POL_line", "kba-wms"]
    }
  ],
])
```

```js
const layer_visibility = (() => {
  for (const entry of toc.values()) {
    for (const layer of entry.layers) {
      if (map.getLayer(layer)) {
        map.setLayoutProperty(
          layer,
          "visibility",
          tableOfContents.map((d) => d.label).includes(entry.label)
            ? "visible"
            : "none"
        );
      }
    }
  }
  return tableOfContents;
})()
```

---

## Map


```js
const map = (() => {
  // Create the "map" object with the maplibregl.Map constructor, referencing the container div

  const mapRef = (container.value = new maplibregl.Map({
    container: container,
    center: [172.5,-43],
    zoom: 4,
    pitch: 0,
    bearing: 0,
    maplibreglLogo: false,
    style: "https://basemaps.cartocdn.com/gl/voyager-gl-style/style.json",
    attributionControl: false
  }));

  // Add some navigation controls.
  mapRef.addControl(new maplibregl.NavigationControl(), "top-right");

  // If this cell is invalidated, dispose of the map.
  invalidation.then(() => mapRef.remove());


  // The map must be loaded before we can add sources.
  new Promise((resolve, reject) => {
    mapRef.on("load", async () => {

      nat_geo(mapRef);
      world_imagery(mapRef);
      terrain(mapRef);
      blue_marble(mapRef);
      topo(mapRef);
      usgs_topo(mapRef);

      kba_wms_view(mapRef);
      kba_2022_10_POL_view(mapRef, kba_2022_10_POL);

    });
  });

  return mapRef;

})();

```


```js
const bounds = map.getBounds();
```


```js
const updateBoundingBoxObject = (mapRef) => {
  // Create an initial bounding box object
  const bbox = {
    north: mapRef.getBounds().getNorth(),
    south: mapRef.getBounds().getSouth(),
    east: mapRef.getBounds().getEast(),
    west: mapRef.getBounds().getWest(),
  };

  // Create a generator function to handle reactivity
  return Generators.observe((push, end) => {
    const update = () => {
      bbox.north = mapRef.getBounds().getNorth();
      bbox.south = mapRef.getBounds().getSouth();
      bbox.east = mapRef.getBounds().getEast();
      bbox.west = mapRef.getBounds().getWest();
      push({...bbox}); // Push updated bbox
    };

    update(); // Initial update

    // Update when the map's view changes
    mapRef.on("moveend", update);

    return () => mapRef.off("moveend", update); // Cleanup function
  });
};
```

```js
const boundingBox = updateBoundingBoxObject(map);
```

```js
display(boundingBox);
```


```js
function getVisibleFeatures(geojson, boundingBox) {
  if (!geojson || !geojson.features || !boundingBox) {
    throw new Error('Invalid arguments');
  }

  // Create a turf bounding box polygon
  const bboxPolygon = turf.bboxPolygon([
    boundingBox.west,
    boundingBox.south,
    boundingBox.east,
    boundingBox.north
  ]);

  // Filter features based on whether they intersect with the bounding box polygon
  const visibleFeatures = geojson.features.filter(feature => {
    // Check if the feature intersects with the bounding box polygon
    return turf.booleanIntersects(bboxPolygon, feature);
  });

  // Return the filtered GeoJSON object
  return {
    type: 'FeatureCollection',
    features: visibleFeatures
  };
}
```

```js
const visibleFeatures = getVisibleFeatures(kba_2022_10_POL, boundingBox);
```



```js
const proxyUrl = 'https://corsproxy.io/?';
```

```js
import * as turf from "npm:@turf/turf";
```


```js
import maplibregl from 'npm:maplibre-gl';
import FeatureService from 'npm:mapbox-gl-arcgis-featureserver';
```
