<script>
  import { onMount } from 'svelte'

  const token = process.env.MAPBOX_TOKEN
  mapboxgl.accessToken = token

  const initialZoom = 10
  const styles = [
    'mapbox://styles/mapbox/streets-v11',
    'mapbox://styles/mapbox/satellite-streets-v11',
    'mapbox://styles/mapbox/satellite-v9'
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
      center: [151.20, -33.86], // sydney
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
        map0timeout = setTimeout(() => map0lock = false, TIMEOUT)
      } else if (num === 1) {
        clearTimeout(map1timeout)
        map1lock = true
        map1timeout = setTimeout(() => map1lock = false, TIMEOUT)
      }
    }

    map0.on('zoom', () => {
      if (map0lock) return
      lock(1)
      const zoom = map0.getZoom()
      requestAnimationFrame(() => {
        map1.setZoom(zoom)  
      })
    }) 
    map1.on('zoom', () => {
      if (map1lock) return
      lock(0)
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
      unit: 'metric'
    })
    map0.addControl(scale)

})
</script>

<main>
  <div id="map0" class="map"></div>
  <div id="map1" class="map"></div>
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

  @media (min-width: 600px) {
    main {
      flex-direction: row;
    }
  }
</style>
