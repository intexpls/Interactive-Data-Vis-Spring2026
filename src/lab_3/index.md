---
title: "Lab 3: Mayoral Mystery"
toc: false
---

<!--
This page is where you can iterate. Follow the lab instructions in the [readme.md](./README.md).
-->
<style>
  body {
    font-size: 14pt;
  }

  svg {
    font-size: 11pt;
  }

  main {
    max-width: 60%;
    margin: 0 auto;
  }

  p, h1, h2, h3, h4 {
    max-width: 100%;
    line-height: 1.35;
  }

  p {
    margin-top: 0px;
    margin-bottom: 10px;
  }

  h2 {
    line-height: 1.1;
    margin-bottom: 18px;
    margin-top: 40px;
  }

  .title-card {
    text-align: center;
    line-height: 1.05;
    margin-bottom: 20px;
    z-index: 1;
  }

  .main-title-card {
    font-weight: 900;
    font-size: 40pt;
    margin-top: 40px;
  }

  .sub-title-card {
    font-weight: 600;
    font-size: 22pt;
    margin-top: 25px;
  }

  .title-graph {
    position: relative;
    z-index: -2;
  }

  .chart-container {
    display: flex;
  }

  .vertical-chart-container {
    flex-direction: column;
  }

  .horizontal-chart-container {
    flex-direction: row;
  }

  .legend {
    font-size: 12pt;
    font-family: Helvetica;
    color: #9F9F9F;
    margin-top: 22px;
    margin-bottom: -5px;
  }

  .chart-subtitle {
    font-size: 10.5pt;
    font-family: Helvetica;
    color: #B6B6B6;
    margin-bottom: 40px;
    font-weight: 700;
  }

  .chart-subtitle-two-lines {
    margin-bottom: 25px;
    line-height: 13px;
  }

  .chart-title-emoji {
    line-height: 28px;
    font-size: 10pt;
  }
</style>

<!-- Import Data -->
```js
const nyc = await FileAttachment("data/nyc.json").json();
const results = await FileAttachment("data/election_results.csv").csv({ typed: true });
const survey = await FileAttachment("data/survey_responses.csv").csv({ typed: true });
const survey_agg = await FileAttachment("data/survey_agg.csv").csv({ typed: true });
const survey_agg_2 = await FileAttachment("data/survey_agg_2.csv").csv({ typed: true });
const survey_flat = await FileAttachment("data/survey_flat.csv").csv({ typed: true });
const events = await FileAttachment("data/campaign_events.csv").csv({ typed: true });
```

<!--
// Note: you don't have to keep this, but some helpful data exposure to see what we've loaded. 
// NYC geoJSON data
display(nyc)
// Campaign data (first 10 objects)
display(results.slice(0,10))
display(survey.slice(0,10))
display(events.slice(0,10))
-->


```js
// The nyc file is saved in data as a topoJSON instead of a geoJSON. Thats primarily for size reasons -- it saves us 3MB of data. For Plot to render it, we have to convert it back to its geoJSON feature collection. 
const districts = topojson.feature(nyc, nyc.objects.districts)

```

```js
const resultsByDistrict = new Map(results.map(d => [d.BoroCD, d.winner_share]))
const resultsByDistrictUs = new Map(results.map(d => [d.BoroCD, d.pct_candidate_win]))
const resultsByDistrictOpp = new Map(results.map(d => [d.BoroCD, d.pct_opponent_win]))
```

<!--
```js
display(districts)
display(resultsByDistrict)
```
-->

<div class="tip">For the best viewing experience, I suggest to view this lab in the full screen without the left-side panel.</div>


<p class='title-card main-title-card'>Welp, we lost</p>
  <img class="title-graph" src="./assets/title_graph.png" alt="A horizontal stacked bar chart for the shares of votes for each candidate." width="100%"/>
<p class='title-card sub-title-card'>But what could we do differently?</p>

***

Our candidate has lost the race. This, however, was a very close one - pulling 30k more votes, or 15k away from the opponent, would be enough to win. It was close enough to think that by doing a few more events, knocking on a few dozen more doors or tweaking our messaging a bit we could've pulled it off. 

Let's look at why our candidate lost and what could be changed to win the next time.

