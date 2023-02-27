# Theme Documentation - mapbox Shortcode


{{< version 0.2.0 >}}

The `mapbox` shortcode supports interactive maps in Hugo with [Mapbox GL JS](https://docs.mapbox.com/mapbox-gl-js) library.

<!--more-->

**Mapbox GL JS** is a JavaScript library that uses WebGL to render interactive maps from [vector tiles](https://docs.mapbox.com/help/glossary/vector-tiles/) and [Mapbox styles](https://docs.mapbox.com/mapbox-gl-js/style-spec/).

The `mapbox` shortcode has the following named parameters to use Mapbox GL JS:

- **lng** _[required]_ (**first** positional parameter)

  Longitude of the inital centerpoint of the map, measured in degrees.

- **lat** _[required]_ (**second** positional parameter)

  Latitude of the inital centerpoint of the map, measured in degrees.

- **zoom** _[optional]_ (**third** positional parameter)

  The initial zoom level of the map, default value is `10`.

- **marked** _[optional]_ (**fourth** positional parameter)

  Whether to add a marker at the inital centerpoint of the map, default value is `true`.

- **light-style** _[optional]_ (**fifth** positional parameter)

  Style for the light theme, default value is the value set in the [front matter](../theme-documentation-content#front-matter) or the [site configuration](../theme-documentation-basics#site-configuration).

- **dark-style** _[optional]_ (**sixth** positional parameter)

  Style for the dark theme, default value is the value set in the [front matter](../theme-documentation-content#front-matter) or the [site configuration](../theme-documentation-basics#site-configuration).

- **navigation** _[optional]_

  Whether to add [NavigationControl](https://docs.mapbox.com/mapbox-gl-js/api#navigationcontrol), default value is the value set in the [front matter](../theme-documentation-content#front-matter) or the [site configuration](../theme-documentation-basics#site-configuration).

- **geolocate** _[optional]_

  Whether to add [GeolocateControl](https://docs.mapbox.com/mapbox-gl-js/api#geolocatecontrol), default value is the value set in the [front matter](../theme-documentation-content#front-matter) or the [site configuration](../theme-documentation-basics#site-configuration).

- **scale** _[optional]_

  Whether to add [ScaleControl](https://docs.mapbox.com/mapbox-gl-js/api#scalecontrol), default value is the value set in the [front matter](../theme-documentation-content#front-matter) or the [site configuration](../theme-documentation-basics#site-configuration).

- **fullscreen** _[optional]_

  Whether to add [FullscreenControl](https://docs.mapbox.com/mapbox-gl-js/api#fullscreencontrol), default value is the value set in the [front matter](../theme-documentation-content#front-matter) or the [site configuration](../theme-documentation-basics#site-configuration).

- **width** _[optional]_

  Width of the map, default value is `100%`.

- **height** _[optional]_

  Height of the map, default value is `20rem`.

Example simple `mapbox` input:

```markdown
{{</* mapbox 121.485 31.233 12 */>}}
Or
{{</* mapbox lng=121.485 lat=31.233 zoom=12 */>}}
```

The rendered output looks like this:

{{< mapbox 121.485 31.233 12 >}}

Example `mapbox` input with the custom style:

```markdown
{{</* mapbox -122.252 37.453 10 false "mapbox://styles/mapbox/navigation-preview-day-v4" "mapbox://styles/mapbox/navigation-preview-night-v4" */>}}
Or
{{</* mapbox lng=-122.252 lat=37.453 zoom=10 marked=false light-style="mapbox://styles/mapbox/navigation-preview-day-v4" dark-style="mapbox://styles/mapbox/navigation-preview-night-v4" */>}}
```

The rendered output looks like this:

{{< mapbox -122.252 37.453 10 false "mapbox://styles/mapbox/navigation-preview-day-v4?optimize=true" "mapbox://styles/mapbox/navigation-preview-night-v4?optimize=true" >}}

