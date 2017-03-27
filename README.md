# README #

Writing stats is a Python script that allows to track progress of writing for several files in specified locations. 
It also allows to push changes to the website and display them on charts. This is the project that was made for tracking my 
[NaNoWriMo](http://nanowrimo.org/) progress.

![alt text](https://github.com/adrijanik/writing-stats/blob/master/writing_stats.png "website")

### Website charts ###

Website is created in Django and it just displays charts generated from Python script word_tracker.


### Words tracking script ###

This script was created because I found it hard to stay on the track of writing daily certain amount of words so I decided to automate the work with handy Python solution and web interface that will clearly show me where am I. As I'm an enthusiastic vim and console user it is a console script. To get information about how to use it simply type: word_tracker --help

### Dependencies ###

*Python 2.7
*Django
*pandas
*numpy
*pygal

based on tutorial: http://tutorial.djangogirls.org/en/django_urls/

