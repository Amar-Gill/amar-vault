TLDR
how to create a custom control element and add it to a leaflet map in a react app

Leaflet is a great package to easily integrate map and GIS functionality into your web application. Bonus points for being open source so you don't need to rely on Google Maps (which is also a great option).

The goal of this article is to add a custom address search control component inside the leaflet map container. I emphasize inside, because the only way to render UI elements ontop of the map tiles is to use the Leaflet controls API.

This article is focused on creating a custom control for the React flavour of Leaflet. There are plenty of articles on the web building custom leaflet controls using Vanilla JS.

### Getting Started
Let's add a standard map to our React app using the `react-leaflet` package.

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

### Creating our Custom Control
Before we make the interactive control element, let's see how to add an arbitrary html element as a control overlay to the leaflet map instance. A leaflet control has the type of `Control` with the additional `onAdd` and `onRemove` properties:

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

			const el = DomUtil.create('div');

			el.className = 'basic-control';

			return el;
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

Inside the options of the `Control.extend` method, we show a trivial example of how to add a simple `<div>`. Leaflet has a built in `DomUtil.create` function, which takes in a string representing an html tag name. We give it the `.basic-control` css class so we can visualize it, and then return the element from the `onAdd` method.

The `<AddressSearch />` component can then be added as a child of the Leaflet `<MapContainer>` component inside `Map.tsx`. This is handy so all control functionality remains encapsulated inside the Map component, and more importantly, the ui element will be rendered on top of the map tiles, which makes for an intuitive map control ux in my opinion.

```tsx
<MapContainer>
	<TileLayer />
	<AddressSearch />
</MapContainer>
```

### The Geoapify Address Search
Let's make something useful now. In this example we will add a text input control that allows us to look up an address, and have the map set it's view to the location. 

We will leverage the [Geoapify]() api for the address look up functionality. It has a generous request limit of 3000 api calls daily for the free tier, which is handy for prototyping. Geoapify also has a handy [address search npm package]() which makes it easy to create the input element with all styling and interactivity we need for the component, including making api calls to the Geoapify api.

Note: there is a separate [npm package for a react component]() that wraps address search functionality and exposes it as a react component. We will **not** be using the react package, because only native DOM elements can be added as a custom control onto a Leaflet map instance. As shown in the previous example, we returned an element which implements the `HTMLDivElement` type.

Before we proceed, you will need to visit the [geoapify]() platform and register so you can obtain an api key.

Next, install the package for the address search element:

```bash
npm i @geoapify/geocoder-autocomplete
```

Import the stylesheet into your `globals.css` file:

```css
@import '~@geoapify/geocoder-autocomplete/styles/minimal.css';
```

Let's modify the  `<AddressSearch />` as follows:

```tsx
import { GeocoderAutocomplete } from '@geoapify/geocoder-autocomplete';
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

			const el = DomUtil.create('div');

			el.className = 'relative minimal';
			
			el.addEventListener('click', (e) => {
				e.stopPropagation();
			});

			el.addEventListener('dblclick', (e) => {
				e.stopPropagation();
			});

			const autocomplete = new GeocoderAutocomplete(
				el,
				`${process.env.NEXT_PUBLIC_GEOAPIFY}`,
				{
					placeholder: 'Enter an address',
				},
			);

			autocomplete.on('select', (location) => {
				map.setView(newPosition, map.getZoom());
			});

			autocomplete.on('suggestions', (suggestions) => {
				return;
			});

			return el;
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

The `GeocoderAutocomplete` class accepts as arguments an html element, the api key, and some optional configurations. That line of code will add the search input element as a child of the parent `<div>` element denoted by `el`. 

more functions.

So when we return `el` from `onAdd`, we get the fully functional address search input element as a map control.

### Optimization
something something production, experienced react devs.

Let's wrap the expensive operations of creating dom elements and adding event listeners with the `useMemo` hook. We can return the instance of the `SearchControl` class from the hook, so that it is available in our `useEffect` hook:

```tsx
export const AddressSearch = () => {
  const mapContainer = useMap();

  const searchControl = useMemo(() => {
    const SearchControl = Control.extend({
      options: {
        position: 'topright',
      },
      onAdd: function (map: Map) {
        const el = DomUtil.create('div');

        el.className = 'relative minimal round-borders';

        el.addEventListener('click', (e) => {
          e.stopPropagation();
        });

        el.addEventListener('dblclick', (e) => {
          e.stopPropagation();
        });

        const autocomplete = new GeocoderAutocomplete(
          el,
          `${process.env.NEXT_PUBLIC_GEOAPIFY}`,
          {
            placeholder: 'Enter an address',
          },
        );

        autocomplete.on('select', (location) => {
          map.setView(newPosition, map.getZoom());
        });

        autocomplete.on('suggestions', (suggestions) => {
          return;
        });

        return el;
      },
      onRemove: function (map: Map) {
        return;
      },
    });

    return new SearchControl();
  }, [setPosition]);

  useEffect(() => {
    mapContainer.addControl(searchControl);
  }, [mapContainer, searchControl]);

  return null;
};
```