---
title: Southwest Gave Me The Ability To Ruin Someone's Day
summary: On July 21st, I was one of the many frustrated flyers across the nation to awake and see that my Southwest flight had been cancelled due to the massive outage. I called SW and got my flight refunded (very easily actually, the support rep was very helpful). I rebooked on Delta and thought nothing else of it. About 3 hours later, I received an email itinerary for another SW flyer on a return trip flying out the same day.
---

On July 21st, I was one of the many frustrated flyers across the nation to awake and see that my Southwest flight had been cancelled due to the massive outage. I called SW and got my flight refunded (very easily actually, the support rep was very helpful). I rebooked on Delta and thought nothing else of it.

About 3 hours later, I received an email itinerary for another SW flyer on a return trip flying out the same day. Everything about it seemed a bit odd: we shared no similarities in name or flight details, we weren’t even in the same part of the country.

![Flight Itinerary]({{ site.baseurl }}/img/southwest/sw1.jpg)
<small>The random person's itinerary that ended up in my inbox</small>

Now, I’m by no means a security expert, and it’s worth noting up front that this seems to be isolated only to those that inadvertently receive flight itineraries other than their own. Even though we’re not talking about a root access-like issue here, I felt the need to dig deeper because I accidentally received a fairly large amount of someone else’s personal data, and in a perfect world, I’d like to know if my data was compromised in this fashion. 

Armed with the flyer’s name, ticket number, confirmation ID, and rapid rewards number, I started looking for ways that a nefarious actor might use such information. The above details would be plenty for a social engineering attack on SW customer support to retrieve the victims’s email address, but the password would still be difficult to obtain. So full account access isn’t quite achievable just from the itinerary.  That doesn’t mean you can’t ruin someone’s day with this information though, so I hit the “Change Flight” button:

![Change Flight Button]({{ site.baseurl }}/img/southwest/sw2.jpg)

which opened up this authentication window in the browser:

![Login Page]({{ site.baseurl }}/img/southwest/sw3.jpg)

Confirmation number? Check. Passenger name? Check. We’re given everything we need to modify the flight reservation IN the itinerary email, no extraneous information is needed. So let’s see how far we can get with this information.

![Date Picker View]({{ site.baseurl }}/img/southwest/sw5.jpg)

We’re directed to another view with a date picker, but perhaps the most interesting bit of this is the yellow warning box above it:

![Yellow Warning Box]({{ site.baseurl }}/img/southwest/sw6.jpg)
<small>Weather, right...</small>

This is the other weird little caveat of this situation. Because of the massive outage, Southwest allowed flyers to reschedule flights free of charge within certain parameters. Because of this, we get to bypass payment authentication if our new flight falls within these restrictions. This gives us 3 airports and two weeks of departure dates to work with without having to authenticate at all. Let’s pick a date and continue.

![Flight View]({{ site.baseurl }}/img/southwest/sw7.jpg)

The next view gives us our flight options, so we can decide if this person likes to get up early or catch the late flight. Early bird it is!

![Confirmation Window]({{ site.baseurl }}/img/southwest/sw10.jpg)

So this is where my little experiment ends. I didn’t want to risk actually messing with someone’s flight for the sake of researching this problem, but based on the fact that this form `POST`s to the `cancelBoardingPassesForChangeItinerary.html` action, I think it’s fair to assume that unauthorized flight record modification is possible with just an email itinerary, at least in this situation.

### Disclosure and Follow-up
It took a couple of phone calls and emails with customer service in order to reach the IT side, and their response indicated that this was an isolated incident. To be completely honest, I didn’t really buy it.

First, my email address inexplicably becoming attached to another SW  account seemed really bizarre. I spoke with the flyer whose itinerary I received and we couldn’t figure out any logical explanation for the email address association from a data perspective. This raised another question that I didn’t get a chance to answer: if my email address was tied to someone else’s account, would I receive a password reset email for that account if I had requested one?

The likelihood that I was the only person intending to fly out on July 21st, the height of the SW outage, that accidentally received someone else’s itinerary can’t be very high. I can’t speculate on what exactly may have happened because I have zero knowledge of their systems, but there’s really nothing here that leads me to believe that I was the only one. Regardless, nobody should ever have the ability to modify someone else's fight so easily.

A few weeks after the Southwest outage, Delta suffered a similar nationwide outage that lasted multiple days. These incidents teach us that it's not only a travel inconvenience when systems go down and flyers lose their flight reservations, we as travelers run the risk of losing control of our personal data because of these legacy systems.
