# sf_smoke
Notes for creating an animated gif of California wildfires

More info in this Medium post: [The fire this time, from San Francisco](https://hi.stamen.com/the-fire-this-time-from-san-francisco-95de4e6553a2)

The end result of following these steps will be an animated gif like this one: [https://twitter.com/stamen/status/919008091017854976](https://twitter.com/stamen/status/919008091017854976)

![Animated gif of smoke from California wildfires](sf_smoke.gif?raw=true)

Downloading files:

```
  for k in $(seq 0 9); do
    for j in $(seq 0 2); do
      for i in $(seq 0 9); do
	      curl -f -O https://haze.airfire.org/bluesky-daily/output/standard/CANSAC-2km/2017101300/forecast/images/three_hour/1RedColorBar/three_hour_2017101${k}${j}${i}00.png;
      done;
    done;
  done
```

This will make a gif:

```
convert -delay 1 -loop 0 *.png sf_smoke.gif
```

But if we want to project them:
(use coordinates found here: https://haze.airfire.org/bluesky-daily/output/standard/CANSAC-2km/2017101000/forecast/grid_info.json)

```
  for k in $(seq 0 9); do
    for j in $(seq 0 2); do
      for i in $(seq 0 9); do
	      gdal_translate -of GTiff -a_srs EPSG:4326 -a_ullr -125.5 42.49999975413084 -112.50000029057264 31.5 three_hour_2017101${k}${j}${i}00.png three_hour_2017101${k}${j}${i}00.tiff
      done;
    done;
  done
```

Then I opened up a tilemill project (download & install tilemill from [the tilemill project](https://github.com/tilemill-project/tilemill)) and projected them all to EPSG:3310, and added some state boundaries (natural earth, from [their download site](https://www.naturalearthdata.com/http//www.naturalearthdata.com/download/10m/cultural/ne_10m_admin_1_states_provinces_lakes.zip) or from [their github](https://github.com/nvkelso/natural-earth-vector/)).

Edit the tilemill template [project.mml.template](https://github.com/stamen/sf_smoke/blob/master/project.mml.template) as needed for the paths on your operating system)

Now to do the scripted tilemill, which uses sed to edit the tilemill project template and replace the raster images for each frame:

```
  rm output*.png;
  for k in $(seq 0 9); do
    for j in $(seq 0 2); do
      for i in $(seq 0 9); do
	      test -e three_hour_2017101${k}${j}${i}00.tiff &&
	      cp project.mml.template ~/Documents/MapBox/project/sf_smoke/project.mml &&
	      sed -i -e s%REPLACESTRING%2017101${k}${j}${i}00% ~/Documents/MapBox/project/sf_smoke/project.mml &&
	      node ~/github/tilemill/index.js export sf_smoke --format=png --width=600 --height=600 output_2017101${k}${j}${i}00.png;
      done;
    done;
  done
  convert -delay 1 -loop 0 output*.png sf_smoke.gif
```
