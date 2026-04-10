# 📰 RSS News Card

![Overview](https://user-images.githubusercontent.com/5878303/152332130-760cf616-5c40-4825-a482-bb8f1f0f5251.png)

## What is RSS News Card ?

A scrollable and highly customisable RSS newsfeed reader card for [Home Assistant][home-assistant] Dashboard, with multi-source support, automatic language detection, and full visual editor. RSS News Card is designed to scrape the content of various types of RSS feeds with pictures and display it on your HA Dashboard the way you want, with many options to tweak.

### Features

- 📰 Multiple RSS sources in a single card, sorted by date/time
- 🌍 Automatic language & date format detection from Home Assistant settings
- 📱 iOS-compatible scrolling, with adjustable card size
- 🎨 Full visual editor with color picker, toggle switches, and font size controls
- ⚠️ Built-in sensor diagnostics with setup instructions
- 🌐 Community localization support (English, Hungarian, German included)
- 🏷️ News source labels
- 💪🏻 Flexible layout
 

## Installation

### HACS (Recommended)

Go to the HACS store, click on the 3 dots in upper right corner, select Custom repos and add the url `https://github.com/suxlala/rss-news-card` and chose Dashboard as a type.

In Home Assistant's global settings, add the resource:

1. Go to **Settings → Dashboards → Three-dots menu → Resources** in the top right
2. Click **+ Add Resource** button in the bottom right
3. Enter in the following:
    * **URL:** /local/rss-news-card/rss-news-card.js
    * **Resource Type:** JavaScript Module
4. Click Create
**Note:** If you do not see the Resources menu, you will need to enable _Advanced Mode_ in your _User Profile_

_or_

### Manual Installation

1. Download `rss-news-card.js` file from the [latest release][release-url].
2. Put `rss-news-card.js` file into your `config/www` folder. (/config/www/rss-news-card/rss-news-card.js)
3. Add reference to `rss-news-card.js` in Dashboard. Either as above, using UI in Dashboard settings
	_or_
   - **Using YAML:** Add following code to `lovelace` section.
     ```yaml
     resources:
       - url: /local/rss-news-card/rss-news-card.js
         type: module
     ```

## Configuration

For every RSS news feed you want to track, create an individual `command_line` sensor in `configuration.yaml`. Copy and paste the YAML code below and amend it with your selected news feed source name and RSS web URL.
Put more sensors one by one under one 'command_line:' line.

```yaml
command_line:
  - sensor:
      name: "Your News Source1"
      unique_id: news_source_1
      icon: "mdi:rss"
      scan_interval: 600
      command: >-
        curl -sk "https://feeds.bbci.co.uk/news/world/europe/rss.xml" -A "Mozilla/5.0" | tr -d '\n' | sed 's/<!\[CDATA\[//g; s/\]\]>//g' | sed 's/<\/item>/\n/g' | grep '<item>' | head -5 | while read -r item; do
          title=$(echo "$item" | sed 's/.*<title>\([^<]*\)<\/title>.*/\1/' | sed 's/^ *//;s/ *$//');
          link=$(echo "$item" | sed 's/.*<link>\([^<]*\)<\/link>.*/\1/');
          desc=$(echo "$item" | sed 's/.*<description>\([^<]*\)<\/description>.*/\1/' | sed 's/^ *//;s/ *$//');
          pub=$(echo "$item" | sed 's/.*<pubDate>\([^<]*\)<\/pubDate>.*/\1/' | awk '{
            split("Jan Feb Mar Apr May Jun Jul Aug Sep Oct Nov Dec", m)
            for(i=1;i<=12;i++) month[m[i]]=sprintf("%02d",i)
            printf "%s-%s-%sT%s+02:00", $4, month[$3], $2, $5
          }');
          img=$(echo "$item" | grep -o 'enclosure url="[^"]*"' | sed 's/enclosure url="//;s/"$//' | head -1);
          if [ -z "$img" ]; then
            img=$(echo "$item" | grep -o 'media:content[^>]*url="[^"]*"' | grep -o 'url="[^"]*"' | sed 's/url="//;s/"$//' | head -1);
          fi;
          if [ -z "$img" ]; then
            img=$(echo "$item" | grep -o 'media:thumbnail[^>]*url="[^"]*"' | grep -o 'url="[^"]*"' | sed 's/url="//;s/"$//' | head -1);
          fi;
          if [ -z "$img" ]; then
            img=$(echo "$item" | grep -oE '<img[^>]+src="[^"]+"' | grep -o 'src="[^"]*"' | sed 's/src="//;s/"$//' | head -1);
          fi;          
          jq -n --arg t "$title" --arg l "$link" --arg d "$desc" --arg p "$pub" --arg i "$img" \
            '{title:$t, link:$l, description:$d, pubDate:$p, image:$i}';
        done | jq -sc '{"articles": .}'     
      value_template: "{{ value_json.articles | count }} cikk"
      json_attributes:
        - articles 
```

> **Note:** This command will check for updates every 10 minutes (600 sec) and collect the headline, summary and picture for the latest 5 news articles from the RSS URL provided. Change the value of `head -5` if you want to gather more or less than 5 articles for your news source.

---

### Create card

Reload Home Assistant to enable the changes

Add a new custom RSS News Card to your Dashboard and use the visual editor or the following example config, by adding your own command line news source sensors created above:

```yaml
type: custom:rss-news-card
title: Latest News
card_height: 400
max_articles: 20
show_description: true
show_source: true
show_date: true
image_width: 100
image_height: 70
sources:
  - entity: sensor.your_news_source1
    name: News source 1
    color: "#e63946"
  - entity: sensor.your_news_source2
    name: News source 2 
    color: "#0077cc"
```

---

## Card config options

| Option | Type | Default | Description |
|---|---|---|---|
| `title` | string | `''` | Card header title |
| `sources` | list | mandatory | RSS sources' list |
| `sources[].entity` | string | mandatory | Command_line sensor entity_id |
| `sources[].name` | string | entity name | News source display name |
| `sources[].color` | string | `#0077cc` | Source label colour (hex, colour name, var) |
| `max_articles` | number | `20` | Number of articles to be displayed |
| `card_height` | number | `400` | Height of scrollable space (px) |
| `show_description` | boolean | `true` | Show article summary text |
| `show_source` | boolean | `true` | Show source label |
| `show_date` | boolean | `true` | Show publication date & time |
| `image_width` | number | `100` | Image width (px) |
| `image_height` | number | `70` | Image height (px) |

---

## Prerequisites

- Home Assistant 2024.1+
- `curl` and `jq` available in the HA container (default in most installations)
- RSS sources configured as `command_line` sensors — see [README](https://github.com/suxlala/rss-news-card#readme) for setup

