---
title: "Lab 4: Clearwater Crisis"
toc: false
---

<style>
  @import url('https://fonts.googleapis.com/css2?family=Cal+Sans&family=DM+Sans:ital,opsz,wght@0,9..40,100..1000;1,9..40,100..1000&family=Inter:ital,opsz,wght@0,14..32,100..900;1,14..32,100..900&family=Stack+Sans+Headline:wght@200..700&display=swap');

  main {
    margin: 0 auto;
  }

  #observablehq-main {
    margin-left: 30px;
  }

  svg {
    font-size: 11pt;
  }


  p, h1, h2, h3, h4 {
    max-width: 70%;
    width: 700px;
  }

  p {
    font-family: "DM Sans", sans-serif;
    font-weight: 400;
    color: rgb(41, 41, 41);
    font-size: 13pt;
    line-height: 1.3em;
    margin-top: 0px;
    margin-bottom: 8px;
  }
  
  h1, h2, h3 {
    font-family: "Cal Sans", sans-serif;
  }

  h1 {
    font-size: 40pt;
    margin-bottom: 8px;

  }
  h2 {
    font-size: 25pt;
    font-weight: 700;
    line-height: 1.1;
    margin-bottom: 13px;
    margin-top: 32px;
  }

  h3 {
    font-size: 22pt;
    font-weight: 300;
    margin-top: 0px;
    margin-bottom: 25px;
    line-height: 1em;
    color: rgb(111, 111, 111);
  }

  #data-summary {
    margin: 15px 0 0 0;
    color: rgb(112, 112, 112);
    font-weight: 600;
  }

  #data-details, #data-summary {
    font-family: "DM Sans", sans-serif;
    width: 60%;
    max-width: 700px;
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
    font-family: "Stack Sans Headline", sans-serif;
    color: #9F9F9F;
    margin-top: 22px;
    margin-bottom: -5px;
  }

  .chart-subtitle {
    font-size: 12pt;
    font-family: "Stack Sans Headline", sans-serif;
    color: #B6B6B6;
    margin-top: 16px;
    margin-bottom: -6px;
    font-weight: 300;
  }

  text {
    font-family: "Stack Sans Headline", sans-serif;
  }

  .custom-pack-tooltip {
    font-size: 20pt;
  }

  .observablehq--block {
    padding: 0 0 7px 0;
  }

</style>

<!-- Import Data -->
```js
const allBreaks = await FileAttachment("data/breaks.csv").csv({ typed: true });
const splitBreaks = await FileAttachment("data/split_breaks.csv").csv({ typed: true });
const breakReasons = await FileAttachment("data/break_reasons.csv").csv({ typed: true });
const songs = await FileAttachment("data/songs_with_breaks.csv").csv({ typed: true });
const breakDurationSplit = await FileAttachment("data/break_duration_split.csv").csv({ typed: true });
const reasonsAgg = await FileAttachment("data/reasons_agg.csv").csv({ typed: true });
const streamgraphData = await FileAttachment("data/filled_streamgraph.csv").csv({ typed: true });
const chartHistory = await FileAttachment("data/chart_history.csv").csv({ typed: true });
const chartDates = await FileAttachment("data/all_chart_dates.csv").csv({ typed: true });
const mileyChart = await FileAttachment("data/miley_full.csv").csv({ typed: true });
```

<h1>Why do they return?</h1><h3>How TV, death and NFL help some songs to return into public eye after decades out</h3>

If you've been tapped in into the TV and movie world, you are familiar with the power a popular show yields over music they decide to feature. Kate Bush's "Running Up That Hill" broke exploded in popularity in 2022 nearly 40 years since release after being featured in the 4th season of "Stranger Things", while "Unchained Melody" by The Righteous Brothers gained enough new listeners after a feature in the 1990 movie "Ghost" for the Californian musical duo to re-release the song half a century after its initial release in 1965.

Movies and TV shows, however, are not the only vehicles that propel previosly charted songs back into the spotlight after decades of being remembered only by biggest fans. For example, "Somebody's Watching Me" by Rockwell jumped back into the charts after 37 years out w/o being a score to something. But why then? Why this and other songs suddenly return into the public eye and at times climb high on charts after years and decades of being forgotten? Let's find out.

