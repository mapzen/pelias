[![Stories in Ready](https://badge.waffle.io/pelias/pelias.png?label=ready&title=Ready)](https://waffle.io/pelias/pelias)
Pelias is a modular, open-source geocoder built on top of ElasticSearch for fast geocoding. 

### What's a geocoder do anyway?

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
- Support multiple data sources (OSM, geonames, quattroshapes etc)
- Flexible software architecture
- Easy to contribute software patches and features to
- Easy to set-up and configure your own instance
- No external dependencies (such as postgres)
- Reliable, configurable & fast import process
- Work equally well for a small city and the entire planet

### I'm a developer, can I get access to the API?

Sure! Our API lives at [pelias.mapzen.com](http://pelias.mapzen.com/), and is usable without an API key and generous
rate-limits. The endpoints are documented [here](https://github.com/pelias/api/wiki/API-Endpoints); happy
geocoding!

```javascript
$ curl "pelias.mapzen.com/reverse?lat=40.74358294846026&lon=-73.99047374725342" | python -m json.tool
{
    "bbox": [
        -73.99051,
        40.74361,
        -73.99051,
        40.74361
    ],
    "date": 1427901226020,
    "features": [
        {
            "geometry": {
                "coordinates": [
                    -73.99051,
                    40.74361
                ],
                "type": "Point"
            },
            "properties": {
                "admin0": "United States",
                "admin1": "New York",
                "admin2": "New York County",
                "alpha3": "USA",
                "id": "9851011",
                "layer": "geoname",
                "name": "Arlington",
                "text": "Arlington, New York County, New York",
                "type": "geoname"
            },
            "type": "Feature"
        }
    ],
    "type": "FeatureCollection"
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
