---
title: Bus Tracker
draft: false
tags:
  - system
  - design
  - python
  - django
  - react
  - restapi
  - fullstack
---

# [Repository](https://github.com/miabobia/CalgaryBusWatcher/tree/django-main)

# What is this Project?
This project is a web app that will display the live coordinates of buses around the City of Calgary on demand. Ideally you can request a route or bus name and be shown a live map view of the buses location.

# Why?
This idea has been implemented many times. I've seen this done in Calgary already, but I wanted to make a more concise version that can run on a web page without needing a login or any unnecessary fluff.
# Technologies  Being Used
- Python
- Django
- React

# How?
## Backend
The backend is using Django as a framework and is seperating into two apps
### Apps:
- core_transit_app
	- models defined here
	- djangorestapi lives here
	- rest api lives here to be consumed by front end
- data_collection_app
	- runs background data collection tasks through cron jobs
	- translate data from City of Calgary's data portal into csv files
	- does version control checks when files are downloaded from data portal
	- updates database based on csv's

## Frontend
React based front tend that has map component and consumes rest api from backend


# Screenshots
![[api_index.png|250]]
![[api_bus.png|250]]
![[api_trip.png|250]]
![[api_route.png|250]]