To figure this out I needed some source of truth that can at least to some extent reflect popularity of songs at least in the U.S. So I decided to use, probably, the biggest music chart in the world - Billboard Hot 100. Every week it compiles songs that get the most attention from public in a week prior to that to provide the music industry with the data about particular songs sucess and general buying trends. The methodology for including and ranking songs has changed over the years, currently it includes sales (physical and digital), online streaming, and radio airplay in the U.S. A song break in this case would be any period when a song left the chart but then came back and charted at least for one more week.
<details id='data-details'>
  <summary id='data-summary'>Data & Methodology</summary>
  <div style="padding: 5px 0 0px 15px">
  I used this Kaggle dataset that pulls daily updates for this and other key Billboard charts. It has entries dating back to the Hot 100 inception on August 4, 1958. 

  Source: <a href='kaggle.com/datasets/ludmin/billboard'>kaggle.com/datasets/ludmin/billboard</a>

  More on <a href='https://www.billboard.com/billboard-charts-legend/'>what data Billboard used for Hot 100</a>

  To calculate breaks, I extracted all songs that had gaps in their consequtive weeks runs. For example, if a song have been on Hot 100 for 14 weeks straight, then was not on for the week 15 but then returned for the week 16. This would be considered a break.
  
  Based on these consecutive gaps I calculated breaks.

  </div>
</details


<!-- SECTION: Songs & breaks counts -->
<h2>Gimme, gimme more</h2>

First, let's see if songs returning to charts is a prominent thing at all. 

<!-- CHART: All breaks - dot plot -->
<p class='chart-subtitle'>Songs that charted on Billboard Hot 100 with at least one break from August 1958 to May 2026</p>

```js
Plot.plot({
  margin: 5,
  width: 700,
  height: 400,
  x: {
    axis: null
    },
  y: {
    axis: null
    },
  marks: [
    Plot.dot(songs, {
        x: "col", 
        y: "row",
        r: 4,
        strokeWidth: 0.4,
        strokeOpacity: 1,
        fill: '#ff9494',
        fillOpacity: 0.6
        }),
    Plot.dot(songs, Plot.pointer({
      x: "col", 
      y: "row", 
      stroke: "#3f3f3f", 
      r: 8, 
      strokeWidth: 3
      }
      )),
    Plot.tip(songs, Plot.pointer({
      fontSize: 14,
      title: (d) => "'" + d.song + "' by " + d.artist + " spent " + d.total_chart + " week(s) in total on Hot 100." + "\n\n" + "It entered the chart on " + d.first_chart.toLocaleDateString("en-US") + " and charted for the last time on " + d.last_chart.toLocaleDateString("en-US") + "\n\n" + "Its highest position was #" + d.max_position + ".",
      x: 'col',
      y: "row",
      fill: "white"
  })),
  ]
})
```

There are nearly 3000 songs that meet this criteria - 2,837 to be exact. Even though this data covers data for almost seven decades, that's an impressive number signaling that the phenomena is pretty prominent. That's, however, not quite what we need - we need to look at the breaks. Looking just at breaks we see even more impressive picture.

<!-- CHART: All breaks - dot plot -->
<p class='chart-subtitle'>Breaks in charting history of songs on Billboard Hot 100 from August 1958 to May 2026</p>

```js
Plot.plot({
  margin: 5,
  width: 950,
  height: 400,
  x: {
    axis: null
    },
  y: {
    axis: null
    },
  marks: [
    Plot.dot(allBreaks, {
        x: "col", 
        y: "row",
        r: 4,
        strokeWidth: 0.4,
        strokeOpacity: 1,
        fill: 'blue',
        fillOpacity: 0.6
        }),
    Plot.dot(allBreaks, Plot.pointer({
      x: "col", 
      y: "row", 
      stroke: "#3f3f3f", 
      r: 7, 
      strokeWidth: 3
      }
      )),
    Plot.tip(allBreaks, Plot.pointer({
      fontSize: 14,
      title: (d) => "'" + d.Song + "' by " + d.Artist + " took a break from charting for " + d.break_duration_days + " days (" + d.break_duration_weeks + " weeks) from " + d.break_start.toLocaleDateString("en-US") + " to " + d.break_end.toLocaleDateString("en-US"), 
      x: 'col',
      y: "row",
      fill: "white",
  })),
  ]
})
```

Now the number climbs closes to four thousands - 3,767. This means that some songs had multiple breaks. Let's look at how many songs had multiple breaks.

<!-- CHART: Songs w/ multiple entries - dot plot w/ highlights -->
<p class='chart-subtitle'>Multiple charting breaks belonging to same <span style='background-color: rgb(255, 152, 152); color: black;'>songs</span></p>

```js
Plot.plot({
  margin: 5,
  width: 950,
  height: 400,
  x: {axis: null},
  y: {axis: null},
  color: {
    type: "categorical",
    domain: ["No", "Yes"],
    range: ["#6b78ff", "#ff9494"]
    },
  marks: [
    Plot.dot(allBreaks, {
      x: "col", 
      y: "row",
      r: 4,
      strokeWidth: 0.4,
      strokeOpacity: 1,
      fill: 'multi_break',
      fillOpacity: 0.6
      }),
    Plot.dot(allBreaks, Plot.pointer({
      x: "col", 
      y: "row", 
      stroke: "#3f3f3f", 
      r: 7, 
      strokeWidth: 3
      }
      )),
    Plot.tip(allBreaks, Plot.pointer({
      x: 'col',
      y: "row",
      fill: "white",
      fontSize: 14,
      title: (d) => "'" + d.Song + "' by " + d.Artist + " took a break from charting for " + d.break_duration_days + " days (" + d.break_duration_weeks + " weeks) from " + d.break_start.toLocaleDateString("en-US") + " to " + d.break_end.toLocaleDateString("en-US")
  })),
    ]
})
```

