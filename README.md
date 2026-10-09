[daily_print.py](https://github.com/user-attachments/files/33269157/daily_print.py)# daily-print-automation-via-windows
A simple Python project that prints a personalized daily sheet every morning — chores, dinners, crafts, learning tasks, Bible verses, and Google Calendar events — all formatted for thermal printers and fully automated with Windows Task Scheduler.

Daily Print Automation — Full Setup Guide (PDF Ready)
Author: Morgan Project: Daily Print Automation Platform: Windows 10/11 Printer: Thermal (Y813BT Label) Language: Python 3.10+

📖 Overview
This document explains how to build a fully automated daily print system that generates a personalized daily sheet including:
•	Bible verse
•	Date
•	Chores
•	Dinner ideas
•	Crafts
•	Learning tasks
•	Google Calendar events (today + tomorrow)
•	Clean thermal printer friendly layout
The system runs automatically every day using Windows Task Scheduler.

🧰 Requirements
•	Windows 10 or 11
•	Python 3.10+
•	Thermal printer
•	Google Calendar API credentials
•	Python packages:
o	google-api-python-client
o	google-auth
o	google-auth-oauthlib
o	pywin32
o	Pillow
Install packages:
Code
pip install google-api-python-client google-auth google-auth-oauthlib pywin32 pillow

🔐 1. Google Calendar API Setup
Step 1 — Create Google Cloud Project
Visit: https://console.cloud.google.com (console.cloud.google.com in Bing) Create a new project.
Step 2 — Enable Calendar API
APIs & Services → Library → “Google Calendar API” → Enable.
Step 3 — Create OAuth Credentials
APIs & Services → Credentials → Create Credentials → OAuth Client ID → Desktop App.
Download:
Code
credentials.json
Place it in:
Code
C:\daily_print\
Step 4 — First Run
Running the script will open a browser window to authorize your Google account. This generates:
Code
token.json

📅 2. Google Calendar Fetch Script
Create:
Code
google_calendar_fetch.py
Paste:

from __future__ import print_function
import datetime
import os.path

from google.auth.transport.requests import Request
from google.oauth2.credentials import Credentials
from google_auth_oauthlib.flow import InstalledAppFlow
from googleapiclient.discovery import build

SCOPES = ['https://www.googleapis.com/auth/calendar.readonly']


def get_calendar_service():
    creds = None

    if os.path.exists('token.json'):
        creds = Credentials.from_authorized_user_file('token.json', SCOPES)

    if not creds or not creds.valid:
        if creds and creds.expired and creds.refresh_token:
            creds.refresh(Request())
        else:
            flow = InstalledAppFlow.from_client_secrets_file(
                'credentials.json', SCOPES)
            creds = flow.run_local_server(port=0)

        with open('token.json', 'w') as token:
            token.write(creds.to_json())

    service = build('calendar', 'v3', credentials=creds)
    return service


def fetch_events_for_day(offset_days):
    service = get_calendar_service()

    now = datetime.datetime.utcnow()
    target_day = now + datetime.timedelta(days=offset_days)

    start_of_day = datetime.datetime(
        target_day.year, target_day.month, target_day.day, 0, 0, 0)
    end_of_day = datetime.datetime(
        target_day.year, target_day.month, target_day.day, 23, 59, 59)

    start_iso = start_of_day.isoformat() + 'Z'
    end_iso = end_of_day.isoformat() + 'Z'

    events_result = service.events().list(
        calendarId='primary',
        timeMin=start_iso,
        timeMax=end_iso,
        singleEvents=True,
        orderBy='startTime'
    ).execute()

    events = events_result.get('items', [])
    formatted = []

    for event in events:
        start = event['start'].get('dateTime', event['start'].get('date'))
        end = event['end'].get('dateTime', event['end'].get('date'))
        formatted.append({
            'title': event.get('summary', 'No Title'),
            'start': start,
            'end': end
        })

    return formatted


def get_today_and_tomorrow_events():
    today = fetch_events_for_day(0)
    tomorrow = fetch_events_for_day(1)
    return today, tomorrow

Code

[google_calendar_fetch.py](https://github.com/user-attachments/files/33269120/google_calendar_fetch.py)


🖨️ 3. Daily Print Script
Create:
Code
daily_print.py
Paste the full working version you already have 

[Uplofrom google_calendar_fetch import get_today_and_tomorrow_events
from PIL import Image, ImageDraw, ImageFont, ImageWin
import win32print
import win32ui
import random
from datetime import date

def wrap_text(text, font, max_width, draw):
    lines = []
    for paragraph in text.split("\n"):
        words = paragraph.split(" ")
        current = ""
        for w in words:
            test = current + w + " "
            if draw.textlength(test, font=font) <= max_width:
                current = test
            else:
                lines.append(current.rstrip())
                current = w + " "
        lines.append(current.rstrip())
    return lines

BIBLE_VERSES = [
    "Genesis 1:1 — In the beginning God created the heaven and the earth.",
   "
]


CHORES = [
    "Laundry wash", "Laundry fold", "Mop", "Sweep", "Clean bathrooms",
    "Clean bedrooms", "Clean dog kennels", "Pick up kitchen",
    "Clean dinner table", "Pick up living rooms", "Vacuum",
    "Change sheets", "Pick up office", "Clean out car"
]

DINNERS = [
    "Chicken tacos", "Chicken stir fry", "Chicken Alfredo", "Chicken noodle soup",
    "Chicken pot pie", "Chicken fajitas", "Chicken parmesan", "BBQ chicken thighs",
    "Honey garlic chicken", "Chicken fried rice", "Chicken quesadillas",
    "Chicken Caesar wraps", "Chicken and dumplings", "Chicken shawarma bowls",
    "Beef stew", "Spaghetti with meat sauce", "Beef tacos", "Cheeseburgers",
    "Beef stroganoff", "Meatloaf", "Beef enchiladas", "Korean beef bowls",
    "Chili", "Beef and broccoli", "
]

CRAFTS = [
    "Paper plate animals", "Handprint art", "Toilet paper roll characters",
    "Paper cup wind chimes", "DIY slime (glitter-free)",
    "Popsicle stick picture frame", "Egg carton caterpillar",
    "Finger painting art", "DIY paper crown", "Salt dough ornaments",
    "Button art canvas", "Paper bag puppet", "Cotton ball snowman",
    "DIY friendship bracelet", "Paper flower bouquet", "Sticker collage art",
    "Foam shape puzzle", "DIY rocket ship craft", "Recycled bottle piggy bank",
    "Paper roll binoculars", "Handmade bookmark", "DIY butterfly craft",
    "Bead necklace", "Paper mask craft", "Bubble-wrap stamp garden cards",
    "Felt animal magnets", "Mini banner notebooks", "Paper ladybug craft",
    "Rainbow paper tunnel", "Lucky charm rainbows"
]

LEARNING_TASKS = [
    "Day 1: Learn Blender navigation (orbit, zoom, pan) — https://youtu.be/TPrnSACZbko",
    "Day 2: Learn how to add/delete objects — https://youtu.be/TPrnSACZbko",
    "Day 3: Learn move tool (G key) — https://youtu.be/TPrnSACZbko",
    ""
]

START_DATE = date(2026, 10, 9)

def get_learning_task():
    days_passed = (date.today() - START_DATE).days
    day_number = days_passed + 1
    if day_number < 1:
        return "Learning starts on October 9."
    if day_number > len(LEARNING_TASKS):
        return "All 120 days completed!"
    return LEARNING_TASKS[day_number - 1]

def format_events_section(events_today, events_tomorrow):
    section = ""
    if events_today:
        section += "\nToday’s Events:\n"
        for e in events_today:
            section += f"[ ] {e['title']}\n"
    if events_tomorrow:
        section += "\nTomorrow’s Events:\n"
        for e in events_tomorrow:
            section += f"[ ] {e['title']}\n"
    return section

def generate_daily_text():
    chores = random.sample(CHORES, 3)
    dinners = random.sample(DINNERS, 2)
    craft = random.choice(CRAFTS)
    learning = get_learning_task()
    verse = random.choice(BIBLE_VERSES)
    events_today, events_tomorrow = get_today_and_tomorrow_events()

    text = "Chores:\n"
    for c in chores:
        text += f"[ ] {c}\n"

    text += "\nDinner Ideas:\n"
    for d in dinners:
        text += f"- {d}\n"

    text += f"\nFun Craft:\n- {craft}\n"
    text += f"\nLearning Task:\n- {learning}\n"
    text += format_events_section(events_today, events_tomorrow)

    return verse, text

def create_image(verse, text, filename="daily.png"):
    img = Image.new("RGB", (800, 1100), "white")
    draw = ImageDraw.Draw(img)

    try:
        font = ImageFont.truetype("arial.ttf", 28)
    except OSError:
        font = ImageFont.load_default()

    max_width = 760

    verse_lines = wrap_text(verse, font, max_width, draw)
    y = 20
    for line in verse_lines:
        bbox = draw.textbbox((0, 0), line, font=font)
        line_width = bbox[2] - bbox[0]
        line_x = (800 - line_width) // 2
        draw.text((line_x, y), line, fill="black", font=font)
        y += 40

    today_str = date.today().strftime("%B %d, %Y")
    bbox_date = draw.textbbox((0, 0), today_str, font=font)
    date_width = bbox_date[2] - bbox_date[0]
    date_x = (800 - date_width) // 2
    draw.text((date_x, y), today_str, fill="black", font=font)
    y += 60

    wrapped_lines = wrap_text(text, font, max_width, draw)
    for line in wrapped_lines:
        draw.text((20, y), line, fill="black", font=font)
        y += 40

    img.save(filename)
    return filename

def print_image(filename):
    printer_name = "YOUR PRINTER"
    hDC = win32ui.CreateDC()
    hDC.CreatePrinterDC(printer_name)

    bmp = Image.open(filename)
    hDC.StartDoc("Daily Print")
    hDC.StartPage()

    dib = ImageWin.Dib(bmp)
    dib.draw(hDC.GetHandleOutput(), (0, 0, bmp.size[0], bmp.size[1]))

    hDC.EndPage()
    hDC.EndDoc()
    hDC.DeleteDC()

if __name__ == "__main__":
    verse, text = generate_daily_text()
    filename = create_image(verse, text)
    print_image(filename)
ading daily_print.py…]()


🖼️ 4. Printer Setup
Install pywin32:
Code
pip install pywin32
Find your printer name:
Control Panel → Devices & Printers → Properties
Set:
Code
printer_name = "your printer name"

🔁 5. Automate Daily Printing (Windows Task Scheduler)
Step 1 — Create launcher file
Create:
Code
run_daily_print.bat
[Uplo@echo off
powershell -Command "& 'C:\Users\bravo\AppData\Local\Python\pythoncore-3.15-64\python.exe' 'C:\daily_print\daily_print.py'"
ading run_daily_print.bat…]()

Contents:
Code
@echo off
powershell -Command "& 'C:\Users\YOURNAME\AppData\Local\Python\python.exe' 'C:\daily_print\daily_print.py'"
Step 2 — Open Task Scheduler
Code
Win + R → taskschd.msc
Step 3 — Create Task
•	Create Task
•	Name: Daily Print
•	Run whether user is logged on or not
•	Run with highest privileges
Step 4 — Trigger
•	Daily
•	8:00 AM
Step 5 — Action
•	Start a program
•	Program/script: run_daily_print.bat
•	Start in: C:\daily_print
Step 6 — Save
Enter your Windows password.

📦 6. Folder Structure
Code
C:\daily_print\
│
├── daily_print.py
├── google_calendar_fetch.py
├── credentials.json        (ignored)
├── token.json              (ignored)
├── run_daily_print.bat
└── daily.png               (generated)

🔒 7. Security & GitHub Safety
Add this .gitignore:
Code
credentials.json
token.json
daily.png
*.pyc
__pycache__/
This prevents leaking your Google credentials.

📜 8. License (Recommended)
MIT License:
Code
MIT License

Copyright (c) 2026 Morgan

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction...



