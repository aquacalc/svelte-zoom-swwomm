<script>
  // *************************** //
  //      Drag, Zoom, & Pan      //
  //      -- SEMANTIC ZOOM --    //
  // *************************** //

  import { onMount } from "svelte";
  import { select } from "d3-selection";
  import { drag } from "d3-drag";

  import { scaleLinear } from "d3-scale";
  import { zoom, zoomIdentity } from "d3-zoom";

  import Icon from "svelte-awesome";
  import mapMarker from "svelte-awesome/icons/mapMarker";
  import mapPin from "svelte-awesome/icons/mapPin";
  import circle from "svelte-awesome/icons/circle";
  // import crosshairs from 'svelte-awesome/icons/crosshairs';

  let data = [];

  let zoomed;
  let bg;

  // Re-scaled x & y
  let newScaleX;
  let newScaleY;
  // $: newScaleX = zoomIdentity;
  // $: newScaleY = zoomIdentity;

  // Add X axis
  $: xScale = scaleLinear().domain([0, 8]).range([0, 300]);

  // Add Y axis
  $: yScale = scaleLinear().domain([0, 8]).range([300, 0]);

  onMount(() => {
    zoomed = select("#viz2 .zoomed");

    bg = select("#viz2 .bg").call(
      zoom() // base d3 pan & zoom behavior
        .scaleExtent([1, 5]) // limit zoom to between 20% and 200% of original size
        .on("zoom", myZoomHandler) // apply pan & zoom transform to 'zoomed' element
      // .on("start", myStartHandler)
    );

    updateData();
    update();
    initDrag();
  });

  const myZoomHandler = ({ transform }) => {
    // console.log(`Evo myZoomHandler`);
    zoomed.attr("transform", transform);

    newScaleX = transform.rescaleX(xScale);
    newScaleY = transform.rescaleY(yScale);
  };

  function initDrag() {
    // console.log(
    //   `In initDrag: `,
    //   select("svg").select(".bg .zoomed .canvas #target-circle")
    // );
    // console.log(`In initDrag: `, select("#wq-target-group"));

    // // see: https://stackoverflow.com/questions/21990857/d3-js-how-to-get-the-computed-width-and-height-for-an-arbitrary-element
    // console.log(`In initDrag: `, select("#wq-target-group").node().getBBox());

    select("svg")
      .selectAll("#wq-target-group")
      .attr("fill", "green")
      .attr("cursor", "grab")
      .call(myDrag);
  }

  let myDrag = drag()
    .on("start", dragstarted)
    .on("drag", dragged)
    .on("end", dragended);

  function dragstarted(e) {
    // console.log(`In dragstarted...`);
    select("svg")
      .selectAll("#wq-target-group")
      .attr("cursor", "grabbing")
      .attr("fill", "red");
  }

  function dragged(e) {
    // console.log(`0. In dragged... [${e.x}, ${e.y}]`);
    // if (e.x <= 250 && e.x > 0) {
    e.subject.x = e.x;
    e.subject.y = e.y;
    // }

    update();
  }

  function dragended(e) {
    select("svg")
      .selectAll("#wq-target-group")
      .attr("fill", "green")
      .attr("cursor", "grab");
  }

  function updateData() {
    console.log(`UPDATE DATA`);

    // // see: https://stackoverflow.com/questions/21990857/d3-js-how-to-get-the-computed-width-and-height-for-an-arbitrary-element
    let heightInit = select("#wq-init-group").node().getBBox().height;
    let widthInit = select("#wq-init-group").node().getBBox().width;

    // Waypoint data
    data = [
      {
        x: newScaleX ? newScaleX(3) : xScale(3),
        y: newScaleY ? newScaleY(4) : yScale(4),
        heightInit,
        widthInit,
        heightTarget: select("#wq-target-group").node().getBBox().height,
        widthTarget: select("#wq-target-group").node().getBBox().width,
      },
    ];
  }

  // got it myself, but the following is confirmation...
  // see: https://stackoverflow.com/questions/47816033/how-can-i-drag-group-g-element-in-d3-js
  function update() {
    select("svg")
      .select("#wq-target-group")
      .data(data)
      .join("wq-target-group")
      .attr("transform", (d) => `translate(${d.x}, ${d.y})`)
      // .attr("transform", (d) => `translate(${
      //   newScaleX ? newScaleX(d.x) : xScale(d.x)
      // },
      // ${
      //   newScaleY ? newScaleY(d.y) : yScale(d.y)
      // })`)
      .attr("cursor", "grabbing");

    // [NB] Need this to syle Svelte Awesome icon(s)
    const myTargetCircle = select("#wq-target-group-circle .fa-icon");

    const myTarget = select("#wq-target-group .fa-icon").attr(
      "opacity",
      "0.85"
    );
    
    if (data[0].x >= 230 || data[0].x <= 60) {
      myTargetCircle.style("fill", "red");
    } else {
      myTargetCircle.style("fill", "#00cc00");
    }

    select("svg")
      .select("#target-line")
      .data(data)
      .join("target-line")
      .attr("x1", (d) => 150 + d.widthInit / 2)
      .attr("y1", (d) => 120 + d.heightInit)
      .attr("x2", (d) => d.x + d.widthTarget / 2)
      .attr("y2", (d) => d.y + d.heightTarget + 1.55) // "scale=2: + 4" to tip of target icon
      .attr("opacity", "0.85");
  }
</script>

<h3>
  Drag target WP (only) plus <span>(semantically)</span> zoom & pan all elements
</h3>
<svg id="viz2" width="300" height="300">
  <g class="bg">
    <rect
      width="100%"
      height="100%"
      fill="whitesmoke"
      style="outline: 1px solid gray"
    />

    <g class="zoomed">
      <!-- In here, this circle can be zoomed, panned, && dragged -->
      <g class="canvas" transform="translate(0, 0)">
        <line
          id="target-line"
          stroke-width="2"
          stroke="olive"
          stroke-linecap="round"
          stroke-opacity="0.5"
        />

        <!-- <g
          id="wq-init-group"
          transform={`translate(${newScaleX ? newScaleX(5) : xScale(5)}, ${
            newScaleY ? newScaleY(3) : yScale(3)
          })`}
        > -->

        <!-- INITIAL WAYPOINT -->
        <g id="wq-init-group" transform={`translate(${150}, ${120})`}>
          <!-- <g 
        id="wq-init-group" 
        transform={`translate(${
          newScaleX ? newScaleX(4) - 9.143 : xScale(4) - 9.143
        }, ${newScaleY ? newScaleY(4) - 32 : yScale(4) - 32})`}
      > -->
          <Icon data={mapPin} scale="1" style="fill: rgb(21, 85, 146);" />
        </g>

        <!-- TARGET WAYPOINT -->
        <g id="wq-target-group" transform={`translate(${100}, ${100})`}>
          <Icon data={mapMarker} scale="2" />
          <g
            id="wq-target-group-circle"
            transform={`translate(${3.75}, ${4.5})`}
          >
            <Icon data={circle} scale="0.8" />
          </g>
        </g>
      </g>
    </g>
  </g>
</svg>

<style>
  /* span {
    text-decoration: line-through;
  } */
</style>
