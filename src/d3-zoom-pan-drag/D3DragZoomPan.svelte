<script>
  import { onMount } from "svelte";
  import { select } from "d3-selection";
  import { drag } from "d3-drag";

  import { scaleLinear } from "d3-scale";
  import { zoom } from "d3-zoom";

  import Icon from "svelte-awesome";
  import mapMarker from "svelte-awesome/icons/mapMarker";
  import mapPin from "svelte-awesome/icons/mapPin";

  let data = [],
    width = 600,
    height = 400,
    numPoints = 1;

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

  const myEndHandler = () => {
    console.log(`END HANDLER`);
    select("svg").selectAll("#target-circle").attr("cursor", "pointer");
  };

  function initDrag() {
    // select("svg").select("#target-circle").call(myDrag);

    // console.log(`In initDrag: `, select("svg").selectAll("circle"));
    // console.log(
    //   `In initDrag: `,
    //   select("svg").select(".bg .zoomed .canvas #target-circle")
    // );
    console.log(`In initDrag: `, select("svg").select("#target-circle"));
    // console.log(`In initDrag: `, select("svg").selectAll("line"));

    // select("svg").selectAll("circle").attr("cursor", "grabbing").call(myDrag);
    select("svg")
      .selectAll("#target-circle").attr('fill', 'green')
      .attr("cursor", "grab")
      .call(myDrag);
  }

  let myDrag = drag()
    .on("start", dragstarted)
    .on("drag", dragged)
    .on("end", dragended);

  function dragstarted(e) {
    select("svg").selectAll("#target-circle").attr("cursor", "grabbing");
  }

  function dragged(e) {
    // console.log(`0. In dragged...`);
    e.subject.x = e.x;
    e.subject.y = e.y;

    update();
  }

  function dragended(e) {
    select("svg").selectAll("#target-circle").attr('fill', 'green').attr("cursor", "grab");
  }

  function updateData() {
    // console.log(`UPDATE-DATA..."`);

    data = [];
    for (let idx = 0; idx < numPoints; idx++) {
      data.push({
        id: idx,
        x: 160 - 50 * idx,
        y: 200,
        fill: idx % 2 === 0 ? "green" : "red",
      });
    }
    // console.log(`DATA: `, data);
  }

  function update() {
    select("svg")
      .select("#target-circle")
      // .selectAll("circle")
      .data(data)
      .join("target-circle")
      // .join("circle")
      .attr("cx", (d) => d.x)
      .attr("cy", (d) => d.y)
      .attr("r", 8)
      .attr("fill", "rebeccapurple")
      // .attr("fill", (d) => d.fill)
      .attr("opacity", "0.55")
      .attr("cursor", "grabbing");
  }
</script>

<svg id="viz2" width="300" height="300">
  <g class="bg">
    <rect width="100%" height="100%" fill="#efc" />
    <!-- This circle can be dragged, but not zoomed or panned -->
    <!-- <circle id="plain-circle" cx="130" cy="80" r="5" fill="rebeccapurple" /> -->
    <g class="zoomed">
      <!-- In here, this circle can be zoomed, panned, && dragged -->
      <g class="canvas" transform="translate(0, 0)">
        <line
          x1="100"
          y1="60"
          x2="200"
          y2="260"
          stroke-width="2"
          stroke="rebeccapurple"
        />

        <line
          x1="160"
          y1="200"
          x2="150"
          y2="120"
          stroke-width="2"
          stroke="green"
        />

        <text id="init-wp" x="130" y="80" fill="rebeccapurple">
          {"\uf3c5"}</text
        >

        <!-- Must have circle here or applied circle from D3 code will not zoom-and-pan -->
        <circle id="target-circle" cx="170" cy="100" r="5" fill="red" />

        <!-- <Icon data={mapMarker} scale='3' style='fill: rgb(21, 85, 146);'/> -->
        <g transform={`translate(${150}, ${120})`}>
          <Icon data={mapPin} scale="1" style="fill: rgb(21, 85, 146);" />
        </g>

        <!-- <circle id="target-circle-2" cx="230" cy="100" r="20" fill="blue" /> -->
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
