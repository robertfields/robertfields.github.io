---
layout: page
title: Robert Fields
permalink: airbnbtableau/
---

# Dublin Airbnb Market Tableau Dashboard

This is a dashboard I created for a Data Visualization class in the Wake Forest MSBA program. It uses data pulled from Airbnb listings in Dublin, Ireland from the [Inside Airbnb](http://insideairbnb.com/get-the-data.html) website.

<a href="#openModal">
  <img src= "/public/Dublin Airbnb Dashboard.png" alt= "dublindash" align= "center">
</a>

<div id="openModal" class="modalDialog">
  <div>
  		<a href="#close" title="Close" class="close">X</a>
      <iframe src="https://public.tableau.com/views/FinalExam_R_Fields/CompetitiveLandscape?:embed=y&:display_count=yes">
      </iframe>
  </div>
</div>


For this assignment, I was to construct two dynamic dashboards, one to evaluate the competitive landscape and another to be used as a pricing tool. It shows the user information for listings across the city, filtered by type of room, neighborhood, property type, accomodation, etc.

The competitive landscape tab evaluates how accomodation or distance from a central location affects listing price. The price comparison tool allows the user to compare a listing price against all other listings based on accomodation, distance from city/neighborhood center, and review scores. A simple transformation was made to calculate the center of listings' latitude and longitude, which can be seen in the packaged workbook download.
