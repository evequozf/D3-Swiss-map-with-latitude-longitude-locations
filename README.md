# Integrate Swiss map with latitude-longitude locations in D3
Some official Swiss maps from [swisstopo](http://www.swisstopo.admin.ch/internet/swisstopo/en/home.html) use a ~~somewhat exotic~~ coordinate system called [CH1903](http://www.swisstopo.admin.ch/internet/swisstopo/en/home/topics/survey/sys/refsys/switzerland.html), in which [the coordinates are already projected](http://www.swisstopo.admin.ch/internet/swisstopo/en/home/topics/survey/sys/refsys/projections.html): locations in this system are specified as 2D coordinates. Most geographical APIs on the web use a latitude and a longitude to represent the position of a point on the globe, in the standard spherical coordinate system [WGS84](http://en.wikipedia.org/wiki/World_Geodetic_System). Therefore, if you want to integrate geodata from other APIs with official swisstopo maps, you need to project the maps back to spherical coordinates.

This project gives a minimal example of integrating regular latitudes/longitudes locations in WGS84, e.g. obtained from an external geo API, into a Swiss map, in [D3](http://d3js.org).

Demo http://evequoz.name/opendata/valais-map-latlon/ (actually only the canton of Valais and 3 cities defined in regular latitudes-longitudes coordinates)

### Generating the maps in spherical coordinates
First, get [this project by Interactive Things](https://github.com/interactivethings/swiss-maps). It helps you generate Swiss maps in TopoJSON from original [swisstopo](http://www.swisstopo.admin.ch/internet/swisstopo/en/home.html) geodata. By default, it generates already projected maps. Those are fine if you are using nothing else than these maps and you are not trying to embed geographical data from other sources (like Google Geocoding API, Leaflet,...) that use spherical coordinates into your maps.

If you want to combine your Swiss map with external APIs, you will have to reproject it to spherical coordinates. Luckily, you can do this using the above mentioned project by simply running

    make clean
    make topo/vs-municipalities.json REPROJECT=true

(this example generates the municipalities boundaries in the canton of Valais)

### Displaying the maps in D3 and integrating locations in spherical coordinates
The example code you'll find in `index.html` is pretty straightforward. On top of the map generated as described above, three red dots are added at the location of three cities specified in spherical coordinates `[latitude, longitude]`. There are two tricks to understand, though. First, your map is in spherical coordinates, therefore, you need to project them to 2D when you draw them, and make sure the paths from the TopoJSON file containing your map gets projected, too.
```javascript
    var projection = d3.geo.albers()
        .rotate([0, 0])
        .center([8.3, 46.8])   
        .scale(16000)          
        .translate([width / 2, height / 2])
        .precision(.1);
    var path = d3.geo.path()
      .projection(projection);
```

Second, the geolocations of the cities are in spherical coordinates. It is usual to have spherical coordinates as latitude first, then longitude. 

    coords = [latitude, longitude]

When projected, the two coordinates are mapped to the x and y axes in 2D: the latitude (North-South) is actually an ``y`` coordinate, and the longitude (East-West) is an ``x`` coordinate. This is exactly what those two seeminlgy counterintuitive functions do:
```javascript
    var x = function(coords) {
      return projection([coords[1],coords[0]])[0];
    }
    var y = function(coords) {
      return projection([coords[1],coords[0]])[1];
    }
```

Those functions are called later on when the cities are actually drawn on top of the map

```javascript    
    svg.selectAll("circle")
        .data(cities)
        .enter().append("circle")
        .attr("r", 5)
        .attr("cx", function(d){return x(d)})
        .attr("cy", function(d){return y(d)});
```

### License
https://github.com/interactivethings/swiss-maps is under BSD License
