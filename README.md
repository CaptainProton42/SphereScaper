---
layout: page
permalink: /
title: Creating an organic grid on a sphere

preview-html:
  assets/demo/index.html
preview-description:
  Interactive grid on a sphere made in the Godot Engine. Left-click to place blocks and right-click to remove them. Click and drag to rotate the sphere. **May take a while to load.**
theme:
  blue-sky
usemath:
  true
disable-navbar:
  true
---

I recently created a tweet about creating an organic looking quad mesh on a sphere and it got rather popular.

<div class="center-align">
<blockquote class="twitter-tweet" data-dnt="true"><p lang="en" dir="ltr">I managed to apply <a href="https://twitter.com/OskSta?ref_src=twsrc%5Etfw">@OskSta</a>&#39;s amazing organic quad grid generation to a sphere! Not sure yet if I should do anything with it 🤔 <a href="https://twitter.com/hashtag/GodotEngine?src=hash&amp;ref_src=twsrc%5Etfw">#GodotEngine</a> <a href="https://t.co/sqA35TQCaf">pic.twitter.com/sqA35TQCaf</a></p>&mdash; CaptainProton42 (@CaptainProton42) <a href="https://twitter.com/CaptainProton42/status/1325752495235330049?ref_src=twsrc%5Etfw">November 9, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>
</div>

Since I got some feedback and people asking me how I achieved this, I decided to give some more context. Don't expect a full-on tutorial here on how to do this in the engine of your choice. Instead, this is supposed to give you some more background and tricks on how to achieve something like this and explain some of the (suprisingly simple) theory. I will attempt to cover the grid generation itself as well as give a brief introduction to marching squares.

As is probably already clear to you, this is directly based on a grid generation algorithm that [Oskar Stålberg](https://twitter.com/osksta) posted on Twitter a while ago and which he uses in his sandbox town building game [Townscaper](https://store.steampowered.com/app/1291340/Townscaper/). He has also written a very in-depth post on the process of building the tech behind Townscaper on [imgur](https://imgur.com/gallery/i364LBr).

<div class="center-align">
<blockquote class="twitter-tweet" data-conversation="none" data-dnt="true"><p lang="en" dir="ltr">I present: <br>Fairly even irregular quads grid in a hex<br><br>Todo: <br>1. Heuristic to reduce (or eliminate) 6 quad verts<br>2. Tile these babies to infinity <a href="https://t.co/o0kU68uovZ">pic.twitter.com/o0kU68uovZ</a></p>&mdash; Oskar Stålberg (@OskSta) <a href="https://twitter.com/OskSta/status/1147881669350891521?ref_src=twsrc%5Etfw">July 7, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>
  </div>

To summarize the algorithm:

1. Start with a triangulation (in this case triangles in a hexagon)
2. Remove random edges triangles to create quads (we will now have a grid consisting of mostly quads but also some triangles)
3. Subdivide *all quads and triangles* into new quads
4. Relax the grid to make it look more natural

So now that we know the basic algorithm for meshes in a plane, what do we need to do to adapt it for a sphere?

The surprisingly simple answer is: *nothing*.

For the slightly more complicated answer, keep reading.

## Grid Generation

### Step 1: Triangulation

The first step of the algorithm requires a triangulation. What is a triangulation? Well, there are many definitions (including but not limited to tracking bad guys in police crima drama series) but in our case we will define it as surface triangulation which, in the words of [Wikipedia](https://en.wikipedia.org/wiki/Surface_triangulation), is "a net of triangles, which covers a given surface partly or totally". In the tweet above we start with the triangulation of a hexagon but we can triangulate any surface, really.

A popular triangulation of the surface of a sphere is the so-called [icosphere](https://en.wikipedia.org/wiki/Geodesic_polyhedron). It is, of course, not the only triangulation but it has some nice properties including the fact that all triangles have the same size.

{{ site.beginInfoBox }}
{{ site.beginInfoBoxTitle }}
Which Software Should You Use?
{{ site.endInfoBoxTitle }}
I am doing all the grid generation in Blender, mainly because Blender's `bmesh` [module](https://docs.blender.org/api/current/bmesh.html) is a breeze to work with and already has most operations (like removing edges, subdividing faces, etc.) defined. Of course, you can do this using whichever software or tool you want and even directly in-engine for grid generation on the fly.
{{ site.endInfoBox }}

Below is an image of our starting icosphere in Blender:

{{ site.beginFigure }}
<img src="assets/icosphere.png" width="50%">
{{ site.beginCaption }}
Our starting triangulation, the icosphere.
{{ site.endCaption }}
{{ site.endFigure }}

### Step 2: Removing Edges

Removing edges is relatively straightforward. We need to
