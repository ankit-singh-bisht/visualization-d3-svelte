<script>
  import { onMount } from "svelte";

  export let labels;
  export let values;
  export let colorScale;
  export let x;
  export let y;
  export let width = 150;
  export let backgroundColor = "white";
  export let opacity = 1;
  export let textColor = "black";
  export let title;
  export let adaptTexts = true;

  const step = 25;
  const paddingLeft = 15;
  const paddingRight = 15;
  const lineLength = 10;
  const spaceBetweenLineText = 3;

  const idContainer = "svg-tooltip-" + Math.random() * 10000;
  const maxTextLength =
    width - paddingLeft - lineLength - spaceBetweenLineText - paddingRight;
  let computedWidth = width;

  onMount(async () => {
    const texts = document
      .getElementById(idContainer)
      .getElementsByClassName("legend-labels");
    const textWidths = [...Array(texts.length).keys()].map((d) => ({
      id: d,
      width: texts[d].getBoundingClientRect().width,
    }));
    const longTexts = textWidths.filter((d) => d.width > maxTextLength);
    if (longTexts.length === 0) return;
    if (adaptTexts) {
      longTexts.map((d) => {
        const textContent = texts[d.id].textContent;
        const numCharsAvailable =
          Math.floor((maxTextLength * textContent.length) / d.width) - 3;
        texts[d.id].textContent =
          textContent.slice(0, numCharsAvailable).trim() + "...";
      });
    } else {
      const maxLength = Math.max(...longTexts.map((d) => d.width));
      computedWidth =
        paddingLeft +
        lineLength +
        spaceBetweenLineText +
        maxLength +
        paddingRight;
    }
  });
</script>

<svg x={x - 10} {y} width={computedWidth + 2} height="200" id={idContainer}>
  <rect
    x="1"
    y="1"
    width={computedWidth}
    height={(labels.length + 1 + (title !== undefined ? 1 : 0)) * step}
    stroke="black"
    stroke-width="1"
    fill={backgroundColor}
    {opacity}
  />
  {#if title !== undefined}
    <text
      x={paddingLeft + 3}
      y={step}
      alignment-baseline="middle"
      font-size="14"
      fill={textColor}>{title}</text
    >
  {/if}
  {#each labels as label, i}
    <g>
      <line
        x1={paddingLeft}
        x2={paddingLeft + lineLength}
        y1={(i + 1 + (title !== undefined ? 1 : 0)) * step - 1}
        y2={(i + 1 + (title !== undefined ? 1 : 0)) * step - 1}
        stroke={colorScale(label)}
        stroke-width="3"
      />
      <text
        x={paddingLeft + lineLength + spaceBetweenLineText}
        y={(i + 1 + (title !== undefined ? 1 : 0)) * step}
        alignment-baseline="middle"
        font-size="14"
        fill={textColor}
        class="legend-labels"
        >{label}{values !== undefined
          ? ": " + values[label].toLocaleString()
          : ""}</text
      >
      <title>{label}</title>
    </g>
  {/each}
</svg>
