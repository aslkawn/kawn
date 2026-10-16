# External URLs used by the studied app (hosts + sample endpoints)

Compiled 2026-10-16 by scanning client code, server code, HTML and bundled public files for http(s) addresses, then dropping test fixtures, placeholders and local hosts. Dev/build-only hosts are listed separately. This is a **static scan**: addresses assembled at run time from settings (e.g. public map-feature query servers, operator-configured upstreams) do not appear. Detail on licences, keys and caching: doc 10; privacy and outbound: doc 11. Census pointers: geocoder chain and Google tile token in 13a section 1; flight, vessel and orbit sources in 13b/13c; ground data sources in 13d; server, benchmark and build in 13f. The censuses name providers but not host names, so the additions below were confirmed against the original source.

Legend: the sample column shows up to 3 shortest endpoints found in production code (query strings trimmed). Count = number of distinct addresses found for that host.


## Basemaps, terrain, 3D tiles

| Host | Count | Sample endpoints |
|---|---|---|
| api.cesium.com | 2 | https://api.cesium.com<br>https://api.cesium.com/v1/assets/{x}/endpoint |
| assets.cesium.com | 1 | https://assets.cesium.com |
| assets.ion.cesium.com | 1 | https://assets.ion.cesium.com |
| dev.virtualearth.net | 1 | https://dev.virtualearth.net |
| ecn.t0.tiles.virtualearth.net | 1 | https://ecn.t0.tiles.virtualearth.net |
| ecn.t1.tiles.virtualearth.net | 1 | https://ecn.t1.tiles.virtualearth.net |
| ecn.t2.tiles.virtualearth.net | 1 | https://ecn.t2.tiles.virtualearth.net |
| ecn.t3.tiles.virtualearth.net | 1 | https://ecn.t3.tiles.virtualearth.net |
| gibs.earthdata.nasa.gov | 1 | https://gibs.earthdata.nasa.gov |
| ion.cesium.com | 1 | https://ion.cesium.com/tokens |
| maps.googleapis.com | 5 | https://maps.googleapis.com<br>https://maps.googleapis.com/maps/api/streetview<br>https://maps.googleapis.com/maps/api/streetview/metadata<br>https://maps.googleapis.com/maps/api/geocode/json (geocoder chain step 4)<br>https://maps.googleapis.com/maps/api/js (maps script host named in the content-security policy) |
| maps.gstatic.com | 1 | https://maps.gstatic.com |
| openfreemap.org | 1 | https://openfreemap.org |
| openmaptiles.org | 1 | https://openmaptiles.org |
| overturemaps.org | 1 | https://overturemaps.org |
| registry.opendata.aws | 1 | https://registry.opendata.aws/noaa-gfs-bdp-pds/ |
| server.arcgisonline.com | 1 | https://server.arcgisonline.com |
| services.arcgisonline.com | 2 | https://services.arcgisonline.com<br>https://services.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer |
| terrain.reearth.land | 4 | https://terrain.reearth.land<br>https://terrain.reearth.land/heights.json<br>https://terrain.reearth.land/cesium-mesh/ellipsoid |
| tile.googleapis.com | 5 | https://tile.googleapis.com<br>https://tile.googleapis.com/v1/createSession<br>https://tile.googleapis.com/v1/3dtiles/root.json (session and tile requests; the short-lived access token from the local server is used here, retried once on 401/403) |
| tile.openstreetmap.org | 2 | https://tile.openstreetmap.org<br>https://tile.openstreetmap.org/ |
| tiles.openfreemap.org | 2 | https://tiles.openfreemap.org<br>https://tiles.openfreemap.org/planet |
| vantor-opendata.s3.amazonaws.com | 2 | https://vantor-opendata.s3.amazonaws.com/events/Nepal-Flooding-Aug-2026/10300100C86CED00.tif<br>https://vantor-opendata.s3.amazonaws.com/events/Nepal-Flooding-Aug-2026/B040001100881410.tif |
| vantor.com | 1 | https://vantor.com/company/open-data-program |
| wvs.earthdata.nasa.gov | 1 | https://wvs.earthdata.nasa.gov/api/v1/snapshot |
| www.naturalearthdata.com | 1 | https://www.naturalearthdata.com |

## Search / geocoding / routing

