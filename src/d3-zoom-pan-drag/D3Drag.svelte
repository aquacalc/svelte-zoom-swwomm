<script>
  import { onMount } from "svelte";
  import { select } from "d3-selection";
  import { drag } from "d3-drag";

  import { scaleLinear } from "d3-scale";
  import { zoom } from "d3-zoom";

  let data = [],
    width = 600,
    height = 400,
    numPoints = 2;

  let zoomed;
  let bg;

  // Re-scaled x & y
  let newScaleX;
  let newScaleY;

  // Add X axis
  $: xScale = scaleLinear().domain([0, 4]).range([0, 300]);

  // Add Y axis
  $: yScale = scaleLinear().domain([0, 4]).range([300, 0]);

  onMount(() => {
    zoomed = select("#viz2 .zoomed");

    bg = select("#viz2 .bg").call(
      zoom() // base d3 pan & zoom behavior
        .scaleExtent([1, 5]) // limit zoom to between 20% and 200% of original size
        .on("zoom", myZoomHandler) // apply pan & zoom transform to 'zoomed' element
      // .on("start", myStartHandler)
      // .on("end", myEndHandler)
    );

    updateData();
    update();
    initDrag();
  });

  const myZoomHandler = ({ transform }) => {
    console.log(`Evo myZoomHandler`);
    zoomed.attr("transform", transform);

    // select("#circle-two").attr("fill", "green").raise();

    newScaleX = transform.rescaleX(xScale);
    newScaleY = transform.rescaleY(yScale);
  };

  function initDrag() {
    // select("svg").select("#target-circle").call(myDrag);
    select("svg").selectAll("circle").attr("cursor", "grabbing").call(myDrag);
  }

  let myDrag = drag().on("drag", handleDrag);

  function handleDrag(e) {
    console.log(`In handleDrag...`);
    e.subject.x = e.x;
    e.subject.y = e.y;
    update();
  }

  function updateData() {
    data = [];
    for (let i = 0; i < numPoints; i++) {
      data.push({
        id: i,
        x: 160,
        y: 200,
        fill: i % 2 === 0 ? "green" : "red",
      });
    }
    // console.log(`DATA: `, data);
  }

  function update() {
    select("svg")
      .selectAll("circle")
      .data(data)
      .join("circle")
      .attr("cx", (d) => d.x)
      .attr("cy", (d) => d.y)
      .attr("r", 20)
      .attr("fill", (d) => d.fill)
      .attr("opacity", "0.5");
  }
</script>

<svg id="viz2" width="300" height="300">
  <g class="bg">
    <rect width="100%" height="100%" fill="#efc" />
<!-- This circle can be dragged, but not zoomed or panned -->
    <circle id="plain-circle" cx="130" cy="80" r="5" fill="rebeccapurple" />
    <g class="zoomed">
      <!-- In here, this circle can be zoomed, panned, && dragged -->
      <g class="canvas" transform="translate(0, 0)">
        <circle id="target-circle" cx="170" cy="100" r="10" fill="red" />
        <!-- <circle id="target-circle-2" cx="230" cy="100" r="20" fill="blue" /> -->
        <line
          x1="100"
          y1="60"
          x2="200"
          y2="260"
          stroke-width="2"
          stroke="rebeccapurple"
        />
        <!-- <circle
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
        /> -->
      </g>
    </g>
  </g>
</svg>
