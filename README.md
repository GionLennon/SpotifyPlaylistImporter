# SpotifyPlaylistImporter
Import all songs of a playlist ordered, into your favorite songs in Spotify.

To confirm the script will do it's job properly it prints the first 20 songs it will import for you to check and confirm with `yes`. 
After that it will iterate through the whole playlist and save every song to the favorites of your profile. 

Because Spotify randomizes the order of the songs, when multiple songs get added to the favourites simultaneously the script waits 1s after every song.

## Dependencies
### Python
  spotipy python package – `pip install spotipy`
### Spotify Developer App
  - Client ID (76xxxxxxxxxxxx…) (your_client_id)
  - Client Secret (76xxxxxxxxxxxx…) (your_client_secret)
  - Redirect URIs (I set mine to http://localhost:8888/callback) (your_deployed_URI)
    
  All three of those can be obtained by creating an app in the [Spotify Developer Dashboard](https://developer.spotify.com/dashboard)
### Spotify Playlist ID
  Playlist ID (can be obtained by sharing the playlist and copying the ID in the link) (`your_playlist_id`, in Step 2 of the code)
### Python Script [Download](./ImportPlaylist.py)
  ```python
import spotipy
from spotipy.oauth2 import SpotifyOAuth
import time

# Spotify API credentials
SPOTIPY_CLIENT_ID = 'your_client_id'
SPOTIPY_CLIENT_SECRET = 'your_client_secret'
SPOTIPY_REDIRECT_URI = 'your_deployed_URI'

# Scopes required for accessing playlists and saving tracks
SCOPES = 'playlist-read-private user-library-modify'

def authenticate_spotify():
    """
    Authenticate the user and return a Spotify client.
    """
    sp = spotipy.Spotify(auth_manager=SpotifyOAuth(
        client_id=SPOTIPY_CLIENT_ID,
        client_secret=SPOTIPY_CLIENT_SECRET,
        redirect_uri=SPOTIPY_REDIRECT_URI,
        scope=SCOPES
    ))
    return sp

def get_playlist_tracks_in_order(sp, playlist_id):
    """
    Retrieve all tracks from a playlist in the order they appear.
    """
    tracks = []
    results = sp.playlist_items(playlist_id, fields='items(track(id,name,artists.name)),next', limit=100)

    while results:
        for item in results['items']:
            track = item['track']
            if track:
                track_info = {
                    'id': track['id'],
                    'name': track['name'],
                    'artists': ", ".join(artist['name'] for artist in track['artists'])
                }
                tracks.append(track_info)  # Preserve order of appearance in playlist
        results = sp.next(results) if results['next'] else None

    return tracks

def save_tracks_slowly(sp, tracks, delay=2):
    """
    Save tracks to the user's favorites with a delay between each save.
    """
    for idx, track in enumerate(tracks, start=1):
        sp.current_user_saved_tracks_add([track['id']])
        print(f"Saved track {idx}: {track['name']} by {track['artists']} to favorites.")
        time.sleep(delay)  # Introduce a delay

if __name__ == '__main__':
    # Step 1: Authenticate
    sp = authenticate_spotify()

    # Step 2: Get Playlist Tracks in Order
    playlist_id = 'your_playlist_id'
    tracks_in_order = get_playlist_tracks_in_order(sp, playlist_id)

    # Step 3: Reverse the Order of Tracks
    tracks_in_order.reverse()  # Invert the playlist order (start from the bottom)

    # Step 4: Display the First 20 Tracks for Confirmation
    print("First 20 tracks in the reversed playlist order:")
    for idx, track in enumerate(tracks_in_order[:20], start=1):
        print(f"{idx}. {track['name']} by {track['artists']}")

    # Step 5: Confirm and Save Tracks Slowly
    confirmation = input("\nDo you want to save all tracks to your favorites in reverse order? (yes/no): ").strip().lower()
    if confirmation == 'yes':
        save_tracks_slowly(sp, tracks_in_order, delay=2)  # Save with a 2-second delay
        print("All tracks saved to favorites in reverse order.")
    else:
        print("Operation canceled.")

  ```

