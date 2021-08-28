<script>
  import { onMount } from 'svelte'

  const token = process.env.MAPBOX_TOKEN
  mapboxgl.accessToken = token

  const initialZoom = 10
  const styles = [
    'mapbox://styles/mapbox/streets-v11',
    'mapbox://styles/mapbox/satellite-streets-v11',
    'mapbox://styles/mapbox/satellite-v9',
  ]

  onMount(async () => {
    const map0 = new mapboxgl.Map({
      container: 'map0', // container ID
      style: styles[0],
      attributionControl: false,
      center: [174.76, -36.85], // auckland
      zoom: initialZoom,
    })

    const map1 = new mapboxgl.Map({
      container: 'map1', // container ID
      style: styles[0],
      center: [151.2, -33.86], // sydney
      zoom: initialZoom,
    })

    let map0lock = false
    let map1lock = false
    let map0timeout = 0
    let map1timeout = 0

    const TIMEOUT = 200
    const lock = (num) => {
      if (num === 0) {
        clearTimeout(map0timeout)
        map0lock = true
        map0timeout = setTimeout(() => (map0lock = false), TIMEOUT)
      } else if (num === 1) {
        clearTimeout(map1timeout)
        map1lock = true
        map1timeout = setTimeout(() => (map1lock = false), TIMEOUT)
      }
    }

    map0.on('zoom', (e) => {
      if (map0lock === true) {
        return
      } else if (map0lock === -1) {
        if (map0.getZoom() < map1.getZoom()) return
      } else if (map0lock === 1) {
        if (map0.getZoom() > map1.getZoom()) return
        map0lock = false // helps the zoom out
      } else {
        lock(1)
      }
      const zoom = map0.getZoom()
      requestAnimationFrame(() => {
        map1.setZoom(zoom)
      })
    })
    map1.on('zoom', (e) => {
      if (map1lock === true) {
        return
      } else if (map1lock === -1) {
        if (map1.getZoom() < map0.getZoom()) return
      } else if (map1lock === 1) {
        if (map1.getZoom() > map0.getZoom()) return
        map1lock = false // helps the zoom out
      } else {
        lock(0)
      }
      const zoom = map1.getZoom()
      requestAnimationFrame(() => {
        map0.setZoom(zoom)
      })
    })

    map0.on('rotate', () => {
      if (map0lock) return
      lock(1)
      const bearing = map0.getBearing()
      requestAnimationFrame(() => {
        map1.setBearing(bearing)
      })
    })
    map1.on('rotate', () => {
      if (map1lock) return
      lock(0)
      const bearing = map1.getBearing()
      requestAnimationFrame(() => {
        map0.setBearing(bearing)
      })
    })

    map0.on('pitch', () => {
      if (map0lock) return
      lock(1)
      const pitch = map0.getPitch()
      requestAnimationFrame(() => {
        map1.setPitch(pitch)
      })
    })
    map1.on('pitch', () => {
      if (map1lock) return
      lock(0)
      const pitch = map1.getPitch()
      requestAnimationFrame(() => {
        map0.setPitch(pitch)
      })
    })

    const scale = new mapboxgl.ScaleControl({
      maxWidth: 80,
      unit: 'metric',
    })
    map0.addControl(scale)

    const geocodeTypes =
      'country, region, postcode, district, place, locality, neighborhood'
    const map0geocoder = new MapboxGeocoder({
      accessToken: mapboxgl.accessToken,
      mapboxgl: mapboxgl,
      types: geocodeTypes,
      marker: false,
    })
    const map1geocoder = new MapboxGeocoder({
      accessToken: mapboxgl.accessToken,
      mapboxgl: mapboxgl,
      types: geocodeTypes,
      marker: false,
    })
    map0.addControl(map0geocoder, 'top-left')
    map1.addControl(map1geocoder, 'top-left')

    const geolocate = new mapboxgl.GeolocateControl({
      showUserLocation: false,
    })
    map1.addControl(geolocate, 'bottom-right')

    const nav = new mapboxgl.NavigationControl()
    map1.addControl(nav, 'bottom-right')

    let map0flying = false
    let map1flying = false
    map0geocoder.on('result', (e) => {
      // if the event is greater than the bounds, we're zooming out
      // otherwise zooming in
      const bounds = map1.getBounds()
      const difference =
        e.result.bbox[2] - e.result.bbox[0] > bounds._ne.lng - bounds._sw.lng
      map0lock = difference ? 1 : -1
      map1lock = true
      map0flying = true
    })
    map1geocoder.on('result', (e) => {
      // if the event is greater than the bounds, we're zooming out
      // otherwise zooming in
      const bounds = map0.getBounds()
      const difference =
        e.result.bbox[2] - e.result.bbox[0] > bounds._ne.lng - bounds._sw.lng
      map0lock = true
      map1lock = difference ? 1 : -1
      map1flying = true
    })

    map0.on('moveend', () => {
      if (map0flying) {
        map0lock = false
        map1lock = false
        map0flying = false
      }
    })
    map1.on('moveend', () => {
      if (map1flying) {
        map0lock = false
        map1lock = false
        map1flying = false
      }
    })
  })
</script>

<main>
  <div id="map0" class="map" />
  <div id="map1" class="map" />
</main>

<style>
  .map {
    flex: 1;
  }

  main {
    display: flex;
    height: 100vh;
    column-gap: 1px;
    row-gap: 1px;
    flex-direction: column;
  }

  @media (min-aspect-ratio: 1/1) {
    main {
      flex-direction: row;
    }
  }
</style>