20% of songs (578 out 2288) returned to the chart more than once. Those returns, however, take a much higher share out of all returns - out of 3,767 breaks, nearly 40% (1,478) belong songs that had multiple returns. Looking at some examples of such songs provides some context - not all returns were born equally relevant for this analysis. For example, "Jaded" from the Miley Cysus' 2023 album "Endless Summer Vacation" had multiple short returns. It entered the chart on March 22, 2023 for a week but then returned for another week on May 31st, then for another week on June 14th, after which it stayed charting for a few weeks.

<p class='chart-subtitle'>Chart history of the 2023 song "Jaded" by Miley Cyrus</p>

```js
Plot.plot({
  width: 700,
  height: 300,
  marginLeft: 35,
  marginBottom: 36,
  x: {
    grid: true,
    label: null,
    domain: [new Date("2023-03-15"), new Date("2023-09-16")]
    },
  y: {
    domain: [100, 0],
    grid: true,
    label: null,
    ticks: 5
  },
  marks: [
    Plot.lineY(mileyChart, {
      filter: (d) => d.Rank != null || d.Rank != undefined,
      x: "Date", 
      y: 'Rank', 
      curve: 'step',
      strokeWidth: 7,
      stroke: 'grey',
      opacity: 0.2
      }),
    Plot.lineY(mileyChart, {
      x: "Date", 
      y: 'Rank', 
      curve: 'step',
      strokeWidth: 7,
      stroke: 'blue',
      opacity: 0.5
      }),
    Plot.dot(mileyChart, {
      x: "Date", 
      y: 'Rank', 
      r: 6,
      fill: 'blue',
      opacity: 0.8,
      stroke: 'black',
      tip: true,
      title: d => "On " + d.Date.toLocaleDateString("en-US") + " '" + d.Song + "' ranked #" + d.Rank + " on Hot 100"
      }),
    Plot.tip(
      [`On May 16th Miley released a music video for 'Jaded' with is what, likely, brought it back to the chart, taking into account that it has a baked-in two-ish weeks lag.`], 
      {x: new Date("2023-05-31"),
      y: 80,
      lineWidth: 15,
      anchor: "bottom"
      }),
    Plot.axisX({
      fontSize: '11pt',
      fontWeight: 900,
      color: 'black'
    }),
    Plot.axisY({
      fontSize: '10.5pt',
      fontWeight: 300,
      color: '#b0b0b0'
    })
  ]
})
```


Quick look at specific songs suggests two things. First, songs returning into charts is not a new phenomena - there are songs from the 70s and 2026. Second, breaks duration varies significantly even for songs that had multiple returns - from 2 weeks to 30 years. We will look into the second finding deeper later but now let's check out the first one. 

<!-- CHART: Songs w/ breaks by decade - dot column chart -->

<p class='chart-subtitle'>Song breaks grouped by their return to the chart decade, each circle = 10 breaks</p>

```js
Plot.plot({
  width: 720,
  height: 375,
  marginLeft: 50,
  marginTop: 22,
  x: {
    domain: ['50s', '60s', '70s', '80s', '90s', '00s', '10s', '20s'],
    label: null
    },
  y: {
    ticks: 6,
    grid: true,
    label: null},
  marks: [
    Plot.waffleY(allBreaks, 
      Plot.groupX(
        {y: "count"}, 
        {x: "return_decade", 
        opacity: 0.6, 
        unit: 10, 
        rx: "100%",
        fill: 'blue',
        })),
      Plot.ruleY([0]),
      Plot.text(allBreaks, Plot.groupX(
        {y: "count", text: "count"}, 
        {
          x: "return_decade", 
          dy: -17,  
          frameAnchor: "middle",
          fontSize: 14,
          fontWeight: 700,
          fill: '#808080'
        }
      )
    ),
    Plot.axisX({
      fontSize: '11pt',
      fontWeight: 900,
      color: 'black'
    }),
    Plot.axisY({
      ticks: 6,
      fontSize: '10.5pt',
      fontWeight: 300,
      color: '#b0b0b0'
    })
  ]
})
```

Wow, there are definitely significantly more chart returns since 2000s and especially 2010s. For all the time before noticable only the 60s stand out. Whether it was the new technology - like tape recorders in the 60s or TikTok in late 10s/early 20s - something has started pushing more songs to return.


<h2>Things are getting faster (and shorter?)</h2>

<!-- SECTION: Breaks duration -->

Now let's get back to the second finding - break durations varied highly. For the sake of answering the main question while preserving my sanity from gruesome manual data collection for 3500+ entries, let's look at how the breaks duration was distributed in case we can cut some less helpful data.

