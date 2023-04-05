---
layout: post
title: "Am I Close Geolocated Reminder App"
date: 2021-11-02
---

Disclaimer: Several iOS updates later and this app is no longer working as intended. I have not had the time to update it, but if you are interested in any aspect of it, do send me an email with your questions.

## Introduction

Am I Close is an iPhone and iPad app I recently developed and published at the Apple App Store. The app sends out a sound notification to the user whenever they are near a shop/location they want to visit. If "Vibrate on Sound" is turned on in Sound and Haptics settings, the phone will vibrate with the notification.

I was motivated to develop the app because I would sometimes forget to buy something I needed despite passing by the store that sells it. Am I Close solves this problem by giving its users an option to create a reminder with a list of items they wish to purchase whenever they remember they need something. Chosen shop or shop locations are also added when creating the reminder. When a user passes next to a store while out on a walk or on some other errand, the reminder issues them a notification to alert them to the fact that now is a good opportunity to make their desired purchases.



## Functionality walk-through

Here is a step by step walk-through of how it works:

When you tap on the app icon ![icon](https://raw.githubusercontent.com/ana0209/ana0209.github.io/master/images/AppIcon.appiconset/29.png) for the first time you are met with a clean interface and an empty list of reminders:


<a id="reminders_list_empty">
<p align="center">
  <img alt="1_png" src="https://raw.githubusercontent.com/ana0209/ana0209.github.io/master/images/iphone-6_5inch/1.png" width="250">
   <br>
   <em>Empty reminder list</em>
</p>


Tapping on the plus sign in the top right corner of the screen directs you to a view where you can add a reminder. You are presented with a map and a search bar:

<a id="add_reminder">
<p align="center">
<kbd>
  <img alt="2_png" src="https://raw.githubusercontent.com/ana0209/ana0209.github.io/master/images/iphone-6_5inch/2.png" width="250" />
</kbd>
    <br>
    <em>Add reminder view</em>
</p> 


Once you enter the name of the shop in the search bar and press the search button, you will see the shop indicated on the map. Sometimes a shop has multiple locations. You can select which locations you are interested in by tapping on the pins corresponding to the shop locations. Grey pins represent the selected locations:

<a id="select_location">
<p align="center">
<kbd>
  <img alt="3_png" src="https://raw.githubusercontent.com/ana0209/ana0209.github.io/master/images/iphone-6_5inch/3.png" width="250" />
</kbd>
    <br>
    <em>Selecting locations of interest</em>
</p> 

By tapping on the items field you are redirected to the items view where you can create your shopping list:

<a id="items">
<p align="center">
<kbd>
  <img alt="4_png" src="https://raw.githubusercontent.com/ana0209/ana0209.github.io/master/images/iphone-6_5inch/4.png" width="250" />
</kbd>
    <br>
    <em>Adding items</em>
</p> 


Finally you can go back and save your reminder. Your list is now ready:

<a id="non_empty_list">
<p align="center">
<kbd>
  <img alt="6_png" src="https://raw.githubusercontent.com/ana0209/ana0209.github.io/master/images/iphone-6_5inch/6.png" width="250" />
</kbd>
    <br>
    <em>List of entered reminders</em>
</p> 


Once you are near a shop, the phone will issue a sound notification. If you have "Vibrate on Ring" turned on in Sounds and Haptics, the phone will vibrate as well:

<a id="notification">
<p align="center">
<kbd>
  <img alt="7_png" src="https://raw.githubusercontent.com/ana0209/ana0209.github.io/master/images/iphone-6_5inch/7.png" width="250" />
</kbd>
    <br>
    <em>Notification</em>
</p> 


Tapping on the notification directs you to the list of nearby shops where you can ask for directions to a nearby location, mark a reminder as completed or snooze it:

<a id="nearby_shops">
<p align="center">
<kbd>
  <img alt="8_png" src="https://raw.githubusercontent.com/ana0209/ana0209.github.io/master/images/iphone-6_5inch/8.png" width="250" />
</kbd>
    <br>
    <em>Nearby shops</em>
</p> 


You snooze a reminder by selecting one of the snooze duration options:

<a id="nearby_shops">
<p align="center">
<kbd>
  <img alt="8_png" src="https://raw.githubusercontent.com/ana0209/ana0209.github.io/master/images/iphone-6_5inch/snooze-options.png" width="250" />
</kbd>
    <br>
    <em>Nearby shops</em>
</p> 

You can restart a snoozed reminder at any time by pressing the play button next to it in the list of snoozed reminders:

<a id="nearby_shops">
<p align="center">
<kbd>
  <img alt="snoozed" src="https://raw.githubusercontent.com/ana0209/ana0209.github.io/master/images/iphone-6_5inch/snoozed.png" width="250" />
</kbd>
    <br>
    <em>Snoozed reminders</em>
</p>


That is it! Hopefully the app helps you pick up your shopping in a convenient fashion and run your errands more efficiently!


## Implementation

The app uses Apple location services. It mostly relies on Apple's [region monitoring](https://developer.apple.com/documentation/corelocation/monitoring_the_user_s_proximity_to_geographic_regions). However, in some cases, when there are a lot of nearby shops, due to the limit on the number of regions that a single app is allowed to use, the app falls back on [user location tracking](https://developer.apple.com/documentation/corelocation/cllocationmanager/1423750-startupdatinglocation). 

The app issues a notification when you are 100-200 meters from the shop. Once the shop is added to the list of the nearby shops it may stay there until you are further away from the shop than you were when the notification was issues. This is due to the way Apple implements region monitoring that is used in the app. In simulation, I found that in most cases the shop was removed from the Nearby Shops list once the user was outside of the region and about 150m away from the region border. 


