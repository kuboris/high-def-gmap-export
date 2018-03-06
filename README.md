# Printing high-res customizable world map posters with Google Maps and Python

### Fair use Warning:

> This is a personal, non-commercial one-time project for my own educational
> purposes. ‘**You may print Content for non-commercial use and enlarge it’**<br>
via [google policy](http://warning this is a personal, non-commercial one-time
project for my own educational purposes. `you may print content for
non-commercial use and enlarge it (for example, a map with directions).` read
google policy before using
http//www.google.ca/permissions/geoguidelines.html#maps-print)*. *Check it*
*before using it for your own projects.

### Drive

Plan was to get a huge poster map for the wall in our apartment. My girlfriend
was very specific with requirements.

1.  The land part should be really light so we can use markers to write custom
stuff.
1.  For this reason there should be no text on the map.
1.  Only show borders of the countries.
1.  Whole map should be in grey-scale
1.  The map should be approximately 2m wide to fit our wall.

After brief research it was obvious that there are no pre-made maps that fit
this description. It was time to make my own.

### Research

In 2017 Google rolled out Map api v3 that enables custom map styling. Which fits
my use case perfectly.

I recommend trying Google Customizer to test how well it works :
[https://mapstyle.withgoogle.com/](https://mapstyle.withgoogle.com/)

![](https://cdn-images-1.medium.com/max/800/1*CnMkzojICy3eZuDQ3bVQDQ.png)
<span class="figcaption_hack">Screenshot of Google customizer</span>

You can specify your color scheme and amount of labels. It super intuitive and
powerful.

However problem is that for that for the wall you need high resolution which you
are unable to get just by getting screenshot.

> Trust me I tried :) , In case you enlarge your browser canvas to 15000x10000 it
> doesn’t render the whole map properly. But if you need smaller resolution this
screenshot approach works well enough.

### Solution

What seemed to be the simplest solution was to use GoogleApi. Google allows you
to get static images of any part of the world that you define. Plan was to get
Tiles with pieces of map and stitch it together for high resolution map.

So that is what I did, I wrote a short python script and downloaded a bunch of
images through the Api. I read about the Microsoft ICE which should be able to
stitch panoramas and maps together. I inputted 180 images and it got stuck and
then produced really messed up map.

It was time to get smart about it deep dive into map making.

### Python script

See whole code at :
[https://github.com/kuboris/high-def-gmap-export](https://github.com/kuboris/high-def-gmap-export)

Requirements Python 3 + Image.

1.  Import dependencies

    #!/usr/bin/python
    import urllib.request
    from PIL import Image
    import os
    import math

2. Google uses tile system for showing Google maps.(*Tiles are 256x256*)We can
specify how wide and tall our static map is in pixels. However we need to
provide latitude and longitude instead of pixels. That’s why we create two
functions that can transform gps coordinates to pixel coordinates.

    #Google uses this tile size
    #based on 
    tileSize = 256
    initialResolution = 2 * math.pi * 6378137 / tileSize
    originShift = 2 * math.pi * 6378137 / 2.0

    def LatLonToPixels( lat, lon,zoom ):
            "Converts given lat/lon in WGS84 Datum to XY in Spherical Mercator EPSG:900913"
            mx = lon * originShift / 180.0
            my = math.log( math.tan((90 + lat) * math.pi / 360.0 )) / (math.pi / 180.0)
            my = my * originShift / 180.0
            res = initialResolution / (2**zoom)
            
            px = (mx + originShift) / res
            py = (my + originShift) / res
            return px, py
          
    def PixelsToLat( px, py, zoom):
        "Converts pixel coordinates in given zoom level of pyramid to EPSG:900913"
        res = initialResolution / (2**zoom)
        mx = px * res - originShift
        my = py * res - originShift
        lon = (mx / originShift) * 180.0
        lat = (my / originShift) * 180.0
        lat = 180 / math.pi * (2 * math.atan(math.exp(lat * math.pi / 180.0)) - math.pi / 2.0)
        return lat, lon

We need to specify function for cycling through all the coordinates, download
image and stitch it to one high res image.

    generateImage(lat,lng,map_tile_width,map_tile_height,zoom,scale,apikey,format_image,maptype,style,size_tile_x,size_tile_y)

Code is too long to paste here but the short explanation of arguments you need
to generate map.

* Lat,lng=Starting GPS location- starting in lower left corner of the map.
* Map_tile_width = Number of tiles you want to get
* Zoom = Google map zoom 1-Whole earth in one picture, 12 super detailed map
* Scale = 1-4 Api allows to get higher resolution with same details. See Google
map Api documetation
* ApiKey =Key that enables you to query google Api. Get Api key from here :
[https://developers.google.com/places/android-api/](https://developers.google.com/places/android-api/)
* Format_Image = PNG
* Maptype = maptype to return. Google offers `roadmap` `satellite` `hybrid`
`terrain`
* Style = coolest part. Allows you to specify how you want the map to look.
Generate it using
[https://mapstyle.withgoogle.com/](https://mapstyle.withgoogle.com/)
* Size_tile = Google allows to specify size of the returned map. I used 512px.

### Action

We can run the whole script to generate the high resolution map. I’ve added some
improvements that adds another tiles to the right edge of the map to get wider
map.

Example of running script. This will generate high_def simple world map with
resolution around 12000 px wide.

    from PIL import Image
    lat,lng=-80.05112877980659, -180.0
    # World size tiles for zoom 5, 512x512 are 17x15
    map_tile_width,map_tile_height=17,15
    zoom=5
    size_tile_x,size_tile_y=512,512
    # Scale doubles the resolution
    scale=2
    #generate your own
    apikey='*'
    format_image='png'
    maptype='roadmap'
    #simplified style
    style='element:geometry%7Ccolor:0xf5f5f5'
    generateImage(lat,lng,map_tile_width,map_tile_height,zoom,scale,apikey,format_image,maptype,style,size_tile_x,size_tile_y)

By increasing zoom you get higher resolution maps. This is the cropped result.

![](https://cdn-images-1.medium.com/max/800/1*DIgURDRDmn3EXR2SIp3jPQ.png)
<span class="figcaption_hack">Cropped generated output. Original is 12000px wide.</span>

### Epilogue

I have spend more time that I’m willing to admit on this. This script should be
universal enough for anyone to create great looking high def maps.

If you want to do it yourself check the whole code :
[https://github.com/kuboris/high-def-gmap-export](https://github.com/kuboris/high-def-gmap-export)

I’d like to thank user Aragon from[ this stack overflow
answer](https://gis.stackexchange.com/questions/46729/corner-coordinates-of-google-static-map-tile).
I’d not be able to solve translating Gps coordinates to pixels without this.