<!-- CHART: All breaks by duration -->

<p class='chart-subtitle'>Song breaks grouped by break duration</p>

```js
Plot.plot({
  width: 730,
  height: 230,
  marginTop: 0,
  marginLeft: 120,
  x: {ticks: 6, domain: [0, 3600], grid: true, label: null},
  y: {label: null, domain: ['Less than 1 year', '1-5 years', '5-10 years', '10-30 years', '30-60 years']},
  color: {
    domain: ['Less than 1 year', '1-5 years', '5-10 years', '10-30 years', '30-60 years'],
    range: ['grey', 'blue', 'blue', 'blue', 'blue']
    },
  marks: [
    Plot.barX(allBreaks, Plot.groupY(
      {x: "count"}, 
      {y: "break_group", tip: true, fill: 'break_group', opacity: 0.7}
    )),
    Plot.ruleX([0]),
    Plot.axisY({
      fontSize: '11pt',
      fontWeight: 900,
      color: 'black'
    }),
    Plot.axisX({
      fontSize: '10.5pt',
      fontWeight: 300,
      color: '#b0b0b0'
    })
    ]
})
```

Most breaks were shorter than a year. To the point that we cannot even see other groups properly.

<p class='chart-subtitle'>Song breaks longer than 1 year</p>

```js
Plot.plot({
  width: 715,
  height: 184,
  marginTop: 0,
  marginLeft: 95,
    x: {grid: true, label: null, domain: [0, 150], ticks: 4},
    y: {label: null, domain: ['1-5 years', '5-10 years', '10-30 years', '30-60 years']},
    color: {legend: true},
    marks: [
      Plot.barX(allBreaks, Plot.groupY(
        {x: "count"}, 
        {y: "break_group", tip: true, fill: 'blue', opacity: 0.7}
      )),
      Plot.ruleX([0]),
    Plot.axisY({
      fontSize: '11pt',
      fontWeight: 900,
      color: 'black'
    }),
    Plot.axisX({
      ticks: 4,
      fontSize: '10.5pt',
      fontWeight: 300,
      color: '#b0b0b0'
    })
    ]
})
``` 

Now we are talking. Most breaks were still shorter than 5 years, which is would be interesting enough already, but there is also a surprisingly high among of breaks longer that 30 years. I wonder if songs that returned from such long breaks were produced around the times that people usually choose for the collective nostalgia - e.g. 80s in the late 2010s. Let's check this out. Due to the sheer number of short breaks, I "stretched" the bottom of the y-axis so breaks shorter than a year do not overpower the chart.

<p class='chart-subtitle'>Songs break duration over time by decades when they returned</p>

```js
Plot.plot({
  width: 720,
  height: 400,
  marginLeft: 45,
  insetLeft: -10,
  insetRight: -10,
  x: {
    padding: 0.2,
    domain: ['50s', '60s', '70s', '80s', '90s', '00s', '10s', '20s'],
    label: null
    },
  y: {
    type: 'pow',
    exponent: 0.5,
    grid: true,
    ticks: 6,
    label: null
    },
  color: {
    domain: ['Less than 1 year', '1-5 years', '5-10 years', '10-30 years', '30-60 years'],
    range: ['#d3ebff', '#adc0ff', '#6e8afc', '#0700cf', '#250076'],
    legend: true
  },
  marks: [
    Plot.barY(
      allBreaks,
      Plot.groupX(
        {y: "count"},
        {
          x: "return_decade",
          fill: "break_group",
          order: ["30-60 years", "10-30 years","5-10 years" , "1-5 years", "Less than 1 year"], 
          tip: true, 
          opacity: 0.75,
          stroke: 'black',
          strokeWidth: 0.5})),
    Plot.ruleY([0]),
    Plot.axisX({
      fontSize: '12pt',
      fontWeight: 900,
      color: 'black'
    }),
    Plot.axisY({
      ticks: 6,
      fontSize: '10.5pt',
      fontWeight: 300,
      color: '#b0b0b0'
    })
  ]
})
```

As we can see, throughout the decades, there were always old songs released 10-30 years ago that jumped back into the charts. However, since the new millenium with acceleration of everything, as we can see even on this chart, there were not songs that returned from 30+ years long breaks. But what songs are why did they return? Now it's finally time to answer these questions.


<!--- SECTION: Reasons --->

<h2>Death, Super Bowl and ever-changing rules</h2>

<div id='reasons'>
</div>

In the lieu of the main question - why do songs return after long breaks - and my sanity, I focused only on songs that were not charted for at least 5 years. There were 103 songs. For each, based on my research and a lot of inference, I marked the best guess for a reason and sub-reason behind their return. I created two categories to capture more nuance. 

<p class='chart-subtitle' style='margin-bottom: 20px;'>Chart return reasons for songs that were not charting for longer than 10 years; circle size = duration of a particular song break</p>

