---
#
# Use the widgets beneath and the content will be
# inserted automagically in the webpage. To make
# this work, you have to use › layout: frontpage
#
layout: frontpage
header:
  image_fullwidth: brett-heenan-cover.jpg
widget1:
  title: "Projects"
  url: '/projects/'
  image: code_snippet.jpg.png
  text: "See what I'm currently working on"
widget2:
  title: "Blog"
  url: '/blog/'
  image: notebook.jpg
  text: "Hear about some of the new things I'm learning and exploring" 
widget3:
  title: "About"
  url: '/about/'
  image: brett-card.jpeg
  text: 'find out a little more about me!'
#
# Use the call for action to show a button on the frontpage
#
# To make internal links, just use a permalink like this
# url: /getting-started/
#
# To style the button in different colors, use no value
# to use the main color or success, alert or secondary.
# To change colors see sass/_01_settings_colors.scss
#
#callforaction:
  #url: https://tinyletter.com/feeling-responsive
  #text: Inform me about new updates and features ›
  #style: alert
permalink: /index.html
#
# This is a nasty hack to make the navigation highlight
# this page as active in the topbar navigation
#
homepage: true
---

<div id="videoModal" class="reveal-modal large" data-reveal="">
  <div class="flex-video widescreen vimeo" style="display: block;">
    <iframe width="1280" height="720" src="https://www.youtube.com/embed/3b5zCFSmVvU" frameborder="0" allowfullscreen></iframe>
  </div>
  <a class="close-reveal-modal">&#215;</a>
</div>
