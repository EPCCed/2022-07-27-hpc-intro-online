---
layout: lesson
root: .
permalink: index.html  # Is the only page that doesn't follow the pattern /:path/index.html
venue: Online
address: 
country: "UK"
language: "English"
latlng: 
humandate: 27 July 2022
humantime: 09:30-16:00 BST
startdate: 2022-07-27
instructor: ["Tony Hallam"]
helper: 
email: ["support@archer2.ac.uk"]
collaborative_notes:
---

{% include gh_variables.html %}

This workshop is an introduction to using high-performance computing systems
effectively. We obviously can't cover every case or give an exhaustive course
on parallel programming in just two days of teaching time. Instead, this
workshop is intended to give students a good introduction and overview of the
tools available and how to use them effectively.

By the end of this workshop, students will know how to:

* Identify problems an HPC system can help solve
* Use the UNIX shell (also known as terminal or command line) to operate a computer,
  connect to an HPC system, and write simple shell scripts.
* Submit and manage jobs on an HPC system using a scheduler, transfer files, and use
  software through environment modules.

This lesson is part of a 2-day ARCHER2 course, you can find the material 
for day 1 at [Introduction to the UNIX shell for High Performance Computing](https://epcced.github.io/2022-07-26-hpc-shell-online/).

<h2 id="general">General Information</h2>

{% comment %}
  LOCATION

  This block displays the address and links to maps showing directions
  if the latitude and longitude of the workshop have been set.  You
  can use https://itouchmap.com/latlong.html to find the lat/long of an
  address.
{% endcomment %}
<p id="where">
  <strong>Where:</strong>
  This course will be taught online via Blackboard Collaborate. All attendees will
  be sent the joining link prior to the event.
</p>

{% comment %}
  DATE

  This block displays the date and links to Google Calendar.
{% endcomment %}
{% if page.humandate %}
<p id="when">
  <strong>When:</strong>
  {{page.humandate}}.
  {% include workshop_calendar.html %}
</p>
{% endif %}

{% comment %}
  SPECIAL REQUIREMENTS

  Modify the block below if there are any special requirements.
{% endcomment %}
<p id="requirements">
  <strong>Requirements:</strong> Participants must bring a laptop with a
  Mac, Linux, or Windows operating system (not a tablet, Chromebook, etc.) that they have administrative privileges
  on. They should have a few specific software packages installed (listed
  <a href="setup.html">on the setup page</a>). They are also required to abide by the <a href="https://www.archer2.ac.uk/training/code-of-conduct/">ARCHER2 Training Code of Conduct</a>.
</p>

{% comment %}
  ACCESSIBILITY

  Modify the block below if there are any barriers to accessibility or
  special instructions.
{% endcomment %}
<p id="accessibility">
  <strong>Accessibility:</strong> We are committed to making this workshop
  accessible to everybody.
</p>
<p>
  Materials will be provided in advance of the workshop. If we can help making learning easier for
  you please get in touch (using contact details below) and we will attempt to provide them.
</p>

{% comment %}
  CONTACT EMAIL ADDRESS

  Display the contact email address set in the configuration file.
{% endcomment %}
<p id="contact">
  <strong>Contact</strong>:
  Please email
  {% if page.email %}
    {% for email in page.email %}
      {% if forloop.last and page.email.size > 1 %}
        or
      {% else %}
        {% unless forloop.first %}
        ,
        {% endunless %}
      {% endif %}
      <a href='mailto:{{email}}'>{{email}}</a>
    {% endfor %}
  {% else %}
    to-be-announced
  {% endif %}
  for more information.
</p>

<!-- <h2>Collaborative Document</h2>

During the course, we will make use of a collaborative document known as an *Etherpad*. You
can find the document at:

 - [Course Etherpad]({{page.collaborative_notes}}) -->


<hr/>

> ## Prerequisites
> Command line experience is necessary for this, second day of the workshop. If you did not already have
this experience, you should have gained it from the first day of the workshop.
{: .prereq}

> ## Requirements
> Participants must bring a laptop with a Mac, Linux, or Windows operating system (not a tablet,
> Chromebook, etc.) that they have administrative privileges on. They should have a few specific software
> packages installed (listed in the Setup section below). They are also required to abide by the
> [ARCHER2 Training Code of Conduct](https://www.archer2.ac.uk/training/code-of-conduct/).
{: .prereq}


{% include links.md %}
