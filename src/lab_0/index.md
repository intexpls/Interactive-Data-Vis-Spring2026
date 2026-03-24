---
title: "Lab 0: Getting Started"
toc: true
---

This page is where you can iterate. Follow the lab instructions in the [readme.md](./README.md).



```js 
Plot.plot({
  inset: 10,
  color: {legend: true},
  marks: [
    d3.groups(penguins, (d) => d.species).map(([s]) =>
      Plot.density(penguins, {
        x: "flipper_length_mm",
        y: "culmen_length_mm",
        weight: (d) => d.species === s ? 1 : -1,
        fill: () => s,
        fillOpacity: 0.2,
        thresholds: [0.05]
      })
    ),
    Plot.dot(penguins, {
      x: "flipper_length_mm",
      y: "culmen_length_mm",
      stroke: "species"
    }),
    Plot.frame()
  ]
})

```