<!-- Section: Overall results -->

<h2>Clear splits<br> and seeming lack of excitement</h2>

For starter, let's look at results granulally <br>- what boroughs and districts our candidate won<br>and by how much.

<!-- Overall results, choropleth -->
<div class='chart-container vertical-chart-container'>
  <div style="position: relative; z-index: 2">

```js
// Simple rendering of the NYC districts topoJSON
Plot.plot({
  height: 10,
  // this projection is already zoomed into NYC
  projection: {
    domain: districts,
    type: "mercator",
  },
  color: {
    type: "linear",
    range: ["#F4AC26", "#FFE4B3", "white", "#B3FFED", "#0CF2BD"],
    unknown: "#ddd",
    legend: true, 
    tickFormat: d => ({ "-100": "100" }[d] || d),
    label: "% voters supported Opponent vs Our candidate", 
    percent: true, 
    domain: [-100, 100] 
  },
  marks: []
})
```
  </div>
  <div style="position: relative; z-index: 1; margin-top: -275px;">

```js
// Simple rendering of the NYC districts topoJSON
Plot.plot({
  width: 1000,
  marginLeft: 50,
  // this projection is already zoomed into NYC
  projection: {
    domain: districts,
    type: "mercator",
  },
  color: {
    type: "linear",
    range: ["#F4AC26", "#FFE4B3", "white", "#B3FFED", "#0CF2BD"],
    unknown: "#ddd",
    percent: true, 
    domain: [-100, 100] 
  },
  marks: [
    Plot.geo(districts),
    Plot.geo(districts, {
      fill: (d) => resultsByDistrict.get(d.properties.BoroCD),
      stroke: "black",
      strokeWidth: 0.2,
      tip: true,
      channels: {
        District: "BoroCD",
        Winner_Vote_Share: (d) => resultsByDistrict.get(d.properties.BoroCD) < 0 ? (resultsByDistrict.get(d.properties.BoroCD) * -100).toFixed() + "%" : (resultsByDistrict.get(d.properties.BoroCD) * 100).toFixed() + "%"
      }
    })
  ]
})
```
  </div>
</div>

The map makes it more clear where our campaign has fallen short. 

First, there are relatively clean splits in how boroughs went us vs the opponent. Our candidate won most of Brooklyn, the Bronx, some parts of Queens and Upper Manhattan (Harlem, Washington Heights, etc), while the opponent took home the rest of Manhattan, most of Queens, the whole Staten Island and some parts of west Brooklyn. 

Second, the opponent wins seem more decisive, with plenty districts giving them 70%+ of votes. We, on the other hand, barely squeezed through in mosts places, only once crossing the 60% of votes.


This disposition suggests that either our campaign efforts were either more intense/effective in some districts or our message resonated better there. Additionally, lower win gaps suggests that there might be the lack of enthusiasm or awareness about our candidate. We'll look at the campaign efforts later, now let's look deeper into districts we won and lost to see what unites them.


<!-- Section: districts won by income level -->
<!-- Chart: three beeswarm charts -->


<h2>Our candidate "sold" their candidacy to low-income voters but completely lost high-earners</h2>

There are different ways to understand voters who resonated with our candidate and campaign. We could look at the message/campaign in the vaccuum, conduct survey or, what makes sense now, look at what groups of votes responded more. 

One data piece we have regarding that is the median household income in the districts. This is not an ideal measure but the best we have now.

Below are all districts split by who won them, ordered by their median income and colored by their borough. 

