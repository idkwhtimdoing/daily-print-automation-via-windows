# daily-print-automation-via-windows
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
Code
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

    return build('calendar', 'v3', credentials=creds)


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

🖨️ 3. Daily Print Script
Create:
Code
daily_print.py
Paste the full working version you already have (the one I provided earlier).

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