| Host | Count | Sample endpoints |
|---|---|---|
| nominatim.openstreetmap.org | 1 | https://nominatim.openstreetmap.org/search |
| photon.komoot.io | 2 | https://photon.komoot.io<br>https://photon.komoot.io/api/ |
| places.googleapis.com | 2 | https://places.googleapis.com/v1/places:searchText<br>https://places.googleapis.com/v1/places:searchNearby |
| routing.openstreetmap.de | 2 | https://routing.openstreetmap.de/about.html<br>https://routing.openstreetmap.de/routed-{x} |
| wiki.openstreetmap.org | 1 | https://wiki.openstreetmap.org/wiki/Slippy_map_tilenames |
| www.census.gov | 1 | https://www.census.gov/geographies/mapping-files/time-series/geo/cartographic-boundary.html |
| www.openstreetmap.org | 2 | https://www.openstreetmap.org/copyright<br>https://www.openstreetmap.org/fixthemap |

## Flights, vessels, satellites, space

| Host | Count | Sample endpoints |
|---|---|---|
| adsb.lol | 2 | https://adsb.lol<br>https://adsb.lol/data/traces/{x}/trace_full_{x}.json |
| aisstream.io | 1 | https://aisstream.io |
| api.adsb.lol | 3 | https://api.adsb.lol/v2/mil<br>https://api.adsb.lol/v2/lat/{x}/lon/{x}/dist/{x}<br>https://api.adsb.lol/v2/lat/{x}/lon/{x}/dist/{x} |
| api.adsbdb.com | 2 | https://api.adsbdb.com/v0/aircraft/{x}<br>https://api.adsbdb.com/v0/callsign/{x} |
| auth.opensky-network.org | 1 | https://auth.opensky-network.org/auth/realms/opensky-network/protocol/openid-connect/token |
| celestrak.org | 3 | https://celestrak.org<br>https://celestrak.org/NORAD/elements/gp.php<br>https://celestrak.org/NORAD/elements/gp.php |
| gpsjam.org | 1 | https://gpsjam.org/faq (documentation link only; the site is never called, only its public scoring method is borrowed) |
| ll.thespacedevs.com | 2 | https://ll.thespacedevs.com/docs/<br>https://ll.thespacedevs.com/2.3.0/launches/ |
| opensky-network.org | 3 | https://opensky-network.org<br>https://opensky-network.org/api/states/all<br>https://opensky-network.org/api/tracks/all |
| thespacedevs.com | 1 | https://thespacedevs.com |
| www.adsbdb.com | 1 | https://www.adsbdb.com |

## Weather, hazards, imagery

| Host | Count | Sample endpoints |
|---|---|---|
| api.gdeltproject.org | 1 | https://api.gdeltproject.org/api/v2/doc/doc |
| api.open-meteo.com | 1 | https://api.open-meteo.com/v1/forecast |
| cmr.earthdata.nasa.gov | 1 | https://cmr.earthdata.nasa.gov/search |
| data-nifc.opendata.arcgis.com | 1 | https://data-nifc.opendata.arcgis.com/ |
| data.ecmwf.int | 1 | https://data.ecmwf.int/forecasts |
| earthdata.nasa.gov | 1 | https://earthdata.nasa.gov/firms |
| earthquake.usgs.gov | 2 | https://earthquake.usgs.gov/earthquakes/eventpage/us7000tbwb/executive<br>https://earthquake.usgs.gov/earthquakes/feed/v1.0/summary/all_day.geojson |
| firms.modaps.eosdis.nasa.gov | 5 | https://firms.modaps.eosdis.nasa.gov/api/map_key/<br>https://firms.modaps.eosdis.nasa.gov/api/area/csv/{KEY}/{SOURCE}/world/2<br>https://firms.modaps.eosdis.nasa.gov/api/area/csv/{KEY}/{SOURCE}/world/{days} |
| inciweb.wildfire.gov | 4 | https://inciweb.wildfire.gov/<br>https://inciweb.wildfire.gov/node/{x}<br>https://inciweb.wildfire.gov/api/single-publication/ |
| lpdaac.usgs.gov | 1 | https://lpdaac.usgs.gov/products/hlss30v002/ |
| mapservices.weather.noaa.gov | 1 | https://mapservices.weather.noaa.gov/tropical/rest/services/tropical/NHC_tropical_weather_summary/MapServer |
| nowcoast.noaa.gov | 2 | https://nowcoast.noaa.gov/<br>https://nowcoast.noaa.gov/geoserver/observations/ |
| ocean.weather.gov | 1 | https://ocean.weather.gov/lightning/lightning_pdd.php |
| oceanservice.noaa.gov | 1 | https://oceanservice.noaa.gov/disclaimer.html |
| open-meteo.com | 2 | https://open-meteo.com/<br>https://open-meteo.com/en/licence |
| www.ecmwf.int | 1 | https://www.ecmwf.int/en/forecasts/datasets/open-data |
| www.gdeltproject.org | 1 | https://www.gdeltproject.org/about.html |
| www.nhc.noaa.gov | 6 | https://www.nhc.noaa.gov<br>https://www.nhc.noaa.gov/<br>https://www.nhc.noaa.gov/CurrentStorms.json |