<!--- CHART: Reasons total grouped: Circle packing --->
```js
const ind_reason_data = {
  "name": "Reason",
  "children": [
    {
      "name": "Artist's Death",
      "children": [
        { "name": "Greatest Love Of All (Whitney Houston)", "value": 25.0 },
        { "name": "I Wanna Dance With Somebody (Who Loves Me) (Whitney Houston)", "value": 24.0 },
        { "name": "I Will Always Love You (Whitney Houston)", "value": 18.0 },
        { "name": "Space Oddity (David Bowie)", "value": 42.0 },
        { "name": "Under Pressure (Queen & David Bowie)", "value": 33.0 },
        { "name": "1999 (Prince)", "value": 17.0 },
        { "name": "Kiss (Prince And The Revolution)", "value": 29.0 },
        { "name": "Let's Go Crazy (Prince And The Revolution)", "value": 31.0 },
        { "name": "Little Red Corvette (Prince)", "value": 32.0 },
        { "name": "Purple Rain (Prince And The Revolution)", "value": 31.0 },
        { "name": "When Doves Cry (Prince)", "value": 31.0 },
        { "name": "I Would Die 4 U (Prince And The Revolution)", "value": 31.0 },
        { "name": "Raspberry Beret (Prince And The Revolution)", "value": 30.0 },
        { "name": "Careless Whisper (Wham! Featuring George Michael)", "value": 31.0 },
        { "name": "Faith (George Michael)", "value": 28.0 },
        { "name": "In The End (Linkin Park)", "value": 15.0 },
        { "name": "Numb (Linkin Park)", "value": 13.0 },
        { "name": "Party Up (Up In Here) (DMX)", "value": 20.0 },
        { "name": "Ruff Ryders' Anthem (DMX)", "value": 21.0 },
        { "name": "X Gon' Give It To Ya (DMX)", "value": 17.0 },
        { "name": "Margaritaville (Jimmy Buffett)", "value": 46.0 },
        { "name": "\"Mama, I'm Coming Home\" (Ozzy Osbourne)", "value": 33.0 }
      ]
    },
    {
      "name": "Rule change",
      "children": [
        {
          "name": "Halloween",
          "children": [
            { "name": "Monster Mash (Bobby \"Boris\" Pickett And The Crypt-Kickers)", "value": 48.0 },
            { "name": "Somebody's Watching Me (Rockwell)", "value": 37.0 },
            { "name": "Thriller (Michael Jackson)", "value": 29.0 }
          ]
        },
        {
          "name": "Christmas",
          "children": [
            { "name": "Mistletoe (Justin Bieber)", "value": 9.0 },
            { "name": "All I Want For Christmas Is You (Mariah Carey)", "value": 12.0 },
            { "name": "Rockin' Around The Christmas Tree (Brenda Lee)", "value": 51.0 },
            { "name": "The Christmas Song (Merry Christmas To You) (Nat \"King\" Cole)", "value": 51.0 },
            { "name": "Jingle Bell Rock (Bobby Helms)", "value": 53.0 },
            { "name": "White Christmas (1947) (Bing Crosby With Ken Darby Singers & John Scott Trotter & His Orchestra)", "value": 56.0 },
            { "name": "Run Rudolph Run (Chuck Berry)", "value": 60.0 },
            { "name": "It's Beginning To Look A Lot Like Christmas (Michael Buble)", "value": 8.0 },
            { "name": "Please Come Home For Christmas (Eagles)", "value": 41.0 }
          ]
        },
        {
          "name": "4th of July",
          "children": [
            { "name": "Party In The U.S.A. (Miley Cyrus)", "value": 13.0 }
          ]
        }
      ]
    },
    {
      "name": "Internet virality",
      "children": [
        {
          "name": "Other",
          "children": [
            { "name": "Billie Jean (Michael Jackson)", "value": 30.0 }
          ]
        },
        {
          "name": "Youtube",
          "children": [
            { "name": "Get Me Bodied (Beyonce)", "value": 6.0 },
            { "name": "Livin' On A Prayer (Bon Jovi)", "value": 26.0 }
          ]
        },
        {
          "name": "Instagram",
          "children": [
            { "name": "My Boo (Ghost Town DJ's)", "value": 19.0 }
          ]
        },
        {
          "name": "TikTok",
          "children": [
            { "name": "Dreams (Fleetwood Mac)", "value": 43.0 },
            { "name": "Sure Thing (Miguel)", "value": 11.0 },
            { "name": "Again (Fetty Wap)", "value": 8.0 },
            { "name": "Lush Life (Zara Larsson)", "value": 9.0 }
          ]
        }
      ]
    },
    {
      "name": "Featured in",
      "children": [
        {
          "name": "Movie",
          "children": [
            { "name": "Ode To Billie Joe (Bobbie Gentry)", "value": 8.0 },
            { "name": "Twist And Shout (The Beatles)", "value": 22.0 },
            { "name": "Stand By Me (Ben E. King)", "value": 25.0 },
            { "name": "Do You Love Me (The Contours)", "value": 25.0 },
            { "name": "Unchained Melody (The Righteous Brothers)", "value": 24.0 },
            { "name": "Bohemian Rhapsody (Queen)", "value": 15.0 },
            { "name": "Tarzan Boy (From \"Teenage Mutant Ninja Turtles III\") (Baltimora)", "value": 6.0 },
            { "name": "Bohemian Rhapsody (Queen)", "value": 26.0 },
            { "name": "Ghostbusters (Ray Parker Jr.)", "value": 37.0 },
            { "name": "Bye Bye Bye (*NSYNC)", "value": 24.0 },
            { "name": "Billie Jean (Michael Jackson)", "value": 11.0 },
            { "name": "Beat It (Michael Jackson)", "value": 42.0 },
            { "name": "Don't Stop 'Til You Get Enough (Michael Jackson)", "value": 46.0 },
            { "name": "Human Nature (Michael Jackson)", "value": 42.0 },
            { "name": "Dirty Diana (Michael Jackson)", "value": 37.0 },
            { "name": "Rock With You (Michael Jackson)", "value": 46.0 }
          ]
        },
        {
          "name": "Ad",
          "children": [
            { "name": "Only Time (Enya)", "value": 11.0 }
          ]
        },
        {
          "name": "TV show",
          "children": [
            { "name": "Running Up That Hill (A Deal With God) (Kate Bush)", "value": 36.0 },
            { "name": "Purple Rain (Prince And The Revolution)", "value": 9.0 }
          ]
        }
      ]
    },
    {
      "name": "Other",
      "children": [
        {
          "name": "Other",
          "children": [
            { "name": "Into The Night (Benny Mardones)", "value": 8.0 },
            { "name": "1999 (Prince)", "value": 15.0 }
          ]
        },
        {
          "name": "Artist's life events",
          "children": [
            { "name": "Trap Queen (Fetty Wap)", "value": 10.0 }
          ]
        }
      ]
    },
    {
      "name": "Performance",
      "children": [
        {
          "name": "Super Bowl",
          "children": [
            { "name": "Get Ur Freak On (Missy \"Misdemeanor\" Elliott)", "value": 13.0 },
            { "name": "Work It (Missy \"Misdemeanor\" Elliott)", "value": 11.0 },
            { "name": "Bad Romance (Lady Gaga)", "value": 6.0 },
            { "name": "Lose Yourself (Eminem)", "value": 18.0 },
            { "name": "Still D.R.E. (Dr. Dre Featuring Snoop Dogg)", "value": 22.0 },
            { "name": "The Next Episode (Dr. Dre Featuring Snoop Dogg)", "value": 21.0 },
            { "name": "Diamonds (Rihanna)", "value": 9.0 },
            { "name": "Umbrella (Rihanna Featuring Jay-Z)", "value": 15.0 },
            { "name": "We Found Love (Rihanna Featuring Calvin Harris)", "value": 10.0 },
            { "name": "Yeah! (Usher Featuring Lil Jon & Ludacris)", "value": 19.0 },
            { "name": "All The Stars (Kendrick Lamar & SZA)", "value": 6.0 },
            { "name": "Humble. (Kendrick Lamar)", "value": 7.0 },
            { "name": "\"Courtesy Of The Red, White And Blue (The Angry American)\" (Toby Keith)", "value": 22.0 },
            { "name": "La Cancion (J Balvin & Bad Bunny)", "value": 6.0 }
          ]
        },
        {
          "name": "Grammy",
          "children": [
            { "name": "Fast Car (Tracy Chapman)", "value": 35.0 }
          ]
        },
        {
          "name": "Coachella",
          "children": [
            { "name": "Baby (Justin Bieber Featuring Ludacris)", "value": 15.0 },
            { "name": "Beauty And A Beat (Justin Bieber Featuring Nicki Minaj)", "value": 13.0 },
            { "name": "Confident (Justin Bieber Featuring Chance the Rapper)", "value": 12.0 }
          ]
        }
      ]
    },
    {
      "name": "Re-release",
      "children": [
        { "name": "Come Rain Or Come Shine (Ray Charles)", "value": 7.0 },
        { "name": "Monster Mash (Bobby \"Boris\" Pickett And The Crypt-Kickers)", "value": 7.0 },
        { "name": "Help The Poor (B.B. King)", "value": 6.0 },
        { "name": "\"They're Coming To Take Me Away, Ha-Haaa!\" (Napoleon XIV)", "value": 7.0 },
        { "name": "Last Kiss (J. Frank Wilson and The Cavaliers)", "value": 9.0 },
        { "name": "Surfin' U.S.A. (The Beach Boys)", "value": 11.0 },
        { "name": "Breaking Up Is Hard To Do (Neil Sedaka)", "value": 13.0 },
        { "name": "Venus (Frankie Avalon)", "value": 16.0 },
        { "name": "Lola (The Kinks)", "value": 9.0 },
        { "name": "Guitar Man (Elvis Presley)", "value": 12.0 },
        { "name": "Solsbury Hill (Peter Gabriel)", "value": 6.0 },
        { "name": "Daydream Believer (The Monkees)", "value": 18.0 },
        { "name": "What About Me (Moving Pictures)", "value": 6.0 },
        { "name": "Fool For Your Loving (Whitesnake)", "value": 9.0 },
        { "name": "I Melt With You (Modern English)", "value": 7.0 },
        { "name": "We Are The Champions (Queen)", "value": 14.0 },
        { "name": "Lights (Journey)", "value": 14.0 },
        { "name": "The Star Spangled Banner (Whitney Houston)", "value": 10.0 },
        { "name": "Blank Space (Taylor Swift)", "value": 8.0 }
      ]
    }
  ]
}

// 2. Horizontal Layout Setup (Widescreen Dimensions)
const width = 3800; 
const height = 1600; 
const format = d3.format(".1f");

// Custom color palette for the 7 top-level rules
const colorScale = d3.scaleOrdinal(d3.schemeObservable10);

// 3. Create the SVG Container
const svg = d3.create("svg")
    .attr("width", width)
    .attr("height", height)
    .attr("viewBox", [20, 20, width, height])
    .attr("style", "width: 95%; height: auto; overflow: visible;");

// 4. Create a Dynamic Floating Tooltip Container
const tooltip = d3.select("#reasons")
  .selectAll(".custom-pack-tooltip")
  .data([null]) 
  .join("div")
    .attr("class", "custom-pack-tooltip")
    .style("position", "absolute")
    .style("visibility", "hidden")
    .style("background-color", "rgba(255, 255, 255, 0.95)") 
    .style("color", "#000000")
    .style("padding", "13px")
    .style("border-radius", "2px")
    .style("font-size", "11pt")
    .style("line-height", "1.4")
    .style("pointer-events", "none")
    .style("box-shadow", "1px 4px 6px 5px rgba(0, 0, 0, 0.1)")
    .style("z-index", "9999");

const mainRules = ind_reason_data.children;
const numRules = mainRules.length;

const packedGroups = mainRules.map((ruleData, index) => {
  const ruleRoot = d3.hierarchy(ruleData)
      .sum(d => d.value)
      .sort((a, b) => b.value - a.value);

  const approxDim = 630; 
  const localPack = d3.pack()
      .size([approxDim, approxDim])
      .padding(10); // Internal gap between sub-rules and songs

  localPack(ruleRoot);

  return {
    ruleData,
    ruleRoot,
    r: ruleRoot.r + 20, 
    targetX: (width / (numRules + 6.5)) * (index + 3.2),
    targetY: height / 2,
    x: (width / (numRules - 40)) * (index - 30), 
    y: height / 2
  };
});

const simulation = d3.forceSimulation(packedGroups)
    .force("x", d3.forceX(d => d.targetX).strength(1)) 
    .force("y", d3.forceY(d => d.targetY).strength(3)) 
    .force("collide", d3.forceCollide(d => d.r + 42)) // Packs the rules tight against each other
    .stop();

for (let i = 0; i < 150; ++i) simulation.tick();

packedGroups.forEach(group => {
  const { ruleRoot, ruleData, x, y } = group;

  // Align internal children nodes based on the final simulation center coordinates
  const xOffset = x - ruleRoot.x + 70;
  const yOffset = y - ruleRoot.y + 30;

  ruleRoot.descendants().forEach(d => {
    d.x += xOffset;
    d.y += yOffset;
  });

  const groupContainer = svg.append("g");

  groupContainer.append("circle")
      .attr("cx", ruleRoot.x)
      .attr("cy", ruleRoot.y)
      .attr("r", ruleRoot.r + 30)
      .attr("fill", "#fbfbff")
      .attr("fill-opacity", 0.5)
      .attr("stroke", "#9ca3af")
      .attr("stroke-width", 1.5)
      .style("pointer-events", "none");

  const subRuleNodes = ruleRoot.descendants().filter(d => d.depth === 1 && d.children);

  groupContainer.append("g")
    .selectAll("circle")
    .data(subRuleNodes)
    .join("circle")
      .attr("cx", d => d.x)
      .attr("cy", d => d.y)
      .attr("r", d => d.r + 5)
      .attr("fill", "#e5e7eb")
      .attr("fill-opacity", 0.5)
      .attr("stroke", "#cfcfcf")
      .attr("stroke-width", 1.5)
      .style("pointer-events", "none");


  const songNodes = ruleRoot.descendants().filter(d => !d.children);

  groupContainer.append("g")
    .selectAll("circle")
    .data(songNodes)
    .join("circle")
      .attr("cx", d => d.x)
      .attr("cy", d => d.y)
      .attr("r", d => d.r - 5)
      .attr('stroke', 'black')
      .attr('stroke-width', 0.5)
      .attr("fill", colorScale(ruleData.name))
      .attr("fill-opacity", 0.75)
      .style("cursor", "pointer")
      
      .on("mouseover", function(event, d) {
        d3.select(this)
          .transition().duration(100)
          .attr("fill-opacity", 1.0)
          .attr("stroke", "#111827")
          .attr("stroke-width", 4);

        tooltip.html(`
          <strong>${d.data.name}</strong><br/>
          <span>Break Duration:</span> <b>${format(d.value)} Years</b>

        `);
        
        return tooltip.style("visibility", "visible");
      })
      
      .on("mousemove", function(event) {
        return tooltip
          .style("top", (event.pageY - 10) + "px")
          .style("left", (event.pageX - 275) + "px");
      })
      
      .on("mouseout", function() {
        d3.select(this)
          .transition().duration(150)
          .attr("fill-opacity", 0.75)
          .attr('stroke', 'black')
          .attr('stroke-width', 0.5)

        return tooltip.style("visibility", "hidden");
      });

  groupContainer.append("g")
    .selectAll("text")
    .data(subRuleNodes)
    .join("text")
      .attr("x", d => d.x)
      .attr("y", d => d.y - d.r - 20)
      .attr("fill", "#4b4b4b")
      .attr("font-size", "30pt")
      .attr("font-weight", "bold")
      .attr("text-anchor", "middle")
      .text(d => d.data.name);

  groupContainer.append("text")
      .attr("x", ruleRoot.x)
      .attr("y", ruleRoot.y - ruleRoot.r - 50)
      .attr("fill", "#1f2937")
      .attr("font-size", "40pt")
      .attr("font-weight", "bold")
      .attr("text-anchor", "middle")
      .text(ruleData.name);
});

// 8. Render directly into layout context
display(svg.node());

```

