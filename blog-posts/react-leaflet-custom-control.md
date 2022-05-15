how to create a custom control element and add it to the map in a react app

packages
- leaflet
- react-leaflet
- @geoapify/geocoder-autocomplete

Leaflet is a great package to easily integrate map and GIS functionality into your web application. Bonus points for being open source so you don't need to rely on Google Maps (which is also a great option).

The goal of this article is to add a custom address search control component inside the leaflet map container. I emphasize inside, because the only way to render UI elements ontop of the map tiles is to use the Leaflet controls API.

This article is focused on creating a custom control for the React flavour of Leaflet. There are plenty of articles on the web building custom leaflet controls using Vanilla JS.

### Getting Started
Let's add a standard map to our web page using the `react-leaflet` package.

```bash
npm i leaflet react-leaflet
npm i @types/leaflet # optional
```

The (react-leaflet docs)[] instruct us to follow the standard set up for a vanilla web project before using the React bindings provided by the package. The set up involves importing the stylesheet for the map ui:

```tsx
 <link
	rel="stylesheet"
    href="https://unpkg.com/leaflet@1.7.1/dist/leaflet.css"
    integrity="sha512-xodZBNTC5n17Xt2atTPuE1HxjVMSvLVW9ocqUKLsCC5CXdbqCmblAshOMAS6/keqq/sMZMZ19scR4PsZChSR7A=="
    crossOrigin=""
/>
```

With that in place let's create a `Map.tsx` component:

```tsx
import { MapContainer, TileLayer } from 'react-leaflet';

const Map = () => {

  return (
    <MapContainer
      className="h-full flex-auto"
      center={[51.505, -0.09]}
      zoom={13}>
      <TileLayer
        attribution='&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors'
        url="https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png"
      />
    </MapContainer>
  );
};

export default Map;
```

This gives us a basic leaflet map centered on London, England. It has a zoom control and click and drag functionality set up by default.

### Adding a Marker with Popup
A common ux enhancement for map is to allow marker placement when a user clicks on a location. `react-leaflet` also makes it simple to add a popup on the marker to provide additional information. In this example we will include the latitude and longtitude of the marker postion in the popup window. Let's make this component in the same `Map.tsx` file:

```tsx
import type { LatLng } from 'leaflet';
import { useState } from 'react';
import { MapContainer, TileLayer, useMapEvents } from 'react-leaflet';

const LocationMarker = () => {
  const map = useMapEvents({
    click(e) {
      setPosition(e.latlng);
      map.flyTo(e.latlng, map.getZoom());
    },
  });

  return position === null ? null : (
    <Marker position={position}>
      <Popup>
        <span>Project Coordinates:</span>
        <br />
        {position.toString()}
      </Popup>
    </Marker>
  );
};

const Map = () => {
  const {position, setPosition } = useState<LatLng>([51.505, -0.09]);

  return (
    <MapContainer center={position}>
      <TileLayer />
      <LocationMarker setPosition={setPosition} />
    </MapContainer>
  );
};

export default Map;
```

Let's break down what we've added.

- We update `<Map>` to have a state for the position value.
- We create a `<LocationMarker />` component, which renders the `<Marker>` and `<Popup>` components imported from `react-leaflet`
- We pass `setPosition` as a prop into `<LocationMarker />` so the map get's centered on the clicked location