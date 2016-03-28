Pelias is a modular, open-source geocoder built on top of ElasticSearch for fast geocoding.

Pelias is the project used to power [Mapzen Search](https://mapzen.com/projects/search)

### What's a geocoder do anyway?

[![Join the chat at https://gitter.im/pelias/pelias](https://badges.gitter.im/pelias/pelias.svg)](https://gitter.im/pelias/pelias?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

Geocoding is the process of transforming input text, such as an address, or a name of a place—to a location on the earth's surface.

![geocode](https://raw.githubusercontent.com/pelias/pelias/master/img/geocoding.gif)

### ... and a reverse geocoder, what's that?

Reverse geocoding is the opposite, it transforms your current geographic location in to a list of places nearby.

![reverse](https://raw.githubusercontent.com/pelias/pelias/master/img/reverse.gif)

### What makes Pelias different from other geocoders?

- It's completely open-source and MIT licensed
- It's based on open-data, so you can run it yourself
- You can install it locally and modify to suit your needs
- It has an impressive list of features, such as fast autocomplete
- It's modular, so you don't need to be an expert to make changes
- It's easy to install and requires no external dependencies
- We run a continuous deployment cycle with a new version shipping weekly

### What are the main goals of the Pelias project?

- Provide accurate search results
- Give users query suggestions (typeahead in the search box)
- Account for location bias (places nearer to you appear higher in the results)
- Support multiple data sources (Defaults include OpenStreetMap, OpenAddresses, geonames, quattroshapes etc)
- Flexible software architecture
- Easy to contribute software patches and features to
- Easy to set-up and configure your own instance
- No external dependencies (such as postgres)
- Reliable, configurable & fast import process
- Work equally well for a small city and the entire planet

### Developer Documentation & API Access

Sure! Our API lives at [search.mapzen.com](http://search.mapzen.com/), and is usable with a free API key ([register here](https://mapzen.com/developers)) and generous
rate-limits. The endpoints are documented [here](https://github.com/pelias/pelias-doc/blob/master/index.md); happy
geocoding!

[This documentation](https://github.com/pelias/pelias-doc/blob/master/index.md) also applies to standalone versions of Pelias, leaving aside API keys, privacy flags, and data sources which may be configured differently for other installations.

```javascript
$ curl -s "search.mapzen.com/v1/reverse?size=1&point.lat=40.74358294846026&point.lon=-73.99047374725342&api_key={YOUR_API_KEY}" | json
{
  "geocoding": {
    "version": "0.1",
    "attribution": "http://search.mapzen.com/v1/attribution",
    "query": {
      "size": 1,
      "private": false,
      "point.lat": 40.74358294846026,
      "point.lon": -73.99047374725342,
      "boundary.circle.lat": 40.74358294846026,
      "boundary.circle.lon": -73.99047374725342,
      "boundary.circle.radius": 50
    },
    "engine": {
      "name": "Pelias",
      "author": "Mapzen",
      "version": "1.0"
    },
    "timestamp": 1443189055394
  },
  "type": "FeatureCollection",
  "features": [
    {
      "type": "Feature",
      "properties": {
        "id": "9851011",
        "gid": "gn:venue:9851011",
        "layer": "venue",
        "source": "gn",
        "name": "Arlington",
        "country_a": "USA",
        "country": "United States",
        "region": "New York",
        "region_a": "NY",
        "county": "New York County",
        "localadmin": "Manhattan",
        "locality": "New York",
        "neighbourhood": "Flatiron District",
        "confidence": 0.9,
        "distance": 0.004,
        "label": "Arlington, Manhattan, NY"
      },
      "geometry": {
        "type": "Point",
        "coordinates": [
          -73.99051,
          40.74361
        ]
      }
    }
  ],
  "bbox": [
    -73.99051,
    40.74361,
    -73.99051,
    40.74361
  ]
}
```

### How can I install my own instance of Pelias?

Check out our [vagrant development environment](https://github.com/pelias/vagrant).

### How does it work?

Magic! Well, like any geocoder, Pelias essentially just executes search queries against an enormous amount of
geographic data that maps longitude/latitude coordinates on the Earth to searchable names (eg `Empire State Building`
or `28 Elm Street`).  We run entirely on open datasets, like [OpenStreetMap](http://www.openstreetmap.org/),
[GeoNames](http://www.geonames.org/about.html), and [OpenAddresses](http://openaddresses.io/).

The underlying architecture has three components:

  * **import pipelines**: the pipelines used to filter, normalize, and ingest geographic datasets into the Pelias database.
  * **database**: the underlying datastore that does all of the query heavy-lifting and powers our search results. We use
    [ElasticSearch](https://www.elastic.co/).
  * **API**: the thing that users interact with. A thin layer sitting on top of the datastore that implements additional
    logic and features.

Here's how they interact:

![A diagram of the Pelias architecture.](https://cloud.githubusercontent.com/assets/4467604/6944539/3b1cdd0e-d862-11e4-995d-0b376caacad6.png)

### What's it built with?
Pelias itself (the import pipelines and API) is written in [Node](https://nodejs.org/), which makes it highly
accessible for other developers and performant under heavy I/O. It aims to be modular and is distributed across a
number of Node packages, each with its own repository under the [Pelias GitHub
organization](https://github.com/pelias). ElasticSearch is our unconventional datastore of choice because of its
unparalleled text functionality, which makes text search *just work* right out of the box, and sufficiently robust
geospatial support.

### Contributing

We built Pelias as an open source project not just because we believe that users should be able to view and play with
the source code of tools they use, but to get the community involved in the project itself.

Anything that we can do to make contributing easier, we want to know about.  Feel free to reach out to us via Github,
[Gitter](https://gitter.im/pelias/pelias), [email](mailto:search@mapzen.com), or [Twitter](https://twitter.com/mapzen])
 We'd love to help people get started working on Pelias, especially
if you're new to open source or programming in general. Both this [meta-repo](https://github.com/pelias/pelias/issues)
and the [API repo](https://github.com/pelias/api/issues) are good places to get started looking for tasks to tackle.
You can also look across all of our issues on our [meta-issue tracker](https://waffle.io/pelias/pelias), Waffle.
 We also welcome reporting issues or suggesting improvements to our [documentation](https://github.com/pelias/pelias-doc).

The current Pelias team can be found on Github as [dianashk](https://github.com/dianashk),
[missinglink](https://github.com/missinglink), [orangejulius](https://github.com/orangejulius),
[riordan](https://github.com/riordan), and [stephenkhess](https://github.com/stephenkhess).

Members emeritus include: [randyme](https://github.com/randyme), [seejohnrun](https://github.com/seejohnrun),
 [sevko](https://github.com/sevko), and [hkrishna](https://github.com/hkrishna).
