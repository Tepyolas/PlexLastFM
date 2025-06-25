# Plex-LastFM
### Why?
- The usual intergration doesn't support track.updateNowPlaying; it only supports track.scrobble.
- This results in a delay of ~50-80% of the song played before it updates last.fm now playing.
  - This updates using track.updateNowPlaying, which takes <10s
  - Allows for instantly calling .fm to show your current playing track
 
### Receives Plex Webhook events
- Extracts artist, track name, album name
  - Sends 'play' and 'resume' events as "track.updateNowPlaying"
  - Sends 'scrobble' (>80% played) events as "track.scrobble"
- Minor error handling

## Setup
### Requires Last.FM API Access + API_KEY env Variables (.env for node, environment variables for Vercel)
- env.LAST_FM_API
- env.LAST_FM_SK (secret key)
- env.LAST_FM_SECRET
- env.API_KEY (simply a string added to the URL call that's checked, an extra security step)

### Run on Vercel (free)
- Simply fork git and setup in Vercel to run from Git
- Add env variables in Vercel (see below)
- Add a firewall rule for 0.0.0.0/0 then another above it for your Plex server IP 
  - *(blacklist all, whitelist plex server IP)*

### Run locally (best if you can run it ON your plex server and hook to 127.0.0.1:3000)
- Download the latest release standalone.zip
- Unzip
- Add .env file as per above/below
- ```node --env-file .env server.js```

Server will be live on (IP):3000. It can run locally on your plex server and the webhook is 127.0.0.1:3000, works best this way.

### Setup Plex
- Add a webhook to *(URL/IP)*:3000/api/webhook?apiKey=*API_KEY*

### Bash script for getting a LastFM permanent session key (env.LAST_FM_SECRET)
- Create an app at https://www.last.fm/api/account/create
  - Change API_KEY in the script to the created API_KEY (also add to **env.LAST_FM_API**)
  - Change SHARED_KEY in the script to the SHARED_KEY (also add to **env.LAST_FM_SK**)
- Run the script
- When prompted, open the URL and authenticate
- Press Enter
- Add the outputted session key **env.LAST_FM_SECRET**

Run the below with -a LAST_FM_API_KEY -s LAST_FM_SECRET_KEY
```
#!/bin/bash
while getopts a:s flag
do
    case "${flag}" in
        a) API_KEY=${OPTARG};;
        s) SECRET_KEY=${OPTARG};;
    esac
done

# --- Functions ---
# Function to calculate MD5 hash, cross-platform compatible
calculate_md5() {
    local input_string="$1"
    # Try md5sum (Linux)
    if command -v md5sum &> /dev/null; then
        echo -n "$input_string" | md5sum | awk '{print $1}'
    # Try openssl md5 (macOS/BSD)
    elif command -v openssl &> /dev/null; then
        echo -n "$input_string" | openssl md5 | awk '{print $NF}'
    else
        echo "Error: Neither 'md5sum' nor 'openssl' found. Cannot calculate MD5 signature." >&2
        exit 1
    fi
}

echo ""
echo "Step 1: User authorization required. Open the URL to authenticate your LastFM account with your API"
echo "URL: https://www.last.fm/api/auth/?api_key=$API_KEY&token=$((curl -s -X GET 'http://ws.audioscrobbler.com/2.0/?format=json&method=auth.getToken&api_key='+$API_KEY) | jq -r .token)"
echo ""
read -p "Press Enter to continue after authorization..."

echo "Step 2: Request the session key using auth.getSession..."
echo ""Session Key: $((curl -s -X POST ["$API_URL"](http://ws.audioscrobbler.com/2.0/?method=auth.getSession&api_key=$API_KEY&token=$TOKEN"&"api_sig=$(calculate_md5 "api_key${API_KEY}methodauth.getSessiontoken${TOKEN}${SECRET_KEY}")&"format=json") | jq -r '.session.key'))"
```
