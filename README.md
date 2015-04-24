# Swiss map with latitude-longitude locations
Official Swiss maps from [swisstopo](http://www.swisstopo.admin.ch/internet/swisstopo/en/home.html) use an exotic coordinate system called [CH1903](http://www.swisstopo.admin.ch/internet/swisstopo/en/home/topics/survey/sys/refsys/switzerland.html), in which [the coordinates are already projected](http://www.swisstopo.admin.ch/internet/swisstopo/en/home/topics/survey/sys/refsys/projections.html). To integrate geodata from other APIs that use spherical coordinates (a latitude and a longitude), you need to reproject the maps.

This project gives a minimal example of integrating regular latitudes/longitudes locations, e.g. obtained from an external geo API, into a Swiss map in [D3](http://d3js.org) (actually only the canton of Valais and 3 cities defined in regular latitudes-longitudes coordinates).

Demo http://evequoz.name/opendata/valais-map-latlon/ 

### Generating the maps in spherical coordinates
First, get [this project by Interactive Things](https://github.com/interactivethings/swiss-maps). It helps you generate Swiss maps from original [swisstopo](http://www.swisstopo.admin.ch/internet/swisstopo/en/home.html) geodata. By default, it generates already projected maps. Those are fine if you are using nothing else than these maps and you are not trying to embed geographical data from other sources (like Google Geocoding API, Leaflet,...) that use spherical coordinates (a latitude and a longitude) into your maps.

If you want to combine your Swiss map with external APIs, you will have to reproject it in spherical coordinates. Luckily, you can do this using the above mentioned project by simply running

    make clean
    make topo/ch-cantons.json REPROJECT=true

(this example generates the cantons boundaries)

### Displaying the maps in D3 and integrating locations in spherical coordinates
The example code you'll find in index.html is pretty straightforward. On top of the map generated as described above, three red dots are added at the location of three cities specified in spherical coordinates [latitude, longitude]. There are two tricks to understand, though. First, your map is in spherical coordinates, therefore, you need to project them to 2D and make sure the paths from the TopoJSON file containing your map gets projected, too.

    var projection = d3.geo.albers()
        .rotate([0, 0])
        .center([8.3, 46.8])   
        .scale(16000)          
        .translate([width / 2, height / 2])
        .precision(.1);
    var path = d3.geo.path()
      .projection(projection);

Second, the geolocation of the cities is in spherical coordinates. It is usual to have spherical coordinates as latitude first, then longitude. 

    coords = [latitude, longitude]

When projected, the two coordinates must be inverted on the x and y axes: the latitude (North-South) is actually an y coordinate, and the longitude (East-West) is an x coordinate. Therefore those two seeminlgy odd functions in the code

    var x = function(coords) {
      return projection([coords[1],coords[0]])[0];
    }
    var y = function(coords) {
      return projection([coords[1],coords[0]])[1];
    }

Those functions are called later on when the cities are actually drawn on top of the map
    
    svg.selectAll("circle")
        .data(cities)
        .enter().append("circle")
        .attr("r", 5)
        .attr("cx", function(d){return x(d)})
        .attr("cy", function(d){return y(d)});


### License
https://github.com/interactivethings/swiss-maps is under BSD License
