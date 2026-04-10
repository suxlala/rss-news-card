# RSS News Card

Scrollable RSS news card for Home Assistant with multi-source support, automatic language detection, and full visual editor.

## Features

- 📰 Multiple RSS sources in a single card, sorted by date
- 🌍 Automatic language & date format detection from Home Assistant settings
- 📱 iOS-compatible scrolling
- 🎨 Visual editor with color picker, toggle switches, and font size controls
- ⚠️ Built-in sensor diagnostics with setup instructions
- 🌐 Community localization support (English, Hungarian, German included)

## Requirements

- `curl` and `jq` available in the HA container (default in most installations)
- RSS sources configured as `command_line` sensors — see [README](https://github.com/yourusername/rss-news-card#readme) for setup

## Quick start

### Command line sensor in 'configuration.yaml'

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

### Card config example

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
    name: News 1
    color: "#e63946"
  - entity: sensor.your_news_source2
    name: News 2 
    color: "#0077cc"
```
