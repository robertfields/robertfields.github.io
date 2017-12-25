---
layout: page
title: Robert Fields
permalink: airbnbtableau/
---

# Dublin Airbnb Market Tableau Dashboard

This is a dashboard I created for a Data Visualization class in the Wake Forest MSBA program. It uses data pulled from Airbnb listings in Dublin, Ireland from the [Inside Airbnb](http://insideairbnb.com/get-the-data.html) website.

```html
<div class='tableauPlaceholder' id='viz1514184497725' style='position:
relative'><noscript><a
href='https:&#47;&#47;robertfields.github.io&#47;airbnbtableau'><img alt=' '
src='https:&#47;&#47;public.tableau.com&#47;static&#47;images&#47;Fi&#47;FinalExam_R_Fields&#47;CompetitiveLandscape&#47;1_rss.png'
style='border: none'/> </a> </noscript> <object class='tableauViz'
style='display:none;'> <param name='host_url'
value='https%3A%2F%2Fpublic.tableau.com%2F' /> <param name='embed_code_version'
value='3' /> <param name='site_root' value='' /><param name='name'
value='FinalExam_R_Fields&#47;CompetitiveLandscape' /><param name='tabs'
value='yes' /><param name='toolbar' value='yes' /><param name='static_image'
value='https:&#47;&#47;public.tableau.com&#47;static&#47;images&#47;Fi&#47;FinalExam_R_Fields&#47;CompetitiveLandscape&#47;1.png' />
<param name='animate_transition' value='yes' /><param
name='display_static_image' value='yes' /><param name='display_spinner'
value='yes' /><param name='display_overlay' value='yes' /><param
name='display_count' value='yes' /></object></div>                <script
type='text/javascript'>                    var divElement =
document.getElementById('viz1514184497725');                    var vizElement =
divElement.getElementsByTagName('object')[0];
vizElement.style.width='100%';vizElement.style.height=(divElement.offsetWidth*0.75)+'px';
var scriptElement = document.createElement('script');
scriptElement.src = 'https://public.tableau.com/javascripts/api/viz_v1.js';
vizElement.parentNode.insertBefore(scriptElement, vizElement);
</script>
```
For this assignment, I was to construct two dynamic dashboards, one to evaluate the competitive landscape and another to be used as a pricing tool. It shows the user information for listings across the city, filtered by type of room, neighborhood, property type, accomodation, etc.

The competitive landscape tab evaluates how accomodation or distance from a central location affects listing price. The price comparison tool allows the user to compare a listing price against all other listings based on accomodation, distance from city/neighborhood center, and review scores. A simple transformation was made to calculate the center of listings' latitude and longitude, which can be seen in the packaged workbook download.
