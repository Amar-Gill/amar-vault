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

The [react-leaflet docs]() instruct us to follow the standard set up for a vanilla web project before using the React bindings provided by the package. The set up involves importing the stylesheet for the map ui:

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
  const {position, setPosition } = useState<LatLng>(null!);

  return (
    <MapContainer center={position ?? [51.505, -0.09]}>
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
- We pass `setPosition` as a prop into `<LocationMarker />` so the map get's centered on the clicked location, when the click event is processed by the `<LocationMarker />` component
- Through the `useMapEvents` hook, we can define an event handler for the click event, and also get access to the `Map` object, so we may call the `flyTo` method, centering the map view on the newly place marker
- Finally the `<Popup>` component can be a child of the `<Marker>` component, and it can include basic html inside; we show the latitude and longitude information of the positon in our example

### Creating our Custom Control
In this example we will add a text input control that allows us to look up an address, and have the map set it's view to the location. It's the same functionality you'd expect for Google maps.

We will leverage the [Geoapify]() api for the address look up functionality. It has a generous request limit of 3000 api calls daily for the free tier, which is handy for prototyping. Geoapify also has a handy [address search npm package]() which makes it easy to create the input element with all styling and interactivity we need for the component, including making api calls to the Geoapify api.

Note: there is a separate [npm package for a react component]() that wraps address search and exposes it as a react component. We will **not** be using the react package, because only native DOM elements can be added as a custom control onto a Leaflet map instance. This will be made obvious when we implement the custom control.

Let's install the package for the address search element:

```bash
npm i @geoapify/geocoder-autocomplete
```

You will need to visit the [geoapify]() platform and register so you can obtain an api key.

Next, let's create a `<AddressSearch />` component for the address search control. We will build this component up incrementally. A leaflet control has the type of `Control` with the additional `onAdd` and `onRemove` properties:

```tsx
import { Control, DomUtil } from 'leaflet';
import { useMap } from 'react-leaflet';
import { useEffect } from 'react';

export const AddressSearch = () => {
	const mapContainer = useMap();
	
	const SearchControl = Control.extend({
		options: {
			position: 'topright',
		},
		onAdd: function(map: Map) {
			//TODO
		},
		onRemove: function(map: Map) {
			return;
		}
	});

	const searchControl = new SearchControl();

	useEffect(() => {
		mapContainer.addControl(searchControl);
	}, []);

	return null;
}
```

Here is the core concept on adding custom leaflet controls in React. The `Control.extend` method returns a class that implements the leaflet `Control` type. We instantiate an instance of the new control, and through the  `useEffect` hook, we add the control with the `addControl` method on the `mapContainer` that is exposed through the `useMap` hook. The component itself then returns `null`.

The `<AddressSearch />` component can be added as a child of our `<Map>` component inside `Map.tsx`. This is handy so all control functionality remains encapsulated inside the Map component, and more importantly, the ui element will be rendered on top of the map tiles, which is a nice ux to have.
