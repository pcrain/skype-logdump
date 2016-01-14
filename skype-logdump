#!/usr/bin/python
#A script for dumping Skype logs
#Author: Patrick Crain

#types
#  30  = call
#  39  = also call
#  50  = sent contact request message
#  51  = ??? [first chat?]
#  60  = /me command
#  61  = normal
#  68  = sent file
#  201 = sent picture

import sqlite3, datetime, re, html, argparse, os, platform

HOME = os.path.expanduser("~")
OUTFILE=None

_lastdate=""
_title=""
_me=""
_partner=""
_outfilename=""

_class=""

class col:
  BLN      ='\033[0m'            # Blank
  UND      ='\033[1;4m'          # Underlined
  INV      ='\033[1;7m'          # Inverted
  CRT      ='\033[1;41m'         # Critical
  BLK      ='\033[1;30m'         # Black
  RED      ='\033[1;31m'         # Red
  GRN      ='\033[1;32m'         # Green
  YLW      ='\033[1;33m'         # Yellow
  BLU      ='\033[1;34m'         # Blue
  MGN      ='\033[1;35m'         # Magenta
  CYN      ='\033[1;36m'         # Cyan
  WHT      ='\033[1;37m'         # White

def hp(s):
  OUTFILE.write(s+"\n")
def htmlopen():
  global OUTFILE
  OUTFILE = open(_outfilename,'w')
  hp("<!DOCTYPE html>")
  hp('<head>')
  hp('<meta charset="UTF-8">')
  hp('<link rel="stylesheet" type="text/css" href="assets/style.css">')
  hp('<script src="./assets/jquery-2.2.0.min.js"></script>')
  hp('<script type="text/javascript">')
  hp('$(document).ready(function(){')

  # hp("$('.newday').nextUntil('tr.newday').hide();")

  hp("$('tr.toggleall').click(function(){")
  hp("  $('table').hide();")
  hp("  $('.newday').nextUntil('tr.newday').toggle();")
  hp("  $('table').show();")
  hp("});")

  hp("$('tr.newday').click(function(){")
  hp("  $(this).nextUntil('tr.newday').toggle();")
  hp("});")
  hp("});")

  hp('</script>')

  hp('</head>')
  hp('<body class="document">')
def htmlclose():
  hp('</body>')
  OUTFILE.close()

def numdate(ts):
  return datetime.datetime.fromtimestamp(int(ts)).strftime('%Y-%m-%d')

def timeonly(ts):
  return datetime.datetime.fromtimestamp(int(ts)).strftime('%I:%M:%S %p')

def fulldate(ts):
  return datetime.datetime.fromtimestamp(int(ts)).strftime('%A, %B %d, %Y')

def handlenormal(m):
  # print(col.WHT + "   " + html.unescape(m[4]) + col.BLN)
  hp('<tr class="'+_class+'">')
  hp("<td class=\"hideable\">")
  hp(timeonly(m[2]))
  hp("</td>")

  hp("<td class=\"hideable\">")
  hp(m[3])
  hp("</td>")
  hp('<td title="'+_title+'">')
  hp(m[4])
  hp("</td>")

def handleme(m):
  # print(col.YLW + "   " + m[4] + col.BLN)
  hp('<tr class="large '+_class+'">')
  hp("<td class=\"hideable\">")
  hp(timeonly(m[2]))
  hp("</td>")

  i=m[3] + ' ' + m[4] + "</td>"

  hp('<td class="mecommand" colspan="2" title="'+_title+'">')
  hp(i)

  hp('<tr class="small '+_class+'">')
  hp('<td></td><td></td><td class="mecommand" title="'+_title+'">')
  hp(i)

