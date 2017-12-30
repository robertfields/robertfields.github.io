---
layout: page
title: Robert Fields
permalink: airbnbtableau/
---
```css
.modalDialog {
	position: fixed;
	font-family: Arial, Helvetica, sans-serif;
	top: 0;
	right: 0;
	bottom: 0;
	left: 0;
	background: rgba(0,0,0,0.8);
	z-index: 99999;
	opacity:0;
	-webkit-transition: opacity 400ms ease-in;
	-moz-transition: opacity 400ms ease-in;
	transition: opacity 400ms ease-in;
	pointer-events: none;
}

.modalDialog:target {
	opacity:1;
	pointer-events: auto;
}

.modalDialog > div {
	width: 400px;
	position: relative;
	margin: 10% auto;
	padding: 5px 20px 13px 20px;
	border-radius: 10px;
	background: #fff;
	background: -moz-linear-gradient(#fff, #999);
	background: -webkit-linear-gradient(#fff, #999);
	background: -o-linear-gradient(#fff, #999);
}

.close {
	background: #606061;
	color: #FFFFFF;
	line-height: 25px;
	position: absolute;
	right: -12px;
	text-align: center;
	top: -10px;
	width: 24px;
	text-decoration: none;
	font-weight: bold;
	-webkit-border-radius: 12px;
	-moz-border-radius: 12px;
	border-radius: 12px;
	-moz-box-shadow: 1px 1px 3px #000;
	-webkit-box-shadow: 1px 1px 3px #000;
	box-shadow: 1px 1px 3px #000;
}

.close:hover { background: #00d9ff; }
```

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
