#!/usr/bin/env python

import os
import csv
import datetime
import sys
import getopt
from numpy import genfromtxt
import pandas as pd
import pygal
from pygal.style import LightStyle
import io
from datetime import date
from datetime import datetime
from dateutil import parser
import re

pth = os.path.dirname(os.path.abspath(__file__)) + "/"
writing_stats = pth + "writing_stats"
daily_count = pth + "daily_count"
config = ".word_tracker.conf"
files_list = []
charts = "charts/"
_p = 0

dirs = []

def atoi(text):
    return int(text) if text.isdigit() else text

def natural_keys(text):
    return [ atoi(c) for c in re.split('(\d+)', text) ]

def usage():
    print("Word tracker - allows to see word counts for each file in\n \
            each given location (directory), it also gives summary word\n \
            count and it allows to draw chart of progress\n \
            Usage:\n \
            -h | --help - shows usage\n \
            -p | --plot - draw chart of progress\n \
            [no arguments] - displays current statistics")

def update_web():
    #copy line_chapters.svg and diff_bar_chapters.svg to writing stats as html
    #add and commit changes in writingstats
    command = "cp " + pth + "charts/line_chapters.svg " + pth + "../writing_stats/tracker/templates/tracker/line_chapters.html\n"
    command += "cp " + pth + "charts/diff_bar_chapters.svg " + pth + "../writing_stats/tracker/templates/tracker/diff_bar_chapters.html\n"
    command += "cd ~/writing_stats\n"
    command += "git add ./tracker/templates/tracker/diff_bar_chapters.html\n"
    command += "git add ./tracker/templates/tracker/line_chapters.html\n"
    command += "git commit -m \"Automatic charts update\"\n"
    command += "git push\n"
    os.system(command)

def words_stats(item, index):
    global pth
    global daily_count
    global writing_stats
    global files_list
    i = 0
    stats = []
    with open(daily_count + '_' + dirs[index] + '.csv') as f:
        for line in f:
                #print(line.split()[0])
                stats.append(line.split()[0])

    for item in files_list[index]:
        print( "%-35s words: %s" % (item,  stats[i]))
        i += 1
    print("------------------------------------------------------------------------")
    print("%-35s words: %s" % ("All", stats[i]))
   

def plot_stats(item,index):
    global pth
    global daily_count
    global writing_stats
    global charts
    data = genfromtxt(writing_stats + '_' + dirs[index] + '.csv', delimiter=',')
    if not os.path.exists(writing_stats + '_' + dirs[index] + '.csv'):
       print("File: " + writing_stats + '_' + dirs[index] + '.csv'+ "doesn't exist, let me create it!")
       update_stats(item,index)
       words_stats(item,index)

    
    #---------Plot bar chart for words count for the prologue----------------
    #print("Plotted stats")
    data = pd.read_csv(writing_stats + '_' + dirs[index] + '.csv')
    print(data)
    bar_chart = pygal.Bar(style=LightStyle, width=800, height=600,
                      legend_at_bottom=True, human_readable=True,
                      title='Prologue - words stats: ' + dirs[index])
    for ind, row in data.iterrows():
            bar_chart.add(row["date"], row["0_file"])
    
    bar_chart.render_to_file(charts + '0_file'+ '_' + dirs[index]+'.svg')
    print("Plotting from: " + dirs[index])
    #---------Plot bar chart for words count for the whole project-----------
    bar_chart = pygal.Bar(style=LightStyle, width=800, height=600,
                      legend_at_bottom=True, human_readable=True,
                      title='All - words stats: ' + dirs[index])
    for ind, row in data.iterrows():
            bar_chart.add(row["date"], row["all"])
    
    bar_chart.render_to_file(charts + 'all'+ '_' + dirs[index]+'.svg')

    #---------Plot line chart for words count for the whole project---------
    dateline = pygal.DateLine(x_label_rotation=25,min_scale=20)
    words = []
    goal = [(parser.parse('2016-09-05 00:00:00'),1),
            (parser.parse('2016-09-22 12:00:00'),10000),
            (parser.parse('2016-10-09 12:00:00'),20000),
            (parser.parse('2016-10-26 12:00:00'),30000),
            (parser.parse('2016-11-12 12:00:00'),40000),
            (parser.parse('2016-11-30 12:00:00'),50000)]
    milestone1k = [(parser.parse('2016-09-04 00:00:00'),10000),(parser.parse('2016-11-30 23:00:00'),10000)]
    milestone2k = [(parser.parse('2016-09-04 00:00:00'),20000),(parser.parse('2016-11-30 23:00:00'),20000)]
    milestone3k = [(parser.parse('2016-09-04 00:00:00'),30000),(parser.parse('2016-11-30 23:00:00'),30000)]
    milestone4k = [(parser.parse('2016-09-04 00:00:00'),40000),(parser.parse('2016-11-30 23:00:00'),40000)]
    milestone5k = [(parser.parse('2016-09-04 00:00:00'),50000),(parser.parse('2016-11-30 23:00:00'),50000)]
    for ind, row in data.iterrows():
        words.append((parser.parse(row["date"]),int(row["all"])))
    dateline.title = 'Words count for the whole project: ' + dirs[index]
    dateline.add('Words', words)
    dateline.add('Goal', goal)
    dateline.add('1k',milestone1k)
    dateline.add('2k',milestone2k)
    dateline.add('3k',milestone3k)
    dateline.add('4k',milestone4k)
    dateline.add('5k',milestone5k)
    dateline.render_to_file(charts + 'line'+ '_' + dirs[index]+'.svg')

    #---------Plot line chart for words count for the session---------------
    difference = pygal.DateLine(x_label_rotation=25)
    words = []
    previous = data["all"][0]
    for ind, row in data.iterrows():
        if ind != 0:
            words.append((parser.parse(row["date"]), int(row["all"]) - int(previous) ))
        previous = row["all"]
    difference.title = 'Daily words count: ' + dirs[index]
    difference.add('Words', words)
    difference.render_to_file(charts + 'diff'+ '_' + dirs[index]+'.svg')

    #---------Plot bar chart for words count for the session---------------
    bar_chart = pygal.Bar(width=800, height=600,
                          title='All - words stats: ' + dirs[index])
    bar_chart.x_labels = []
    tmp =[]
    chapts = []
    for i in range(len(files_list[index])):
        chapts.append([])