The final step would be to look at how reasons might have changed over time - did new factors became more prominent?

<p class='chart-subtitle'>Reasons behind returns from longest breaks over time</p>

<!--- CHART: Reasons over time | Stacked dots --->
```js
Plot.plot({
  marginLeft: 30,
  width: 700,
  height: 350,
  x: {
    domain: ['60s', '70s', '80s', '90s', '00s', '10s', '20s'],
    label: null,
    },
  y: {
    ticks: 4,
    grid: true,
    label: null,
    },
  color: {
    domain: ["Artist's death", "Re-release", "Movie/TV show feature", "Performance", "Internet virality", "Rule change", "Other"],
    range: ['#8A44D9', '#D03FF1', '#77DBC3', '#FF77C4', '#FF7281', '#EECA62', '#3CCF81'],
    legend: true,
      },
  marks: [
    Plot.waffleY(breakReasons, 
      Plot.groupX(
        {y: "count"}, 
        {x: "break_decade", 
        opacity: 0.8, 
        gap: 4,
        unit: 1,
        rx: "100%",
        fill: 'Rule',
        tip: true,
        title: d => d.Rule + "\n" + d.Song + d.Artist
        })),
      Plot.ruleY([0]),
    Plot.axisX({
      fontSize: '11pt',
      fontWeight: 900,
      color: 'black'
    }),
    Plot.axisY({
      fontSize: '10.5pt',
      fontWeight: 300,
      color: '#b0b0b0'
    })
        ]
})
```