def handleimage(m):
  hp('<tr class="large '+_class+'">')
  hp("<td class=\"hideable\">")
  hp(timeonly(m[2]))
  hp("</td>")

  url=re.search(r'''uri=\"(.*?)\"''',m[4]).group(1)+ "/views/imgpsh_fullsize"
  # print(col.MGN + "   " + url + col.BLN)

  i=m[3] + ' sent an image: ' + "<a href=\""+url+"\">"+url+"</a></td>"

  hp('<td class="inlineimage" colspan="2" title="'+_title+'">')
  hp(i)

  hp('<tr class="small '+_class+'">')
  hp('<td></td><td></td><td class="inlineimage" title="'+_title+'">')
  hp(i)

def largesmallrow(classes,html):
  hp('<tr class="large '+classes+'"><td colspan="3">'+html+'</td></tr>')
  hp('<tr class="small '+classes+'"><td></td><td></td><td>'+html+'</td></tr>')

def parseline(m):
  global _lastdate, _title, _class
  if (m[0] != 61 and m[0] != 60 and m[0] != 201):
    return

  newdate = numdate(m[2])
  if _lastdate != newdate:
    largesmallrow("newday",fulldate(m[2]))
    _lastdate = newdate

  _title="From "+m[3]+" on "+fulldate(m[2])+" at "+timeonly(m[2])

  if m[1] == _me:
    color = col.GRN
    _class="myself"
  else:
    color = col.CYN
    _class="other"

  if (m[0] == 61):
    handlenormal(m)
  if (m[0] == 60):
    handleme(m)
  if (m[0] == 201):
    handleimage(m)

  hp('</tr>')

#Configure command line arguments for the parser
def configParser():
  parser = argparse.ArgumentParser()
  # mode = parser.add_mutually_exclusive_group()
  # mode.add_argument("-c", "--clear",    help="clear the logfile",                  action="store_true")
  parser.add_argument("-u", "--user",    type=str,   help="your Skype username", required=True)
  parser.add_argument("-p", "--partner", type=str,   help="conversation partner's username")
  parser.add_argument("-a", "--all",                 help="dump all single user conversations", action="store_true")
  parser.add_argument("-o", "--outfile", type=str,   help="output file name")
  return parser

def dumppartner(part,cursor):
  global _partner, _outfilename
  _partner = part
  c = cursor

  user = "\'%" + _partner + "%\'"
  if not _outfilename:
    _outfilename="./_dump-"+_partner+".html"
  print("Dumping " + _partner)
  c.execute(
    "SELECT type,author,timestamp,from_dispname,body_xml \
     FROM messages \
     WHERE chatname LIKE" + user +
    "ORDER BY timestamp ASC"
  )

  htmlopen()
  hp('<table>')
  largesmallrow("toggleall","Click here to expand / collapse all days")

  for m in c.fetchall():
    parseline(m)

  hp('</table>')
  htmlclose()

def main():
  global _me, _outfilename

  args = configParser().parse_args()
  _me = args.user

  OS=platform.system()
  if OS == "Linux":
    dbpath = HOME+"/.Skype/"+_me+"/main.db"
  elif OS == "Windows":
    dbpath = HOME+"/AppData/Roaming/Skype/"+_me+"/main.db"
  elif OS == "Darwin":
    dbpath = HOME+"/Library/Application Support/Skype/"+_me+"/main.db"
  else:
    print(col.RED + "Can't find path to main.db" + col.BLN)
    exit()

  conn = sqlite3.connect(dbpath)
  c = conn.cursor()

  if args.partner:
    c.execute(
      "SELECT DISTINCT author FROM messages WHERE author LIKE \'%" + args.partner + "%\' \
      AND author NOT LIKE \'" + _me + "\'; "
    )
    u = c.fetchall()[0]
    if u[0]:
      if args.outfile:
        _outfilename = args.outfile
      dumppartner(u[0],c)
  elif args.all:
    c.execute(
      "SELECT DISTINCT author FROM messages WHERE author NOT LIKE \'" + _me + "\';"
    )
    for u in c.fetchall():
      if u[0]:
        _outfilename=None
        dumppartner(u[0],c)
  else:
    print("Must specify -p <partner> or -a")
    exit()

if __name__ == "__main__":
  main()
