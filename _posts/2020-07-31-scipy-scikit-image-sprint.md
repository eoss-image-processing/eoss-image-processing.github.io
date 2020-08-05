---
layout: post
title:  "scikit-image sprint at Scipy "
author:	Emmanuelle Gouillart
---

![group picture of sprint attendees - including toddlers!](/assets/sprint_scikit_image.png)

On July 18-19, we've had our first all-remote scikit-image sprint, as part of
the scipy conferences. Of course we missed seeing old friends and meeting new
ones around the Scipy BBQ, but this remote sprint has been a great experience
and brought many useful contributions to the project. Here are a few take-aways
from this sprint.

- All-remote means that all participants are on an equal footing: when
  sprinting at a conference, core devs tend to pay more attention to people in
  the same room than to people who are sprinting remotely, or to people they
  know better / who are more self-confident to ask questions, etc. Here all
  conversations were taking place on [our Zulip chat forum](https://skimage.zulipchat.com/#narrow/stream/176555-sprint), 
  which made conversations more "distributed", with everybody being able to
  ask questions or chime in on specific topics.

- As usual, preparation is key. It was definitely useful to
  go through our list on open issues before the sprint and label some as "good
first issue" or "sprint". By "good first issue" we often think of issues
adapted to contributors who are starting to contribute to open source in
general. In a sprint, however, first-time contributors can already have a lot
of experience in other domains, and it's worth labeling a few issues for this
type of contributors. This is how we've been lucky to have a very experienced
developer and new contributor writing our first Github action for an improved deployment of the
documentation!

- Regarding tools, zulip is very nice for chatting with its topic feature. We
  missed a bit not having a video platform with the possibility to create
breakout rooms (although it was nice to have a common Google Meet room to be
able to see other sprinters!). Other projects have had a great experience with
the [discord platform](https://discord.com/new) for sprinting, we'll probably
give it a try at the next sprint. 

- Even if you still need to make adjustments when sprinting on weekends, a
  remote sprint is more compatible with family-life as an in-person one. As you
  can see from our group picture, we even had three toddlers enjoying the
  sprint for a little while! 

Here is a short summary of what we've been doing during the sprint and as
follow-up work. These improvements will be available in our 0.18 release, or
right now if you install the dev version.

- new biology and medical-imaging datasets have been [added to our data
  repo](https://gitlab.com/scikit-image/data/-/merge_requests/6), opening the door to new tutorials for life science users.
- we have added our first github action workflow for an improved deployment of
  the development documentation ([#4843](https://github.com/scikit-image/scikit-image/pull/4843) and [#4852](https://github.com/scikit-image/scikit-image/pull/4852))
- bug fixes for the ransac algorithm ([4844](https://github.com/scikit-image/scikit-image/pull/4844)) and color string mapping in `label2rgb` ([#4840](https://github.com/scikit-image/scikit-image/pull/4840))
- automatic formatting of docstrings for improved consistency ([#4849](https://github.com/scikit-image/scikit-image/pull/4849))
- improved docstring for `rgb2lab` ([#4839](https://github.com/scikit-image/scikit-image/pull/4839)) and `marching_cubes` [#4846](https://github.com/scikit-image/scikit-image/pull/4846)
- improved consistency of `structure_tensor` with the rest of the code base ([#4841](https://github.com/scikit-image/scikit-image/pull/4841)) 
- tutorial on [visualizing 3D data](https://scikit-image.org/docs/dev/auto_examples/applications/plot_3d_image_processing.html) ([#4850](https://github.com/scikit-image/scikit-image/pull/4850))
- default value for `level` parameter in `find_contours` ([#4862](https://github.com/scikit-image/scikit-image/pull/4862))
- ongoing work on a boundary tracing algorithm ([#4853](https://github.com/scikit-image/scikit-image/pull/4853)) and a faster convex-hull algorithm


Many thanks to all of our sprinters!
- Alex de Siqueira
- Clement Ng
- Corey Harris
- Emmanuelle Gouillart
- Felipe Rodrigues
- Gregory Lee
- Joris Vankerschaver
- Juan Nunez-Iglesias
- Lars Grüter
- Louis Maddox
- Marianne Corvellec
- Mathias Buissonier
- Ruby Werman
- Stéfan van der Walt
- Volker Hilsenstein
- Wendy Mak
- Yogendra (@cedarfall)

As usual, a sprint is just how the story begins ;-)... we hope to see new contributors again on Github, and maybe become maintainers in the future!


