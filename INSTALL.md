# Installing Pelias

Mapzen offers the Mapzen Search service in hopes that as many people as possible will use it,
but we also encourage people to set up their own Pelias instance.

For most cases, it's useful to have much of the installation process automated, so we suggest
looking at the [Pelias Vagrant image](https://github.com/pelias/vagrant).

However, for more in-depth usage, to learn more about the working of Pelias, or to contribute back,
manual setup is useful. These instructions will help you install Pelias from scratch manually.

## Installation Overview

The steps for fully installing Pelias look like this:

1. Decide which datasets and settings will be used
2. Download appropriate data
3. Download Pelias code, using the appropriate branches
4. Set up Elasticsearch
5. Install the Elasticsearch schema using pelias-schema
6. Use one or more importers to load data into Elasticsearch
7. Install the libpostal text analyzer (optional)
8. Start the API server to begin handling queries

## System Requirements

In general, Pelias will require:

* A working [Elasticsearch](https://www.elastic.co/products/elasticsearch) 2.3 cluster. It can be on
  a single machine or across several
* [Node.js](https://nodejs.org/) 4.0 or newer (the latest in the Node 4 or 6 series is recommended). Node.js 0.10 and 0.12 are no longer supported
* Up to 100GB disk space to download and extract data
* Lots of RAM, 8GB is a good minimum. A full North America OSM import just fits in 16GB RAM


## Choose your datasets

Pelias can currently import data from four different sources. The contents and description of these
sources are available on our [data sources page](https://github.com/pelias/pelias-doc/blob/master/data-sources.md). Here we'll just focus on what to
download for each one.

### Who's on First

The [Who's on First](https://github.com/pelias/whosonfirst#data) importer can download all the Who's
on First data quickly and easily. See the README for the most up to date instructions.

### Geonames

The [pelias/geonames](https://github.com/pelias/geonames/#importing-data) importer contains code and
instructions for downloading Geonames data automatically. Individual countries, or the entire planet
(1.3GB compressed) can be specified.

### OpenAddresses
The OpenAddresses project includes [numerous download options](https://results.openaddresses.io/),
all of which are `.zip` downloads. The full dataset is just over 6 gigabytes compressed (the
extracted files are around 30GB), but there are numerous subdivision options. In any case, the
`.zip` files simply need to be extracted to a directory of your choice, and Pelias can be configured
to either import every `.csv` in that directory, or only selected files.

### OpenStreetMap

OpenStreetMap has a nearly limitless array of download options, and any of them should work as long as
they're in [PBF](http://wiki.openstreetmap.org/wiki/PBF_Format) format. Generally the files will
have the extension `.osm.pbf`. Good sources include the [Mapzen Metro Extracts](https://mapzen.com/data/metro-extracts/)
(which has popular cities available immediately, or custom areas that take only
a few minutes to build), and planet files listed on the [OSM wiki](http://wiki.openstreetmap.org/wiki/Planet.osm).
A full planet PBF file is about 36GB.

## Choose your import settings

There are several options that should be discussed before starting any data imports, as they require
a compromise between import speed and resulting data quality and richness.

### Admin Lookup

Most data that is imported by Pelias comes to us incomplete: many data sources don't supply what we
call admin hierarchy information: the neighbourhood, city, country, or other region that contains
the record. In OpenAddresses, for example, many records contain only a housenumber, street name, and
coordinates.

Fortunately, Who's on First contains a well-developed set of geometries for all admin regions from the
neighbourhood to continent level. Through
[point-in-polygon](https://en.wikipedia.org/wiki/Point_in_polygon) lookup, our importers can
[derive](https://github.com/pelias/wof-admin-lookup) this information!

The downsides to enabling admin lookup are increased memory requirements and longer import times.
Because geometry data is quite large, expect to use about 6GB of RAM (not disk) during import just
for this geometry data. And because of the complexity of the required calculations, imports with
admin lookup are up to 10 times slower than without.

Who's on First, of course, always includes full hierarchy information because it's built into the
dataset itself, so there's no tradeoff to be made. Who's on First data will always import quite fast
and with full hierarchy information.

### Address Deduplication

OpenAddresses data contains lots of addresses, but it also contains lots of duplicate data. To help
reduce this problem we've built an [address-deduplicator](https://github.com/pelias/address-deduplicator)
that can be run at import. It uses the [OpenVenues deduplicator](https://github.com/openvenues/address_deduper)
to remove records that are near each other and have names that are likely to be duplicates. Note
that it's considerably smarter than simply doing exact comparisons of names and coordinates: it uses
[Geohash prefixes](https://en.wikipedia.org/wiki/Geohash) to compare nearby records, and the
[libpostal address normalizer](https://github.com/openvenues/libpostal#examples-of-normalization) to
compare names, so it can tell that records with `101 Main St` and `101 Main Street` are likely to
refer to the same place.

Unfortunately, our current implementation is very slow, and requires about 50GB of scratch disk
space during a full planet import. It's worth noting that Mapzen Search currently does _not_
deduplicate any data, although we hope to improve the performance of deduplication and resume using
it eventually.

## Considerations for full-planet builds

As may be evident from the dataset section above, importing all the data in all four supported datasets is
worthy of its own discussion. Current [full planet builds](https://pelias-dashboard.mapzen.com/pelias)
weigh in at over 320 million documents, and require about 230GB total storage in Elasticsearch.
Needless to say, a full planet build is not likely to succeed on most personal computers.

Fortunately, because of services like AWS and the scalability of Elasticsearch, full planet builds
are possible without too much extra effort. To set expectations, a cluster of 4
[r3.xlarge](https://aws.amazon.com/ec2/instance-types/) AWS instances running Elasticsearch, and one
c4.8xlarge instance running the importers can complete a full planet build in about two days.

## Choose your Pelias code branch

As part of the setup instructions below, you'll be downloading several Pelias packages from source
on Github. All of these packages offer 3 branches for various use cases. Based on your needs, you
should pick one of these branches and use the same one across all of the Pelias packages.

`production`: contains only code that has been tested against a full-planet build and is live on
Mapzen Search. This is the "safest" branch and it will change the least frequently, although we
generally release new code at least once a week.

`staging`: these branches contain the code that is currently being tested against a full planet
build for imminent release to Mapzen Search. It's useful to track what code will be going out in the
next release, but not much else.

`master`: master branches contain the latest code that has passed code review, unit/integration
tests, and is ready to be included in the next release. While we try to avoid it, the nature of the
master branch is that it will sometimes be broken. That said, these are the branches to use for
development of new features.

## Installation

### Download the Pelias repositories

At a minimum, you'll need the Pelias [schema](https://github.com/pelias/schema/) and
[api](https://github.com/pelias/api/) repositories, as well as at least one of the importers. Here's
a bash snippet that will download all the repositories (they are all small enough that you don't
have to worry about the space of the code itself), check out the production branch (which is
probably the one you want), and install all the node module dependencies.

```bash
for repository in schema api whosonfirst geonames openaddresses openstreetmap; do
	git clone git@github.com:pelias/${repository}.git
	pushd $repository > /dev/null
	git checkout production # or staging, or remove this line to stay with master
	npm install
	popd > /dev/null
done
```

### Customize Pelias Config

Nearly all configuration for Pelias is driven through a single config file: `pelias.json`. By
default, Pelias will look for this file in your home directory, but you can configure where it
looks. For more details, see the [pelias-config](https://github.com/pelias/config) repository.

The two main things of note to configure are where on the network to find Elasticsearch, and where
to find the downloaded data files.

Pelias will by default look for Elasticsearch on `localhost` at port 9200 (the standard
Elasticsearch port).

By taking a look at the [default config](https://github.com/pelias/config/blob/master/config/defaults.json#L2),
you can see the Elasticsearch configuration looks something like this:

```js
{
  "esclient": {
  "hosts": [{
    "host": "localhost",
    "port": 9200
  }]

  ... // rest of config
}
```

If you want to connect to Elasticsearch somewhere else, change `localhost` as needed. You can
specify multiple hosts if you have a large cluster. In fact, the entire `esclient` section of the
config is sent along to the [elasticsearch-js](https://github.com/elastic/elasticsearch-js) module, so
any of its [configuration options](https://www.elastic.co/guide/en/elasticsearch/client/javascript-api/current/configuration.html)
are valid.

The other major section, `imports`, defines settings for each importer. The defaults look like this:

```json
{
 "imports": {
    "geonames": {
      "datapath": "./data",
      "adminLookup": false
    },
    "openstreetmap": {
      "datapath": "/mnt/pelias/openstreetmap",
	  "adminLookup": false,
      "leveldbpath": "/tmp",
      "import": [{
        "filename": "planet.osm.pbf"
      }]
    },
    "openaddresses": {
      "datapath": "/mnt/pelias/openaddresses",
      "adminLookup": false,
      "files": []
    },
    "whosonfirst": {
      "datapath": "/mnt/pelias/whosonfirst"
    }
  }
}
```

As you can see, the default datapaths are meant to be changed. This is also where you can enable
admin lookup by overriding the default value.

### Elasticsearch Configuration

Of special note in `pelias.json` are Elasticsearch settings. The [default](https://github.com/pelias/config/blob/master/config/defaults.json)
settings (see the `elasticsearch` section) will be fine for development, but in particular the shard count should be
increased for production use. Mapzen Search uses 24 shards in production (for a full planet build).
Smaller installations should probably at least use the Elasticsearch default of 5 shards:

```js
{
  "elasticsearch": {
    "settings": {
      "index": {
        "number_of_shards": "5",
      }
    }
  }
}
```

### Install Elasticsearch

Other than requiring Elasticsearch 2.3, nothing special in the Elasticsearch setup is required for
Pelias, so please refer to the [official 2.3 install docs](https://www.elastic.co/guide/en/elasticsearch/reference/2.3/setup.html).

Older versions of Elasticsearch are not supported.

Make sure Elasticsearch is running and connectable, and then you can continue with the Pelias
specific setup and importing. Using a plugin like [head](https://mobz.github.io/elasticsearch-head/)
or [Marvel](https://www.elastic.co/products/marvel) can help monitor Elasticsearch as you import
data.

If you're using a terminal, you can also search and/or monitor Elasticsearch using their [APIs.](https://www.elastic.co/guide/en/elasticsearch/reference/2.3/api-conventions.html)

**Note:** On large imports, Elasticsearch can be very sensitive to memory issues. Be sure to modify it's [heap size](https://www.elastic.co/guide/en/elasticsearch/guide/2.x/heap-sizing.html) from the default confiration to something more appropriate to your machine.

### Set up the Elasticsearch Schema

The Elasticsearch Schema is analogous to the layout of a table in a traditional relational database,
like MySQL or PostgreSQL. While Elasticsearch attempts to auto-detect a schema that works when
inserting new data, this generally leads to non-optimal results. In the case of Pelias, inserting
data without first applying the Pelias schema will cause all queries to fail completely: Pelias
requires specific configuration settings for both performance and accuracy reasons.

Fortunately, now that your `pelias.json` file is configured with how to connect to Elasticsearch,
the Schema repository can automatically create the Pelias index and configure it exactly as needed:

```bash
cd schema # assuming you've just run the bash snippet to download the repos from earlier
node scripts/create_index.js
```

If you want to reset the schema later (to start over with a new import or because the schema code
has been updated), you can drop the index and start over like so:

```bash
# !! WARNING: this will remove all your data from pelias!!
node scripts/drop_index.js # it will ask for confirmation first
node scripts/create_index.js
```

Note that Elasticsearch has no analogy to a database migration, so you generally have to delete and
reindex all your data after making schema changes.

### Run the importers

Now that the schema is set up, you're ready to begin importing data.

For all importers except for Geonames, you can start the import process with the `npm start`
command:

```bash
cd $importer_directory; npm start
```

For the [Geonames](https://github.com/pelias/geonames/) importer, please see its
[README](https://github.com/pelias/geonames/blob/master/README.md) file for the most up to date
instructions.  We are working towards making all the importers have [the same interface](https://github.com/pelias/pelias/issues/255),
so the Geonames importer will behave the same as the others soon.

Depending on how much data you've imported, now may be a good time to grab a coffee. Without admin
lookup, the fastest speeds you'll see are around 10,000 records per second. With admin lookup,
expect around 800-2000 inserts per second.

### Install Libpostal (optional, but recommended)

Pelias is now able to use the [libpostal](https://github.com/openvenues/libpostal) address parser,
which greatly increases the quality of search results. Libpostal must be installed on the machines
running the Pelias API, and requires about 4GB of disk space to download all the required data. This
data represents a statistical natural language processing model of address parsing trained on
OpenStreetMap data. The API will also require about 2GB of memory (it used only a few hundred
before), to store the needed data for queries.

First, install libpostal following its [installation docs](https://github.com/openvenues/libpostal#installation).
This will also download the training data, so be sure to have enough free disk space.

Next, configure the Pelias API to use libpostal (it won't by default) by adding a section like this
to `pelias.json`:

```json
{
  "api": {
    "textAnalyzer": "libpostal"
  }
}
```

In the future, libpostal may become the default, and we may drop support for
[addressit](https://github.com/DamonOehlman/addressit), the current default text parser. Until then,
the `textAnalyzer` property can be changed back to `addressit` (or removed) to stop using libpostal.

Once configured, the API will use libpostal via the [node-postal](https://github.com/openvenues/node-postal)
NPM module.

### Start the API

As soon as you have any data in Elasticsearch, you can start running queries against the
[Pelias API server](https://github.com/pelias/api/).

Again thanks to `pelias.json`, the API already knows how to connect to Elasticsearch, so all that's
required to star the API is `npm start`. You can now send queries to `http://localhost:3100/`!
