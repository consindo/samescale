<script>
  import MapView from './Map.svelte'

  let loadMap = false
  let initialLocation = []
  const triggerClick = (coords) => () => {
    initialLocation = coords
    loadMap = true
  }
</script>

{#if loadMap}
  <MapView {initialLocation} />
{/if}

<div
  class="container"
  style="--opacity: {loadMap ? 0 : 1}; --pointer-events: {loadMap
    ? 'none'
    : 'auto'}"
>
  <h1>Same Scale</h1>
  <p>
    Compare two maps with the same scale <br />and see the difference in our
    built environments
  </p>
  <nav>
    <ul>
      <li
        class="akl-syd"
        on:click={triggerClick([
          [174.76, -36.85],
          [151.2, -33.86],
        ])}
      >
        Auckland &middot; Sydney
      </li>
      <li
        class="nyc-hkg"
        on:click={triggerClick([
          [-74, 40.71],
          [114.16, 22.32],
        ])}
      >
        New York City &middot; Hong Kong
      </li>
      <li
        class="cbr-wdc"
        on:click={triggerClick([
          [149.13, -35.28],
          [-77.04, 38.91],
        ])}
      >
        Canberra &middot; Washington, D.C.
      </li>
    </ul>
  </nav>
  <aside>
    <p>
      Note: This visualization uses the web mercator projection, so it's not
      great for larger areas!
    </p>
  </aside>
  <footer>
    <a href="https://jono.nz">{new Date().getFullYear()} &copy; Jono Cooper</a>
  </footer>
</div>

<style>
  .container {
    text-align: center;
    position: absolute;
    top: 0;
    left: 0;
    height: 100%;
    height: -webkit-fill-available;
    overflow-y: auto;
    width: 100%;
    z-index: 25;
    background-color: #fafafa;
    background-image: url(/bg.png);
    background-size: 100px;
    transition: 500ms ease opacity;
    opacity: var(--opacity);
    pointer-events: var(--pointer-events);
  }
  @media screen and (min-device-pixel-ratio: 1.5) {
    .container {
      background-size: 50px;
    }
  }
  h1 {
    font-size: 3rem;
    letter-spacing: -1px;
    text-shadow: 0 1px 1px #fff;
  }
  p {
    font-size: 1.25rem;
    max-width: 425px;
    padding: 0 15px;
    margin: 0 auto;
    letter-spacing: -0.5px;
    text-shadow: 0 1px 1px #fff;
  }

  aside p {
    font-size: 0.9rem;
    max-width: 300px;
    color: #666;
    letter-spacing: 0;
    text-shadow: 0 1px 1px #fff;
  }

  ul {
    list-style-type: none;
    display: flex;
    column-gap: 15px;
    row-gap: 15px;
    padding: 0 15px;
    margin: 2.5rem auto;
    justify-content: center;
    align-items: center;
  }
  @media (max-width: 800px) {
    h1 {
      font-size: 2.5rem;
    }
    p {
      font-size: 1.1rem;
    }
    ul {
      flex-direction: column;
    }
  }
  li {
    width: 300px;
    height: 200px;
    box-sizing: border-box;
    display: block;
    background-color: #444;
    background-size: cover;
    background-position: 50% 50%;
    border-radius: 10px;
    font-size: 15px;
    font-weight: bold;
    color: #fff;
    text-shadow: 0 1px 2px rgba(0, 0, 0, 0.75);
    padding-top: 165px;
    box-shadow: 0 1px 3px rgba(0, 0, 0, 0.3);
    transition: 150ms ease transform;
  }
  li:hover {
    transform: scale(1.02);
  }
  li:active {
    transform: scale(0.98);
  }

  .akl-syd {
    background-image: url(/akl-syd.jpg);
  }
  .nyc-hkg {
    background-image: url(/nyc-hkg.jpg);
  }
  .cbr-wdc {
    background-image: url(/cbr-wdc.jpg);
  }

  footer {
    padding: 0.75rem 0 2rem;
  }

  footer a {
    color: #888;
    font-size: 0.75rem;
  }
</style>
