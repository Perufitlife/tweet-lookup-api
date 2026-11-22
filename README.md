# Tweet lookup API: hydrate tweet IDs and URLs into JSON with likes, retweets and media

How to turn a list of tweet IDs or tweet links into full JSON without the X API, a developer account or a login: the text, date, language, likes, retweets, replies, quotes, bookmarks, hashtags, mentions, links, media, reply and quote flags, and the author with their follower count.

The data comes from the Apify Actor [Tweet Scraper by URL or ID](https://apify.com/renzomacar/tweet-scraper-by-url-or-id). It reads the same public data an embedded tweet shows. Paste x.com links, twitter.com links or bare numeric IDs, one per line.

## Why not the official API

Looking up tweets by ID on the X API needs a developer account, and reading more than a handful needs a paid tier. Research datasets are usually shared as lists of tweet IDs, because X's terms limit sharing the full tweets, and someone has to "hydrate" them back into tweets.

## Example: eight famous tweets, hydrated

Input, mixing the three accepted forms:

```json
{
  "urls": [
    "https://x.com/jack/status/20",
    "266031293945503744",
    "440322224407314432",
    "https://twitter.com/carterjwm/status/849813577770778624",
    "1349129669258448897",
    "https://x.com/elonmusk/status/1519480761749016577",
    "https://x.com/BarackObama/status/896523232098078720",
    "https://x.com/NASA/status/1546995244960440321"
  ]
}
```

Run on 2026-11-22: 7 tweets in 3.5 seconds. The eighth link no longer exists; the run log names it ("Tweet is deleted, private, or from a suspended account") and it is not charged.

| Posted | Author | Text | Likes | Retweets | Replies | Quotes | Bookmarks | Retweets per like |
|---|---|---|---|---|---|---|---|---|
| 2006-03-21 | @jack | just setting up my twttr | 324,581 | 126,351 | 19,258 | 8,045 | 25,517 | 0.39 |
| 2012-11-07 | @BarackObama | Four more years. | 454,242 | 683,387 | 46,764 | 16,622 | 576 | 1.50 |
| 2014-03-03 | @TheEllenShow | If only Bradley's arm was longer. Best photo ever. | 1,890,277 | 2,505,522 | 158,680 | 16,070 | 11,231 | 1.33 |
| 2017-04-06 | @carterjwm | HELP ME PLEASE. A MAN NEEDS HIS NUGGS | 1,130,186 | 2,878,880 | 32,554 | 71,929 | 16,300 | 2.55 |
| 2017-08-13 | @BarackObama | "No one is born hating another person because of… | 3,391,789 | 1,213,783 | 58,166 | 58,245 | 2,568 | 0.36 |
| 2021-01-12 | @elonmusk | Hey you … Yeah you Queen … You're gonna make it! | 504,289 | 93,411 | 12,533 | 13,495 | 6,449 | 0.19 |
| 2022-04-28 | @elonmusk | Next I'm buying Coca-Cola to put the cocaine back in | 4,168,923 | 573,170 | 166,998 | 168,424 | 22,926 | 0.14 |

The three tweets that held the retweet record in turn (Obama 2012, then Ellen's Oscars selfie, then Carter Wilkerson's nuggets, which was a campaign to beat Ellen) are the only ones with more retweets than likes. Every other tweet, from any year, gets between 2.5 and 7 likes per retweet.

All seven rows in full: [sample-output.json](sample-output.json). One of them, shortened:

```json
{
  "id": "440322224407314432",
  "url": "https://x.com/TheEllenShow/status/440322224407314432",
  "text": "If only Bradley's arm was longer. Best photo ever. #oscars http://t.co/C9U5NOtGap",
  "createdAt": "2014-03-03T03:06:13.000Z",
  "lang": "en",
  "likeCount": 1890277,
  "retweetCount": 2505522,
  "replyCount": 158680,
  "quoteCount": 16070,
  "bookmarkCount": 11231,
  "hashtags": ["oscars"],
  "media": [{ "type": "photo", "url": "https://pbs.twimg.com/media/BhxWutnCEAAtEQ6.jpg", "width": 1920, "height": 1080 }],
  "isReply": false,
  "isQuote": false,
  "authorHandle": "TheEllenShow",
  "authorName": "The Ellen Show",
  "authorFollowers": 62097752,
  "requestedId": "440322224407314432"
}
```

`requestedId` is what you passed in, so the output can be joined back to your list.

## Call it from code

You need an Apify account and its API token.

```bash
curl -X POST "https://api.apify.com/v2/acts/renzomacar~tweet-scraper-by-url-or-id/run-sync-get-dataset-items?token=$APIFY_TOKEN" \
  -H "content-type: application/json" \
  -d '{"urls":["https://x.com/jack/status/20"]}'
```

Python, hydrate an ID file and write a CSV:

```python
import csv
from apify_client import ApifyClient

ids = [line.strip() for line in open("tweet_ids.txt") if line.strip()]
client = ApifyClient("YOUR_APIFY_TOKEN")
run = client.actor("renzomacar/tweet-scraper-by-url-or-id").call(run_input={"urls": ids})
cols = ["requestedId", "createdAt", "authorHandle", "text", "likeCount", "retweetCount", "replyCount"]
with open("tweets.csv", "w", newline="", encoding="utf-8") as f:
    w = csv.DictWriter(f, fieldnames=cols, extrasaction="ignore")
    w.writeheader()
    for row in client.dataset(run["defaultDatasetId"]).iterate_items():
        w.writerow(row)
```

JavaScript, which IDs came back empty:

```js
import { ApifyClient } from "apify-client";

const ids = ["20", "440322224407314432", "1546995244960440321"];
const client = new ApifyClient({ token: process.env.APIFY_TOKEN });
const run = await client.actor("renzomacar/tweet-scraper-by-url-or-id").call({ urls: ids });
const { items } = await client.dataset(run.defaultDatasetId).listItems();
const found = new Set(items.map((t) => t.requestedId));
console.log("missing:", ids.filter((id) => !found.has(id)));
```

## Use it from Claude, ChatGPT or another AI agent (MCP)

```json
{
  "mcpServers": {
    "apify": {
      "url": "https://mcp.apify.com/?tools=renzomacar/tweet-scraper-by-url-or-id",
      "headers": { "Authorization": "Bearer YOUR_APIFY_TOKEN" }
    }
  }
}
```

Then ask, for example: "Get these tweets and tell me which one got the most replies relative to its likes."

## Input fields

| Field | What it does |
|---|---|
| `urls` | Tweet links (x.com or twitter.com) or numeric IDs, one per line. |
| `maxItems` | Stop after this many tweets. |
| `postedAfter`, `postedBefore` | Keep only tweets posted in a date range (a date, a timestamp or "7 days"). |
| `preferSyndication` | Try X's embed endpoint first; useful for very large lists. |

## Limits

- It looks up tweets you already have. It does not search X or list an account's timeline.
- Deleted tweets, protected accounts and suspended accounts return nothing and are not charged.
- View counts are not part of the public tweet data it reads.

## Price

Pay per tweet returned, with the price shown on the Actor page. Tweets that come back empty are not charged.