## Transit and bikeshare

| Host | Count | Sample endpoints |
|---|---|---|
| api.entur.io | 1 | https://api.entur.io/realtime/v1/gtfs-rt/vehicle-positions |
| api.tfl.gov.uk | 1 | https://api.tfl.gov.uk/Place/Type/JamCam |
| austin.publicbikesystem.net | 2 | https://austin.publicbikesystem.net/customer/gbfs/v2/en/station_status.json<br>https://austin.publicbikesystem.net/customer/gbfs/v2/en/station_information.json |
| cdn.mbta.com | 2 | https://cdn.mbta.com/realtime/VehiclePositions.pb<br>https://cdn.mbta.com/sites/default/files/2023-08/mbta-massdot-develop-license-agreement.pdf |
| chat.publicbikesystem.net | 2 | https://chat.publicbikesystem.net/customer/gbfs/v2/en/station_status.json<br>https://chat.publicbikesystem.net/customer/gbfs/v2/en/station_information.json |
| developer.entur.org | 1 | https://developer.entur.org/pages-intro-authentication |
| gbfs.bcycle.com | 6 | https://gbfs.bcycle.com/{x}/station_status.json<br>https://gbfs.bcycle.com/bcycle_indego/station_status.json<br>https://gbfs.bcycle.com/bcycle_lametro/station_status.json |
| gbfs.biketownpdx.com | 2 | https://gbfs.biketownpdx.com/gbfs/2.3/en/station_status.json<br>https://gbfs.biketownpdx.com/gbfs/2.3/en/station_information.json |
| gbfs.bluebikes.com | 2 | https://gbfs.bluebikes.com/gbfs/en/station_status.json<br>https://gbfs.bluebikes.com/gbfs/en/station_information.json |
| gbfs.cogobikeshare.com | 2 | https://gbfs.cogobikeshare.com/gbfs/2.3/en/station_status.json<br>https://gbfs.cogobikeshare.com/gbfs/2.3/en/station_information.json |
| gbfs.lyft.com | 8 | https://gbfs.lyft.com/gbfs/2.3/bay/en/station_status.json<br>https://gbfs.lyft.com/gbfs/2.3/bkn/en/station_status.json<br>https://gbfs.lyft.com/gbfs/2.3/chi/en/station_status.json |
| gtfs.ovapi.nl | 2 | https://gtfs.ovapi.nl/README<br>https://gtfs.ovapi.nl/nl/vehiclePositions.pb |
| gtfsrt.api.translink.com.au | 1 | https://gtfsrt.api.translink.com.au/api/realtime/seq/VehiclePositions |
| hon.publicbikesystem.net | 2 | https://hon.publicbikesystem.net/customer/gbfs/v2/en/station_status.json<br>https://hon.publicbikesystem.net/customer/gbfs/v2/en/station_information.json |
| realtime.hsl.fi | 1 | https://realtime.hsl.fi/realtime/vehicle-positions/v2/hsl |
| svc.metrotransit.org | 2 | https://svc.metrotransit.org/<br>https://svc.metrotransit.org/mtgtfs/vehiclepositions.pb |
| tfl.gov.uk | 1 | https://tfl.gov.uk/info-for/open-data-users/ |
| translink.com.au | 1 | https://translink.com.au/about-translink/open-data |
| www.capmetro.org | 1 | https://www.capmetro.org/developertools |
| www.hsl.fi | 1 | https://www.hsl.fi/en/hsl/open-data |

## Traffic and public cameras