```js
function beeswarm(
  data,
  { gap = 2, ticks = 50, dynamic, direction, xStrength = 1, yStrength = 0.05, ...options }
) {
  const dots = Plot.dot(data, options);
  const { render } = dots;

  dots.render = function () {
    const g = render.apply(this, arguments);
    const circles = d3.select(g).selectAll("circle");

    const nodes = [];
    const [cx, cy, x, y, forceX, forceY] =
      direction === "x"
        ? ["cx", "cy", "x", "y", d3.forceX, d3.forceY]
        : ["cy", "cx", "y", "x", d3.forceY, d3.forceX];
    for (const c of circles) {
      const node = {
        x: +c.getAttribute(cx),
        y: +c.getAttribute(cy),
        r: +c.getAttribute("r")
      };
      nodes.push(node);
    }
    const force = d3
      .forceSimulation(nodes)
      .force("x", forceX((d) => d[x]).strength(xStrength))
      .force("y", forceY((d) => d[y]).strength(yStrength))
      .force(
        "collide",
        d3
          .forceCollide()
          .radius((d) => d.r + gap)
          .iterations(5)
      )
      .tick(ticks)
      .stop();
    update();
    if (dynamic) force.on("tick", update).restart();
    return g;

    function update() {
      circles.attr(cx, (_, i) => nodes[i].x).attr(cy, (_, i) => nodes[i].y);
    }
  };

  return dots;
}

function beeswarmX(data, options = {}) {
  return beeswarm(data, { ...options, direction: "x" });
}
```

<p class='legend'>Median household income by district by borough: <span style="background-color: #3763B1; color: white;">Bronx</span>, <span style="background-color: #FF7018; color: white;">Brooklyn</span>, <span style="background-color: #00BC3D; color: white;">Manhattan</span>, <span style="background-color: #E10021; color: white;">Queens</span>, <span style="background-color: #A600B9; color: white;">States Island</span></p>


<div class="chart-container vertical-chart-container">
  <div>

```js
Plot.plot({
  height: 170,
  width: 755,
  marginLeft: 105,
  marginBottom: -10,
  x: {
    domain: [25000, 145000],
    label: null, 
    axis: "top", 
    grid: true,
    color: "#A9A9A9",
    tickPadding: 5,
  },
  color: {
    scheme: "category10",
    type: "categorical"
  },
  marks: [
    Plot.axisY({
      labelAnchor: "center", 
      tickFormat: (d) => "Our candidate",
      style: {fontSize: 20}
    }),
    beeswarmX(results, {
      filter: (d) => d.we_won == "Yes",
      x: "median_household_income", 
      y: 0,
      stroke: "borough",
      strokeWidth: 2,
      r: 6,
      tip: true,
      title: (d) => 'District - ' + d.BoroCD + " (" + d.borough +  ")" + '\n' + 'Median household income - ' + '$' + (d.median_household_income/1000).toFixed(1) + "k"
    }),


  ]
})
```

  </div>
  <div style='margin-top: -32px'>

```js
Plot.plot({
  width: 755,
  height: 100,
  marginLeft: 105,
  marginTop: -1,
  insetTop: -5,
  x: {
    domain: [25000, 145000],
    label: null, 
    axis: "top", 
    grid: true
  },
  color: {
    scheme: "category10",
    type: "categorical"
  },
  marks: [
    Plot.axisY({
      labelAnchor: "center", 
      tickFormat: (d) => "Opponent",
      style: {fontSize: 20}
    }),
    beeswarmX(results, {
    filter: (d) => d.we_won == "No",
    x: "median_household_income", 
    y: 0,
    stroke: "borough",
    strokeWidth: 2,
    r: 6,
    tip: true,
    title: (d) => 'District - ' + d.BoroCD + " (" + d.borough +  ")" + '\n' + 'Median household income - ' + '$' + (d.median_household_income/1000).toFixed(1) + "k"
  })
  ]
})
```
  </div>
</div> 

That's a pretty clean split. This also strengthens the hypothesis - our message resonated more with low-income voters who consituted a most of our voters. Low-income voters prefered out candidate across boroughs - even districts won by us in Manhattan, have lower median income. At the same time even though we were able to pull of a few medium income districts, the most of them went for the opponent. Why is that? Was something wrong with our campaign effors? Let's see.


<h2>Wrong focus</h2>

The final step in taking the helicopter view at the campaign (that we can take without detailed look into the crosstabs) is to look how we campaigned and whether it paid off or not.

To do that, we selected all districts lost by our candidate and a few comparable ones where we won. For each district we mapped their median housegold income level and data about campaign efforts - door knocking, how much hours our candidate personally spent in the district and campaign events. Districts are sorted by the win/loss gap between two candidates.

<!-- Section: Campaign gaps -->
<!-- Chart: dumbbell chart -->

