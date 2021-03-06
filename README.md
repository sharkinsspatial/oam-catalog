<h1 align="center">OAM Catalog API
  <a href="https://travis-ci.org/hotosm/oam-catalog">
    <img src="https://api.travis-ci.org/hotosm/oam-catalog.svg?branch=master" alt="Build Status"></img>
  </a></h1>

<div align="center">
  <h3>
  <a href="https://docs.openaerialmap.org/ecosystem/getting-started/">Ecosystem</a>
  <span> | </span>
  <a href="https://github.com/hotosm/oam-browser">Imagery Browser</a>
  <span> | </span>
  <a href="https://github.com/hotosm/openaerialmap.org">OAM Homepage</a>
  </h3>
</div>

[![Deploy](https://www.herokucdn.com/deploy/button.png)](https://heroku.com/deploy)

A catalog for OpenAerialMap imagery. The application indexes all metadata available within Open Imagery Network and creates an API to search and find imagery. The API powers the frontend search tool, OAM Imagery Browser. Read the [ecosystem documentation](https://docs.openaerialmap.org/ecosystem/getting-started/) for more information about OpenAerialMap.

## Installation and Usage

The steps below will walk you through setting up your own instance of the oam-catalog.

### Install Project Dependencies

- [MongoDB](https://www.mongodb.org/)
- [Node.js](https://nodejs.org/) (v4.5.x)

### Install Application Dependencies

If you use [`nvm`](https://github.com/creationix/nvm), activate the desired Node version:

```
nvm install
```

Install Node modules:

```
npm install
```

### Usage

#### Starting the database:

```
mongod
```

The database is responsible for storing metadata about the imagery and analytics.

#### Starting the API:

```
node index.js
```

The API exposes endpoints used to access information form the system via a RESTful interface.

#### Starting the background worker:

```
node worker.js
```

The worker process runs on a schedule (every 5 minutes by default) and checks for new data, update database when it finds anything to add.

### Environment Variables

- `OAM_DEBUG` - Debug mode `true` or `false` (default)
- `AWS_SECRET_KEY_ID` - AWS secret key id for reading OIN buckets
- `AWS_SECRET_ACCESS_KEY` - AWS secret access key for reading OIN buckets
- `OIN_REGISTER_URL` - URL to register file containing location of imagery buckets
- `DBURI` - MongoDB connection url
- `CRON_TIME` - A valid cron string (default is every 5 minutes) for the worker schedule
- `SECRET_TOKEN` - The token used for post requests to `/tms` endpoint
- `NEW_RELIC_LICENSE_KEY` - Your New Relic API monitoring license key

If you are running a local OAM bucket, here are additional environment variables you can configure (more information in the Docker > Local Indexing section)
- `HOST_PREFIX`: - Used by the localoam service to construct URIs that point to the images (default: `http://localoam`)
- `LOCAL_OAM_BUCKET`: - Used by the localoam service as the location of the HTTP server (default is a docker context service name: `http://localoam:4999`)

For development purposes, `NEW_RELIC_LICENSE_KEY` can be omitted. Although the system will work some functionality will not be available and errors may be triggered.


## Endpoints and Parameters

More API documentation can be found at: [tbd]. 

### Available Endpoints

-  `/meta` -XGET
-  `/meta/add` -XPOST
-  `/tms` -XGET
-  `/tms` -XPOST
-  `/analytics` -XGET

### POST parameters for `/tms`:

To add/update `/tms` endpoint, the following JSON format should be used:

```json
{
    "uri": "http://example.com/tms_uri",
    "images": [
        {
            "uuid": "http://example.com/image_uri.tif"
        }
    ]
}
```
*Note that the `/tms` endpoint requires authenticated access.*

### Search parameters for `/meta`:

#### bbox:

- format: `?bbox=[lon_min],[lat_min],[lon_max],[lat_max]`
- example: `/meta?bbox=-66.15966796875,46.45678142812658,-65.63232421875,46.126556302418514`

#### title:

- format: `?title=string`
- example: `/meta?title=sometitle`


#### provider:

- format: `?provider=string`
- example: `/meta?provider=someprovider`

#### GSD (Resolution):

- format: `?gsd_from=value&gsd_to=value`
- example: `/meta?gsd_from=0.005&gsd_to=20`

*Note that gsd_from and gsd_to can be used on their own. Values should be provided in meters.*

#### has tiled service?:

- format: `?has_tiled`
- example: `/meta?has_tiled`

#### page:

- format: `?page=number`
- example: `/meta?page=2`

#### date:
- format: `?acquisition_from=date&acquisition_to=date`
- example: `/meta?acquisition_from=2015-04-10&acquisition_to=2015-05-01`

*Note that acquisition_from and acquisition_to can be used on their own.*

#### limit:

default is `100`.

- format: `?limit=number`
- example: `/meta?limit=1000`

#### sorting and ordering:

- format: `?order_by=property&sort=asc|desc`
- example: `/meta?order_by=acquisition_start&sort=asc`

*Note that `sort` and `order_by` are required together and one alone will not be recognized. Default is to show higher resolution and newer imagery first.*


### Documentation

The documentation for the different endpoints can be generated by running:
```
npm run docs
```
This will compile the documentation and place it inside `docs`. The docs site can then be run by any web server.

The documentation is also automatically built and deployed by [Travis CI](https://travis-ci.org/) whenever a Pull Request is merged to the production branch (in this case `master`). The deployment is done to `gh-pages`. (http://hotosm.github.io/oam-catalog/)

### Deployment

Changes to the `master` branch are automatically deployed via Travis to https://oam-catalog.herokuapp.com.

## Docker
To package the app as a container:

- Ensure that docker and docker-compose are installed
- Copy `local.sample.env` to `local.env` and fill the values according to the instructions above.
- Run `docker-compose up`.

The app will be available at `http://localhost:4000`


### Local indexing
For local indexing, an alternative `docker-compose` configuration file can be used. This will launch an additional service, a file server for a folder with JSON metadata files that fit the OAM spec. The metadata files need a `HOST_PREFIX` (such as the default `http://localhost:4999`) that points to the location of the `localoam` service running on the network. 

- Copy `.env.sample` to `.env` and add the path to the volume you want to index as well as the `HOST_PREFIX`
- Copy `local.sample.env` to `local.env` and fill the values according to the instructions above.
- Run `docker-compose -f docker-compose-local.yml up`

## License
Oam Catalog is licensed under **BSD 3-Clause License**, see the [LICENSE](LICENSE) file for more details.

