---
title: "Lab 1: Passing Pollinators"
toc: true
theme: [parchment]
---

This page is where you can iterate. Follow the lab instructions in the [readme.md](./README).


```js
const bees = FileAttachment("./data/pollinator_activity_data.csv").csv({typed: true})
```


#### ❓Q1 What is the body mass and wing span distribution of each pollinator species observed?
## Bigger bees can spread their wings wider compared to smaller ones

Practically all Carpenter bees (except for one skinny legend) weight more and have wider wings span compared to Bumble bees are Honey Bees.


```js echo
Plot.plot({
  color: {legend: true},
  x: {label: "Average mass, g"},
  y: {label: "Average wing span, mm"},
  marks: [
    Plot.dot(bees, {x: "avg_body_mass_g", y: "avg_wing_span_mm", stroke: "pollinator_species"})
  ]
})
```


#### ❓Q2 What is the ideal weather (conditions, temperature, etc) for pollinating?
## Bees prefer cooler temperature and less sun but ultimately they are not that picky

```js echo
Plot.plot({
  marginLeft: 80,
  x: {label: "# of observations"},
  y: {label: "nectar produced, microliters"},
  color: {legend: true},
  marks: [
    Plot.barX(bees, {y: "weather_condition", x: 1, inset: 0.5, fill: "temperature", sort: "temperature"}),
    Plot.ruleX([0])
  ]
})
```



#### ❓Q3 Which flower has the most nectar production?
## Sunflowers produced the most nectar

```js echo
Plot.plot({
    x: {label: ""},
  marks: [
    Plot.barY(bees, Plot.groupX({y: "sum"}, {x: "flower_species", y: "nectar_production", fill: "flower_species", sort: {x: "-y"}})),
    Plot.ruleY([0])
  ]
})
```