
The openstreetmap importer provides a way of parsing, mapping and augmenting OSM data in to elasticsearch.

## Prerequisites

In order to use the importer you must first install and configure `elasticsearch` and `nodejs`.

You can follow the instructions here: https://github.com/pelias/pelias/blob/master/INSTALL.md to get set up.

The recommended version for nodejs is `0.12+` and elasticsearch is `1.3.4+` although we also test against `nodejs@0.10`.

Before continuing you should confirm that you have these tools correctly installed and elasticsearch is running on port `9200`.

## Elasticsearch plugin

In order to perform more complex analysis of text we have a custom elasticsearch plugin which needs to be installed.

You can find more information about the plugin here: https://github.com/pelias/elasticsearch-plugin

## Clone and Install dependencies

```bash
$ git clone https://github.com/pelias/openstreetmap.git && cd openstreetmap;
$ npm install
```

## Download data

You will need to download quattroshapes in order to build an admin hierarchy for each record, you can pull-down a tarball from http://data.mapzen.com/quattroshapes/quattroshapes-simplified.tar.gz which you will need to extract, preferably to an SSD if you have one. 

The importer will accept any valid `pbf` extract you have, this can be a full planet file (25GB+) from http://planet.openstreetmap.org/ or a smaller extract from https://mapzen.com/metro-extracts/ or http://download.geofabrik.de/

## Configuration

In order to tell the importer the location of your downloads, temp space and enviromental settings you will first need to create a `~/pelias.json` file.

As a minumum, it should contain the following:

```javascript
{
  "logger": {
    "level": "debug"
  },
  "esclient": {
    "hosts": [{
      "env": "development",
      "protocol": "http",
      "host": "localhost",
      "port": 9200
    }]
  },
  "imports": {
    "quattroshapes": {
      "datapath": "/data/quattro"
    },
    "openstreetmap": {
      "leveldbpath": "/tmp",
      "datapath": "/data/pbf",
      "import": [{
        "type": { "node": "osmnode", "way": "osmway" },
        "filename": "london_england.osm.pbf"
      }]
    }
  }
}
```

Make sure you change the following settings to reflect your environment:

- `imports.quattroshapes.datapath` - this is the directory where you extracted the `.shp` and other files from the quattroshapes download
- `imports.openstreetmap.datapath` - this is the directory which you downloaded the pbf file to
- `imports.openstreetmap.import[0].filename` - this is the name of the pbf file you downloaded

You can optionally change:

- `imports.openstreetmap.leveldbpath` - this is the directory where temporary files will be stored in order to denormalize osm ways, in the case of a planet import it is best to have 100GB free so you don't run out of disk.

If your paths point to an SSD rather than a HDD then you will get a significant speed boost, although this is not required.

## Configuring the elasticsearch mappings

While `elasticsearch` is technically schema-less, the data will not be correctly stored unless you first tell it how the data is to be indexed.

```bash
$ git clone https://github.com/pelias/schema.git && cd schema;
$ npm install
$ node scripts/create_index.js
```

In order to confirm that the mappings have been correctly inserted in to elasticsearch you can now query http://localhost:9200/pelias/_mapping

## Running an import

This will start the import process, it will take around 30 seconds to prime it's in-memory data and then you should see regular debugging output in the terminal.

```bash
$ node index.js
```

## Issues

If you have any issues getting set up or the documentation is missing something, please open an issue here: https://github.com/pelias/openstreetmap/issues

## Contributing

Please fork and pull request against upstream master on a feature branch.

Pretty please; provide unit tests and script fixtures in the `test` directory.

### Running Unit Tests

```bash
$ npm test
```

### Continuous Integration

Travis tests every release against node version `0.10` & `0.12`.

[![Build Status](https://travis-ci.org/pelias/openstreetmap.png?branch=master)](https://travis-ci.org/pelias/openstreetmap)