#    chapts = [[],[],[],[],[],[],[],[],[],[],[]]
    previous = data["all"][0]

    prev = []
    for i in range(len(chapts)):
        prev.append(data[str(i)+"_file"][0])
            
    for ind, row in data.iterrows():
        if ind != 0:
            for i in range(len(chapts)): 
                chapts[i].append(int(row[str(i)+"_file"]) - int(prev[i]))
            tmp.append( int(row["all"]) - int(previous)) 
#            bar_chart.add( int(row["all"]) - int(previous) )
        previous = row["all"]
        for i in range(len(prev)):
            prev[i] = row[str(i)+"_file"]
        bar_chart.x_labels.append(parser.parse(row["date"]).date().day)
 
    i = 0
    for chap in chapts:
        bar_chart.add(str(i),chap)
        i = i + 1
    #bar_chart.add('words',tmp)
    bar_chart.render_to_file(charts + 'diff_bar_files'+ '_' + dirs[index]+'.svg')

    #---------Plot bar chart for words count for the session---------------
    bar_chart = pygal.Bar(width=800, height=600,
                          title='All - words stats: ' + dirs[index])
    bar_chart.x_labels = []
    tmp =[]
    previous = data["all"][0]
            
    for ind, row in data.iterrows():
        if ind != 0:
            
            tmp.append( int(row["all"]) - int(previous)) 
#            bar_chart.add( int(row["all"]) - int(previous) )
        previous = row["all"]
        bar_chart.x_labels.append(parser.parse(row["date"]).date().day)
 
    bar_chart.add('words',tmp)
    bar_chart.render_to_file(charts + 'diff_bar'+ '_' + dirs[index]+'.svg')


def update_stats(item,index):
    global pth
    global daily_count
    global writing_stats
    global files_list
    command = "wc -w "
    for file_name in files_list[index]:
        command += pth + item  + "/" + file_name + " "
    command += "> " + daily_count + '_' + dirs[index] + '.csv'
    os.system(command)

    stats = []
    with open(daily_count + '_' + dirs[index] + '.csv') as f:
        for line in f:
                #print(line.split()[0])
                stats.append(line.split()[0])
    
    now = datetime.now()
    stats.append(now)
    
    if not os.path.exists(writing_stats + '_' + dirs[index] + '.csv'):
        print("File " + writing_stats + '_' + dirs[index] + '.csv' + " doesn't exist, let me create it.")
        header = ""
        i = 0
        for file_name in files_list[index]:
            header += str(i) + "_file,"
            i = i + 1
        header +="all,date\n"
        #header = "zero,one,two,three,four,five,six,seven,eight,nine,ten,all,date\n"
        with io.FileIO(writing_stats + '_' + dirs[index] + '.csv', "w") as file:
                file.write(header)
    else:
        print("File " + writing_stats + '_' + dirs[index] + '.csv'  + " does exist. Let me update it.")
    
    #print("Stats: " + str(stats))
    
    with open(writing_stats + '_' + dirs[index] + '.csv', 'ab') as f:
        writer = csv.writer(f)
        writer.writerow(stats)


def main(argv):                         
    global pth
    global daily_count
    global writing_stats
    global chapters
    global _p
    global dirs

    if os.path.isdir(pth + "chapters"):
        chapters = os.listdir(pth + "chapters")

    chapters.sort(key=natural_keys)

    if(len(chapters) == 0):
        print("Error! Please create chapters before you run the script again!")
        usage()
        sys.exit(3)

    if not os.path.exists(config):
        print("File " + config + "doesn't exist, let me create it.")
        var = raw_input("Please enter directories to track: ")
        with io.FileIO(pth + config, "w") as file:
                file.write(var)


    with open(config) as f:
        for line in f:
                dirs = line.split()
    print(dirs)
    for item in dirs:
        if os.path.isdir(pth + item):
            tab = []
            tab = os.listdir(pth + item)
            tab.sort(key=natural_keys)
            files_list.append(tab)



    try:                                
        opts, args = getopt.getopt(argv, "hp", ["help", "plot"])
    except getopt.GetoptError: 
        usage()                         
        sys.exit(2)                     
    for opt, arg in opts:                
        if opt in ("-h", "--help"):      
            usage()                     
            sys.exit()                  
                 
        elif opt in ("-p", "--plot"): 
            
            for index in range(len(dirs)):                
                print("I'm plotting: " + dirs[index] + str(index))
                plot_stats(dirs[index],index)         
            sys.exit()                
    for index in range(len(dirs)):
        update_stats(dirs[index],index)
        words_stats(dirs[index],index)
        plot_stats(dirs[index],index)
    update_web()


            
if __name__ == "__main__":
    main(sys.argv[1:])
