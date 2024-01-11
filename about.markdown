---
layout: page
title: "About"
permalink: /
---

Hi, I'm Augusto, a MsCEng student at a double degree program with CentraleSupélec and the University of São Paulo

I recently finished a software engineering internship at Datadog, focusing on enhancing an APM benchmarking system.

Before that, I was pursuing a specialization on mathematics and data science within CentraleSupélec, where I worked alongside research groups and companies on multiple machine learning projects:
- Denoising of X-ray images with General Electric Healthcare/Laboratoire Signaux et Systèmes.
- Detection and reconstruction of roads from radar satellite images with SONDRA/European Space Agency.
- Speech separation in the context of automatic tracking of speakers with Orange.

My passion for math, statistics, machine learning and computer science was fostered at the University of São Paulo, where I authored an undergraduate research project on deep learning with complex numbers and worked on a SWE internship at BTG Pactual.

I also have competing passions in climbing, lifting, and making music.

You can reach me at [igoragoli@gmail.com](mailto:igoragoli@gmail.com) for my CV.

# Blog

<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>

