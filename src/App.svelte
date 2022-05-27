<script>
  import { csv } from "d3-fetch";
  import { autoType } from "d3-dsv";
  import { scaleLinear, scaleBand } from "d3-scale";

  let w = 960;
  let h = 600;
  let barsH = 400;
  let xScale = scaleLinear().domain([-6, 6]).range([0, w]);
  let yScale = scaleLinear().domain([-4, 4]).range([0, h]);
  let data = null;

  csv("./tsne_output.csv", autoType()).then((rawData) => (data = rawData));
  let selectedSound = 1;

  const dims = [
    "breedte",
    "gewicht",
    "hoogte",
    "humiditeit",
    "lengte",
    "materie",
    "temperatuur",
    "textuur",
    "vorm",
  ];
  let yScaleBars = scaleBand().domain(dims).range([0, barsH]).paddingInner(0.3);
  let xScaleBars = scaleLinear().domain([0, 10]).range([0, w]);
</script>

{#if data}
  {#if selectedSound}
    <div>
      Speel geluid {selectedSound} af, of klik op een cirkel om een ander geluid
      te selecteren.<br />
      <audio src={`./sounds/geluid ${selectedSound}.mp3`} controls />
    </div>
  {:else}
    <div>Klik op een cirkel om het geluid af te spelen</div>
  {/if}
  <svg width={960} height={600}>
    {#if data}
      {#each data as sound}
        <g
          class={"sound-circle"}
          on:click={() => (selectedSound = sound.geluid)}
        >
          <circle
            cx={xScale(sound.V1)}
            cy={yScale(sound.V2)}
            r={20}
            stroke={"none"}
            stroke-width={"1px"}
          >
            {#if sound.geluid == selectedSound}
              <animate
                attributeName="r"
                from="16"
                to="36"
                dur="1.5s"
                begin="0s"
                repeatCount="indefinite"
              />
              <animate
                attributeName="opacity"
                from="1"
                to="0"
                dur="1.5s"
                begin="0s"
                repeatCount="indefinite"
              />
            {/if}
          </circle>
          <circle cx={xScale(sound.V1)} cy={yScale(sound.V2)} r="20" />
          <text
            x={xScale(sound.V1)}
            y={yScale(sound.V2)}
            dy={5}
            fill={"white"}
            text-anchor={"middle"}>{sound.geluid}</text
          >
        </g>
      {/each}
    {/if}
  </svg>
  <h2>Geluidprofiel geluid {selectedSound}</h2>
  <svg width={w} height={h}>
    {#each dims as dim}
      <rect
        x={0}
        y={yScaleBars(dim)}
        width={xScaleBars(+data.find((d) => d.geluid == selectedSound)[dim])}
        height={yScaleBars.bandwidth()}
      />
      <text x={4} y={yScaleBars(dim) + 22} fill={"white"}>{dim}</text>
      <text
        x={xScaleBars(+data.find((d) => d.geluid == selectedSound)[dim]) + 4}
        y={yScaleBars(dim) + 22}
        fill={"black"}
        >{Math.round(+data.find((d) => d.geluid == selectedSound)[dim] * 10) /
          10}</text
      >
    {/each}
  </svg>
{/if}

<style>
  .sound-circle {
    cursor: pointer;
  }
</style>
