---
layout: dsp_post
title: How to find N-nearest places using PostGIS
categories: database postgres "how to"
description: Leveraging PostGIS extension to solving non-trivial problem of solving N-nearest locations to a certain point
---

# Description of the problem

Couple weeks ago I've come across a really interesting problem I would like to describe here. The problem was about how to effectively find K-nearest points on the map to a given location. I have never been faced with this kind of problem, so I super excited to tackle it. I've definitely learned a lot along the way and I would like to explain this problem in simple words to you.

In the application I'm currently working on we store locations of certain places - for the sake of this article let's assume we store locations of the gyms. We store their coordinates (longitude and latitude) as numeric values.

The feature we would like to add is to select 5 nearest gyms for a given location.

## How to solve this problem?

The first thing that might come to our minds is, since we already have coordinates of the gyms in the database, that we can calculate the distance between given location and any gym's location, and return 5 with the lowest value.

As you may expect, this operation would be extremely slow and would certainly become a bottleneck of our application with the growth of the number of the gyms in the database. It definitely is not good enough even as a temporary solution.

Fortunately, there is a good solution to this problem - [PostGIS](http://www.postgis.net/).

## What is PostGIS

In a nutshell, [PostGIS](http://www.postgis.net/) is an extension of the [Postgres](https://www.postgresql.org/) database that adds very powerful geographical data processing capabilities to this database. It helps to perform spatial operations, like calculating distance or area, and adds spatial geometry data types to the database. It's open source, very fast and freely available.

## Solution

Firstly, thing we need to do is to install [PostGIS](http://www.postgis.net/). Assuming, that you already have [Postgres](https://www.postgresql.org/) installed, to add [PostGIS](http://www.postgis.net/) you have to connect to your database using **_psql_** or **_PgAdmin_** and run the following command

```
CREATE EXTENSION postgis;
```

Remember that you have to run this command separately on each database you want to use.

Next, we will add a new column to the `gyms` table. This column will be using a data type called **_geometric_** provided by the [PostGIS](http://www.postgis.net/). **_Geometry_** is the planar spatial data type and we will use it to calculate distance. Another possible option here is **_geography_** data type. **_Geography_** uses geodetic calculation instead of planar geometry, so it will give you greater precision but it's significantly slower.

Another thing worth mentioning here is the scale you want to operate. If you want to calculate distances within a couple of kilometers, let's call it a "city scale" - **_geometry_** might be sufficient for you. If you want to calculate distances on the "continental-scale", **_geography_** data type will give you better results.

In our case, since domain of the problem defines scale to couple of kilometers (we are not interested in gyms in Berlin when we live in London), **_geometry_** is good enough.

Type this SQL command in the console:

```
ALTER table gyms add column geom geometry(POINT,4326);
```

You might wonder, what this mysterious 4326 number is? This is a [SRID - Spatial Reference System Identifier](https://en.wikipedia.org/wiki/Spatial_reference_system#Identifier), and number 4326 means we will us [WGS 84 - (World Geodetic System)](https://en.wikipedia.org/wiki/World_Geodetic_System#WGS84). You can read more about Spatial Reference System [here](https://en.wikipedia.org/wiki/Spatial_reference_system).

Since we already have some gyms with their longitudes and latitudes in the database, our next task will be updating newly created column using those values. To do so run this SQL command:

```
UPDATE gyms set geom = ST_SetSRID(ST_MakePoint(longitude,latitude),4326);
```

We use **_ST_MakePoint_** with coordinates as an arguments to create point geometry, and then we set Spatial Reference System Identifier of this point.

The very last thing we need to add is the index on the **_geom_** column.

```
create index idx_gyms_geom on gyms using GIST(geom);
```

What is worth pointing out here is the **_GIST_** function. It's Postgres's implementation of the [Generic Search Tree](https://en.wikipedia.org/wiki/GiST), which allows to build indexes in almost any kind of data type. This is the reason indexing of the geographic data using [PostGIS](http://www.postgis.net/) or bioinformatics data using [BioPostgres](http://pgfoundry.org/projects/biopostgres) is possible.

## Find 5 nearest gyms to a given location

Now we have everything we need to solve our problem. We can test if everything works fine by running this command:

```
select * from gyms order by gyms.geom <-> '0101000020E610000082E2C798BB7E66C00000000000000000' limit 5;
```

This should return 5 records within blink a of an eye. Note, that '0101000020E610000082E2C798BB7E66C00000000000000000' is just the representation of the point (I took ti from the **_geom_** column), and **_<->_** is the [PostGIS](http://www.postgis.net/) operator that returns 2D distance between point A and B.

## Summary

When I first learned about this, I was intimidated by all the new concepts, like SRID, the difference between **_geometry_** and **_geography_** etc, so my goal here was to make it a little more approachable. I hope you find this article useful.
