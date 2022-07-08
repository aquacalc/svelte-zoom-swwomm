<script>
  import { onMount } from "svelte";
  import { select } from "d3-selection";
  import { drag } from "d3-drag";

  import { scaleLinear } from "d3-scale";
  import { zoom } from "d3-zoom";

  import Icon from "svelte-awesome";
  import mapMarker from "svelte-awesome/icons/mapMarker";
  import mapPin from "svelte-awesome/icons/mapPin";
  import circle from "svelte-awesome/icons/circle";
  // import crosshairs from 'svelte-awesome/icons/crosshairs';

  let data = [],
    width = 600,
    height = 400,
    numPoints = 1;

  let zoomed;
  let bg;

  // Re-scaled x & y
  let newScaleX;
  let newScaleY;

  // target Icon's <g> dimensions
  let targetWidth;
  let targetHeight;

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

  // const myEndHandler = () => {
  //   console.log(`END HANDLER`);
  //   select("svg").selectAll("#target-circle").attr("cursor", "pointer");
  // };

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
    // console.log(`0. In dragged...`);
    e.subject.x = e.x;
    e.subject.y = e.y;

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
    data = [];
    for (let idx = 0; idx < numPoints; idx++) {
      data.push({
        id: idx,
        x: 160 - 50 * idx,
        y: 200,
        fill: idx % 2 === 0 ? "green" : "red",
        heightInit: select("#wq-init-group").node().getBBox().height,
        widthInit: select("#wq-init-group").node().getBBox().width,
        heightTarget: select("#wq-target-group").node().getBBox().height,
        widthTarget: select("#wq-target-group").node().getBBox().width,

        // height: 27.428569793701172,
        // width: 18.285715103149414
      });
    }
  }

  // got it myself, but the following is confirmation...
  // see: https://stackoverflow.com/questions/47816033/how-can-i-drag-group-g-element-in-d3-js

  function update() {
    // // see: https://stackoverflow.com/questions/21990857/d3-js-how-to-get-the-computed-width-and-height-for-an-arbitrary-element
    // console.log(`In initDrag: `, select("#wq-target-group").node().getBBox());

    let targetGsDimensions = select("#wq-target-group").node().getBBox();
    targetWidth = targetGsDimensions.width;
    targetHeight = targetGsDimensions.Height;

    // console.log(`targetWidth: ${targetWidth} (${typeof targetWidth})`);

    console.log(`0. update: `, data[0].x);

    select("svg")
      .select("#wq-target-group")
      .data(data)
      .join("wq-target-group")
      .attr("transform", (d) => `translate(${d.x}, ${d.y})`)
      .attr("fill", "red")
      // .style("fill", d => {d.x >= 180 ? 'red' : 'blue'})
      // .attr("opacity", "0.75")
      .attr("cursor", "grabbing");

    // [NB] Need this to syle Svelte Awesome icon(s)
    const myTarget = select("#wq-target-group .fa-icon").attr("opacity", "0.9");
    if (data[0].x >= 230 || data[0].x <= 60) {
      console.log(`1-A. update: `, data[0].x);
      myTarget.style("fill", "red");
    } else {
      console.log(`1-B. update: `, data[0].x);
      myTarget.style("fill", "olive");
    }

    const myTargetCircle = select("#wq-target-group-circle .fa-icon");
    myTargetCircle.style("fill", "red");

    select("svg")
      .select("#target-line")
      .data(data)
      .join("target-line")
      .attr("x1", (d) => 150 + d.widthInit / 2)
      .attr("y1", (d) => 120 + d.heightInit)
      .attr("x2", (d) => d.x + d.widthTarget / 2)
      .attr("y2", (d) => d.y + d.heightTarget + 4); // "+ 4" to tip of target icon
    // .attr("x2", (d) => d.x + 18.285715103149414 / 2)
    // .attr("y2", (d) => d.y + 27.428569793701172)
    // .attr("fill", "red")
    // .attr("opacity", "0.65");
    // .attr("cursor", "grabbing");
  }

  $: isTargetSafe = false;
</script>

<h3>
  Drag target WP (only) plus <span>(semantically)</span> zoom & pan all elements
</h3>
<svg id="viz2" width="300" height="300">
  <g class="bg">
    <rect width="100%" height="100%" fill="#efc" />
    <!-- This circle can be dragged, but not zoomed or panned -->
    <!-- <circle id="plain-circle" cx="130" cy="80" r="5" fill="rebeccapurple" /> -->
    <g class="zoomed">
      <!-- In here, this circle can be zoomed, panned, && dragged -->
      <g class="canvas" transform="translate(0, 0)">
        <line
          id="target-line"
          x1="100"
          y1="60"
          x2="200"
          y2="260"
          stroke-width="1.5"
          stroke="#202020"
          stroke-linecap="round"
        />

        <!-- <g transform={`translate(${150}, ${120})`}>
          <Icon data={mapMarker} scale="4" />
          <g transform={`translate(${7.5}, ${8.5})`}>
            <Icon data={circle} scale="1.6" style="fill:skyblue" />
          </g>
        </g> -->

        <!-- <text id="init-wp" x="130" y="80" fill="rebeccapurple">
          {"\uf3c5"}</text
        > -->

        <!-- Must have circle here or applied circle from D3 code will not zoom-and-pan -->
        <!-- <circle id="target-circle" cx="170" cy="100" r="5" fill="red" /> -->

        <!-- <Icon data={mapMarker} scale='3' style='fill: rgb(21, 85, 146);'/> -->
        <g id="wq-init-group" transform={`translate(${150}, ${120})`}>
          <Icon data={mapPin} scale="1" style="fill: rgb(21, 85, 146);" />
        </g>

        <g id="wq-target-group" transform={`translate(${100}, ${100})`}>
          <Icon data={mapMarker} scale="4" />
          <g
            id="wq-target-group-circle"
            transform={`translate(${7.5}, ${8.5})`}
          >
            <Icon data={circle} scale="1.6" />
            <!-- <Icon
              data={circle}
              scale="1.6"
              style="fill:{isTargetSafe ? 'green' : 'red'}"
            /> -->
          </g>
        </g>
      </g>
    </g>
  </g>
</svg>

<style>
  span {
    text-decoration: line-through;
  }
</style>