```js
const focus_chart_domain = [405, 404, 408, 103, 501, 314, 107, 208, 108, 502, 407, 402, 401, 307, 503, 102, 406, 105, 410, 306, 413, 305, 207, 311, 403, 301]

const chart_height_main = 570
const chart_height_two = 595
const chart_height_three = 545
```

<div class='chart-container horizontal-chart-container' style="width: 120%; max-height: 640px; margin-top: 30px;">
  <div>
    <p class='chart-subtitle' style='margin-left: 60px; margin-bottom: 25px;'>Income<br> Level, $</p>

  ```js
  Plot.plot({
    height: chart_height_main,
    width: 130,
    marginTop: 11,
    insetRight: 20,
    insetLeft: 20,
    grid: true,
    y: {
      type: "point",
      label: null,
      domain: focus_chart_domain,
      stroke: "gray",
    },
    marks: [
      Plot.text(results, {
        frameAnchor: "left",
        fontSize: 12,
        text: d => "💵".repeat(d.income_level),
        dx: 20,
        y: "BoroCD"
      }),
      Plot.axisY({
        fontSize: 13,
        fontWeight: 700,
        textAnchor: "end",
        fill: "#6A6A6A"
      })
    ]
  })
  ```
  </div>
  <div>
    <p class='chart-subtitle'>Win/Lose gap, %</p>

  ```js
  Plot.plot({
    height: chart_height_main,
    width: 155,
    marginTop: 23,
    marginLeft: 0,
    marginRight: 0,
    insetLeft: 10,
    insetRight: 30,
    insetTop: -6,
    x: {
      percent: 'true',
      label: null,
      axis: "top",
      domain: [-50, 50],
      grid: true,
    },
    y: {
      grid: true,
      stroke: "gray",
      label: null,
      labelAnchor: "top",
      domain: focus_chart_domain
    },
    color: {
      range: ["#FF8284", "#ACFF82"],
  },
    style: {
      fontSize: 11
    },
    marks: [
      Plot.barX(results, {
        y: "BoroCD",
        x: d => d.pct_diff,
        fill: d => Math.sign(d.pct_diff),
        tip: true
    })
    ]
  })
  ```
  </div>
  <div>
    <p class='chart-subtitle chart-subtitle-two-lines'><span class='chart-title-emoji'>🚪</span><br>Doors<br>knocked</p>

  ```js
  Plot.plot({
    height: chart_height_two,
    width: 73,
    marginTop: -10,
    marginBottom: 60,
    marginLeft: 0,
    insetLeft: -50,
    x: {
      axis: null,
      label: ""
    },
    y: {
      label: null,
      domain: focus_chart_domain,
      grid: true,
      stroke: "gray",
      axis: null
    },
    color: {
      scheme: "blues",
      domain: [-10000, 10000]
    },
    style: {
      overflow: "visible"
    },
    marks: [
      Plot.tip(results, Plot.pointer({
        fontSize: 15,
        title: (d) => (d.gotv_doors_knocked/1000).toFixed(1) + "k doors knocked on.",
        dx: -25,
        y: "BoroCD",
        fill: "white"
      })),
      Plot.dot(results, {
        y: "BoroCD",
        x: 0,
        symbol: "square",
        fill: "steelblue",
        opacity: 0.5,
        stroke: "black",
        strokeWidth: 0.5,
        r: "gotv_doors_knocked"
      })
    ]
  })
  ```
  </div>
  <div>
    <p class='chart-subtitle  chart-subtitle-two-lines'><span class='chart-title-emoji'>🕖</span><br>Hours<br>spend</p>

  ```js
  Plot.plot({
    height: chart_height_two,
    width: 80,
    marginTop: -10,
    marginBottom: 60,
    marginLeft: 0,
    insetLeft: -50,
    x: {
      axis: null
    },
    y: {
      label: null,
      domain: focus_chart_domain,
      grid: true,
      axis: null,
      stroke: "gray"
    },
    color: {
      domain: [-50, 30]
    },
    style: {
      overflow: "visible"
    },
    marks: [
      Plot.tip(results, Plot.pointer({
        fontSize: 15,
        title: (d) => d.candidate_hours_spent + " hours spent by our candidate campaigning in the district.",
        y: "BoroCD",
        dx: -25,
        fill: "white"
      })),
      Plot.dot(results, {
        y: "BoroCD",
        x: 0,
        symbol: "square",
        fill: "purple",
        opacity: 0.4,
        stroke: "black",
        strokeWidth: 0.5,
        r: "candidate_hours_spent"
      })
    ]
  })
  ```
  </div>
  <div>
    <p class='chart-subtitle' style="margin-bottom: -4px;"><span class='chart-title-emoji'>💬</span><br>Campaign events</p>

  ```js
  Plot.plot({
    height: chart_height_three,
    width: 350,
    marginTop: 25,
    marginRight: 0,
    marginLeft: 0,
    insetLeft: 10,
    insetRight: 10,
    insetTop: -2,
    y: {
      axis: null,
      grid: true,
      stroke: "gray",
      label: null,
      labelAnchor: "top",
      domain: focus_chart_domain
    },
    style: {
      fontSize: 14
    },
    marks: [
      Plot.axisX({
        fontSize: "8pt",
        anchor: "top",
        label: null,
        tickPadding: 3,
        ticks: "month",
        tickFormat: "%b"
      }),
      Plot.dot(events, {
        y: "boro_cd",
        x: "event_date",
        r: d => d.estimated_attendance * 0.06,
        fill: "orange",
        opacity: 0.55,
        stroke: 'black',
        strokeWidth: 0.5,
        tip: true 
      })
    ], 
    r: { type: "identity"}
  })
  ```
  </div>
