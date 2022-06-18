---
title: "Canvas to Notion Rest API Link"
date: 2022-05-30T17:37:52-07:00
tags: [API, Coding, Notion]
toc: true
draft: true
---

## Intentions/Inspirations
I've used Notion for everything life and school related since discovering it in the summer of 2020. I've played around with relational databases, spent hours making uber specific formulas, and tried to get anyone I can to take a look at the large tree of content I've slowly been collecting and sorting over the months. So, needless to say, when Notion released it's beta API in May 2020 I was on the edge of my seat!

Every new school quarter I go through my assignments on Canvas LMS for each class and input each one into one big Notion database that I refer to throughout the quarter. It's a super useful product in the end but all that data entry takes hours to complete. So by connecting Notion and Canvas using their respective restful APIs I have an opportunity to save myself hours of data entry each quarter and learn about APIs while doing it.

## The Journey

I started out pretty confident and went straight to [Notion's API documentation](https://developers.notion.com/reference/intro) to try and figure out how it worked. Needless to say it didn't take long before I realized there was no way for me to understand what was going on without seeing it in context.

<div class="row">
    <div class="column post-text-col">
        I then found <a href="https://www.youtube.com/watch?v=n8WzcnZYOIM&t=784s">Shashank Kalanithi's video</a> listed on Notion's website as an opportunity to see the API in action. 30 minutes in and I was still stumped and realized as much as I kind of understand what APIs do, I don't understand them well enough to get a sense of what's happening even in this context.
    </div>
    <div class="column post-image-col">
        <img alt="youtube thumbnail with text Notion API Discord Webhooks" class="post-image" src="https://i.ytimg.com/vi/n8WzcnZYOIM/hq720.jpg?sqp=-oaymwEcCNAFEJQDSFXyq4qpAw4IARUAAIhCGAFwAcABBg==&rs=AOn4CLDH7IEbmKqFxYblPwmMxAE-tHFpmQ"/>
    </div>
</div>
<div class="row">
    <div class="column post-text-col">
        That's usually where <a href="http://freeCodeCamp.org">freeCodeCamp.org</a> comes in to shine! After <a href="https://www.youtube.com/watch?v=GZvSYJDk-us&t=6796s">2 hours</a> of a very detailed and kind explanation and some experimenting with Postmate I felt like I finally learned the language behind working with APIs in python and could move on to the specialized uses.
    </div>
    <div class="column post-image-col">
        <img alt="youtube thumbnail with text APIs for beginners" class="post-image" src="https://i.ytimg.com/vi/GZvSYJDk-us/hq720.jpg?sqp=-oaymwEcCNAFEJQDSFXyq4qpAw4IARUAAIhCGAFwAcABBg==&rs=AOn4CLD5RPyYTzv2_Nq8bHHd5UnKSLNXyQ"/>
    </div>
</div>

From there it was just a matter of time consulting both Notion and Canvas's API documentations and trouble shooting the bits of python I was relearning and voila! I had myself a script that could talk to both Notion and Canvas and populate a database.

## The Results of Iteration #1 - Learning how to use restful APIs
The first stopping point was when I was able to get 

{{< expandable label="first round of code" level="2" >}}

```python
# import necessary libraries/files
import requests
import json
import secrets #contains the notion token, the notion database ID, and the canvas token.

# Create the necessary url and headers for notion
notion_base_url = "https://api.notion.com/v1/pages"
notion_headers = {
    "Authorization": "Bearer " + secrets.notion_token,
    "Content-Type": "application/json",
    "Notion-Version": "2021-05-13"
}

# Create the necessary url and headers for canvas
course_id = "1451055" # URBDP 200
canvas_base = "https://canvas.uw.edu/api/v1/courses/"
canvas_URL = canvas_base + course_id + "/assignments"
canvas_header = {
    "Authorization": "Bearer " + secrets.canvas_token
}

# Scrape canvas for what we need
response = requests.get(canvas_URL, headers=canvas_header)

# Convert it into readable json format
richList = response.json()

# for every assignment in the list, we want to clean up the data and send a message to notion with the right data (title, due date, URL)
for i in range(0, len(richList)):
    # Pop off the assignment we need right now
    current = richList[i]

    # Format part of the assignment URL we need to make it accessible as a UW student
    better_URL = current["html_url"].replace("https://canvas.instructure.com", "https://canvas.uw.edu")

    # Create a new dictionary that will populate the new notion page of this assignment
    pageData = {
        "parent": { "database_id": secrets.notion_database_id},
        "properties": {
            "Title": {
                "title": [
                    {
                        "text": {
                            "content": current["name"]
                        }
                    }
                ]
            },
            "Due Date": {
                "date": {
                    "start": current["due_at"]
                }
            },
            "URL" : {
                "url": better_URL
            }
        }
    }

    # Convert the new dictionary pageData to json for sending
    data = json.dumps(pageData)

    # Send the new page out to notion
    res = requests.request("POST", notion_base_url, headers=notion_headers, data=data)

    # Troubleshoot helpers
    print(res.status_code)
    print(res.text)

```
{{< /expandable >}}

## The Results of Iteration #2 - Creating a useful product
Roadblocks:
- working with datetime library in python to convert timezones 
- 

## In Progress Iteration #3 -