| Host | Count | Sample endpoints |
|---|---|---|
| 511on.ca | 3 | https://511on.ca/<br>https://511on.ca/map/Cctv/<br>https://511on.ca/api/v2/get/cameras |
| api.tomtom.com | 2 | https://api.tomtom.com/traffic/map/4/tile/flow/relative/<br>https://api.tomtom.com/traffic/map/4/tile/flow/relative/{z}/{x}/{y}.pbf |
| cctv.austinmobility.io | 1 | https://cctv.austinmobility.io/image/{x}.jpg |
| cwwp2.dot.ca.gov | 2 | https://cwwp2.dot.ca.gov/<br>https://cwwp2.dot.ca.gov/data/d{x}/cctv/cctvStatusD${String(district).padStart(2 |
| data.austintexas.gov | 2 | https://data.austintexas.gov<br>https://data.austintexas.gov/api/views/b4k4-adkb/rows.json |
| data.calgary.ca | 2 | https://data.calgary.ca/resource/k7p9-kppz.json<br>https://data.calgary.ca/stories/s/Open-Calgary-Terms-of-Use/u45n-7awa |
| data.livetraffic.com | 1 | https://data.livetraffic.com/cameras/traffic-cam.json |
| data.norge.no | 1 | https://data.norge.no/nlod/en/2.0 |
| data.texas.gov | 1 | https://data.texas.gov/download/eiei-9rpf/application%2Foctet-stream |
| deldot.gov | 1 | https://deldot.gov/map/ |
| docs.tomtom.com | 1 | https://docs.tomtom.com/pricing/ |
| its.txdot.gov | 2 | https://its.txdot.gov<br>https://its.txdot.gov/ |
| kamera.atlas.vegvesen.no | 1 | https://kamera.atlas.vegvesen.no/api/images/ |
| kamera.vegvesen.no | 1 | https://kamera.vegvesen.no/public/{x}/manifest.m3u8 |
| my.tomtom.com | 1 | https://my.tomtom.com/keys |
| ogckart-sn1.atlas.vegvesen.no | 1 | https://ogckart-sn1.atlas.vegvesen.no/ogc/features/v1/collections/datex_3_1:CctvSimple/items |
| ristmikud.tallinn.ee | 1 | https://ristmikud.tallinn.ee/ |
| tarktee.transpordiamet.ee | 4 | https://tarktee.transpordiamet.ee/<br>https://tarktee.transpordiamet.ee/images/<br>https://tarktee.transpordiamet.ee/api/v1/datex/roadCameraImages |
| tie.digitraffic.fi | 1 | https://tie.digitraffic.fi/api/weathercam/v1/stations |
| tmc.deldot.gov | 1 | https://tmc.deldot.gov/json/videocamera.json |
| trafficcam.calgary.ca | 1 | https://trafficcam.calgary.ca/ |
| video.deldot.gov | 1 | https://video.deldot.gov |
| weathercam.digitraffic.fi | 1 | https://weathercam.digitraffic.fi/ |
| webcam.warendorf.de | 1 | http://webcam.warendorf.de/ |
| webcams.nyctmc.org | 3 | https://webcams.nyctmc.org/<br>https://webcams.nyctmc.org/api/cameras<br>https://webcams.nyctmc.org/api/cameras/ |
| webcams.transport.nsw.gov.au | 1 | https://webcams.transport.nsw.gov.au/ |
| www.digitraffic.fi | 1 | https://www.digitraffic.fi/en/ |
| www.drivebc.ca | 3 | https://www.drivebc.ca/<br>https://www.drivebc.ca/api/webcams/<br>https://www.drivebc.ca/images/{x}.jpg |
| www.kreis-warendorf.de | 1 | https://www.kreis-warendorf.de/ |
| www.livetraffic.com | 1 | https://www.livetraffic.com/ |
| www.ontario.ca | 1 | https://www.ontario.ca/page/open-government-licence-ontario |
| www.tomtom.com | 1 | https://www.tomtom.com |
| www.warendorf.de | 1 | https://www.warendorf.de/ |
| www2.gov.bc.ca | 1 | https://www2.gov.bc.ca/gov/content/data/open-data/open-government-licence-bc |

## Street-level imagery

