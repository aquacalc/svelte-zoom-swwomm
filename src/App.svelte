<script>
  // Zoom & pan only
  // import D3ZoomPan from "./d3-zoom-pan-drag/D3ZoomPan.svelte";

  // Drag target circle plus (geometric) zoom and pan
  // import D3Drag from "./d3-zoom-pan-drag/D3Drag.svelte";

  // Drag target WP (only) with zoom (GEOMETRIC) & pan entire chart content
  import D3DragZoomPan from "./d3-zoom-pan-drag/D3DragZoomPan.svelte";

  // Drag target WP (only) with zoom (SEMANTIC) & pan entire chart content
  import D3_D_Z_P_semantic from "./d3-zoom-pan-drag/D3_D_Z_P_semantic.svelte";
</script>

<main>
  <h3>Svelte-ish D3 zoom-and-pan with element drag for WQ Map</h3>
  <p>
    Visit <a href="https://blog.swwomm.com/2021/03/d3v6-pan-and-zoom.html"
      >this blog</a
    > for the HTML zoom-and-pan version...
  </p>
  <p>
    ...and <a href="https://www.d3indepth.com/interaction/">this blog</a> for the
    HTML drag version
  </p>

  <section>
    <!-- <D3ZoomPan /> -->
  </section>

  <section>
    <!-- <D3Drag /> -->
  </section>

  <section>
    <!-- <D3DragZoomPan /> -->
  </section>

  <section>
    <D3_D_Z_P_semantic />
  </section>
</main>

<style>
  main {
    text-align: center;
    padding: 1em;
    max-width: 240px;
    margin: 0 auto;
  }

  h3 {
    color: #ff3e00;
    /* text-transform: uppercase; */
    font-size: 1.2em;
    font-weight: 100;
  }

  @media (min-width: 640px) {
    main {
      max-width: none;
    }
  }
</style>
