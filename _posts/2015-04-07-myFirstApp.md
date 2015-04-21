---
layout: post
title: "My First App"
date: 2015-04-07

---


#Background
   <p> About two months ago I started my foray into Java Development. Things went mostly smoothly, the biggest challenge I had was actually just getting Android Studio running, after my 10th complete uninstall/reinstall of Java and Studio, I was in business.
	 I surprised myself as to how quick I seem to have taken to Android development. Within a week I was able to apply my, then still, beginner-level Java knowledge and Android development experience into a practical app for work.  </p>
<p>The app solved a serious annoyance at my job. several times during the course of a shift I need to access some information (usually pricing). My company does provide a tool to access said information, however getting access to the information is painfully slow.    I should clarify, the tool itself isn’t slow, but it does require me to log into 4 different systems before I can even access it. If I could remake the tool as an app I could solve my problem. </p>
<p> Conceptually the tool isn’t that complicated; it pulls information from four different companies and formats the information into an easy to read table. And because the information the tool uses  is publically available from each of the respective company’s website, I figured my app could pull the information from the companies’ respective websites, allowing me to access the information I need without having to login to any systems in my company. </p>
<p>The first version of my app was functional, but not much else (almost all of the data was “hand scraped” and hard coded). I was still learning a lot about Java itself and Android Development, even as I developed the app. As I got close to finishing the app it was very tempting to scrap my earlier code, very “primitive” code and replace it with code much neater stuff. </p>

<p>Eventually school starter picking up in terms of work-load and I wasn’t able to finish the app to my liking, but I never forgot about it since I stopped working on it mid-February.  I knew I’d come back and re-do the entire thing, this time I’d be a lot more prepared.</p>

<p> Since I left the app I’ve come up with several things I’d like to do to improve it. Priority Number one being some-sort of automation in collecting the information displayed in the app. I also want to redesign the app from a visual and functional stand point so that the app flows better, scales better among different devices and improve the overall usability.  </p>
<p>Automating the information collection process is my first priority and it’s what my next blog post will be about. 
I've got a couple screenshots of one of the activities from my original app to show what the old App looked like.</p>

![img1](/images/appCalculatorSpin.png)

This is what one of the activities looks like. It's a mess. The two spinners are actually a custom 'mySpinner' class that inherit the Spinner class. 
There is a little more to it, but basically the MySpinner overrides the setSelection method of the Spinner class. This allows me/user to select the same item twice, which is useful incase you're buying multiples of an item.

![img2](/images/appCalculator.png)

Unfortunately with the My Spinner class not displaying visually meant spot-checking the alignment of objects visually was rather difficult. 