| Host | Count | Sample endpoints |
|---|---|---|
| api.kartaview.org | 1 | https://api.kartaview.org |
| cdn.kartaview.org | 1 | https://cdn.kartaview.org |
| connect.facebook.net | 1 | https://connect.facebook.net/en_US/sdk.js#xfbml=0&version=v24.0 |
| graph.mapillary.com | 1 | https://graph.mapillary.com |
| kartaview.org | 1 | https://kartaview.org |
| streetviewpixels-pa.googleapis.com | 2 | https://streetviewpixels-pa.googleapis.com<br>https://streetviewpixels-pa.googleapis.com/v1/tile |
| tiles.mapillary.com | 1 | https://tiles.mapillary.com/maps/vtp |
| www.facebook.com | 3 | https://www.facebook.com/plugins/video.php<br>https://www.facebook.com/reel/1571491657757829<br>https://www.facebook.com/watch/ |
| www.facebook.com'] | 1 | https://www.facebook.com'] |
| www.mapillary.com | 4 | https://www.mapillary.com<br>https://www.mapillary.com/app/<br>https://www.mapillary.com/dashboard/developers |

## Radio / media / news

| Host | Count | Sample endpoints |
|---|---|---|
| all.api.radio-browser.info | 1 | https://all.api.radio-browser.info/json/servers |
| de1.api.radio-browser.info | 1 | https://de1.api.radio-browser.info |
| de2.api.radio-browser.info | 1 | https://de2.api.radio-browser.info |
| i.ytimg.com | 6 | https://i.ytimg.com<br>https://i.ytimg.com/vi/DbqRexFxv3k/hqdefault.jpg<br>https://i.ytimg.com/vi/FE5HRRFhDQg/hqdefault.jpg |
| news.google.com | 1 | https://news.google.com/rss/search |
| nl1.api.radio-browser.info | 1 | https://nl1.api.radio-browser.info |
| platform.twitter.com | 1 | https://platform.twitter.com/widgets.js |
| www.radio-browser.info | 1 | https://www.radio-browser.info/ |
| www.youtube-nocookie.com | 2 | https://www.youtube-nocookie.com<br>https://www.youtube-nocookie.com/embed/{x} |
| www.youtube.com | 17 | https://www.youtube.com<br>https://www.youtube.com/iframe_api<br>https://www.youtube.com/watch |
| x.com | 5 | https://x.com/BNODesk/status/2092568283098575057<br>https://x.com/Emeka2026/status/2093196357242662947<br>https://x.com/Indowatchosint/status/2092640554568007958 |

## AI / voice

| Host | Count | Sample endpoints |
|---|---|---|
| api.openai.com | 3 | https://api.openai.com/v1/responses<br>https://api.openai.com/v1/realtime/calls<br>https://api.openai.com/v1/realtime/client_secrets |
| developers.openai.com | 3 | https://developers.openai.com/api/docs/pricing<br>https://developers.openai.com/api/docs/models/gpt-realtime-2<br>https://developers.openai.com/api/docs/models/gpt-realtime-2.1-mini |
| platform.openai.com | 1 | https://platform.openai.com/api-keys |
| generativelanguage.googleapis.com | 1 | wss host of the Gemini Live bidirectional generate-content service (developer voice benchmark only, not the running app) |
| ai.google.dev | 1 | https://ai.google.dev/gemini-api/docs/pricing (benchmark rate table source; also listed under dev-only below) |

## Infrastructure datasets

| Host | Count | Sample endpoints |
|---|---|---|
| sketchfab.com | 18 | https://sketchfab.com/oyan3D<br>https://sketchfab.com/e0057537<br>https://sketchfab.com/IProZenoN |
| tiles.dontgetflocked.com | 2 | https://tiles.dontgetflocked.com<br>https://tiles.dontgetflocked.com/cameras-{x}-hourly.json |
| www.submarinecablemap.com | 1 | https://www.submarinecablemap.com |

## Fonts / docs / licences / misc

| Host | Count | Sample endpoints |
|---|---|---|
| creativecommons.org | 2 | https://creativecommons.org/licenses/by/4.0/<br>https://creativecommons.org/licenses/by-nc/4.0/ |
| developers.google.com | 2 | https://developers.google.com/maps/documentation/tile/get-api-key<br>https://developers.google.com/maps/documentation/places/web-service/get-api-key |
| fonts.googleapis.com | 3 | https://fonts.googleapis.com<br>https://fonts.googleapis.com/css2<br>https://fonts.googleapis.com/css2 |
| fonts.gstatic.com | 1 | https://fonts.gstatic.com |
| opendatacommons.org | 1 | https://opendatacommons.org/licenses/odbl/1-0/ |
| policies.google.com | 1 | https://policies.google.com/terms |
| www.esri.com | 1 | https://www.esri.com |
| www.google.com | 2 | https://www.google.com/streetview/<br>https://www.google.com/maps/@ |
| www.opengis.net | 1 | http://www.opengis.net/kml/2.2 |