</div>

Comparing districts, a few reasons for our loss become clear. 

First, reaching out pays off. We won all low-income districts and in all of them we've knocked on a lot more doors than in districts in other income brackets - 7-14 times more in most cases. This trends continues with middle income districts - the ones we won had twice as many doors we knocked on compared to the ones we lost. Those numbers are still miles away from door knocking effors in low-income districts but at the same time, maybe because of that, our victory margins in those districts are also small.

Second, it's more of a hunch that requires additional research but - more intense door knocking specifically in low-income districts seems to work. Low-income districts have the highest volumes of door knocking but the margins of victory are not just not astonishing. Moreover, they are lower than in middle-income districts with fractions of door-knocking efforts. The hunch is that low-income districts may have more low-propensity voters who may naturally face more hurdles in life due to their lack of financial stability and thus may need more outreach from the campaign to learn about the candidate, the platform and how voting this candidate in can help them. This hunch, though requiring additional research to confirm, is supported by some survey responses. For example, a voter who supported our candidate but from the district with a low turnout (43%) that we lost by just 1k votes, said: "I think I got one mailer but didn't pay attention. Busy with work and kids". 

Third, the campaign team misdirected their efforts by focusing that much more on high-income districts. Both in terms of hours personally spent by the candidate and the number of events, high-income districts are way ahead even of low-income ones that we swept. Despite this enhanced efforts, we lost each and every high-income district by the biggest margins. At the same time high-income districts are among ones with least door knocks, which may be part of the reason for losing them that could've been justified, if we focused more on reaching out to voters in middle-income districts. This idea is also reflected in the survey. A potential voter, who ended up skipping this election in the middle-income district 307 (lost by 3.5k votes with 49% turnout), echoes this sentiment: "I wish I knew more about the candidates. Feel like I didn't hear much about what they wanted to do."

<!-- Section: recommendations -->

<h2>Start, stop, continue: re-do advice</h2>


All of these findings suggest that, without looking deeper into the platform and communication, the campaign should've focus on bringing in more voters in middle-income districts with which we already had some pull hence we won some of them. They should've been the priority districts alongside with voters in the low-income ones - pulling just 15-20k votes away from the opponents would be enough. Additionally, door knocking and candidate apperances in priority disticts is a must.

In particular:

1. Figure out who are the primary voters and check that the platform and messaging are aligned with them.
2. Figure out what disticts we need to turnout and bring in to win the election. If the platform and messaging stays mostly the same, it makes sense to prioritise low- and middle-income districts since seemingly it resonated with people.
3. Prioritise those districts in our campaigning and make sure that we do enough and a little more door knocking, candidate appearances and events. 


