---
layout: post
title:  "downtoearth: Visualizing Astronomical Scales and More"
# title:  "downtoearth: Visualizing astronomical scales and more"
---

TL;DR -- I made a tool for visualizing stuff in the context of Earth's map. Go straight to **downtoearth**.

<br />

> #### If in a scaled-down universe the Sun had a diameter of 1 cm, what would be the distance to the nearest star?

With this simple thought experiment, I had an incredible insight into the sheer enormity of our universe. Where, indeed, would the nearest star be? A few hundred meters away? Maybe a kilometer or two?

As it turns out, in a scaled universe where the Sun is a mere 1 cm in diameter, our closest star neighbor -- [Proxima Centauri](https://en.wikipedia.org/wiki/Proxima_Centauri) -- would be **approximately 288 km away**. "Hell", I thought, "if this tiny Sun was located in my living room in Rio de Janeiro, Proxima Centauri would be almost as far as São Paulo!".

I certainly cannot wrap my head around the distance of 4.2 light-years, or 40 trillion km, but I do have an inherent feeling for the Rio-São Paulo distance. I was born and raised in Rio. I have seen both Rio and São Paulo represented on a map countless times, from geography classes at school, to weatherman reports on TV and the routine usage of Google Maps. I have personally visited São Paulo quite a few times since a young age, treading the distance by car, bus, and plane.

And there it was, in the context of such mundane path, the sheer enormity of our universe. For a brief moment, I could visualize a tiny 1 cm Sun floating in front of me in the darkness, and far away -- so far away -- Proxima Centauri, our closest neighbor. All in a somewhat comprehensible scale of distance. 

It was such an awe-inspiring moment. And thus the concept of **downtoearth** was born. Scale down the universe, plot it on a map of the Earth, and from the knowledge of local distances get insight into astronomical scales.

![Proxima Centauri from Rio de Janeiro](/assets/images/downtoearth/proxima-centauri-rio-sp-small-framed.png)

## Visualization tool

The Rio-São Paulo distance insight only works for those of us who *know* this distance by heart. If you live in Jacarta or Oslo, my personal insight can be as easily visualized as the unfathomable 40 trillion km figure. In order for this type of visualization to work for any person, it would have to be tailor-made for individual localities and scales. That's how downtoearth turned into a software project.

Initially, I took the idea as an excuse to learn the Vue frontend framework. I did manage to get far with it, but using a framework felt overkill for such a simple project. I also wanted to feel more in control of the tool implementation, and had not ventured into vanilla JavaScript territory for a long time. So downtoearth became a Vanilla JS project (actually TypeScript, since I feel naked without my types).

For mapping I chose the [Mapbox GL JS](https://github.com/mapbox/mapbox-gl-js) library, which is quite simple to work with and has tons of features out of the box. I also used [Turf.js](https://turfjs.org/) for easily calculating shapes, bounding boxes, and distances on the Earth's surface.

Without any framework, I relied on a simple, custom-made store utility that uses [Events](https://developer.mozilla.org/en-US/docs/Web/API/Event) to orchestrate state changes between the application, the UI elements, and the components renderized on the map. I know, not exactly elegant or production-ready, but for a simple frontend project it seems to hold up just fine.

[interface choices]

[Not only astronomical scales, visualize anything.]

[JSON configuration]