## Other

| Host | Count | Sample endpoints |
|---|---|---|
| localhost').pathname | 1 | http://localhost').pathname |
| localhost').searchparams | 1 | http://localhost').searchParams |
| s3-eu-west-1.amazonaws.com | 1 | https://s3-eu-west-1.amazonaws.com/jamcams.tfl.gov.uk/ |
| services3.arcgis.com | 1 | https://services3.arcgis.com/T4QMspbfLg3qTGWY/arcgis/rest/services/ |

## Dev, build and tooling only (not part of the running app)

ai.google.dev, data.sfgov.org, overturemaps-us-west-2.s3.us-west-2.amazonaws.com, raw.githubusercontent.com, www.topografix.com, www2.census.gov

Note: storage.googleapis.com is not build-only. A public Google sample-video bucket (generic sample files, two addresses of the form https://storage.googleapis.com/gtv-videos-bucket/sample/...mp4) is used by the demo-only Tokyo (Shinjuku) camera pack, licensed only as a projection-pipeline test and not in the default catalogue (doc 10 section 2.6).


## Not visible to a static scan (known from docs 10/11)
- Public map-feature query servers (installations, traffic roads, plate-reader extracts): configured per deployment because the shared public servers refuse the app; the app needs a self-hosted or commercial endpoint.
- Hosts for ADS-B receivers, SDR feeds and user-entered stream addresses: supplied by the user at run time.
- Regional camera feeds: many camera image/stream hosts come from bundled camera lists, not from code strings.
- Operator-configured upstreams for map features: set in environment, read server-side.
- A local model runtime and any OpenAI-compatible chat endpoint used by the developer voice benchmark: supplied by the developer at run time (loopback or vendor address), not part of the product.
- Operator-supplied camera packs (local JSON file or inline JSON): their image and stream hosts are whatever the operator enters.
- Panoramax, a planned fourth street-level provider, has no calls in the studied app; a custom provider id slot is reserved only.
- The built-in camera seed list (eight cities including Dubai) uses no outbound host of its own.
- Photon biases only by a point and the Nominatim proxy is reached through the local server, so neither exposes a browser-visible Nominatim host other than the one listed above.

## Kawn notes
- Use this list as a **source-candidate checklist**, not a to-copy list: each source needs its own licence check (commercial use, redistribution, attribution) before Kawn uses it.
- UAE-relevant hosts that apply directly: OpenSky/adsb.lol (flights), AISstream (ships), CelesTrak (satellites), Open-Meteo and ECMWF (weather), NASA FIRMS (heat spots), USGS (quakes), NASA GIBS (cloud imagery), Esri/OSM/OpenFreeMap/Google (basemaps), Photon/Nominatim/Google Places (search), Mapillary/KartaView/Google (street level), radio-browser (radio), OpenAI (voice).
- Section 3 of doc 10 now carries the benchmark vendors (Gemini Live, local runtime) and the harmonised Landsat/Sentinel imagery path as source rows; none is a UAE production dependency.
- Gulf-specific additions to plan for (not in the studied app): a Gulf-aware, Arabic-capable geocoder to sit ahead of Photon and Nominatim; self-hosted tiles and routing; AIS and ADS-B sources licensed for commercial UAE use; owner decisions (Dubai default view, Scope feather 100, location tray) do not add hosts.
- Everything US/EU-specific (transit agencies, DOT cameras, NHC cyclones, NIFC fires, bikeshare operators) has no UAE equivalent; Kawn needs UAE sources: RTA open data/GTFS, Dubai Pulse, DLD open data, Dubai Municipality, NCM weather, Abu Dhabi Open Data.

## Revision log
- Pass 3 (census merge): added the Google 3D tile session and street-view metadata endpoints and the maps script host; marked the GPSJAM address as documentation only; added the Gemini Live benchmark host and pricing page; corrected the storage.googleapis.com classification (demo camera pack bucket, not build-only); extended the not-visible-to-a-scan list (benchmark runtimes, operator camera packs, planned Panoramax, seed list); added Kawn planning notes. Confidence: hosts confirmed against the original source strings; no network capture.
