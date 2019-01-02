---
layout: post
title: Making a playlist from local radio
playlist-id: playlist/0YNiCpiDoBymDKZSGSOaMa
---

When I moved to a new city, I missed tuning in to my favorite radio station in the car. I wanted a way to bring the radio station with me. Most radio stations have a live stream, but they are typically low quality and costly to stream on the go. Not to mention those pesky ads. This is my solution.

# Writing the script
A simple python script was sufficient to download a snipped of radio, identify the song, and add the song to a Spotify playlist. The script was scheduled to run every few minutes to capture most songs. 

## Recording the radio
First we have to record a snippet of radio audio to use for identification. We can get a direct link from a radio stream website by inspecting the webpage's source. I found a stream of my favorite radio station using [TuneIn](https://tunein.com/).

~~~~html
<div id="player" style="width: 0px; height: 0px;">
   <img id="jp_poster_0" style="width: 0px; height: 0px; display: none;">
   <audio id="jp_audio_0" preload="metadata" src="http://54.213.1.131/alphatyler-kooifmmp3-ibc2?session-id=905d1dce346e446ebf6bd47342232bfa&amp;source=TuneIn" title="106.5 Jack FM - Playing What We Want"></audio>
</div>
~~~~

We can open the page and download raw audio to a `.wav` using standard python libraries. I found that 10 seconds was long enough for identification.

~~~~python
import time, sys
from urllib.request import urlopen
url = 'http://18363.live.streamtheworld.com/WZSTFM.mp3?tdtok=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiIsImtpZCI6ImZTeXA4In0.eyJpc3MiOiJ0aXNydiIsInN1YiI6IjIxMDY0IiwiaWF0IjoxNTM3MDM4NjY0LCJ0ZC1yZWciOmZhbHNlfQ.aB_AKuD_Vwmu_3t8poeAoWq9E2K_aJfVz8piMpKZeis'
print ("Connecting to "+url)
response = urlopen(url, timeout=10.0)
fname = "Sample"+str(time.clock())[2:]+".wav"
f = open(fname, 'wb')
block_size = 1024
print ("Recording roughly 10 seconds of audio now - Please wait")
limit = 10
start = time.time()
while time.time() - start < limit:
    try:
        audio = response.read(block_size)
        if not audio:
            break
        f.write(audio)
        sys.stdout.write('.')
        sys.stdout.flush()
    except Exception as e:
        print ("Error "+str(e))
f.close()
sys.stdout.flush()
print("")
print("10 seconds from "+url+" have been recorded in "+fname)
~~~~

## Song recognition
We can identify the audio snippet using Automatic Content Recognition service (ACR). ACR generates an audio fingerprint using distinguishing characteristics such as average zero crossing rate, tempo, and average spectrum. It identifies the fingerprint by comparing to a database of reference fingerprints.

We can use the free API provided by [ACRCloud](https://www.acrcloud.com/) to return song metadata such as Spotify song ID and track title.

~~~~python
from acrcloud.recognizer import ACRCloudRecognizer
import json
config = {
    'host': 'identify-us-west-2.acrcloud.com',
    'access_key': '82640bba7076975b5fcaf79d00795d16',
    'access_secret': 'JaLNPFDhjQbAuJyQKU8HfU6xZU2YzwzTFFiPH3Z7',
    'debug': True,
    'timeout': 10
}
acrcloud = ACRCloudRecognizer(config)
track_data = json.loads(acrcloud.recognize_by_file(fname, 0))
# Cleanup .wav
import os
import glob
for f in glob.glob('*.wav'): os.remove(f)
try:
    track_title = track_data['metadata']['music'][0]['external_metadata']['spotify']['track']['name']
    track_id = track_data['metadata']['music'][0]['external_metadata']['spotify']['track']['id']
    print('Identified track as', '\"'+track_title+'\"', track_id)
except:
    print('Audio could not be identified')
    sys.exit()
~~~~

## Connecting to Spotify
I utilized the [spotipy](https://github.com/plamere/spotipy) library written by [Paul Lamere](https://musicmachinery.com/) to simplify interaction with Spotify's API. We need to provide IDs for the identified track, radio playlist, and a client token tied to the Spotify account.

~~~~python
import pprint
import sys
import spotipy
import spotipy.util as util

username = 'ajpieface2'
playlist_id = '0YNiCpiDoBymDKZSGSOaMa'
client_id = '10432503769b49b99aecc7acf15c1821'
client_secret = 'a1c5ce5b2d80496f91ed6210182bbff2'
redirect_uri = 'http://localhost/'
track_ids = [track_id]

scope = 'playlist-modify-public'
token = util.prompt_for_user_token(username, scope, client_id, client_secret, redirect_uri)

if token:
    sp = spotipy.Spotify(auth=token)
    sp.trace = False
    sp.user_playlist_remove_all_occurrences_of_tracks(username, playlist_id, track_ids)
    sp.user_playlist_add_tracks(username, playlist_id, track_ids)
    print('\"'+track_title+'\"', '('+track_id+')', 'added to playlist '+playlist_id)

else:
    print("Can't get token for", username)
~~~~

# Results
The script does a great job of identifying songs using ACRCloud. The script does not protect against duplicates, so it is necessary to occasionally run [JMPerez's](https://jmperezperez.com/) [Spotify Deduplicator](https://jmperezperez.com/spotify-dedup/). A future version could include the spotify deduplicator, or compare the identified song against the current list before adding.

The full source can be found on [Github](https://github.com/acjensen/radio-playlist).

{% include spotify.html %}