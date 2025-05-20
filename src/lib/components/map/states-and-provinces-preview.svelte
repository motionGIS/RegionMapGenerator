<script lang="ts">
  import type maplibregl from 'maplibre-gl'; // @ts-ignore
  import MapLibre from 'svelte-maplibre/MapLibre.svelte'; // @ts-ignore
  import GeoJSON from 'svelte-maplibre/GeoJSON.svelte'; // @ts-ignore
  import FillLayer from 'svelte-maplibre/FillLayer.svelte'; // @ts-ignore
  import LineLayer from 'svelte-maplibre/LineLayer.svelte';
  import { hoverStateFilter } from 'svelte-maplibre/filters.js';
  import type { ExpressionSpecification } from 'maplibre-gl';
  import states from '$lib/components/map/states.json';
  import statesToNamesJson from '$lib/components/map/states-to-names.json';
  
  import { Checkbox } from "$lib/components/ui/checkbox/index.js";
  import { Label } from "$lib/components/ui/label/index.js";
  import { Switch } from "$lib/components/ui/switch";
  import * as toGeoJSON from '@tmcw/togeojson';

  let globeBool = $state(true);
  let projection = $derived(globeBool ? "globe" : "mercator");

  const statesToNames: Record<string, string> = statesToNamesJson;

  const { selectedStates = [], gpxTraces = [] } = $props<{
    selectedStates?: string[];
    gpxTraces?: { id: string; color: string; data: string; name: string }[];
  }>();
  let fillColor = $state('#000000');
  let borderColor = $state('#ffffff');

  let showBorder = $state(true);
  let showFill = $state(true);

  let map: maplibregl.Map | undefined = $state();
  let loaded = $state(false);

  const allowedNames = $derived(() =>
    selectedStates.map((code: string | number) => statesToNames[code]).filter(Boolean)
  );

  let filter: ExpressionSpecification | undefined = $derived(
      ["any",
        ['in', ['get', 'name'], allowedNames().toString()],
      ],
    );

  function getSelectedBbox() {
    const features = states.features.filter(
      f => allowedNames().includes(f.properties.name)
    );
    if (features.length === 0) return null;

    let minX = Infinity, minY = Infinity, maxX = -Infinity, maxY = -Infinity;
    for (const feature of features) {
      const coords = feature.geometry.type === 'Polygon'
        ? feature.geometry.coordinates.flat(1)
        : feature.geometry.type === 'MultiPolygon'
          ? feature.geometry.coordinates.flat(2)
          : [];
      for (const coord of coords) {
        if (Array.isArray(coord) && coord.length >= 2 && typeof coord[0] === 'number' && typeof coord[1] === 'number') {
          const [x, y] = coord;
          if (x < minX) minX = x;
          if (y < minY) minY = y;
          if (x > maxX) maxX = x;
          if (y > maxY) maxY = y;
        }
      }
    }
    return [[minX, minY], [maxX, maxY]];
  }

  let prevSelection = "";

  $effect(() => {
    const names = allowedNames();
    const selection = JSON.stringify(names);
    if (map && names.length && selection !== prevSelection) {
      prevSelection = selection;
      const bbox = getSelectedBbox();
      if (bbox) {
        map.fitBounds([bbox[0][0], bbox[0][1], bbox[1][0], bbox[1][1]], { padding: 40, duration: 500 });
      }
    }
  });

  const gpxFeatures = $derived(() =>
    gpxTraces
      .map((trace: { data: string; color: any; id: any; name: any; }) => {
        try {
          const dom = new window.DOMParser().parseFromString(trace.data, "application/xml");
          const geojson = toGeoJSON.gpx(dom);
          return geojson.features
            .filter(f => f.geometry.type === "LineString")
            .map(f => ({
              ...f,
              properties: {
                ...(f.properties || {}),
                color: trace.color,
                id: trace.id,
                name: trace.name
              }
            }));
        } catch (e) {
          return [];
        }
      })
      .flat()
  );

  const gpxFeatureCollection = $derived(() => ({
    type: "FeatureCollection",
    features: Array.isArray(gpxFeatures()) ? gpxFeatures() : []
  }));
</script>

<div class="flex items-center space-x-2">
  <Label for="projection">2d (Map view)</Label>
  <Switch id="projection" bind:checked={globeBool} />
  <Label for="projection">3d (Globe View)</Label>
</div>



<div class="flex w-full max-w-md items-center gap-y-2 self-start">
  <label>Fill <input type="checkbox" bind:checked={showFill} /> <input type="color" bind:value={fillColor} /></label>
  <label>Outlines <input type="checkbox" bind:checked={showBorder} /><input type="color" bind:value={borderColor} /></label>
</div>

<MapLibre
  bind:map
  bind:loaded
  style={{
    "version": 8,
    "sources": {},
    "layers": []
  }}
  class="h-[50vh] w-full"
  center={[-98.137, 40.137]}
  zoom={4}
  projection={{ type: projection }}
>
<GeoJSON id="states" data={states} promoteId="STATEFP">
  <FillLayer
    {filter}
    paint={{
      "fill-color": fillColor,
      "fill-opacity": 1
    }}
  />
  <LineLayer
    {filter}
    paint={{
      "line-color": borderColor,
      "line-width": 2
    }}
  />
  <LineLayer
    {filter}
    paint={{
      "line-color": "rgba(0,0,0,0)",
      "line-width": 0
    }}
  />
</GeoJSON>
  <GeoJSON id="gpx-traces" data={gpxFeatureCollection()}>
    <LineLayer
      paint={{
        // Use the color property from each feature
        "line-color": ["get", "color"],
        "line-width": 3
      }}
    />
  </GeoJSON>
</MapLibre>