Hm-m-m, that's interesting. Re-releasing old songs was the main reasons why after a decade or two returned to the chart during the odds in the 60-80s. However, the 80-90s change the game. Being featured in a movie became a real factor. 

But then another 20 years came and went and with 2010s things have changed again. Huge musician from 70-80s like Prince of MJ have started passing away which propelled their hits back into the charts. At the same with more content than even in the 2020s, more songs got a chance to be featured in something big and got the new waves of fans, which is precisely what happened to "Running Up that Hill" by Kate Bush. Another avenue that picked up in the 10s and got even stronger in the 20s - big performances that go viral (for the right or the wrong reasons). That's what happened to both "Fast Car" by Tracy Chapman (GRAMMYs) and three Jubstin Bieber's songs (Coachella).

Finally, the reason that popped out of seemingly nowhere in the 10s and broing back songs that often have been returning to charts every yearc even since is "Rule change". This reasons includes all cases when Billboard changed the Hot 100 methodology to account for some additional data source OR change entry rules for songs despite their previous charting history. This is why since 2012 "All I Want For Christmas Is You" by Mariah Carey has been on Hot 100 every December, and in recent years even in November. It is hard to untanlge what is the chicken and the egg in this case. On one hand, Billboard tries to tune in to the people preference and include songs that are listened to. On the other, getting in the top 5, especially for the 10th time, definitely generate headlines and streams keeping the whole cylce going.