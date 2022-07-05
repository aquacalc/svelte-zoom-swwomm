<script>
  import { onMount } from "svelte";
  import { select } from "d3-selection";
  import { zoom } from "d3-zoom";

  import { scaleLinear } from "d3-scale";
  import { drag } from "d3-drag";

  let zoomed;
  let bg;

  // Re-scaled x & y
  let newScaleX;
  let newScaleY;

  // Add X axis
  $: xScale = scaleLinear().domain([0, 4]).range([0, 300]);

  // Add Y axis
  $: yScale = scaleLinear().domain([0, 4]).range([300, 0]);

  // Without omMount(), there is no zoom-and-pan
  onMount(() => {
    // see: https://blog.swwomm.com/2021/03/d3v6-pan-and-zoom.html

    // "The rect inside the .bg group will ensure that the user's click-n-drag
    // or mouse wheeling will be captured as long as the mouse pointer is
    // anywhere inside the svg element
    // (without this rect, the mouse would be captured
    //  only when the mouse is over a graphical element inside the .bg group
    //  — like the circle in this example).""
    // "Then attach the pan & zoom behavior to the .bg group
    // — but apply the pan & zoom transform to the .zoomed group it contains.""
    // "This prevents stuttering when panning as the .bg group remains fixed;
    // and it avoids messing with any transforms or other
    // styling/positioning already on the inner .canvas group"

    zoomed = select("#viz2 .zoomed");

    bg = select("#viz2 .bg").call(
      zoom() // base d3 pan & zoom behavior
        .scaleExtent([1, 5]) // limit zoom to between 20% and 200% of original size
        .on("zoom", myZoomHandler) // apply pan & zoom transform to 'zoomed' element
        .on("start", myStartHandler)
        .on("end", myEndHandler)
    );

    select("#circle-two").attr("fill", "orange");
  });

  const myZoomHandler = ({ transform }) => {
    // console.log(`Evo myZoomHandler`)
    // zoomed.attr("transform", transform);

    select("#circle-two").attr("fill", "green").raise();

    newScaleX = transform.rescaleX(xScale);
    newScaleY = transform.rescaleY(yScale);
  };

  const myStartHandler = () => {
    // console.log(`Evo myStartHandler`)

    select("#circle-two").attr("fill", "red");
  };

  const myEndHandler = () => {
    // console.log(`Evo myEndHandler`)

    select("#circle-two").attr("fill", "orange");
  };

  // ------------------------------------ //

  let myDrag = drag().on("drag", handleDrag);

  function handleDrag(e) {
    e.subject.x = e.x;
    e.subject.y = e.y;
    update();
  }

  function update() {
    select("svg")
      .selectAll("circle")
      .data(data)
      .join("circle")
      .attr("cx", function (d) {
        return d.x;
      })
      .attr("cy", function (d) {
        return d.y;
      })
      .attr("r", 40);
  }

  function initDrag() {
    select("svg").selectAll("circle").call(drag);
  }
</script>

<svg id="viz2" width="300" height="300">
  <g class="bg">
    <rect width="100%" height="100%" fill="#efc" />
    <g class="zoomed">
      <g class="canvas" transform="translate(200, 200)">
        <!-- <circle cx="0" cy="0" r="15" fill="#69c" /> -->
        <circle
          id="circle-one"
          cx={newScaleX ? newScaleX(0) : xScale(0)}
          cy={newScaleY ? newScaleY(4) : yScale(4)}
          r="18"
          fill="#69c"
        />
        <circle
          id="circle-two"
          cx={newScaleX ? newScaleX(1) : xScale(1)}
          cy={newScaleY ? newScaleY(4) : yScale(4)}
          r="12"
        />
      </g>
    </g>
  </g>
</svg>

<style>
  svg {
    outline: 3px solid green;
  }
  .bg {
    outline: 2px solid rebeccapurple;
  }
  .bg > rect {
    cursor: move;
  }
</style>
